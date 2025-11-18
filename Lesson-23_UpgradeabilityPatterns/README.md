# Lesson 23 - Upgradeability patterns

## Patterns overview

* Upgrading the immutable
* The simple Interface Pattern
* The problem with migrating storage
* The problem with logic changes
* Flexible and storage efficient solutions
  * Proxy contracts
  * Unstructured and transparent proxies
  * Storage patterns
  * Diamond patterns
  * Metamorphic contracts
    * Not possible anymore after [EIP-6780](https://eips.ethereum.org/EIPS/eip-6780)

### References for Upgradeability Patterns

<https://medium.com/@shub.sharma350/upgradability-patterns-in-solidity-part-1-13e23ce1f144>

<https://mirror.xyz/0xB38709B8198d147cc9Ff9C133838a044d78B064B/M7oTptQkBGXxox-tk9VJjL66E1V8BUF0GF79MMK4YG0>

<https://medium.com/@shub.sharma350/upgradability-patterns-in-solidity-part-2-8a2e531d80f8>

<https://medium.com/@shub.sharma350/upgradability-patterns-in-solidity-part-3-cba09b164497>

<https://medium.com/@shub.sharma350/upgradability-patterns-in-solidity-part-4-99a2ae29876e>

## OpenZeppelin plugins and contracts

* Upgrade Plugin
* Upgradeable variants
* Building upgradeable contracts with OpenZeppelin
* Constructor and Initializers
* Upgradeable Libraries
* Creating contracts
* Unsafe operations
* Restrictions

### References for OpenZeppelin Plugins and Contracts

<https://docs.openzeppelin.com/upgrades-plugins/1.x/proxies>

<https://docs.openzeppelin.com/contracts/4.x/api/proxy>

<https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable>

<https://docs.openzeppelin.com/contracts/4.x/upgradeable>

<https://docs.openzeppelin.com/upgrades>

<https://docs.openzeppelin.com/learn/upgrading-smart-contracts>

## Implementing a simple contract upgrade

* Running upgrades with hardhat
* Implementing a proxy pattern
* (Review) Delegate calls
* Extending storage
* Changing logic
* Understanding the risks
* Calling updates

### References for Implementing a Simple Contract Upgrade

<https://docs.openzeppelin.com/upgrades-plugins/1.x/hardhat-upgrades>

<https://docs.ethers.org/v6/api/contract/#ContractFactory>

### Code references

* Install `@openzeppelin/hardhat-upgrades` dependency

```bash
npm install --save-dev @openzeppelin/hardhat-upgrades @openzeppelin/contracts @openzeppelin/contracts-upgradeable
```

* Modify `hardhat.config.ts` to include the plugin

```typescript
import '@openzeppelin/hardhat-upgrades';
```

* Start with a **non-upgradeable** contract example

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import {AccessControl} from "@openzeppelin/contracts/access/AccessControl.sol";

contract Token is ERC20, AccessControl {
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

    constructor() ERC20("Token", "TKN") {
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(MINTER_ROLE, msg.sender);
    }

    function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
        _mint(to, amount);
    }
}
```

* Make it upgradeable

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {ERC20Upgradeable} from "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
import {AccessControlUpgradeable} from "@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol";

contract TokenUpgradableV1 is ERC20Upgradeable, AccessControlUpgradeable {
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    bytes32 public someVariableNotConstant;

    function initialize() public initializer {
        __ERC20_init("TokenUpgradable", "TKN+");
        __AccessControl_init();
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(MINTER_ROLE, msg.sender);
        someVariableNotConstant = keccak256("VARIABLE");
    }

    function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
        _mint(to, amount);
    }
}
```

* Create a contract for `TokenUpgradableV2` with the following changes:

  1. Add a state variable for `auditedSupply`:

     ```solidity
     uint256 public auditedSupply;
     ```
  
  2. Add a role for `AUDIT_ROLE`:

     ```solidity
     bytes32 public constant AUDIT_ROLE = keccak256("AUDIT_ROLE");
     ```

  3. Add a function to audit the supply:

     ```solidity
     function auditReport(uint256 amount) public onlyRole(AUDIT_ROLE) {
         auditedSupply = amount;
     }
     ```
  
  4. Modify the `mint` function to use the `auditedSupply`:

     ```solidity
     function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
         require(
             auditedSupply >= totalSupply(),
             "Supply is already bigger than the value from last audited report"
         );
         require(
             auditedSupply - totalSupply() >= amount,
             "Mint value is greater than what is available to be minted"
         );
         _mint(to, amount);
     }
     ```

