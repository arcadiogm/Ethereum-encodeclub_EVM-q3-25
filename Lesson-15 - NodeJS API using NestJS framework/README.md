# Lesson 15 - NodeJS API using NestJS framework

## NestJS Framework

* Running services with node
* Web server with node
* Using Express to run a node web server
* About using frameworks
* Using NestJS framework
* Overview of a NestJS project
* Using the CLI
* Initializing a project with NestJS
* Swagger plugin

### References

<https://nodejs.org/en/docs/guides/getting-started-guide/>

<https://devdocs.io/express-getting-started/>

<https://docs.nestjs.com/>

<https://docs.nestjs.com/openapi/introduction>

<https://www.coreycleary.me/what-is-the-difference-between-controllers-and-services-in-node-rest-apis>

<https://restfulapi.net/idempotent-rest-apis/>

<https://docs.nestjs.com/openapi/types-and-parameters>

## Implementing the API

* The NestJS CLI
* Creating Resources
* Controllers, Services and Routes in NestJS
* Modules and injections (overview)
* Server configuration
* Serving script operations as API services
* Params, DTOs and Payloads
* HTTP errors and messages
* (Review) Environment
* Implementing the features

## Read-only data

* GET methods:
  * GET contract address
  * GET token name
  * GET total supply
  * GET balance of a given address
  * GET transaction receipt of a transaction by transaction hash

### Example GET Method for Contract Address

* Controller method for GET `contract-address`:

```typescript
  @Get('contract-address')
  getContractAddress(){
    return {result: this.appService.getContractAddress()};
  }
```

* Implementing `getContractAddress` service at `app.service.ts`:

```typescript
  getContractAddress(): string {
    return "0x2282A77eC5577365333fc71adE0b4154e25Bb2fa";
  }
```

### Example GET Method for Token Name

* Installing Viem

```bash
npm i viem
```

* Controller method for GET `token-name`:

```typescript
  @Get('token-name')
  async getTokenName() {
    return {result: await this.appService.getTokenName()};
  }
```

* Fetching the ABI for creating a `Contract` with Viem:
  * Pick your `MyToken.json` from any ERC20 that you compiled before
  * Copy it and paste at `src/assets/`
  * Import the file at `app.service.ts`:

```typescript
import * as tokenJson from "./assets/MyToken.json";
```

* Add the `resolveJsonModule` configuration to `compilerOptions` inside `tsconfig.json`:

```json
"resolveJsonModule": true
```

* Implementing `getTokenName` service at `app.service.ts`:

```typescript
  async getTokenName(): Promise<string> {
    const publicClient = createPublicClient({
      chain: sepolia,
      transport: http(`>Put your RPC endpoint URL here<`),
    });
    const name = await publicClient.readContract({
      address: this.getContractAddress(),
      abi: tokenJson.abi,
      functionName: "name"
    });
    return name;
  }
```

### Using environment variables

* Add `dotenv` to your project and configure it at `main.ts`:

```bash
npm i --save dotenv
```

```typescript
...
import 'dotenv/config';

async function bootstrap() {
...
```

* Create your `.env` file at the root of your project:

```txt
PRIVATE_KEY=****************************************************************
RPC_ENDPOINT_URL="https://****************************************************************"
TOKEN_ADDRESS="0x****************************************"
```

* Modify the `getTokenName` service:

```typescript
  getContractAddress(): string {
    return process.env.TOKEN_ADDRESS;
  }

  async getTokenName(): Promise<string> {
    const publicClient = createPublicClient({
      chain: sepolia,
      transport: http(process.env.RPC_ENDPOINT_URL),
    });
    const name = await publicClient.readContract({
      address: this.getContractAddress(),
      abi: tokenJson.abi,
      functionName: "name"
    });
    return name;
  }
```

### Using NestJS Configurations Module

* Remove `dotenv` configurations at `main.ts`:

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
...
```

* Add `@nestjs/config` to your project

```bash
npm i --save @nestjs/config
```

* Import `ConfigModule` in `app.module.ts`

```typescript
import { Module } from "@nestjs/common";
import { AppController } from "./app.controller";
import { AppService } from "./app.service";
import { ConfigModule } from "@nestjs/config";

