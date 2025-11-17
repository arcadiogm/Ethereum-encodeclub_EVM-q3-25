# Lesson 1 - Introduction

## Wallets and transactions

* What is a wallet
* Finding wallet applications
* Using metamasks
* Creating a public-private key pair
* Connecting to Sepolia testnet
* Funding my wallet with a Faucet
* Sending a native token (ETH) transaction

## How ethereum works

* Explaining Etherscan
* Transactions
* Gas
* Blocks
* Consensus and finality
* State changes
* The EVM
* Accounts

## Trying it out

* Using Uniswap as an example
* What is a dApp
* Swap transaction example

## Contract interaction: Token Swap

* What is a state change inside a smart contract operation

## Hands on coding - Remix

* Remix interface (overview)

### References for Remix

<https://remix-ide.readthedocs.io/en/latest/>

## Coding HelloWorld.sol contract with Solidity

* Solidity philosophy
* OOP basics of Solidity
* Contract Structure
  * SPDX License Identifier
  * Pragmas
  * Imports
  * Comments
  * Contract definition
* State Variables
* Contract Storage
* Constructor function
* Functions
* Return values

### Code Reference

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.7.0 <0.9.0;

contract HelloWorld {

    constructor() {}

    function helloWorld() public view returns (string memory) {}

}
```

### References for Solidity

<https://docs.soliditylang.org/en/latest/>

## Compiling and deploying

* Compilation parameters
  * Compiler version
  * EVM Version
  * Optimization
* Bytecode
* ABI
* Deployment parameters
  * Environment
  * Account
  * Gas
  * Contract
* Deploying
* Attaching
