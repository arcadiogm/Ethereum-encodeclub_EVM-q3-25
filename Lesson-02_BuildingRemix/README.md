# Lesson 2 - Building HelloWorld.sol in Remix

## Coding HelloWorld.sol contract with Solidity

* Solidity philosophy
* OOP basics of Solidity
* Contract Structure
  * SPDX License Identifier
  * Pragmas
  * Imports
  * Comments
  * Contract definition
* Variables
* Storage areas
  * Account storage
  * Memory
  * Stack
* Constructor function
* Functions
* Visibility
* Typing
* Return values

## Detailed contract structure

* (Review) Contract structure
* (Review) Variables
* (Review) Storage
* What are objects (contracts and interfaces)
* Multiple objects per file

### References for Solidity and Smart Contracts

<https://docs.soliditylang.org/en/latest/grammar.html>

<https://docs.soliditylang.org/en/latest/structure-of-a-contract.html>

<https://docs.soliditylang.org/en/latest/contracts.html#visibility-and-getters>

<https://docs.soliditylang.org/en/latest/contracts.html#functions>

<https://docs.soliditylang.org/en/latest/contracts.html#interfaces>

### Code Reference

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.7.0 <0.9.0;

contract HelloWorld {
    string private text;

    constructor() {
        text = "Hello World";
    }

    function helloWorld() public view returns (string memory)  {
        return text;
    }

    function setText(string memory newText) public {
        text = newText;
    }
}
```

## Contract interaction

### Part 1

* Contract interactions using virtual environments in Remix
* Viewing state changes through functions

### Part 2

* State changing calls in public testnets
* Using interfaces to interact with other contracts

```solidity
interface HelloWorldInterface {
    function helloWorld() external view returns (string memory);
    function setText(string memory newText) external;
}

contract HelloWorld is HelloWorldInterface {
    ...
}
```

### Part 3

* Payable functions
* Experimenting with payable calls

```solidity
function setText(string memory newText) public payable {
    text = newText;
}
```

## Clean code and documentation

* The Style guide
* Documentation
* NatSpec Format

### References for clean code and documentation

<https://docs.soliditylang.org/en/latest/style-guide.html>

<https://docs.soliditylang.org/en/latest/natspec-format.html#natspec>

