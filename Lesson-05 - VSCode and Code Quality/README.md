# Lesson 5 - Vscode setup and code quality

## Environment setup

* Tooling:
  * [Node](https://nodejs.org/en/docs/guides/getting-started-guide/)
  * Package managers
    * [NPM](https://docs.npmjs.com/cli/v9/configuring-npm/install)
    * [Yarn](https://yarnpkg.com/getting-started/install)
    * [Bun](https://bun.sh/docs/installation)
  * [Git CLI](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup)
  * [VS Code](https://code.visualstudio.com/docs/setup/setup-overview)
* RPC Providers
  * [Infura](https://infura.io/)
  * [Alchemy](https://www.alchemy.com/)
  * [Etherscan](https://etherscan.io/register)

## Programming setup

* OpenZeppelin contracts as a reference repository
* Best practices to copy from them
* Recommended tools and configurations

### References for tools

<https://github.com/OpenZeppelin/openzeppelin-contracts>

<https://git-scm.com/book/en/v2/Getting-Started-The-Command-Line>

<https://yarnpkg.com/getting-started>

<https://yarnpkg.com/getting-started/usage>

<https://code.visualstudio.com/docs>

### Getting started with our development environment

>Recommendation:
>Before starting, make sure that you have the correct version of Node installed in your system.
>>You can check your current version by running `node -v` in your terminal.

* Use [Node Version Manager](https://github.com/nvm-sh/nvm) to install and use Node v20 LTS version to avoid problems

```bash
nvm install --lts
...
nvm use --lts
...
node -v
v20.11.1
```

>Note: on windows the `nvm` command is not available, use `nvm-windows` instead from [Corey Butler's community package](https://github.com/coreybutler/nvm-windows).

```bash
# For windows
nvm install lts
...
nvm use lts
...
node -v
v20.11.1
```

Lets start by cloning OpenZeppelin contracts repository:

```bash
git clone https://github.com/OpenZeppelin/openzeppelin-contracts.git
cd .\openzeppelin-contracts\
npm install
```

>Note: You might see some warnings, but that's ok. You can ignore anything that is not an error, even vulnerability warnings (for now).
>>Note 2: on windows the `scripts/prepare.sh` might give an error. This is not a problem for running the project.

### Output example

```bash
npm run compile
...
> openzeppelin-solidity@5.0.2 compile
> hardhat compile
...
Compiled 231 Solidity files successfully
```

### Package.json scripts

```json
    "scripts": {
      "compile": "hardhat compile",
      ...
      "test": "hardhat test",
      ...
    },
```

### Test example

```bash
npm run test ./test/token/ERC20/ERC20.test.js
...
> openzeppelin-solidity@5.0.2 test
> hardhat test ./test/token/ERC20/ERC20.test.js
...
126 passing (5s)
```

>Note: on Windows the paths should use `\` instead of `/`, for example: `.\test\token\ERC20\ERC20.test.js`

### Breaking change

Open a file to edit:

```bash
code ./contracts/token/ERC20/ERC20.sol
```

Change line `78`:

```solidity
    function decimals() public view virtual override returns (uint8) {
        return 42;
    }
```

Run the test again:

```bash
npm run test ./test/token/ERC20/ERC20.test.js
...
> openzeppelin-solidity@5.0.2 test
> hardhat test ./test/token/ERC20/ERC20.test.js
...
  124 passing (6s)
  2 failing

  1) ERC20
       $ERC20
         has 18 decimals:

      AssertionError: expected 42 to equal 18.
      + expected - actual

      -42
      +18
      
      at Context.<anonymous> (test/token/ERC20/ERC20.test.js:47:48)
      at processTicksAndRejections (node:internal/process/task_queues:95:5)

  2) ERC20
       $ERC20ApprovalMock
         has 18 decimals:

      AssertionError: expected 42 to equal 18.
      + expected - actual

      -42
      +18
      
      at Context.<anonymous> (test/token/ERC20/ERC20.test.js:47:48)
      at processTicksAndRejections (node:internal/process/task_queues:95:5)
```

### Quality of code

* Composability
* Upgradeability
* Maintainability and readability
* Managing work flow and progress
* Reaching _peace of mind_

## Getting started with a new project using Hardhat

* Creating a base repository
* Setup hardhat with typescript
* Configure VS Code
* Hardhat scripts and tasks
* VS Code extensions recommended

### References

<https://hardhat.org/getting-started/>

<https://hardhat.org/guides/typescript.html>

<https://hardhat.org/guides/vscode-tests.html>

### Steps

* Creating a new project named `project`:

```bash
# Exit your working folder:
cd ..
# Alternatively you could pick a directory in another place, like "cd ~/desktop" or any other you may prefer
# Create a new folder called "project" or any other name of your choice
mkdir project
# Enter that folder
cd project
# Make sure that your folder is not inside any other project folder in your device
# For example, check if don't have any other files named "package.json" in any of the directories above your current directory
```

* Initializing a repository:

>Hardhat is used through a local installation in your project.
>>This way your environment will be reproducible, and you will avoid future version conflicts.

This requires initializing a repository with `npm` or `yarn` before proceeding

* Starting a new project using `npm`:

```bash
npm init
# After running this, complete the prompt with the default options (hit enter for every option)
# These options can be changed later in the package.json file
```

* Installing hardhat locally using `npm`:

```bash
npm install --save-dev hardhat
# Complete the prompt with the default options (hit enter for every option)
# These options can be changed later in the package.json file
```

>Note: To use `bun` or `yarn` instead of `npm`, you can run `bun add -d <name of package>` or `yarn add --dev <name of package>` instead of `npm install --save-dev <name of package>` to install dependencies, and you can use `bun <command>` or `yarn <command>` instead of `npx <command>` to run packages.

* Initializing Hardhat:

```bash
npx hardhat init
    "Create a TypeScript project (with Viem)"
# Pick all default options
code .
```

>Note: Depending on your system and/or your package manager, hardhat will automatically install all the dependencies in the same command. Sometimes it won't install them, but instead it will just output a list of suggested dependencies versions in your terminal.

### Recommended extensions

[Mocha Test Explorer](https://marketplace.visualstudio.com/items?itemName=hbenl.vscode-mocha-test-adapter)

[Hardhat](https://hardhat.org/hardhat-vscode/docs/overview)

### Environment setup for VSCode tests

* For our tests extension to work, we need to create a configuration file for it
  * This configuration file will not be there when you initialize the repo, you must create it
* Create a file named exactly `.mocharc.json` in the root of your project folder with the following contents:

```json
{
  "require": "hardhat/register",
  "timeout": 40000,
  "_": ["test*/**/*.ts"]
}
```

* For storing our personal credentials and environment variables, we need to create an environment file for it
  * This file will not be there when you initialize the repo, you must create it
* Create a file named exactly `.env` in root project folder with the following contents:

```env
MNEMONIC="here is where your extracted twelve words mnemonic phrase should be put"
PRIVATE_KEY="<your wallet private key should go here>"
POKT_API_KEY="********************************"
INFURA_API_KEY="********************************"
INFURA_API_SECRET="********************************"
ALCHEMY_API_KEY="********************************"
ETHERSCAN_API_KEY="********************************"
```

>Note: you may choose as many RPC providers as you want, but you must have at least one.
>> Having more RPC providers can be useful as fallback options in case one of them is down. For POKT you can create one free key in each provider available in their network too.

Edit the environment with your keys, and make sure that this file is in your `.gitignore` file.

>Do never share your `.env` file with anyone that you don't trust.
>> If you end up revealing your private keys **you may lose all your assets**, even if you were using testnets, since your wallet is the same regardless of the network you are connected to.

* Test if your hardhat configuration is working :

```bash
npx hardhat compile 
npx hardhat test 
```

### Hardhat task example

* Create an _Accounts_ task in `hardhat.config.ts` file:

```typescript
import { task, type HardhatUserConfig } from "hardhat/config";
...
task("accounts", "Prints the list of accounts", async (taskArgs, hre) => {
  const accounts = await hre.viem.getWalletClients();
  for (const account of accounts) {
    console.log(account.account.address);
  }
});
```

* Test the new task:

```bash
npx hardhat accounts 
```

## Coding in VS Code

* Syntax for typescript scripts
* How the project operates
* Writing a test file
* Using Ethers.js library
* Using Hardhat toolbox
* Using Typechain library
* Testing syntax
* Running a test file

### References for next lessons

<https://viem.sh/>

<https://mochajs.org/>

<https://hardhat.org/hardhat-chai-matchers/docs/overview>

<https://www.chaijs.com/guide/>

### Clearing template files

```bash
rm ./contracts/*
rm ./ignition/*
rm ./test/*
npx hardhat clean
```

>You can also remove the `node_modules` folder and the `package-lock.json` file if you want to start fresh. Just remember to run `npm install` or `yarn install` to install the dependencies again next time.
>> This can be useful if you want to use this folder as a starting point for your next projects in the bootcamp.

## Homework
* Get to know Hardhat and Viem documentations
