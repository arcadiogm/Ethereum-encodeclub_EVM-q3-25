# Lesson 22 - DeFi

## What is DeFi

* Overview of financial services
* What is DeFi

### References for DeFi Introduction

<https://www.investopedia.com/ask/answers/030315/what-financial-services-sector.asp>

<https://www.investopedia.com/decentralized-finance-defi-5113835>

## Projects overview

* Locked value
* Stablecoins, marketmakers and lending protocols
* Swaps and DEXs
* Assets and derivatives
* Payments
* Money flow

### Projects Overview References

<https://defillama.com/>

<https://docs.makerdao.com/>

<https://docs.aave.com/hub/>

<https://uniswap.org/developers>

## Token Economics

* Incentives
* Yield Farming
* Sustainable yeld
* Systemic overview

### References for Token Economics

<https://academy.binance.com/en/articles/what-is-real-yield-in-defi>

<https://x.com/shivsakhuja/status/1530712622374342658> (Twitter thread)

### Swap example

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;
import "@uniswap/v3-periphery/contracts/interfaces/ISwapRouter.sol";
import "@uniswap/v3-periphery/contracts/libraries/TransferHelper.sol";

contract SwapExamples {
    // For the scope of these swap examples,
    // we will detail the design considerations when using `exactInput`, `exactInputSingle`, `exactOutput`, and  `exactOutputSingle`.
    // It should be noted that for the sake of these examples we pass in the swap router as a constructor argument instead of inheriting it.
    // More advanced example contracts will detail how to inherit the swap router safely.
    // This example swaps DAI/WETH9 for single path swaps and DAI/USDC/WETH9 for multi path swaps.

    ISwapRouter public immutable swapRouter;

    constructor(ISwapRouter _swapRouter) {
        swapRouter = _swapRouter;
    }

    address public constant DAI = 0x6B175474E89094C44Da98b954EedeAC495271d0F;
    address public constant WETH9 = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;
    address public constant USDC = 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48;

    // For this example, we will set the pool fee to 0.3%.
    uint24 public constant poolFee = 3000;

    /// @notice swapExactInputSingle swaps a fixed amount of DAI for a maximum possible amount of WETH9
    /// using the DAI/WETH9 0.3% pool by calling `exactInputSingle` in the swap router.
    /// @dev The calling address must approve this contract to spend at least `amountIn` worth of its DAI for this function to succeed.
    /// @param amountIn The exact amount of DAI that will be swapped for WETH9.
    /// @return amountOut The amount of WETH9 received.
    function swapExactInputSingle(uint256 amountIn)
        external
        returns (uint256 amountOut)
    {
        // msg.sender must approve this contract

        // Transfer the specified amount of DAI to this contract.
        TransferHelper.safeTransferFrom(
            DAI,
            msg.sender,
            address(this),
            amountIn
        );

        // Approve the router to spend DAI.
        TransferHelper.safeApprove(DAI, address(swapRouter), amountIn);

        // Naively set amountOutMinimum to 0. In production, use an oracle or other data source to choose a safer value for amountOutMinimum.
        // We also set the sqrtPriceLimitx96 to be 0 to ensure we swap our exact input amount.
        ISwapRouter.ExactInputSingleParams memory params = ISwapRouter
            .ExactInputSingleParams({
                tokenIn: DAI,
                tokenOut: WETH9,
                fee: poolFee,
                recipient: msg.sender,
                deadline: block.timestamp,
                amountIn: amountIn,
                amountOutMinimum: 0,
                sqrtPriceLimitX96: 0
            });

        // The call to `exactInputSingle` executes the swap.
        amountOut = swapRouter.exactInputSingle(params);
    }
}
```

## Flash swaps and flash loans

* Collateralized and unsecured loans
* Flash loans
* Flash loans attacks
* Flash swaps
* Multiple swap steps
* Gas fees

### References for Flash Loans

<https://academy.binance.com/en/articles/what-are-flash-loans-in-defi>

<https://docs.aave.com/faq/flash-loans>

<https://coinmarketcap.com/alexandria/article/what-are-flash-loan-attacks>

<https://docs.uniswap.org/protocol/guides/flash-integrations/inheritance-constructors>

<https://medium.com/coinmonks/tutorial-of-flash-swaps-of-uniswap-v3-73c0c846b822>

## Implementing ERC20FlashMint

* EIP 3156 fo Flash loans
* Implementing ERC20FlashMint
* Loan fees
* Implementing IERC3156FlashLender and IERC3156FlashBorrower
* Running a flash loan
* Ensuring that the loan can be paid

### References for Implementation

<https://eips.ethereum.org/EIPS/eip-3156>

<https://docs.openzeppelin.com/contracts/5.x/api/token/erc20#ERC20FlashMint>

<https://docs.openzeppelin.com/contracts/5.x/api/interfaces#IERC3156FlashLender>

<https://docs.openzeppelin.com/contracts/5.x/api/interfaces#IERC3156FlashBorrower>

### Code examples

* Coding `FlashMinter.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import {ERC20FlashMint} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20FlashMint.sol";
import {AccessControl} from "@openzeppelin/contracts/access/AccessControl.sol";