* Create a `TokenUpgradableV2Wrong` with incorrect upgrade logic:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {ERC20Upgradeable} from "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
import {AccessControlUpgradeable} from "@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol";

contract TokenUpgradableV2Wrong is ERC20Upgradeable, AccessControlUpgradeable {
    uint256 public auditedSupply;
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    bytes32 public constant AUDIT_ROLE = keccak256("AUDIT_ROLE");
    bytes32 public someVariableNotConstant;

    function initialize() public initializer {
        __ERC20_init("TokenUpgradable", "TKN+");
        __AccessControl_init();
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(MINTER_ROLE, msg.sender);
        someVariableNotConstant = keccak256("VARIABLE");
    }

    function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
        require(
            auditedSupply >= totalSupply(),
            "Supply is already bigger than the value from last audited report"
        );
        require(
            auditedSupply - totalSupply() >= amount,
            "Mint value is greater than what is available to be minted"
        );
        _mint(to, amount);
    }

    function auditReport(uint256 amount) public onlyRole(AUDIT_ROLE) {
        auditedSupply = amount;
    }
}
```

* Create a test file for testing upgrades with these different contracts

```typescript
import { expect, use as chaiUse } from "chai";
import chaiAsPromised from "chai-as-promised";
import { ethers, upgrades } from "hardhat";
import {
  Token,
  TokenUpgradableV1,
  TokenUpgradableV2,
  Token__factory,
  TokenUpgradableV1__factory,
  TokenUpgradableV2Wrong__factory,
  TokenUpgradableV2__factory,
} from "../typechain-types";
import { HardhatEthersSigner } from "@nomicfoundation/hardhat-ethers/signers";

chaiUse(chaiAsPromised);

const MINT_TEST_VALUE = 1;

