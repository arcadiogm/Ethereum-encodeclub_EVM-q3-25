# Lesson 8 - Scripts for Ballot.sol

## Using scripts to automate operations

* Running a script with hardhat and ts-node
* Ballot deployment  script

### Ballot Deployment Script References

<https://hardhat.org/hardhat-runner/docs/guides/deploying>

<https://hardhat.org/hardhat-runner/docs/guides/tasks-and-scripts>

### Creating a deployment script with Typescript and Viem

* Creating a script for _DeployWithHardhat.ts_ operation:

```typescript
import { viem } from "hardhat";
const PROPOSALS = ["Proposal 1", "Proposal 2", "Proposal 3"];

async function main() {
  console.log("Proposals: ");
  PROPOSALS.forEach((element, index) => {
    console.log(`Proposal N. ${index + 1}: ${element}`);
  });
  // TODO
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

* Adding commands:

```typescript
import { viem } from "hardhat";
import { toHex, hexToString, formatEther } from "viem";
const PROPOSALS = ["Proposal 1", "Proposal 2", "Proposal 3"];

async function main() {
  ...
  console.log("\nDeploying Ballot contract");
  const ballotContract = await viem.deployContract("Ballot", [
    PROPOSALS.map((prop) => toHex(prop, { size: 32 })),
  ]);
  console.log("Ballot contract deployed to:", ballotContract.address);
  console.log("Proposals: ");
  for (let index = 0; index < PROPOSALS.length; index++) {
    const proposal = await ballotContract.read.proposals([BigInt(index)]);
    const name = hexToString(proposal[0], { size: 32 });
    console.log({ index, name, proposal });
  } 
}
```

### Running scripts

```bash
npx hardhat run ./scripts/DeployWithHardhat.ts
```

## Connecting to a public blockchain

* Environment files
* Providers
* Connecting to a testnet with a RPC Provider
* Running scripts on chain

### Configuring hardhat for deploying a contract to a public testnet

* Configuring Hardhat network options
* Using environment variables
* Running scripts with the `--network` option

### References

<https://hardhat.org/hardhat-runner/docs/config>

<https://hardhat.org/hardhat-runner/docs/guides/configuration-variables>

<https://www.npmjs.com/package/dotenv>

### Configuration file

* Edit your `hardhat.config.ts` file:

```typescript
const config: HardhatUserConfig = {
  solidity: "0.8.24",
  networks: {
    sepolia: {
      url: "https://ethereum-sepolia-rpc.publicnode.com",
    }
  },
};
```

* Running the deployment script with the `--network` option

```bash
npx hardhat run ./scripts/DeployWithHardhat.ts --network sepolia
```

>You should get a `DefaultWalletClientNotFoundError` error. This is expected to happen because deploying a contract requires sending a transaction, and thus we need to specify an account to sign and pay for it.

### Setting up environment variables

* Create a `.env` file in the root of your project:

```env
PRIVATE_KEY=****************************************************************
INFURA_API_KEY="********************************"
INFURA_API_SECRET="********************************"
ALCHEMY_API_KEY="********************************"
ETHERSCAN_API_KEY="********************************"
```

>Note: You can use any RPC provider you prefer by adding the corresponding environment variable. For example `INFURA_API_KEY="********************************"` or `ETHERSCAN_API_KEY="********************************"`.
>>The more providers you add, the more flexibility you have if some of them are facing issues.

* Install the `dotenv` package:

```bash
npm install dotenv
```

* Import the `dotenv` package in your script:

```typescript
import * as dotenv from "dotenv";
dotenv.config();
```

* Access the environment variables in your script:

```typescript
const providerApiKey = process.env.ALCHEMY_API_KEY || "";
const deployerPrivateKey = process.env.PRIVATE_KEY || "";
```

* Use the environment variables in your script's logic:

```typescript
const config: HardhatUserConfig = {
  solidity: "0.8.24",
  networks: {
    sepolia: {
      url: `https://eth-sepolia.g.alchemy.com/v2/${providerApiKey}`,
      accounts: [deployerPrivateKey],
    }
  },
};
```

* View chain and account information before deploying

```typescript
async function main() {
  const publicClient = await viem.getPublicClient();
  const blockNumber = await publicClient.getBlockNumber();
  console.log("Last block number:", blockNumber);
  const [deployer] = await viem.getWalletClients();
  console.log("Deployer address:", deployer.account.address);
  const balance = await publicClient.getBalance({
    address: deployer.account.address,
  });
  console.log(
    "Deployer balance:",
    formatEther(balance),
    deployer.chain.nativeCurrency.symbol
  );
  ...
```

* Running the deployment again

```bash
npx hardhat run ./scripts/DeployWithHardhat.ts --network sepolia
```

### Running scripts with arguments

* Passing arguments
* Passing variables to the deployment script
* Receiving arguments in the script

### References for Running Scripts

<https://hardhat.org/hardhat-runner/docs/advanced/scripts>

<https://hardhat.org/hardhat-runner/docs/guides/typescript#running-your-tests-and-scripts-directly-with--ts-node>

<https://nodejs.org/docs/latest/api/process.html#processargv>

### Testing the parameters

* Create a new script for _DeployWithViem.ts_ operation:

```typescript
async function main() {
  const proposals = process.argv.slice(2);
  if (!proposals || proposals.length < 1)
    throw new Error("Proposals not provided");
  console.log("Proposals:");
  proposals.forEach((element, index) => {
    console.log(`Proposal N. ${index + 1}: ${element}`);
  });
}
```

* Try to run with hardhat

```bash
npx hardhat run ./scripts/DeployWithViem.ts "arg1" "arg2" "arg3"
```

>This should give an error saying `Error HH308: Unrecognized positional argument arg1`

* Passing arguments using `ts-node` directly:

```bash
npx ts-node --files ./scripts/DeployWithViem.ts "arg1" "arg2" "arg3"
```

>Note: the [`--files`](https://typestrong.org/ts-node/docs/options/#files) flag is needed for some dependencies to run correctly later on

### Replacing Hardhat helpers with basic Viem functions

* Public Providers
* Wallet Clients
* Running scripts onchain
* Script for giving voting rights to a given address
* Dealing with transactions in scripts
* Contract factory and json imports
* Transaction receipts and async complexities when running onchain

### References for Public and Wallet Clients

<https://viem.sh/docs/clients/public>

<https://viem.sh/docs/clients/wallet>

<https://viem.sh/docs/contract/deployContract>

### Creating a public client

* Import `createPublicClient`, `http` and `sepolia` from `viem`
* Create a public client connected to the same url as you used in the `hardhat.config.ts` file

```typescript
import { createPublicClient, http } from "viem";
import { sepolia } from "viem/chains";
import * as dotenv from "dotenv";
dotenv.config();

const providerApiKey = process.env.ALCHEMY_API_KEY || "";

async function main() {
  const proposals = process.argv.slice(2);
  if (!proposals || proposals.length < 1)
    throw new Error("Proposals not provided");
  const publicClient = createPublicClient({
    chain: sepolia,
    transport: http(`https://eth-sepolia.g.alchemy.com/v2/${providerApiKey}`),
  });
  const blockNumber = await publicClient.getBlockNumber();
  console.log("Last block number:", blockNumber);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});