contract MyFlashMinter is ERC20FlashMint, AccessControl {
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    address public immutable receiver;
    uint256 public immutable fee;

    constructor(
        string memory name,
        string memory symbol,
        address receiver_,
        uint256 fee_
    ) ERC20(name, symbol) {
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(MINTER_ROLE, msg.sender);
        receiver = receiver_;
        fee = fee_;
    }

    function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
        _mint(to, amount);
    }

    function flashFee(address token, uint256 amount)
        public
        view
        override
        returns (uint256)
    {
        require(token == address(this), "FlashMinter: Unsupported currency");
        return _flashFee(token, amount);
    }

    function _flashFee(address, uint256 amount)
        internal
        view 
        override
        returns (uint256)
    {
        return (amount * fee) / 10000;
    }

    function _flashFeeReceiver() internal view override returns (address) {
        return receiver;
    }
}
```

* Coding `MyFlashSwap.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import {IERC3156FlashBorrower} from "@openzeppelin/contracts/interfaces/IERC3156FlashBorrower.sol";
import {IERC3156FlashLender} from "@openzeppelin/contracts/interfaces/IERC3156FlashLender.sol";

contract MyFlashSwap is IERC3156FlashBorrower {
    IERC3156FlashLender lender;

    constructor(IERC3156FlashLender lender_) {
        lender = lender_;
    }

    /// @dev ERC-3156 Flash loan callback
    function onFlashLoan(
        address initiator,
        address token,
        uint256 amount,
        uint256 fee,
        bytes calldata data
    ) external override returns (bytes32) {
        require(
            msg.sender == address(lender),
            "FlashBorrower: Untrusted lender"
        );
        require(
            initiator == address(this),
            "FlashBorrower: Untrusted loan initiator"
        );
        (bytes32 myData) = abi.decode(
            data,
            (bytes32)
        );
        
        // Sending the money somewhere for it to profit
        //TODO

        // Collecting the money from the destination
        //TODO

        return keccak256("ERC3156FlashBorrower.onFlashLoan");
    }

    /// @dev Initiate a flash loan
    function flashBorrow(
        address token,
        uint256 amount,
        bytes32 myData
    ) public {
        bytes memory data = abi.encode(myData);
        uint256 _fee = lender.flashFee(token, amount);
        uint256 _repayment = amount + _fee;
        uint256 _allowance = IERC20(token).allowance(
            address(this),
            address(lender)
        );
        IERC20(token).approve(address(lender), _allowance + _repayment);
        lender.flashLoan(this, token, amount, data);
    }
}
```

* Emulating a profit with a dummy contract `MagicSwapFaucet.sol`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import {AccessControl} from "@openzeppelin/contracts/access/AccessControl.sol";

contract MagicSwapFaucet {
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

    function doManyTradesAndProfitStonks(
        address emulatedTokenAddress,
        uint256 tokensThatWouldBeConsumed,
        uint256 emulatedProfit
    ) public {
        require(
            emulatedProfit > tokensThatWouldBeConsumed,
            "This operation is not profitable"
        );
        require(
            IERC20(emulatedTokenAddress).balanceOf(address(this)) >=
                emulatedProfit - tokensThatWouldBeConsumed,
            "Trying to profit more than what this contract holds of balance"
        );
        IERC20(emulatedTokenAddress).transferFrom(
            msg.sender,
            address(this),
            tokensThatWouldBeConsumed
        );
        IERC20(emulatedTokenAddress).approve(msg.sender, emulatedProfit);
    }
}
```

* Using the dummy contract to profit:

```solidity
        (address targetTradeContract, uint256 emulatedProfit) = abi.decode(
            data,
            (address, uint256)
        );
        
        // Sending the money somewhere for it to profit
        IERC20(token).approve(targetTradeContract, amount);
        MagicSwapFaucet(targetTradeContract).doManyTradesAndProfitStonks(
            token,
            amount,
            amount + emulatedProfit + fee
        );

        // Collecting the money from the destination
        IERC20(token).transferFrom(
            targetTradeContract,
            address(this),
            amount + emulatedProfit + fee
        );
```

```solidity
    function flashBorrow(
        address token,
        uint256 amount,
        address targetTradeContract,
        uint256 emulatedProfit
    ) public {
        bytes memory data = abi.encode(targetTradeContract, emulatedProfit);
```

