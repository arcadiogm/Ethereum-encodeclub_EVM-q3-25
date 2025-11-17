# Lesson 13 - Frontend

## Architecture overview

* The browser
* HTML
* CSS
* Javascript
* Dynamic content
* Languages and frameworks
* Responsiveness

### Web Application References

<https://en.wikipedia.org/wiki/Web_application>

## Frameworks

* Modern programming languages and frameworks
* Picking a framework
* React
* NextJS

### Frontend Framework References

<https://stackdiary.com/front-end-frameworks/>

<https://merehead.com/blog/top-css-frameworks-developers-designers-2023/>

<https://react.dev/learn>

<https://nextjs.org/learn/react-foundations/from-react-to-nextjs>

## Implementing the dApp using Scaffold-ETH 2

* Creating the project
* Running the toolkit
* Using the Burner wallet
* Project structure
* Components
* Overview of jsx syntax
* Adding content to pages

### Scaffold-ETH References

<https://github.com/scaffold-eth/scaffold-eth-2>

<https://react.dev/learn/writing-markup-with-jsx>

<https://docs.scaffoldeth.io/>

<https://www.rainbowkit.com/docs/introduction>

<https://wagmi.sh/react/getting-started>

<https://tailwindcss.com/>

<https://daisyui.com/docs/install/>

### Creating the project

* Install yarn if you don't have it

```bash
corepack enable
```

* Clone the initial project base

```bash
git clone https://github.com/scaffold-eth/scaffold-eth-2.git
```

* Initialize the project

```bash
cd scaffold-eth-2
yarn install
```

* Start the application

```bash
yarn start
```

> **Note:** If you navigate to `http://localhost:3000/` you will see the UI in place, but with some errors since there is no local chain running yet.
>
> To fix that, you could configure the project to point to a public blockchain, or you could run a local chain in your development environment.

* Configuring the application chain

```bash
code ./packages/nextjs/scaffold.config.ts
```

Now change `targetNetworks: [chains.hardhat]` to `[chains.sepolia]` or any other.

The problem of using public networks is that it makes the development experience very slow.

This is why we will use a local chain for development.

* Running a local chain

Open another terminal window in the same folder or split your current terminal.

```bash
yarn chain
```

* Testing with a sample contract

Open another terminal window in the same folder or split your current terminal.

```bash
yarn deploy
```

### Using your own contract

* Contract code for _HelloWorld.sol_

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

* Create a new deployment script named `01_deploy_hello_world.ts` at `packages/hardhat/deploy`

```typescript
import { HardhatRuntimeEnvironment } from "hardhat/types";
import { DeployFunction } from "hardhat-deploy/types";

const deployHelloWorld: DeployFunction = async function (hre: HardhatRuntimeEnvironment) {
  const { deployer } = await hre.getNamedAccounts();
  const { deploy } = hre.deployments;

  await deploy("HelloWorld", {
    from: deployer,
    log: true,
    autoMine: true,
  });
};

export default deployHelloWorld;

deployHelloWorld.tags = ["HelloWorld"];
```

* Run deployment again

```bash
yarn deploy
```

### Updating the Home Screen

* Open the Home page to edit

```bash
code ./packages/nextjs/app/page.tsx
```

```tsx
import type { NextPage } from "next";

const Home: NextPage = () => {
  return (
    <>
      <div className="flex items-center flex-col flex-grow pt-10">
        <div className="px-5">
          <h1 className="text-center mb-8">
            <span className="block text-2xl mb-2">Welcome to</span>
            <span className="block text-4xl font-bold">Scaffold-ETH 2</span>
          </h1>
          <p className="text-center text-lg">
            Get started by editing{" "}
            <code className="italic bg-base-300 text-base font-bold max-w-full break-words break-all inline-block">
              packages/nextjs/pages/index.tsx
            </code>
          </p>
          <PageBody />
        </div>
      </div>
    </>
  );
};

function PageBody() {
  return (
    <>
      <p className="text-center text-lg">Here we are!</p>
    </>
  );
}

export default Home;
```

>Usually new components are created inside new files, organized under separated folders, but for simplicity we are adding it directly to the page for this example.

### Blockchain integration

* Adding a [hook](https://docs.scaffoldeth.io/hooks/) to read the contract

```typescript
"use client";
import { useScaffoldReadContract } from "~~/hooks/scaffold-eth";
```

* Create a new component for reading the contract

```tsx
function ExampleContractRead() {
  const {
    data: helloWorld,
    isPending,
    isError,
    dataUpdatedAt,
  } = useScaffoldReadContract({
    contractName: "HelloWorld",
    functionName: "helloWorld",
  });

  if (isPending) return <p className="text-center text-lg">Loading...</p>;
  if (isError) return <p className="text-center text-lg">Error getting information from your contract</p>;

  return (
    <>
      <p className="text-center text-lg">The text from the contract is {helloWorld}</p>
      <p className="text-center text-sm">This data was last updated at {new Date(dataUpdatedAt).toLocaleString()}</p>
    </>
  );
}
```

* Display the component in the page below `PageBody`

```tsx
          ...
          </p>
          <PageBody />
          <ExampleContractRead />
        </div>
        ...
```

## Understanding Scaffold-ETH Blockchain Integration

* Scaffold-ETH 2 uses the [Reown WalletKit](https://docs.reown.com/walletkit/overview) (formely known as WalletConnect) to connect to the blockchain
* When setting up a project with Scaffold ETH,you can modify the `scaffold.config.ts` file to use your own `Project ID` configuration
  * Create an account in the [Reown Cloud](https://cloud.reown.com/)
  * Create a project and copy the `Project ID`
  * Create an entry for the `NEXT_PUBLIC_WALLET_CONNECT_PROJECT_ID` in your project `.env` file or in your device environment variables, and place the `Project ID` value there
* It's possible to create a whole app using the `AppKit`component in [any EVM chain](https://reown.com/blog/how-to-get-started-with-reown-appkit-on-any-evm-chain)
  * You can check a demonstration of the `AppKit` implementation for `NextJS` with `Wagmi` in this [demo project](https://appkit-web-examples-next-wagmi.reown.com/)
  * If you want to experiment with this project, you can fork it from this [StackBlitz example](https://stackblitz.com/github/reown-com/appkit-web-examples/tree/main/nextjs/next-wagmi-app-router?file=README.md)

## Homework

* Create a Project ID in the [Reown Cloud](https://cloud.reown.com/) for your project and add it to the `.env` file
