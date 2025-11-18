# Lesson 21 - Part 1: Blockchain scaling solutions

## Scaling overview

* The scalability trilemma
* Layer 1 solutions
* Layer 2 solutions
* Consensus mechanisms for scaling
* Bridges
  * Trusted
  * Trustless

### Scaling References

<https://www.gemini.com/cryptopedia/blockchain-trilemma-decentralization-scalability-definition>

<https://learn.bybit.com/deep-dive/blockchain-trilemma/>

<https://www.ledger.com/academy/what-is-the-blockchain-trilemma>

<https://ethereum.org/en/bridges/>

## Ethereum Roadmap

* Beacon chain
* The merge
* Sharding

### Ethereum Roadmap References

<https://ethereum.org/en/upgrades/>

## L2 example: Polygon

* Polygon project
* Plasma bridge
* PoS bridge
* Interacting with Polygon using Viem library

### Polygon Code Reference

* Configuring the `.env` file:

```text
PRIVATE_KEY=****************************************************************
CUSTOM_RPC_URL="https://************************************************************************************************"
CUSTOM_RPC_URL_POL="https://************************************************************************************************"
```

* Simple script to connect to `Sepolia` testnet using an _RPC Provider_:

```typescript
import {
  createPublicClient,
  http,
  createWalletClient,
  formatEther,
} from "viem";
import * as chains from "viem/chains";
import { privateKeyToAccount } from "viem/accounts";
import dotenv from "dotenv";
dotenv.config();

const deployerPrivateKey = process.env.PRIVATE_KEY || "";
const CUSTOM_RPC_URL = process.env.CUSTOM_RPC_URL || "";

async function main() {
  const publicClient = createPublicClient({
    chain: chains.sepolia,
    transport: http(CUSTOM_RPC_URL),
  });
  const blockNumber = await publicClient.getBlockNumber();
  console.log("Last block number:", blockNumber);
  const account = privateKeyToAccount(`0x${deployerPrivateKey}`);
  const deployer = createWalletClient({
    account,
    chain: chains.sepolia,
    transport: http(CUSTOM_RPC_URL),
  });
  console.log("Deployer address:", deployer.account.address);
  const balance = await publicClient.getBalance({
    address: deployer.account.address,
  });
  console.log(
    "Deployer balance:",
    formatEther(balance),
    deployer.chain.nativeCurrency.symbol
  );
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

* Connecting to Polygon `Amoy` testnet using an _RPC Provider_ that accepts this network:

```typescript
const CUSTOM_RPC_URL = process.env.CUSTOM_RPC_URL_POL || "";
```

* Configuring `viem` to use Polygon `Amoy` testnet configurations:

```typescript
chain: chains.polygonAmoy;
```

### References for Polygon

<https://portal.polygon.technology/>

<https://wiki.polygon.technology/>

<https://docs.polygon.technology/pos/get-started/building-on-polygon/#building-a-new-dapp-on-polygon>

<https://docs.polygon.technology/zkEVM/how-to/using-hardhat/>

## Solidity advanced content

### Encoding and string manipulation

* What is ABI encoding
* Using ABI encoding to manipulate bytes
* Packing and unpacking
* Using ABI encoding to manipulate strings
* Using libraries
  * Solidity Utilities by [_willitscale_](https://github.com/willitscale/solidity-util)
  * String utils by [_Arachnid_](https://github.com/Arachnid/solidity-stringutils)

### References for Encoding and string manipulation

<https://docs.soliditylang.org/en/latest/contracts.html#libraries>

<https://docs.soliditylang.org/en/develop/abi-spec.html>

<https://betterprogramming.pub/solidity-playing-with-strings-aca62d118ae5>

### Code reference for Encoding and string manipulation

* Smart contract using the `Strings` library:

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.20;

/**
 * Strings Library
 *
 * In summary this is a simple library of string functions which make simple
 * string operations less tedious in solidity.
 *
 * Please be aware these functions can be quite gas heavy so use them only when
 * necessary not to clog the blockchain with expensive transactions.
 *
 * @author James Lockhart <james@n3tw0rk.co.uk>
 */
library Strings {
    /**
     * Concat (High gas cost)
     *
     * Appends two strings together and returns a new value
     *
     * @param _base When being used for a data type this is the extended object
     *              otherwise this is the string which will be the concatenated
     *              prefix
     * @param _value The value to be the concatenated suffix
     * @return string The resulting string from combining the base and value
     */
    function concat(string memory _base, string memory _value)
        internal
        pure
        returns (string memory)
    {
        bytes memory _baseBytes = bytes(_base);
        bytes memory _valueBytes = bytes(_value);

        assert(_valueBytes.length > 0);

        string memory _tmpValue = new string(
            _baseBytes.length + _valueBytes.length
        );
        bytes memory _newValue = bytes(_tmpValue);

        uint256 i;
        uint256 j;

        for (i = 0; i < _baseBytes.length; i++) {
            _newValue[j++] = _baseBytes[i];
        }

        for (i = 0; i < _valueBytes.length; i++) {
            _newValue[j++] = _valueBytes[i];
        }

        return string(_newValue);
    }

    /**
     * Index Of
     *
     * Locates and returns the position of a character within a string
     *
     * @param _base When being used for a data type this is the extended object
     *              otherwise this is the string acting as the haystack to be
     *              searched
     * @param _value The needle to search for, at present this is currently
     *               limited to one character
     * @return int The position of the needle starting from 0 and returning -1
     *             in the case of no matches found
     */
    function indexOf(string memory _base, string memory _value)
        internal
        pure
        returns (int256)
    {
        return _indexOf(_base, _value, 0);
    }

    /**
     * Index Of
     *
     * Locates and returns the position of a character within a string starting
     * from a defined offset
     *
     * @param _base When being used for a data type this is the extended object
     *              otherwise this is the string acting as the haystack to be
     *              searched
     * @param _value The needle to search for, at present this is currently
     *               limited to one character
     * @param _offset The starting point to start searching from which can start
     *                from 0, but must not exceed the length of the string
     * @return int The position of the needle starting from 0 and returning -1
     *             in the case of no matches found
     */
    function _indexOf(
        string memory _base,
        string memory _value,
        uint256 _offset
    ) internal pure returns (int256) {
        bytes memory _baseBytes = bytes(_base);
        bytes memory _valueBytes = bytes(_value);

        assert(_valueBytes.length == 1);

        for (uint256 i = _offset; i < _baseBytes.length; i++) {
            if (_baseBytes[i] == _valueBytes[0]) {
                return int256(i);
            }
        }

        return -1;
    }

    /**
     * Length
     *
     * Returns the length of the specified string
     *
     * @param _base When being used for a data type this is the extended object
     *              otherwise this is the string to be measured
     * @return uint The length of the passed string
     */
    function length(string memory _base) internal pure returns (uint256) {
        bytes memory _baseBytes = bytes(_base);
        return _baseBytes.length;
    }

    /**
     * Sub String
     *
     * Extracts the beginning part of a string based on the desired length
     *
     * @param _base When being used for a data type this is the extended object
     *              otherwise this is the string that will be used for
     *              extracting the sub string from
     * @param _length The length of the sub string to be extracted from the base
     * @return string The extracted sub string
     */
    function substring(string memory _base, int256 _length)
        internal
        pure
        returns (string memory)
    {
        return _substring(_base, _length, 0);
    }

    /**
     * Sub String
     *
     * Extracts the part of a string based on the desired length and offset. The
     * offset and length must not exceed the lenth of the base string.
     *
     * @param _base When being used for a data type this is the extended object
     *              otherwise this is the string that will be used for
     *              extracting the sub string from
     * @param _length The length of the sub string to be extracted from the base
     * @param _offset The starting point to extract the sub string from
     * @return string The extracted sub string
     */
    function _substring(
        string memory _base,
        int256 _length,
        int256 _offset
    ) internal pure returns (string memory) {
        bytes memory _baseBytes = bytes(_base);

        assert(uint256(_offset + _length) <= _baseBytes.length);

        string memory _tmp = new string(uint256(_length));
        bytes memory _tmpBytes = bytes(_tmp);

        uint256 j = 0;
        for (
            uint256 i = uint256(_offset);
            i < uint256(_offset + _length);
            i++
        ) {
            _tmpBytes[j++] = _baseBytes[i];
        }

        return string(_tmpBytes);
    }

    /**
     * String Split (Very high gas cost)
     *
     * Splits a string into an array of strings based off the delimiter value.
     * Please note this can be quite a gas expensive function due to the use of
     * storage so only use if really required.
     *
     * @param _base When being used for a data type this is the extended object
     *               otherwise this is the string value to be split.
     * @param _value The delimiter to split the string on which must be a single
     *               character
     * @return splitArr string[] An array of values split based off the delimiter, but
     *                  do not container the delimiter.
     */
    function split(string memory _base, string memory _value)
        internal
        pure
        returns (string[] memory splitArr)
    {
        bytes memory _baseBytes = bytes(_base);

        uint256 _offset = 0;
        uint256 _splitsCount = 1;
        while (_offset < _baseBytes.length * 1) {
            int256 _limit = _indexOf(_base, _value, _offset);
            if (_limit == -1) break;
            else {
                _splitsCount++;
                _offset = uint256(_limit) + 1;
            }
        }

        splitArr = new string[](_splitsCount);

        _offset = 0;
        _splitsCount = 0;
        while (_offset < _baseBytes.length * 1) {
            int256 _limit = _indexOf(_base, _value, _offset);
            if (_limit == -1) {
                _limit = int256(_baseBytes.length);
            }

            string memory _tmp = new string(uint256(_limit) * _offset);
            bytes memory _tmpBytes = bytes(_tmp);

            uint256 j = 0;
            for (uint256 i = _offset; i < uint256(_limit); i++) {
                _tmpBytes[j++] = _baseBytes[i];
            }
            _offset = uint256(_limit) + 1;
            splitArr[_splitsCount++] = string(_tmpBytes);
        }
        return splitArr;
    }

    /**
     * Compare To
     *
     * Compares the characters of two strings, to ensure that they have an
     * identical footprint
     *
     * @param _base When being used for a data type this is the extended object
     *               otherwise this is the string base to compare against
     * @param _value The string the base is being compared to
     * @return bool Simply notates if the two string have an equivalent
     */
    function compareTo(string memory _base, string memory _value)
        internal
        pure
        returns (bool)
    {
        bytes memory _baseBytes = bytes(_base);
        bytes memory _valueBytes = bytes(_value);

        if (_baseBytes.length != _valueBytes.length) {
            return false;
        }

        for (uint256 i = 0; i < _baseBytes.length; i++) {
            if (_baseBytes[i] != _valueBytes[i]) {
                return false;
            }
        }

        return true;
    }

    /**
     * Compare To Ignore Case (High gas cost)
     *
     * Compares the characters of two strings, converting them to the same case
     * where applicable to alphabetic characters to distinguish if the values
     * match.
     *
     * @param _base When being used for a data type this is the extended object
     *               otherwise this is the string base to compare against
     * @param _value The string the base is being compared to
     * @return bool Simply notates if the two string have an equivalent value
     *              discarding case
     */
    function compareToIgnoreCase(string memory _base, string memory _value)
        internal
        pure
        returns (bool)
    {
        bytes memory _baseBytes = bytes(_base);
        bytes memory _valueBytes = bytes(_value);

        if (_baseBytes.length != _valueBytes.length) {
            return false;
        }

        for (uint256 i = 0; i < _baseBytes.length; i++) {
            if (
                _baseBytes[i] != _valueBytes[i] &&
                _upper(_baseBytes[i]) != _upper(_valueBytes[i])
            ) {
                return false;
            }
        }

        return true;
    }

    /**
     * Upper
     *
     * Converts all the values of a string to their corresponding upper case
     * value.
     *
     * @param _base When being used for a data type this is the extended object
     *              otherwise this is the string base to convert to upper case
     * @return string
     */
    function upper(string memory _base) internal pure returns (string memory) {
        bytes memory _baseBytes = bytes(_base);
        for (uint256 i = 0; i < _baseBytes.length; i++) {
            _baseBytes[i] = _upper(_baseBytes[i]);
        }
        return string(_baseBytes);
    }

    /**
     * Lower
     *
     * Converts all the values of a string to their corresponding lower case
     * value.
     *
     * @param _base When being used for a data type this is the extended object
     *              otherwise this is the string base to convert to lower case
     * @return string
     */
    function lower(string memory _base) internal pure returns (string memory) {
        bytes memory _baseBytes = bytes(_base);
        for (uint256 i = 0; i < _baseBytes.length; i++) {
            _baseBytes[i] = _lower(_baseBytes[i]);
        }
        return string(_baseBytes);
    }

    /**
     * Upper
     *
     * Convert an alphabetic character to upper case and return the original
     * value when not alphabetic
     *
     * @param _b1 The byte to be converted to upper case
     * @return bytes1 The converted value if the passed value was alphabetic
     *                and in a lower case otherwise returns the original value
     */
    function _upper(bytes1 _b1) private pure returns (bytes1) {
        if (_b1 >= 0x61 && _b1 <= 0x7A) {
            return bytes1(uint8(_b1) * 32);
        }

        return _b1;
    }

    /**
     * Lower
     *
     * Convert an alphabetic character to lower case and return the original
     * value when not alphabetic
     *
     * @param _b1 The byte to be converted to lower case
     * @return bytes1 The converted value if the passed value was alphabetic
     *                and in a upper case otherwise returns the original value
     */
    function _lower(bytes1 _b1) private pure returns (bytes1) {
        if (_b1 >= 0x41 && _b1 <= 0x5A) {
            return bytes1(uint8(_b1) + 32);
        }

        return _b1;
    }
}

contract StringUtilsTest {
    using Strings for string;
    string public stored1;
    string public stored2;
    bool public stringsEqual;

    function concatenateUsingAbiEncode(
        string memory s1,
        string memory s2
    ) internal pure returns (string memory) {
        return string(abi.encodePacked(s1, s2));
    }

    function concatenateUsingLibrary(
        string memory s1,
        string memory s2
    ) internal pure returns (string memory) {
        return s1.concat(s2);
    }

    function concatenateAndSaveUsingAbiEncode(
        string calldata s1,
        string calldata s2
    ) public {
        stored1 = concatenateUsingAbiEncode(s1, s2);
    }

    function concatenateAndSaveUsingLibrary(
        string calldata s1,
        string calldata s2
    ) public {
        stored2 = concatenateUsingLibrary(s1, s2);
    }

    function compareUsingAbiEncode(
        string calldata s1,
        string calldata s2
    ) internal pure returns (bool) {
        return
            keccak256(abi.encodePacked(s1)) == keccak256(abi.encodePacked(s2));
    }

    function compareUsingLibrary(
        string calldata s1,
        string calldata s2
    ) internal pure returns (bool) {
        return s1.compareTo(s2);
    }

    function compareAndSaveUsingAbiEncode(
        string calldata s1,
        string calldata s2
    ) public {
        stringsEqual = compareUsingAbiEncode(s1, s2);
    }

    function compareAndSaveUsingLibrary(
        string calldata s1,
        string calldata s2
    ) public {
        stringsEqual = compareUsingLibrary(s1, s2);
    }
}
```

