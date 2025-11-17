# Lesson 3 - Syntax and structure

## Important concepts

* Storage locations
  * Account storage
* Replacing memory with calldata for saving gas
* Definitions
  * Visibility
  * State mutability
  * Modifiers
  * Virtual
  * Override

### Solidity Concepts References

<https://docs.soliditylang.org/en/latest/introduction-to-smart-contracts.html#accounts>

<https://docs.soliditylang.org/en/latest/introduction-to-smart-contracts.html#storage-memory-and-the-stack>

<https://docs.soliditylang.org/en/latest/types.html#bytes-and-string-as-arrays>

### Code Examples

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

    function setText(string calldata newText) public {
        text = newText;
    }
}
```

### Variable and Function References

<https://docs.soliditylang.org/en/latest/types.html>

<https://docs.soliditylang.org/en/latest/contracts.html#visibility-and-getters>

<https://docs.soliditylang.org/en/latest/contracts.html#functions>

<https://docs.soliditylang.org/en/latest/cheatsheet.html#mathematical-and-cryptographic-functions>

## Exercises

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.7.0 <0.9.0;

contract HelloWorld {
    string private text;

    constructor() {
        text = initialText();
    }
    
    function helloWorld() public view returns (string memory) {
        return text;
    }

    function setText(string calldata newText) public {
        text = newText;
    }

    function initialText() public pure returns (string memory) {
        return "Hello World";
    }

    function _isUnchanged() internal view returns (bool) {
        return keccak256(bytes(text)) == keccak256(bytes(initialText()));
    }

    function textHasChanged() public view returns (bool returnValue_) {
        returnValue_ = !_isUnchanged();
    }

    function _restore() internal {
        text = initialText();
    }
    
    function restore() public returns (bool) {
        if (_isUnchanged()) return false;
        _restore();
        return true;
    }
}
```

## Assertion and Modifiers

* How errors are handled on solidity (briefly)
* Assertion
* Require statements
* Modifiers
* Where to use modifiers

### Error Handling References

<https://docs.soliditylang.org/en/latest/control-structures.html#error-handling-assert-require-revert-and-exceptions>

<https://docs.soliditylang.org/en/latest/structure-of-a-contract.html#function-modifiers>

## Blockchain Global Variables References

* Reserved words and global variables that a programmer should know
* Global variables about blockchain state
* Global variables about the transaction
* Global variables about the transaction message

<https://docs.soliditylang.org/en/latest/units-and-global-variables.html>

<https://docs.soliditylang.org/en/latest/introduction-to-smart-contracts.html#index-8>

<https://docs.soliditylang.org/en/latest/introduction-to-smart-contracts.html#blocks>