```

### Creating a wallet client

* Import `createWalletClient` and `privateKeyToAccount` from `viem`
* Create an account using the private key from the env file
* Create a wallet client similar to the public client, but connected to the account above

```typescript
import { createPublicClient, http, createWalletClient, formatEther } from "viem";
import { privateKeyToAccount } from "viem/accounts";
...
const deployerPrivateKey = process.env.PRIVATE_KEY || "";

async function main() {
...
  const account = privateKeyToAccount(`0x${deployerPrivateKey}`);
  const deployer = createWalletClient({
    account,
    chain: sepolia,
    transport: http(`https://eth-sepolia.g.alchemy.com/v2/${providerApiKey}`),
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
```

### Deploying a contract

* Import `abi` and `bytecode` from your compiled contract `json` _artifact_
* Using Hardhat, your `json` files should be under `artifacts/contracts` folder after you compile
* Each contract's artifacts should be inside the folders with the contract's name

```typescript
import { abi, bytecode } from "../artifacts/contracts/Ballot.sol/Ballot.json";
...

async function main() {
  ...
  console.log("\nDeploying Ballot contract");
  const hash = await deployer.deployContract({
    abi,
    bytecode: bytecode as `0x${string}`,
    args: [proposals.map((prop) => toHex(prop, { size: 32 }))],
  });
  console.log("Transaction hash:", hash);
  console.log("Waiting for confirmations...");
  const receipt = await publicClient.waitForTransactionReceipt({ hash });
  console.log("Ballot contract deployed to:", receipt.contractAddress);
```

### Reading information from a deployed contract

* Use `readContract` function from a `Public Client`
* Use the same ABI as used before to create a contract instance

```typescript
  console.log("Proposals: ");
  for (let index = 0; index < proposals.length; index++) {
    const proposal = (await publicClient.readContract({
      address: receipt.contractAddress,
      abi,
      functionName: "proposals",
      args: [BigInt(index)],
    })) as any[];
    const name = hexToString(proposal[0], { size: 32 });
    console.log({ index, name, proposal });
  }
```

## Running operations scripts

* Creating a script for _CastVote.ts_ operation

```bash
npx ts-node --files ./scripts/CastVote.ts
```

* receiving parameters

```typescript
  const parameters = process.argv.slice(2);
  if (!parameters || parameters.length < 2)
    throw new Error("Parameters not provided");
  const contractAddress = parameters[0] as `0x${string}`;
  if (!contractAddress) throw new Error("Contract address not provided");
  if (!/^0x[a-fA-F0-9]{40}$/.test(contractAddress))
    throw new Error("Invalid contract address");
  const proposalIndex = parameters[1];
  if (isNaN(Number(proposalIndex))) throw new Error("Invalid proposal index");
```

* Attaching the contract and checking the selected option

```typescript
  console.log("Proposal selected: ");
  const proposal = (await publicClient.readContract({
    address: contractAddress,
    abi,
    functionName: "proposals",
    args: [BigInt(proposalIndex)],
  })) as any[];
  const name = hexToString(proposal[0], { size: 32 });
  console.log("Voting to proposal", name);
  console.log("Confirm? (Y/n)");
```

* Sending transaction on user confirmation

```typescript
  const stdin = process.openStdin();
  stdin.addListener("data", async function (d) {
    if (d.toString().trim().toLowerCase() != "n") {
      const hash = await voter.writeContract({
        address: contractAddress,
        abi,
        functionName: "vote",
        args: [BigInt(proposalIndex)],
      });
      console.log("Transaction hash:", hash);
      console.log("Waiting for confirmations...");
      const receipt = await publicClient.waitForTransactionReceipt({ hash });
      console.log("Transaction confirmed");
    } else {
      console.log("Operation cancelled");
    }
    process.exit();
  });
```

## Weekend Project

* Develop and run scripts for “Ballot.sol” within your group to give voting rights, casting votes, delegating votes and querying results
* Consider function execution and the transaction hash, if successful, or the revert reason, if failed

