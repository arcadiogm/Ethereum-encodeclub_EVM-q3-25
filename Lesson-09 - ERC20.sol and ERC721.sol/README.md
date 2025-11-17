# Lesson 9 - MyERC20.sol and MyERC721.sol

## Quickstart with OpenZeppelin wizard

* Overview about Ethereum Improvement Proposals (EIPs)
* Overview about Application-level standards and conventions (ERCs)
* Explain about OpenZeppelin Contracts library
* (Review) Objects in smart contracts
* Inheritance overview
* Overview about ERC20
* Overview about ERC721
* Using OpenZeppelin wizard

### References for ERCs

<https://eips.ethereum.org/>

<https://eips.ethereum.org/erc>

<https://docs.openzeppelin.com/contracts/5.x/>

<https://docs.openzeppelin.com/contracts/5.x/erc20>

<https://docs.openzeppelin.com/contracts/5.x/erc721>

<https://docs.soliditylang.org/en/latest/contracts.html#inheritance>

<https://solidity-by-example.org/inheritance/>

<https://docs.openzeppelin.com/contracts/5.x/wizard>

<https://docs.openzeppelin.com/contracts/5.x/backwards-compatibility>

### Installation

```bash
npm install --save-dev @openzeppelin/contracts
```

or

```bash
yarn add --dev @openzeppelin/contracts
```

or

```bash
bun add -d @openzeppelin/contracts
```

### Plain ERC20 Code reference

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MyToken is ERC20 {
    constructor() ERC20("MyToken", "MTK") {}
}
```

### Plain ERC721 Code reference

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract MyNFT is ERC721 {
    constructor() ERC721("MyNFT", "NFT") {}
}
```

## Contract structure

* Syntax about inheritance
* Overview about OpenZeppelin features for ERC20 and ERC721
* Overview about OpenZeppelin features for Access Control
* Overview about OpenZeppelin utilities and components
* Adding minting feature
* Adding RBAC feature

### References for Contract Structure

<https://www.npmjs.com/package/@openzeppelin/contracts>

<https://docs.openzeppelin.com/contracts/5.x/extending-contracts>

<https://docs.openzeppelin.com/contracts/5.x/access-control>

## Operating the contracts with scripts

* (Review) Script operation
* (Review) Accounts and funding
* (Review) Providers
* (Review) Async operations
* (Review) Running scripts on test environment
* (Review) Contract factory and json imports
* (Review) Transaction receipts and async complexities when running onchain

### Code reference

* Script structure

```typescript
import { viem } from "hardhat";
import { parseEther } from "viem";

async function main() {
    const publicClient = await viem.getPublicClient();
    const [deployer, account1, account2] = await viem.getWalletClients();
    // TODO
}

main().catch((err) => {
  console.error(err);
  process.exitCode = 1;
});
```

* Deploying with hardhat helper functions

```typescript
const tokenContract = await viem.deployContract("MyToken");
console.log(`Contract deployed at ${tokenContract.address}`);
```

* Fetching total supply

```typescript
const totalSupply = await tokenContract.read.totalSupply();
console.log({ totalSupply });
```

* Implementing initial supply

```solidity
    constructor() ERC20("MyToken", "MTK") {
        _mint(msg.sender, 10 * 10 ** decimals());
    }
```

* Implementing RBAC for supply control

```solidity
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";

contract MyToken is ERC20, AccessControl {
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

    constructor() ERC20("MyToken", "MTK") {
        _mint(msg.sender, 10 * 10 ** decimals());
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
    }

    function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
        _mint(to, amount);
    }
}
```

* Handling roles

```typescript
// Fetching the role code
const code = await tokenContract.read.MINTER_ROLE();
```

* Minting tokens without role fails

```typescript
const mintTx = await tokenContract.write.mint(
    [deployer.account.address, parseEther("10")],
    { account: account2.account }
);
await publicClient.waitForTransactionReceipt({ hash: mintTx });
```