@Module({
  imports: [ConfigModule.forRoot()],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

* Import `ConfigService` inside `app.service.ts`:

```typescript
  constructor(private configService: ConfigService) {}
```

* Refactor `getContractAddress` and `getTokenName`:

```typescript
  getContractAddress(): string {
    return this.configService.get<string>('TOKEN_ADDRESS');
  }

  async getTokenName(): Promise<string> {
    const publicClient = createPublicClient({
      chain: sepolia,
      transport: http(this.configService.get<string>('RPC_ENDPOINT_URL')),
    });
    const name = await publicClient.readContract({
      address: this.getContractAddress(),
      abi: tokenJson.abi,
      functionName: "name"
    });
    return name;
  }
```

### Adding Swagger Module

* Install `@nestjs/swagger` to your project

```bash
npm install --save @nestjs/swagger
```

* Configure your `main.ts` file:

```typescript
import { NestFactory } from "@nestjs/core";
import { SwaggerModule, DocumentBuilder } from "@nestjs/swagger";
import { AppModule } from "./app.module";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const config = new DocumentBuilder()
    .setTitle("API example")
    .setDescription("The API description")
    .setVersion("1.0")
    .addTag("example")
    .build();
  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup("api", app, document);

  await app.listen(3000);
}
bootstrap();
```

### Other Controller GET Methods

* Adding the GET methods at `app.controller.ts`:

```typescript
  @Get('total-supply')
  async getTotalSupply() {
    return {result: await this.appService.getTotalSupply()};
  }

  @Get('token-balance/:address')
  async getTokenBalance(@Param('address') address: string) {
    return {result: await this.appService.getTokenBalance(address)};
  }

  @Get('transaction-receipt')
  async getTransactionReceipt(@Query('hash') hash: string) {
    return {result: await this.appService.getTransactionReceipt(hash)};
  }
```

* Organizing the `AppService` class:

```typescript
export class AppService {
  publicClient;

  constructor(private configService: ConfigService) {
    this.publicClient = createPublicClient({
      chain: sepolia,
      transport: http(this.configService.get<string>('RPC_ENDPOINT_URL')),
    });
  }
  ...
```

## Minting tokens

* GET methods:
  * Get the address of the server wallet
  * Check if address has MINTER_ROLE role
* POST methods:
  * Request tokens to be minted

### Implementation

* Controller methods:

```typescript
  @Get('server-wallet-address')
  getServerWalletAddress() {
    return {result: this.appService.getServerWalletAddress()};
  }

  @Get('check-minter-role')
  checkMinterRole(@Query('address') address: string) {
    return {result: await this.appService.checkMinterRole(address)};
  }

  @Post('mint-tokens')
  async mintTokens(@Body() body: any) {
    return {result: await this.appService.mintTokens(body.address)};
  }
```

* Creating the `MintTokenDto` class at `src/dtos/mintToken.dto.ts`:

```typescript
import { ApiProperty } from "@nestjs/swagger";

export class MintTokenDto {
  @ApiProperty({ type: String, required: true, default: "My Address" })
  address: string;
}
```

* Using `MintTokenDto` for the `mintTokens` method:

```typescript
...
import { MintTokenDto } from './dtos/mintToken.dto';

@Controller()
export class AppController {
  ...

  @Post('mint-tokens')
  async mintTokens(@Body() body: MintTokenDto) {
    return {result: await this.appService.mintTokens(body.address)};
  }
}
```

* Implementing the `WalletClient` in the `AppService` constructor:

```typescript
export class AppService {
  publicClient;
  walletClient;

  constructor() {
    const account = privateKeyToAccount(`0x${process.env.PRIVATE_KEY}`);
    this.publicClient = createPublicClient({
      chain: sepolia,
      transport: http(process.env.RPC_ENDPOINT_URL),
    });
    this.walletClient = createWalletClient({
      transport: http(process.env.RPC_ENDPOINT_URL),
      chain: sepolia,
      account: account,
    });
  }
```

* Checking the wallet address and minter role:

```typescript
  getServerWalletAddress(): string {
    return this.walletClient.account.address;
  }

  async checkMinterRole(address: string): Promise<boolean> {
    const MINTER_ROLE = "0x9f2df0fed2c77648de5860a4cc508cd0818c85b8b8a1ab4ceeef8d981c8956a6";
    // const MINTER_ROLE =  await this.publicClient.readContract({
    //   address: this.getContractAddress(),
    //   abi: tokenJson.abi,
    //   functionName: 'MINTER_ROLE'
    // });
    const hasRole = await this.publicClient.readContract({
      address: this.getContractAddress(),
      abi: tokenJson.abi,
      functionName: 'hasRole',
      args: [MINTER_ROLE, address],
    });
    return hasRole;
  }
```