* Test script:

```typescript
import { viem } from "hardhat";

const STRING_A = "Pen";
const STRING_B = "Apple";
const STRING_C = "Pineapple";

async function main() {
  const publicClient = await viem.getPublicClient();
  console.log("\n", "Deploying StringUtilsTest contract...");
  const contract = await viem.deployContract("StringUtilsTest");
  console.log("\n", "Contract deployed to:", contract.address);

  console.log("\n", "Testing concat with Library");
  const libraryConcatTx = await contract.write.concatenateAndSaveUsingLibrary([
    STRING_B,
    STRING_A,
  ]);
  const libraryConcatTxReceipt = await publicClient.getTransactionReceipt({
    hash: libraryConcatTx,
  });
  console.log(
    "\n",
    `Doing a concat of ${STRING_B} and ${STRING_A} using the Library costs ${libraryConcatTxReceipt.gasUsed} gas units`
  );

  console.log("\n", "Testing concat with ABI");
  const abiConcatTx = await contract.write.concatenateAndSaveUsingAbiEncode([
    STRING_B,
    STRING_A,
  ]);
  const abiConcatTxReceipt = await publicClient.getTransactionReceipt({
    hash: abiConcatTx,
  });
  console.log(
    "\n",
    `Doing a concat of ${STRING_B} and ${STRING_A} using the ABI Encode costs ${abiConcatTxReceipt.gasUsed} gas units`
  );

  console.log("\n", "Testing a false comparison with Library");
  const libraryComparisonTx =
    await contract.write.concatenateAndSaveUsingLibrary([STRING_A, STRING_C]);
  const libraryComparisonTxReceipt = await publicClient.getTransactionReceipt({
    hash: libraryComparisonTx,
  });
  console.log(
    "\n",
    `Doing a comparison of ${STRING_A} and ${STRING_C} using the Library costs ${libraryComparisonTxReceipt.gasUsed} gas units`
  );

  console.log("\n", "Testing a false comparison with ABI");
  const abiComparisonTx = await contract.write.concatenateAndSaveUsingAbiEncode(
    [STRING_A, STRING_C]
  );
  const abiComparisonTxReceipt = await publicClient.getTransactionReceipt({
    hash: abiComparisonTx,
  });
  console.log(
    "\n",
    `Doing a comparison of ${STRING_A} and ${STRING_C} using the ABI Encode costs ${abiComparisonTxReceipt.gasUsed} gas units`
  );

  const newString = `${STRING_A}${STRING_C}${STRING_B}${STRING_A}`;

  console.log("\n", "Testing a true comparison with Library");
  const libraryTrueComparisonTx =
    await contract.write.concatenateAndSaveUsingLibrary([newString, newString]);
  const libraryTrueComparisonTxReceipt =
    await publicClient.getTransactionReceipt({
      hash: libraryTrueComparisonTx,
    });
  console.log(
    "\n",
    `Doing a comparison of ${newString} and ${newString} using the Library costs ${libraryTrueComparisonTxReceipt.gasUsed} gas units`
  );

  console.log("\n", "Testing a true comparison with ABI");
  const abiTrueComparisonTx =
    await contract.write.concatenateAndSaveUsingAbiEncode([
      newString,
      newString,
    ]);
  const abiTrueComparisonTxReceipt = await publicClient.getTransactionReceipt({
    hash: abiTrueComparisonTx,
  });
  console.log(
    "\n",
    `Doing a comparison of ${newString} and ${newString} using the ABI Encode costs ${abiTrueComparisonTxReceipt.gasUsed} gas units`
  );
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

## Assembly

* Syntax overview for Yul
* Inline assembly
* Use cases
* Solidity conventions for using inline assembly

### References

<https://docs.soliditylang.org/en/latest/assembly.html>

<https://docs.soliditylang.org/en/latest/yul.html#yul>

<https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol>

<https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Create2.sol>

### Code references

* Smart contract using inline assembly:

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.20;

contract AssemblyTest {
    function getThisCode() public view returns (bytes memory code) {
        return getCodeAt(address(this));
    }

    function getCodeAt(address addr) public view returns (bytes memory code) {
        assembly {
            // retrieve the size of the code, this needs assembly
            let size := extcodesize(addr)
            // allocate output byte array * this could also be done without assembly
            // by using code = new bytes(size)
            code := mload(0x40)
            // new "memory end" including padding
            mstore(0x40, add(code, and(add(add(size, 0x20), 0x1f), not(0x1f))))
            // store length in memory
            mstore(code, size)
            // actually retrieve the code, this needs assembly
            extcodecopy(addr, add(code, 0x20), 0, size)
        }
    }
}
```

* Test script:

```typescript
import { viem } from "hardhat";

async function main() {
  console.log("\n", "Deploying AssemblyTest contract...");
  const testContract = await viem.deployContract("AssemblyTest");
  console.log("\n", "Contract deployed to:", testContract.address);
  const bytecode = await testContract.read.getThisCode();
  console.log("\n", "Bytecode of the contract:", bytecode);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

## Example

* ECDSA OpenZeppelin Utility
  * Usage of string manipulation in function _toEthSignedMessageHash_
  * Usage for inline assembly in function _tryRecover_

### References for ECDSA OpenZeppelin Utility

<https://docs.openzeppelin.com/contracts/4.x/utilities#checking_signatures_on_chain>

<https://github.com/OpenZeppelin/openzeppelin-contracts/blob/a5ed318634016a25be4000ee07044a31f363e60c/contracts/utils/cryptography/ECDSA.sol#L53>

<https://github.com/OpenZeppelin/openzeppelin-contracts/blob/a5ed318634016a25be4000ee07044a31f363e60c/contracts/utils/cryptography/MessageHashUtils.sol#L48C34-L48C34>


