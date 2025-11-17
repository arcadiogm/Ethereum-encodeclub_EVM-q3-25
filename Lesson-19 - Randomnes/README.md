# Lesson 19 - Randomness

## Randomness sources

* (Review) Block hash and timestamp
* (Review) Hashing and bytes conversion
* Creating a random number from block.hash and/or block.timestamp
* Mining exploitation
* False randomness

### Code references

Coding `NotQuiteRandom.sol`:

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.7.0 <0.9.0;

contract NotQuiteRandom {
    function getRandomNumber()
        public
        view
        returns (uint256 notQuiteRandomNumber)
    {
        // TODO: get randomness from block hash
    }

    function tossCoin() public view returns (bool heads) {
        // TODO: make the random number be translated to a boolean
    }
}
```

Coding `getRandomNumber.ts`:

```typescript
import { viem } from "hardhat";
import { recoverMessageAddress, hexToSignature } from "viem";
import * as readline from "readline";
import { mine } from "@nomicfoundation/hardhat-network-helpers";

async function blockHashRandomness() {
  const publicClient = await viem.getPublicClient();
  const currentBlock = await publicClient.getBlock();
  const contract = await viem.deployContract("NotQuiteRandom");
  const randomNumber = await contract.read.getRandomNumber();
  console.log(
    `Block number: ${currentBlock?.number}\nBlock hash: ${currentBlock?.hash}\nRandom number from this block hash: ${randomNumber}`
  );
  await mine(1);
  const currentBlock2 = await publicClient.getBlock();
  const randomNumber2 = await contract.read.getRandomNumber();
  console.log(
    `Block number: ${currentBlock2?.number}\nBlock hash: ${currentBlock2?.hash}\nRandom number from this block hash: ${randomNumber2}`
  );
  await mine(1);
  const currentBlock3 = await publicClient.getBlock();
  const randomNumber3 = await contract.read.getRandomNumber();
  console.log(
    `Block number: ${currentBlock3?.number}\nBlock hash: ${currentBlock3?.hash}\nRandom number from this block hash: ${randomNumber3}`
  );
  await mine(1);
  const currentBlock4 = await publicClient.getBlock();
  const randomNumber4 = await contract.read.getRandomNumber();
  console.log(
    `Block number: ${currentBlock4?.number}\nBlock hash: ${currentBlock4?.hash}\nRandom number from this block hash: ${randomNumber4}`
  );
  await mine(1);
  const currentBlock5 = await publicClient.getBlock();
  const randomNumber5 = await contract.read.getRandomNumber();
  console.log(
    `Block number: ${currentBlock5?.number}\nBlock hash: ${currentBlock5?.hash}\nRandom number from this block hash: ${randomNumber5}`
  );
}

async function tossCoin() {
  const publicClient = await viem.getPublicClient();
  const currentBlock = await publicClient.getBlock();
  const contract = await viem.deployContract("NotQuiteRandom");
  const heads = await contract.read.tossCoin();
  console.log(
    `Block number: ${currentBlock?.number}\nBlock hash: ${
      currentBlock?.hash
    }\nThe coin landed as: ${heads ? "Heads" : "Tails"}`
  );
  await mine(1);
  const currentBlock2 = await publicClient.getBlock();
  const heads2 = await contract.read.tossCoin();
  console.log(
    `Block number: ${currentBlock2?.number}\nBlock hash: ${
      currentBlock2?.hash
    }\nThe coin landed as: ${heads2 ? "Heads" : "Tails"}`
  );
  await mine(1);
  const currentBlock3 = await publicClient.getBlock();
  const heads3 = await contract.read.tossCoin();
  console.log(
    `Block number: ${currentBlock3?.number}\nBlock hash: ${
      currentBlock3?.hash
    }\nThe coin landed as: ${heads3 ? "Heads" : "Tails"}`
  );
  await mine(1);
  const currentBlock4 = await publicClient.getBlock();
  const heads4 = await contract.read.tossCoin();
  console.log(
    `Block number: ${currentBlock4?.number}\nBlock hash: ${
      currentBlock4?.hash
    }\nThe coin landed as: ${heads4 ? "Heads" : "Tails"}`
  );
  await mine(1);
  const currentBlock5 = await publicClient.getBlock();
  const heads5 = await contract.read.tossCoin();
  console.log(
    `Block number: ${currentBlock5?.number}\nBlock hash: ${
      currentBlock5?.hash
    }\nThe coin landed as: ${heads5 ? "Heads" : "Tails"}`
  );
}

blockHashRandomness().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

### Randomness Best Practices References

<https://github.com/wissalHaji/solidity-coding-advices/blob/master/best-practices/timestamp-can-be-manipulated.md>

<https://fravoll.github.io/solidity-patterns/randomness.html>

## Organizing the script with _readline_

* Operation scripts
* User input
* Async logic

### References for _readline_

<https://nodejs.org/api/readline.html#readline>

<https://nodejs.org/api/readline.html#readline_example_tiny_cli>

### Code references for using _readline_

