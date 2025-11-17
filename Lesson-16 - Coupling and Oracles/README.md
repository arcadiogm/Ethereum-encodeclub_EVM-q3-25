# Lesson 16 - Coupling and Oracles

## Review

* Frontend
* Integrating frontend with blockchain
* Coupling frontend in API
* Using off-chain data in dApps

## Coupling frontend and APIs

* On-chain and off-chain features
* Keeping user Private Key private
* Mapping interactions, resources and payloads
* Handling errors

### Coupling References

<https://en.wikipedia.org/wiki/Loose_coupling>

<https://react.dev/reference/react/useEffect#fetching-data-with-effects>

### Implementing the coupling

* Change port to `3001` at `main.ts` (backend):

```typescript
await app.listen(3001);
```

* Edit Home page in the frontend at `packages/nextjs/pages/index.tsx`:

```bash
cd ../..
code ./scaffold-eth-2/packages/nextjs/pages/index.tsx
```

* Create a new Component for `ApiData`:

```tsx
function ApiData(params: { address: `0x${string}` }) {
  return (
    <div className="card w-96 bg-primary text-primary-content mt-4">
      <div className="card-body">
        <h2 className="card-title">Testing API Coupling</h2>
        <p>TODO</p>
      </div>
    </div>
  );
}
```

* Modify `WalletInfo` to add `ApiData` Component:

```tsx
if (address)
  return (
    <div>
      <p>Your account address is {address}</p>
      <p>Connected to the network {chain?.name}</p>
      <WalletAction></WalletAction>
      <WalletBalance address={address as `0x${string}`}></WalletBalance>
      <TokenInfo address={address as `0x${string}`}></TokenInfo>
      <ApiData address={address as `0x${string}`}></ApiData>
    </div>
  );
```

* Create a new Component for `TokenAddressFromApi`:

```tsx
function TokenAddressFromApi() {
  const [data, setData] = useState<{ result: string }>();
  const [isLoading, setLoading] = useState(true);

  useEffect(() => {
    fetch("http://localhost:3001/contract-address")
      .then((res) => res.json())
      .then((data) => {
        setData(data);
        setLoading(false);
      });
  }, []);

  if (isLoading) return <p>Loading token address from API...</p>;
  if (!data) return <p>No token address information</p>;

  return (
    <div>
      <p>Token address from API: {data.result}</p>
    </div>
  );
}
```

* Put `TokenAddressFromApi` inside `ApiData` Component:

```tsx
        <TokenAddressFromApi></TokenAddressFromApi>
        <p>TODO</p>
```

## CORS settings

* Cross-origin resource sharing
* Allow origin errors

### CORS Documentation References

* Modifying `main.ts` (backend):

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableCors();
  ...
```

### References

<https://docs.nestjs.com/security/cors>

<https://github.com/expressjs/cors#configuration-options>

---

## Sending a POST call

* Building a Payload
* Using DTOs

### POST Request References

<https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST>

<https://jasonwatmore.com/post/2020/02/01/react-fetch-http-post-request-examples>

<https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type>

### Implementation example

* Create the `RequestTokens` Component:

```tsx
function RequestTokens(params: { address: string }) {
  const [data, setData] = useState<{ result: boolean }>();
  const [isLoading, setLoading] = useState(false);

  const body = { address: params.address };

  if (isLoading) return <p>Requesting tokens from API...</p>;
  if (!data)
    return (
      <button
        className="btn btn-active btn-neutral"
        onClick={() => {
          setLoading(true);
          fetch("http://localhost:3001/mint-tokens", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify(body),
          })
            .then((res) => res.json())
            .then((data) => {
              setData(data);
              setLoading(false);
            });
        }}
      >
        Request tokens
      </button>
    );

  return (
    <div>
      <p>Result from API: {data.result ? "worked" : "failed"}</p>
    </div>
  );
}
```

* Put `RequestTokens` inside `ApiData` Component:

```tsx
        <TokenAddressFromApi></TokenAddressFromApi>
        <RequestTokens address={params.address}></RequestTokens>
```

* Implement `mintTokens` at `app.service.ts` to return something:

```typescript
  async mintTokens(address: string) {
    return { result: true };
  }