* Testing the contracts

```typescript
import "dotenv/config";
import { viem } from "hardhat";
import { parseEther, formatEther } from "viem";

const FLASH_LOAN_FEE = 1000n;
const FLASH_LOAN_AMOUNT = 10n;
const flashLoanFeeString = ((FLASH_LOAN_FEE * 100n) / 10000n).toString() + "%";

async function main() {
  const publicClient = await viem.getPublicClient();
  const [signer] = await viem.getWalletClients();
  
  async function deployContracts() {
    console.log("Deploying FlashMint ERC20 Contract\n");
    const flashMintErc20Contract = await viem.deployContract("MyFlashMinter", [
      "Stonks Token",
      "Stt",
      signer.account.address,
      FLASH_LOAN_FEE,
    ]);
    console.log("Awaiting confirmations\n");
    console.log("Completed!\n");
    console.log(`Contract deployed at ${flashMintErc20Contract.address}\n`);
    console.log("Deploying FlashSwap Contract\n");
    const flashSwapContract = await viem.deployContract("MyFlashSwap", [
      flashMintErc20Contract.address,
    ]);
    console.log("Awaiting confirmations\n");
    console.log("Completed!\n");
    console.log(`Contract deployed at ${flashSwapContract.address}\n`);
    console.log("Deploying Magic Swap Faucet Contract\n");
    const magicSwapFaucetContract = await viem.deployContract("MagicSwapFaucet");
    console.log("Awaiting confirmations\n");
    console.log("Completed!\n");
    console.log(`Contract deployed at ${magicSwapFaucetContract.address}\n`);
    console.log(
      "Minting some tokens to the Magic Swap Faucet Contract at FlashMint ERC20 Contract\n"
    );
    const mintTokensTx = await flashMintErc20Contract.write.mint([
      magicSwapFaucetContract.address,
      parseEther((FLASH_LOAN_AMOUNT * 100n).toString()),
    ]);
    console.log("Awaiting confirmations\n");
    await publicClient.getTransactionReceipt({ hash: mintTokensTx });
    console.log("Completed!\n");
    return { flashMintErc20Contract, flashSwapContract, magicSwapFaucetContract };
  }

  const { flashMintErc20Contract, flashSwapContract, magicSwapFaucetContract } = await deployContracts();

  async function checkBalances() {
    console.log("Checking the current balances\n");
    const SignerBalanceBN = await flashMintErc20Contract.read.balanceOf([
      signer.account.address,
    ]);
    const SignerBalance = formatEther(SignerBalanceBN);
    const totalSupplyBN = await flashMintErc20Contract.read.totalSupply();
    const swapContractBalanceBN = await flashMintErc20Contract.read.balanceOf([
      flashSwapContract.address,
    ]);
    const magicSwapFaucetContractBalanceBN = await flashMintErc20Contract.read.balanceOf([
      magicSwapFaucetContract.address,
    ]);
    const totalSupply = formatEther(totalSupplyBN);
    const swapContractBalance = formatEther(swapContractBalanceBN);
    const magicSwapFaucetContractBalance = formatEther(
      magicSwapFaucetContractBalanceBN
    );
    console.log(`Total supply of tokens: ${totalSupply}\n`);
    console.log(
      `Current token balance of the contract deployer: ${SignerBalance} Stt\n`
    );
    console.log(
      `Current token balance inside the swap contract: ${swapContractBalance} Stt\n`
    );
    console.log(
      `Current token balance inside the magic swap faucet contract: ${magicSwapFaucetContractBalance} Stt\n`
    );
  }

  await checkBalances();

  async function doSwap() {
    console.log(
      `Initiating flash swap to borrow ${FLASH_LOAN_AMOUNT} Stt trying to profit ${FLASH_LOAN_AMOUNT / 2n} Stt\n`
    );
    const swapTx = await flashSwapContract.write.flashBorrow([
      flashMintErc20Contract.address,
      parseEther(FLASH_LOAN_AMOUNT.toString()),
      magicSwapFaucetContract.address,
      parseEther((FLASH_LOAN_AMOUNT / 2n).toString()),
    ]);
    console.log("Awaiting confirmations\n");
    const receipt = await publicClient.getTransactionReceipt({ hash: swapTx });
    const gasUsed = receipt.gasUsed;
    const gasPrice = receipt.effectiveGasPrice;
    console.log(
      `Flash Swap completed!\n\n${gasUsed} gas units spent (${formatEther(
        gasUsed * gasPrice
      )} ETH)\nPaid ${(FLASH_LOAN_AMOUNT * FLASH_LOAN_FEE) / 10000n} Stt of lending fees (${flashLoanFeeString})\n`
    );
  }

  await doSwap();

  await checkBalances();
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