>This should fail with the error `AccessControlUnauthorizedAccount`

* Minting tokens with the proper Minter Role

```typescript
// Giving role
const roleTx = await tokenContract.write.grantRole([
  code,
  account2.account.address,
]);
await publicClient.waitForTransactionReceipt({ hash: roleTx });
```

* Fetching token data with `Promise.all()`

```typescript
const [name, symbol, decimals, totalSupply] = await Promise.all([
  tokenContract.read.name(),
  tokenContract.read.symbol(),
  tokenContract.read.decimals(),
  tokenContract.read.totalSupply(),
]);
console.log({ name, symbol, decimals, totalSupply });
```

* Sending a transaction

```typescript
// Sending a transaction
const tx = await tokenContract.write.transfer([
  account1.account.address,
  parseEther("2"),
]);
await publicClient.waitForTransactionReceipt({ hash: tx });
```

* Viewing balances

```typescript
const myBalance = await tokenContract.read.balanceOf([deployer.account.address]);
console.log(`My Balance is ${myBalance} decimals units`);
const otherBalance = await tokenContract.read.balanceOf([account1.account.address]);
console.log(
  `The Balance of Acc1 is ${otherBalance} decimals units`
);
```

* Viewing converted balances

```typescript
import { parseEther, formatEther } from "viem";
...
const myBalance = await tokenContract.read.balanceOf([deployer.account.address]);
console.log(`My Balance is ${formatEther(myBalance)} ${symbol}`);
const otherBalance = await tokenContract.read.balanceOf([account1.account.address]);
console.log(
  `The Balance of Acc1 is ${formatEther(otherBalance)} ${symbol}`
);
```

* Viewing converted balances with decimals conversion

```solidity
function decimals() public pure override returns (uint8) {
    return 8;
}
```

```typescript
import { parseEther, formatUnits } from "viem";
...
const myBalance = await tokenContract.read.balanceOf([deployer.account.address]);
console.log(`My Balance is ${formatUnits(myBalance, decimals)} ${symbol}`);
const otherBalance = await tokenContract.read.balanceOf([account1.account.address]);
console.log(
  `The Balance of Acc1 is ${formatUnits(otherBalance, decimals)} ${symbol}`
);
```

## Events with solidity

* Event syntax
* Event storage
* Event indexing
* Topics and filters
* Transaction structure
* State changes with events

### References for Events

<https://docs.soliditylang.org/en/latest/contracts.html#events>

<https://viem.sh/docs/glossary/terms#event-log>

<https://viem.sh/docs/actions/public/watchEvent.html>

<https://viem.sh/docs/contract/getContractEvents>

## Watching for events in tests

* Event syntax with Hardhat Chai Matchers
* Triggering an event
* Checking arguments

### References for Watching for Events

<https://hardhat.org/hardhat-runner/docs/advanced/using-viem>

<https://hardhat.org/hardhat-runner/plugins/nomicfoundation-hardhat-viem>

### Code Reference

* Writing a test to trigger the `Transfer` event

```typescript
import { expect } from "chai";
import { viem } from "hardhat";

describe("Basic tests for understanding ERC20", async () => {
  it("triggers the Transfer event with the address of the sender when sending transactions", async () => {
    const tokenContract = await viem.deployContract("MyToken");
    const publicClient = await viem.getPublicClient();
    const [deployer, account1] = await viem.getWalletClients();
    const hash = await tokenContract.write.transfer([account1.account.address, 1n]);
    await publicClient.waitForTransactionReceipt({ hash });
    const withdrawalEvents = await tokenContract.getEvents.Transfer();
    expect(withdrawalEvents).to.have.lengthOf(1);
    expect(withdrawalEvents[0].args.from?.toLowerCase()).to.equal(deployer.account.address);
    expect(withdrawalEvents[0].args.to?.toLowerCase()).to.equal(account1.account.address);
    expect(withdrawalEvents[0].args.value).to.equal(1n);
  });
});
```