describe("Upgrade", async () => {
  let tokenFactory: Token__factory;
  let tokenUpgradableV1Factory: TokenUpgradableV1__factory;
  let tokenUpgradableV2WrongFactory: TokenUpgradableV2Wrong__factory;
  let tokenUpgradableV2Factory: TokenUpgradableV2__factory;

  let accounts: HardhatEthersSigner[];

  beforeEach(async () => {
    accounts = await ethers.getSigners();
    const [
      tokenFactory_,
      tokenUpgradableV1Factory_,
      tokenUpgradableV2WrongFactory_,
      tokenUpgradableV2Factory_,
    ] = await Promise.all([
      ethers.getContractFactory("Token"),
      ethers.getContractFactory("TokenUpgradableV1"),
      ethers.getContractFactory("TokenUpgradableV2Wrong"),
      ethers.getContractFactory("TokenUpgradableV2"),
    ]);
    tokenFactory = tokenFactory_ as Token__factory;
    tokenUpgradableV1Factory =
      tokenUpgradableV1Factory_ as TokenUpgradableV1__factory;
    tokenUpgradableV2WrongFactory =
      tokenUpgradableV2WrongFactory_ as TokenUpgradableV2Wrong__factory;
    tokenUpgradableV2Factory =
      tokenUpgradableV2Factory_ as TokenUpgradableV2__factory;
  });

  describe("When deploying the common ERC20 token", async () => {
    let token: Token;
    beforeEach(async () => {
      token = await tokenFactory.deploy();
      await token.waitForDeployment();
    });

    it("Mints tokens correctly using a valid minter", async () => {
      const totalSupplyBefore = await token.totalSupply();
      const mintTx = await token.mint(
        accounts[0].address,
        ethers.parseEther(MINT_TEST_VALUE.toFixed(18))
      );
      await mintTx.wait();
      const totalSupplyAfter = await token.totalSupply();
      const diff = totalSupplyAfter - totalSupplyBefore;
      expect(Number(ethers.formatEther(diff))).to.eq(MINT_TEST_VALUE);
      const balanceAfter = await token.balanceOf(accounts[0].address);
      expect(Number(ethers.formatEther(balanceAfter))).to.eq(MINT_TEST_VALUE);
    });

    describe("When trying to upgrade", async () => {
      it("Fails", async () => {
        await expect(
          upgrades.deployProxy(tokenFactory)
        ).to.eventually.be.rejectedWith("Contract `contracts/ERC20.sol:Token` is not upgrade safe");
      });
    });
  });

  describe("When deploying the upgradeable ERC20 token", async () => {
    let tokenProxy: TokenUpgradableV1;

    beforeEach(async () => {
      const tokenProxy_ = await upgrades.deployProxy(tokenUpgradableV1Factory);
      tokenProxy = tokenProxy_ as unknown as TokenUpgradableV1;
    });

    describe("When the contract is deployed", async () => {
      it("Mints tokens correctly using a valid minter", async () => {
        const totalSupplyBefore = await tokenProxy.totalSupply();
        const mintTx = await tokenProxy.mint(
          accounts[0].address,
          ethers.parseEther(MINT_TEST_VALUE.toFixed(18))
        );
        await mintTx.wait();
        const totalSupplyAfter = await tokenProxy.totalSupply();
        const diff = totalSupplyAfter - totalSupplyBefore;
        expect(Number(ethers.formatEther(diff))).to.eq(MINT_TEST_VALUE);
        const balanceAfter = await tokenProxy.balanceOf(accounts[0].address);
        expect(Number(ethers.formatEther(balanceAfter))).to.eq(MINT_TEST_VALUE);
      });
    });

    describe("When the upgrade contract is wrong", async () => {
      it("Fails", async () => {
        const tokenProxyAddress = await tokenProxy.getAddress();
        await expect(
          upgrades.upgradeProxy(
            tokenProxyAddress,
            tokenUpgradableV2WrongFactory
          )
        ).to.eventually.be.rejectedWith("New storage layout is incompatible");
      });
    });

    describe("When the upgrade contract is correct", async () => {
      const TEST_AUDIT_VALUE = 1000;
      const TEST_UPDATE_MINT_VALUE = TEST_AUDIT_VALUE + 1;

      let tokenProxyUpdated: TokenUpgradableV2;
      beforeEach(async () => {
        const tokenProxyAddress = await tokenProxy.getAddress();
        const tokenProxyUpdated_ = await upgrades.upgradeProxy(
          tokenProxyAddress,
          tokenUpgradableV2Factory
        );
        tokenProxyUpdated = tokenProxyUpdated_ as unknown as TokenUpgradableV2;
        const AUDIT_ROLE = await tokenProxyUpdated.AUDIT_ROLE();
        const grantRoleTX = await tokenProxyUpdated.grantRole(
          AUDIT_ROLE,
          accounts[1].address
        );
        await grantRoleTX.wait();
        const submitAudit = await tokenProxyUpdated
          .connect(accounts[1])
          .auditReport(ethers.parseEther(TEST_AUDIT_VALUE.toFixed(18)));
        await submitAudit.wait();
      });

      it("Mints tokens correctly using a valid minter after the audit report is registered", async () => {
        expect(TEST_AUDIT_VALUE >= MINT_TEST_VALUE);
        const totalSupplyBefore = await tokenProxyUpdated.totalSupply();
        const mintTx = await tokenProxyUpdated.mint(
          accounts[0].address,
          ethers.parseEther(MINT_TEST_VALUE.toFixed(18))
        );
        await mintTx.wait();
        const totalSupplyAfter = await tokenProxyUpdated.totalSupply();
        const diff = totalSupplyAfter - totalSupplyBefore;
        expect(Number(ethers.formatEther(diff))).to.eq(MINT_TEST_VALUE);
        const balanceAfter = await tokenProxyUpdated.balanceOf(
          accounts[0].address
        );
        expect(Number(ethers.formatEther(balanceAfter))).to.eq(MINT_TEST_VALUE);
      });

      it("Fails when minting more than registered", async () => {
        expect(TEST_UPDATE_MINT_VALUE >= TEST_AUDIT_VALUE);
        await expect(
          tokenProxyUpdated.mint(
            accounts[0].address,
            ethers.parseEther(TEST_UPDATE_MINT_VALUE.toFixed(18))
          )
        ).to.be.revertedWith(
          "Mint value is greater than what is available to be minted"
        );
      });
    });
  });
});
```

