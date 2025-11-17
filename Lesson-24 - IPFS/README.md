# Lesson 24 - Part 1: IPFS

## File storage for the Web3.0

* Files on the server
* Scalable centralized solutions
* Databases vs File servers
* Decentralized file storage principles
* Introducing IPFS
* IPFS as a File Server
* IPFS alternatives

### References for File Storage

<https://ipfs.io/>

<https://www.arweave.org/>

<https://filecoin.io/>

## Using The IPFS

* Hosting and Co-Hosting files with IPFS
* Running a node
* Connecting to a local node
* Uploading files
* Downloading files
* File hashes
* File lifecycle

### References for Using The IPFS

<https://docs.ipfs.io/>

<https://docs.ipfs.tech/install/command-line/#install-official-binary-distributions>

<https://docs.ipfs.io/how-to/command-line-quick-start/>

<https://docs.ipfs.tech/concepts/>

<https://docs.ipfs.tech/concepts/lifecycle/>

## Part 2: Smart Contract Security

### The Security Toolbox

* Docker
* Running images
* Tools
  * Slither
  * Echidna
  * Manticore
  * Mythx
  * Securfy2
* Fakerdao sample project

### References for The Security Toolbox

<https://github.com/crytic/slither>

<https://github.com/crytic/echidna>

<https://github.com/trailofbits/manticore>

<https://mythx.io/>

<https://github.com/eth-sri/securify2>

<https://medium.com/coinmonks/ethereum-security-analysis-tools-an-introduction-and-comparison-1096194e64d5>

<https://docs.python.org/3/library/venv.html>

<https://github.com/scopelift/fakerdao>

<https://www.graphviz.org/>

<https://dreampuf.github.io/GraphvizOnline/>

### Instructions for Using The Security Toolbox

Setup:

* Clone `Fakerdao`:

```bash
git clone https://github.com/scopelift/fakerdao
```

* Create a new Hardhat project in another folder (eg. _myProject_):

```bash
mkdir myProject
cd myProject
npm init
...
```

* Copy the `contracts` folder from `Fakerdao` to your new project:

```bash
cd ..
cp -r ./fakerdao/contracts ./myProject
```

* Enter your new project folder:

```bash
cd myProject
```

* Install the dependencies:

```bash
npm i
npm i @openzeppelin/contracts-ethereum-package@2.5.0 --save-dev
```

* Change your hardhat settings on _hardhat.config.ts_:
  
```typescript
const config: HardhatUserConfig = {
  solidity: {
    version: "0.5.16",
  },
};
```

* Compile the contracts:

```bash
npx hardhat compile
```

* Setup [Python](https://docs.python.org/3/) and [Pip](https://pip.pypa.io/en/stable/)

* Install and run `slither` using a [Python venv](https://docs.python.org/3/library/venv.html):

```bash
python -m venv venv
. venv/bin/activate 
pip install slither-analyzer
slither .
```

* Using slither [printers](https://github.com/crytic/slither/wiki/Printer-documentation):

```bash
slither . --print contract-summary --hardhat-ignore-compile
slither . --print "call-graph,inheritance-graph" --hardhat-ignore-compile 
```

* Viewing `.dot` files with `graphviz`:

  * Install [graphviz](https://graphviz.org/download/)

```bash
dot -Tpng inheritance-graph.dot > inheritance-graph.png
dot -Tpng all_contracts.call-graph.dot > all_contracts.call-graph.png 
```