```typescript
async function main() {
  const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout,
  });
  rl.question(
    "Select operation: \n Options: \n [1]: Random from block hash \n [2]: Toss a coin \n [3]: Message signature \n [4]: Random from a sealed seed \n [5]: Random from block hash plus a sealed seed \n [6]: Random from randao \n",
    (answer) => {
      console.log(`Selected: ${answer}`);
      const option = Number(answer);
      switch (option) {
        case 1:
          blockHashRandomness();
          break;
        case 2:
          tossCoin();
          break;
        case 3:
          // TODO
          break;
        case 4:
          // TODO
          break;
        case 5:
          // TODO
          break;
        case 6:
          // TODO
          break;
        default:
          console.log("Invalid");
          break;
      }
      rl.close();
    }
  );
}
```

## Using signatures

* Elliptic encryption (review)
* Signing messages (review)
* Verifying signatures (review)
* Workaround for information asymmetry with commit-reveal

### Code references for using signatures

Coding `PseudoRandom.sol`:

```solidity
//SPDX-License-Identifier: MIT
pragma solidity >=0.7.0 <0.9.0;

import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";

contract PseudoRandom {
    using ECDSA for bytes32;

    struct Signature {
        uint8 v;
        bytes32 r;
        bytes32 s;
    }

    Signature savedSig;

    function setSignature(
        uint8 _v,
        bytes32 _r,
        bytes32 _s
    ) public {
        savedSig = Signature({v: _v, r: _r, s: _s});
    }

    function getRandomNumber(string calldata seed)
        public
        view
        returns (uint256 pseudoRandomNumber)
    {
        address messageSigner = verifyString(
            seed,
            savedSig.v,
            savedSig.r,
            savedSig.s
        );
        require(msg.sender == messageSigner, "Invalid seed");
        pseudoRandomNumber = uint256(keccak256(abi.encodePacked(seed)));
    }

    function getCombinedRandomNumber(string calldata seed)
        public
        view
        returns (uint256 pseudoRandomNumber)
    {
        address messageSigner = verifyString(
            seed,
            savedSig.v,
            savedSig.r,
            savedSig.s
        );
        require(msg.sender == messageSigner, "Invalid seed");
        pseudoRandomNumber = uint256(
            keccak256(abi.encodePacked(seed, blockhash(block.number - 1)))
        );
    }

    // Don't worry about this function for now
    function verifyString(
        string memory message,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) public pure returns (address signer) {
        string memory header = "\x19Ethereum Signed Message:\n000000";
        uint256 lengthOffset;
        uint256 length;
        assembly {
            length := mload(message)
            lengthOffset := add(header, 57)
        }
        require(length <= 999999);
        uint256 lengthLength = 0;
        uint256 divisor = 100000;
        while (divisor != 0) {
            uint256 digit = length / divisor;
            if (digit == 0) {
                if (lengthLength == 0) {
                    divisor /= 10;
                    continue;
                }
            }
            lengthLength++;
            length -= digit * divisor;
            divisor /= 10;
            digit += 0x30;
            lengthOffset++;
            assembly {
                mstore8(lengthOffset, digit)
            }
        }
        if (lengthLength == 0) {
            lengthLength = 1 + 0x19 + 1;
        } else {
            lengthLength += 1 + 0x19;
        }
        assembly {
            mstore(header, lengthLength)
        }
        bytes32 check = keccak256(abi.encodePacked(header, message));
        return ecrecover(check, v, r, s);
    }
}
```

Modifying the script:

```typescript
async function signature() {
  const signers = await viem.getWalletClients();
  const signer = signers[0];
  console.log(
    `Signing a message with the account of address ${signer.account.address}`
  );
  const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout,
  });
  rl.question("Enter a message to be signed:\n", async (answer) => {
    const signedMessage = await signer.signMessage({ message: answer });
    console.log(`The signed message is:\n${signedMessage}`);
    rl.close();
    testSignature();
  });
}

async function testSignature() {
  console.log("Verifying signature\n");
  const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout,
  });
  rl.question("Enter message signature:\n", (userSignature) => {
    rl.question("Enter message:\n", async (message) => {
      const signature = userSignature as `0x${string}` | Uint8Array;
      const address = await recoverMessageAddress({ message, signature });
      console.log(`This message signature matches with address ${address}`);
      rl.question("Repeat? [Y/N]:\n", (answer) => {
        rl.close();
        if (answer.toLowerCase() === "y") {
          testSignature();
        }
      });
    });
  });
}

async function sealedSeed() {
  console.log("Deploying contract");
  const contract = await viem.deployContract("PseudoRandom");
  const signers = await viem.getWalletClients();
  const signer = signers[0];
  console.log(
    `Signing a message with the account of address ${signer.account.address}`
  );
  const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout,
  });
  rl.question("Enter a random seed to be signed:\n", async (seed) => {
    const signedMessage = await signer.signMessage({ message: seed });
    rl.close();
    console.log(`The signed message is:\n${signedMessage}`);
    const sig = hexToSignature(signedMessage);
    console.log("Saving signature at contract");
    const sigV = sig.v ? Number(sig.v) : 0;
    await contract.write.setSignature([sigV, sig.r, sig.s]);
    try {
      console.log("Trying to get a number with the original seed");
      const randomNumber = await contract.read.getRandomNumber([seed]);
      console.log(`Random number result:\n${randomNumber}`);
      console.log("Trying to get a number without the original seed");
      const fakeSeed = "FAKE_SEED";
      const randomNumber2 = await contract.read.getRandomNumber([fakeSeed]);
      console.log(`Random number result:\n${randomNumber2}`);
    } catch (error) {
      console.log("Operation failed");
    }
  });
}

async function randomSealedSeed() {
  console.log("Deploying contract");
  const contract = await viem.deployContract("PseudoRandom");
  const signers = await viem.getWalletClients();
  const signer = signers[0];
  console.log(
    `Signing a message with the account of address ${signer.account.address}`
  );
  const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout,
  });
  rl.question("Enter a random seed to be signed:\n", async (seed) => {
    const signedMessage = await signer.signMessage({ message: seed });
    rl.close();
    console.log(`The signed message is:\n${signedMessage}`);
    const sig = hexToSignature(signedMessage);
    console.log("Saving signature at contract");
    const sigV = sig.v ? Number(sig.v) : 0;
    await contract.write.setSignature([sigV, sig.r, sig.s]);
    try {
      console.log("Trying to get a number with the original seed");
      const randomNumber = await contract.read.getCombinedRandomNumber([seed]);
      console.log(`Random number result:\n${randomNumber}`);
      console.log("Trying to get a number without the original seed");
      const fakeSeed = "FAKE_SEED";
      const randomNumber2 = await contract.read.getCombinedRandomNumber([
        fakeSeed,
      ]);
      console.log(`Random number result:\n${randomNumber2}`);
    } catch (error) {
      console.log("Operation failed");
    }
  });
}
```

### Signature Randomness References

<https://viem.sh/docs/actions/wallet/signMessage.html>

<https://blockchain-academy.hs-mittweida.de/courses/solidity-coding-beginners-to-intermediate/lessons/solidity-11-coding-patterns/topic/commit-reveal/>

## Using RANDAO

* Block consensus info
* Beacon chain data

### Code references for using RANDAO

Random.sol:

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.7.0 <0.9.0;

contract Random {
    function getRandomNumber()
        public
        view
        returns (uint256 randomNumber)
    {
        randomNumber = block.prevrandao;
    }

    function tossCoin() public view returns (bool heads) {
        heads = getRandomNumber() % 2 == 0;
    }
}
```

Scripts:

```typescript
async function randao() {
  const publicClient = await viem.getPublicClient();
  console.log("Deploying contract");
  const contract = await viem.deployContract("Random");
  const currentBlock = await publicClient.getBlock();
  const randomNumber = await contract.read.getRandomNumber();
  console.log(
    `Block number: ${currentBlock?.number}\nBlock difficulty: ${currentBlock?.difficulty}\nRandom number from this block difficulty: ${randomNumber}`
  );
  await mine(1);
  const currentBlock2 = await publicClient.getBlock();
  const randomNumber2 = await contract.read.getRandomNumber();
  console.log(
    `Block number: ${currentBlock2?.number}\nBlock difficulty: ${currentBlock2?.difficulty}\nRandom number from this block difficulty: ${randomNumber2}`
  );
  await mine(1);
  const currentBlock3 = await publicClient.getBlock();
  const randomNumber3 = await contract.read.getRandomNumber();
  console.log(
    `Block number: ${currentBlock3?.number}\nBlock difficulty: ${currentBlock3?.difficulty}\nRandom number from this block difficulty: ${randomNumber3}`
  );
  await mine(1);
  const currentBlock4 = await publicClient.getBlock();
  const randomNumber4 = await contract.read.getRandomNumber();
  console.log(
    `Block number: ${currentBlock4?.number}\nBlock difficulty: ${currentBlock4?.difficulty}\nRandom number from this block difficulty: ${randomNumber4}`
  );
  await mine(1);
  const currentBlock5 = await publicClient.getBlock();
  const randomNumber5 = await contract.read.getRandomNumber();
  console.log(
    `Block number: ${currentBlock5?.number}\nBlock difficulty: ${currentBlock5?.difficulty}\nRandom number from this block difficulty: ${randomNumber5}`
  );
}
```

### RANDAO Implementation References

<https://eips.ethereum.org/EIPS/eip-4399>

<https://github.com/ethereum/solidity/releases/tag/v0.8.18>

<https://github.com/randao/randao>

<https://soliditydeveloper.com/prevrandao>

## Theory: Other randomness sources

* Bias in decentralized randomness generation
* Oracles
* Data sources
* Oracle patterns
* On-chain data
* VRF

### Additional Randomness Sources References

<https://fravoll.github.io/solidity-patterns/oracle.html>

<https://tellor.io/blog/tellor-rng/>

<https://docs.chain.link/docs/chainlink-vrf/>