```

* TODO: Implement the mint transaction and return the hash for displaying it in the frontend.

## Oracles

* Using off-chain data in Smart Contracts
* Trust and decentralization in data sources
* Oracle patterns

### Oracle References

<https://fravoll.github.io/solidity-patterns/oracle.html>

<https://ethereum.org/en/developers/docs/oracles/>

## Tellor

* Tellor oracle network
* Getting data
* Data sources
* Query IDs
* Tests

### Tellor Documentation References

<https://docs.tellor.io/tellor/the-basics/readme>

<https://docs.tellor.io/tellor/getting-data/solidity-integration>

<https://docs.tellor.io/tellor/the-basics/contracts-reference#ethereum>

### Code reference

* Installing Tellor:

```bash
npm install usingtellor
```

* Contract for CallOracle.sol:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "usingtellor/contracts/UsingTellor.sol";

contract CallOracle is UsingTellor {
    constructor(address payable _tellorAddress) UsingTellor(_tellorAddress) {}

    function getBtcSpotPrice(uint256 maxTime) external view returns (uint256) {
        bytes memory _queryData = abi.encode(
            "SpotPrice",
            abi.encode("btc", "usd")
        );
        bytes32 _queryId = keccak256(_queryData);

        (bytes memory _value, uint256 _timestampRetrieved) = _getDataBefore(
            _queryId,
            block.timestamp - 20 minutes
        );
        if (_timestampRetrieved == 0) return 0;
        require(
            block.timestamp - _timestampRetrieved < maxTime,
            "Maximum time elapsed"
        );
        return abi.decode(_value, (uint256));
    }
}
```

* Script for `DeployCallOracle.ts`:

```typescript
import {
  createPublicClient,
  http,
  createWalletClient,
  formatEther,
} from "viem";
import { privateKeyToAccount } from "viem/accounts";
import { sepolia } from "viem/chains";
import {
  abi,
  bytecode,
} from "../artifacts/contracts/CallOracle.sol/CallOracle.json";
import * as dotenv from "dotenv";
dotenv.config();

const providerApiKey = process.env.ALCHEMY_API_KEY || "";
const deployerPrivateKey = process.env.PRIVATE_KEY || "";

const TELLOR_ORACLE_ADDRESS = "0xB19584Be015c04cf6CFBF6370Fe94a58b7A38830";

async function main() {
  const publicClient = createPublicClient({
    chain: sepolia,
    transport: http(`https://eth-sepolia.g.alchemy.com/v2/${providerApiKey}`),
  });
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
  console.log("Deploying CallOracle contract");
  const hash = await deployer.deployContract({
    abi,
    bytecode: bytecode as `0x${string}`,
    args: [TELLOR_ORACLE_ADDRESS],
  });
  console.log("Transaction hash:", hash);
  console.log("Waiting for confirmations...");
  const receipt = await publicClient.waitForTransactionReceipt({ hash });
  console.log("CallOracle contract deployed to:", receipt.contractAddress);

  const btcSpotPrice = await publicClient.readContract({
    address: receipt.contractAddress as `0x${string}`,
    abi,
    functionName: "getBtcSpotPrice",
    args: [360 * 24 * 60 * 60],
  });

  console.log(
    `The last value for BTC Spot Price for the ${
      sepolia.name
    } network is ${formatEther(btcSpotPrice as bigint)} USD`
  );
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

## Weekend Project

* Complete the projects together with your group
* Create a voting dApp to cast votes, delegate and query results on chain
* Request voting tokens to be minted using the API
* (bonus) Store a list of recent votes in the backend and display that on frontend
* (bonus) Use an oracle to fetch off-chain data
  * Use an oracle to fetch information from a data source of your preference
  * Use the data fetched to create the proposals in the constructor of the ballot

### Voting dApp integration guidelines

* Build the frontend using Scaffold ETH 2 as a base
* Build the backend using NestJS to provide the `Mint` method
  * Implement a single POST method
    * Request voting tokens from API
* Use these tokens to interact with the tokenized ballot
* All other interactions must be made directly on-chain
