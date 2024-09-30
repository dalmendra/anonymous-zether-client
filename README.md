# Anonymous Zether Client
## Overview

**This project was forked from [Kaleido's anonymous-zether-client](https://github.com/kaleido-io/anonymous-zether-client/tree/real-digital) with the main objective of implementing Delivery vs Payment transactions with privacy using Anonymous Zether. This README document is based on the original, but was updated to include the modifications made on the application.**

This is a sample client implementation for [anonymous-zether](https://github.com/dalmendra/anonymous-zether) (forked from [Kaleido's anonymous-zether](http://github.com/kaleido/anonymous-zether)) to enable private transactions using Anonymous Zether.

**Main features:**

- a REST interface (app.js)
- key management: the client manages 3 types of signing accounts:
  - one-time signing keys for submitting zether transfer transactions
  - managed wallet that can easily dispense billions of signing keys to use by authenticated users. This can be handy if an application wants to present to their end users a web2 experience without requiring them to manage signing keys. A typical technique is mapping user identities (such as OIDC subject ID) to signing keys such that each authenticated user has their own signing account
  - an admin account that was used to deploy the ERC20 contract, which has the minting privileges
- pre-calculated cache to resolve the balance. After decrypting the onchain state that represents an account's Zether balance, the application gets the `g^b` value, where `b` is the actual balance. This step requires brute force computation to resolve the value `b`. A cache is provided for all values of `b` in the range 0 - 100,000 to allow this step to be completed instantaneously.

## Dependencies

anonymous-zether is not published as an NPM module, so you must check out the repository to fulfill the dependency.

Checkout the repository [anonymous-zether](https://github.com/dalmendra/anonymous-zether) so that it's peer to the root of this repository.

> Note: the default branch of the repository is `hardhat`. This is the branch you need.

Then you can `npm i` or `yarn` to install the dependencies.

To run the application's main script, you can use Node.js and to run a local Ethereum node, Hardhat can be used:
* [Node.js](https://nodejs.org/en/download/) - tested with version v20.12.0.
* [Hardhat Network](https://github.com/NomicFoundation/hardhat) - tested with version v2.22.12.
* [Postman](https://www.postman.com/downloads/) can be used to simplify the process of sending HTTP requests and analyzing responses.

The node_modules dir in the anonymous-zether project must be in the PATH environment variable. Update this variable for the client application to work without any dependency issues. In Windows Powershell, the command below should update the PATH - please modify the directory below to match your environment.
```console
$env:NODE_PATH = "(...)\anonymous-zether\packages\protocol\node_modules"
```


## Running the application

Follow the steps below to have a working system.

### Blockchain network

The client works with any Ethereum JSON-RPC endpoint, such as a local node for testing purposes.

You can get a local Ethereum node running easily by using [Hardhat](https://github.com/NomicFoundation/hardhat) or [ganache-cli](https://www.npmjs.com/package/ganache-cli). Example using hardhat:

```console
$ npx hardhat node
Started HTTP and WebSocket JSON-RPC server at http://127.0.0.1:8545/

Accounts
========

WARNING: These accounts, and their private keys, are publicly known.
Any funds sent to them on Mainnet or any other live network WILL BE LOST.

Account #0: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266 (10000 ETH)
Private Key: 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80

Account #1: 0x70997970C51812dc3A010C7d01b50e0d17dc79C8 (10000 ETH)
Private Key: 0x59c6995e998f97a5a0044966f0945389dc9e86dae88c7a8412f4603b6b78690d

Account #2: 0x3C44CdDdB6a900fa2b585dd299e03d12FA4293BC (10000 ETH)
Private Key: 0x5de4111afa1a4b94908f83103eb1f1706367c2e68ca870fc3fb9a804cdab365a

(...)

WARNING: These accounts, and their private keys, are publicly known.
Any funds sent to them on Mainnet or any other live network WILL BE LOST.
```

### Deploy Anonymous Zether Contracts

First make sure the smart contracts are deployed properly. To make Anonymous Zether work and allow for private DvP transactions using an ERC-1155 and an ERC-20 tokens, you need to deploy these tokens contracts as well as the Anonymous Zether contracts for each of them.

The easiest way to deploy is using the Hardhat script in the project [anonymous-zether](https://github.com/dalmendra/anonymous-zether) in the `packages/protocol` folder:

```console
$ cd anonymous-zether/packages/protocol
$ npm i
$ npm run deploy:local

> @anonymous-zether/protocol@0.1.0 deploy:local
> npx hardhat run scripts/deploy.js --network localTest

Deploying contracts with the account: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
Deployer private key: 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
CashToken contract deployed to  0x5FbDB2315678afecb367f032d93F642f64180aa3
CashToken owner: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
ERC1155Token contract deployed to  0xe7f1725E7734CE288F8367e1Bb143E90bb3F0512
ERC1155Token owner: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
InnerProductVerifier deployed to  0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0
ZetherVerifier deployed to  0xCf7Ed3AccA5a467e9e704C703E8D87F634fB0Fc9
BurnVerifier deployed to  0xDc64a140Aa3E981100a9becA4E685f962f0cF6C9
ZSCRestricted deployed to  0x5FC8d32690cc91D4c39d9d3abcBD16989F875707
ZSCERC1155Restricted deployed to  0x0165878A594ca255338adfa4d48449f69242Eb8F
DvpZSC deployed to  0xa513E6E4b8f2a923D98304ec87F64353C4D5C853
```

## Configurations

The app takes configurations in the form of environment variables. They can be provided via the `.env` file in the same folder as the `app.js`. A sample copy has been provided in the file `.env.sample` so you just need to copy that to file named `.env` in the same folder and adjust the values according to your blockchain environment and the output of the deployment script run in the previous step. The  `.env.sample` file has tips to help you on that task.
> The existence of the `.env` file will interfere with unit tests, which relies on test instrumentation for some of the failure tests. Please make sure to delete `.env` before running the unit test suite

The main contribution of this fork is that now the client enables DvP transactions using an ERC-20 and an ERC-1155 token. You can simulate two users by running two clients simultaneously. For that, you must have two copies of the anonymous-zether-client directory (e.g. anonymous-zether-client-seller and anonymous-zether-client-buyer).

In the seller's directory:
* Edit the `.env` file to point ZSC_ADDRESS to the ERC-1155 token (the one that will be traded for the ERC-20 token).
* Leave the line `const PORT = parseInt(process.env.PORT || 3000);` in the `app.js` file unchanged.

In the buyer's directory:
* Edit the `.env` file to point ZSC_ADDRESS to the ERC-20 token (CashToken - the one that will be traded for the ERC-1155 token).   
* Change the application port to 3001: `const PORT = parseInt(process.env.PORT || 3001);` in the `app.js` file.

> Note: to allow enough time for the DvP transactions to be executed, the epoch length may need to be increased. To perform that change, increase the parameter `ZSC_EPOCH_LENGTH` in the `.env` file to 60 seconds or more, up to 240 seconds. The epoch must also be changed accordingly in the `packages/protocol/script/deploy.js` script in [anonymous-zether](https://github.com/dalmendra/anonymous-zether).

In each directory, install the project packages and their dependencies:

```console
$ cd anonymous-zether-client-seller
$ npm i
```
```console
$ cd anonymous-zether-client-buyer
$ npm i
```

### Launch the server for both users (seller and buyer)

```console
$ cd anonymous-zether-client-seller
$ node app.js
2024-09-27T01:58:11.445Z [INFO] Populating the balance cache from file...
2024-09-27T01:58:11.451Z [INFO] Subscribed to events (subscription ID: 0x1)
2024-09-27T01:58:12.125Z [INFO] Populated the cache successfully with file \lib\trade-manager\resources\recover-balance-cache.csv
2024-09-27T01:58:12.126Z [INFO] Initialized HD Wallet for submitting transactions
2024-09-27T01:58:12.130Z [INFO] Successfully opened connection to key DB at tmp\zether\keysdb
2024-09-27T01:58:12.132Z [INFO] Configurations:
2024-09-27T01:58:12.132Z [INFO]             data dir: ./tmp/zether
2024-09-27T01:58:12.132Z [INFO]              eth URL: ws://localhost:8545
2024-09-27T01:58:12.132Z [INFO]             chain ID: 1337
2024-09-27T01:58:12.132Z [INFO]                erc20: 0x5FbDB2315678afecb367f032d93F642f64180aa3
2024-09-27T01:58:12.132Z [INFO]                  DvP: 0xa513E6E4b8f2a923D98304ec87F64353C4D5C853
2024-09-27T01:58:12.133Z [INFO]         epoch length: 6 seconds
2024-09-27T01:58:12.134Z [INFO] Listening on port 3000
```

```console
$ cd anonymous-zether-client-buyer
$ node app.js
2024-09-27T01:59:16.480Z [INFO] Populating the balance cache from file...
2024-09-27T01:59:16.484Z [INFO] Subscribed to events (subscription ID: 0x2)
2024-09-27T01:59:17.153Z [INFO] Populated the cache successfully with file \lib\trade-manager\resources\recover-balance-cache.csv
2024-09-27T01:59:17.154Z [INFO] Initialized HD Wallet for submitting transactions
2024-09-27T01:59:17.158Z [INFO] Successfully opened connection to key DB at tmp\zether\keysdb
2024-09-27T01:59:17.160Z [INFO] Configurations:
2024-09-27T01:59:17.160Z [INFO]             data dir: ./tmp/zether
2024-09-27T01:59:17.160Z [INFO]              eth URL: ws://localhost:8545
2024-09-27T01:59:17.160Z [INFO]             chain ID: 1337
2024-09-27T01:59:17.160Z [INFO]                erc20: 0x5FbDB2315678afecb367f032d93F642f64180aa3
2024-09-27T01:59:17.160Z [INFO]                  DvP: 0xa513E6E4b8f2a923D98304ec87F64353C4D5C853
2024-09-27T01:59:17.161Z [INFO]         epoch length: 6 seconds
2024-09-27T01:59:17.162Z [INFO] Listening on port 3001
```

## API Reference

Each function available in the API (`app.js`) will be described below. You can manually send requests (e.g. using `curl`) as shown in the examples below or you can use [Postman](https://www.postman.com/downloads/) to simplify the process of sending HTTP requests and analyzing responses. To facilitate using Postman, please import the collection `zether-client-DvP.postman_collection.json` available in the root folder of this project.

>The Postman collection does not include all available requests, but it encompasses the needed requests - from both users (buyer and seller) - to perform a DvP transaction with privacy using Anonymous Zether.

### Create a pair of ethereum and shielded accounts

The client uses a 1-1 mapping between Ethereum signing accounts and Shielded accounts.

```console
$ curl -H "Content-Type: application/json" -d {} http://localhost:3000/api/v1/accounts
{
  "eth": "0x1A3e76010Fa50764378078aD5CfE71b271559D70",
  "shielded": [
    "0x17872c1b0dbe8f96f3909025d8628d30d479ebcacfb1fcf809f8f3a2778691d9",
    "0x071cb2add5a05a8544bc8ea1b267d9fc5330ac20b2aeb4a36a459da8ea5a6e68"
  ]
}
```

### Look up hosted accounts

```console
$ curl -H "Content-Type: application/json" http://localhost:3000/api/v1/accounts
{
  "0xd6538eb66ED15247dD4C0370fc4388e197DF7F95": [
    "0x254dd333d42266a3482817b70c9d0820a2a61f3d083db0bf067c2c4c48979609",
    "0x12ca7a2944688563b17223652501d6e2ceaaf3bb0586cc03903b5810e0556bbf"
  ],
  "0x6d6052B851E6EBfae1458e23BbD1119F182cda05": [
    "0x22c5f8e5aa98b26c741756c2c06887ab42cf5512649afb70461229214ba4b9c2",
    "0x230778e08977c0d03cad5ab1d5d67439bb886b912745ebc66495ae2229f60094"
  ],
  "0x173Fdf2C4845D61dB0CB93B789154475aff30729": [
    "0x088e0c151ce3828ac25245a6fd56977f986ae3bb3724651b4bf7b49ed45f1269",
    "0x083c23057e981fc55494300414be6ddb4b42f9c6a4c8e6f989cf9419c2cff3c8"
  ]
}
```

### Mint ERC20 tokens

This gives ethereum accounts some ERC20 tokens, which then can be deposited to their corresponding shielded accounts for secure transfers.

```console
$ curl -H "Content-Type: application/json" -d '{"ethAddress":"0x173Fdf2C4845D61dB0CB93B789154475aff30729","amount":1000,"isERC20":true}' http://localhost:3000/api/v1/mint/erc20
{
  "success": true,
  "transactionHash": "0x480eb9c7093f528c2fcd1c481e3dc90962c90fc5a2f2b28177ed1a21d72b3cf5"
}
```

> You can also mint ERC1155 tokens using the endpoint `/api/v1/mint/erc1155`. In this case, the parameter "isERC20" must be set to false.

### Look up ERC20 balances

Query the ERC20 balances for an Ethereum account.

```console
$ curl -H "Content-Type: application/json" http://localhost:3000/api/v1/accounts/0x173Fdf2C4845D61dB0CB93B789154475aff30729/balance
{
  "balance": "1000"
}
```

### Deposit Zether

Call the following endpoint to deposit ERC20 tokens to Zether and get the corresponding shielded account funded with the equal amount of zether tokens.

```console
$ curl -H "Content-Type: application/json" -d '{"ethAddress":"0x173Fdf2C4845D61dB0CB93B789154475aff30729","amount":100,"zsc":"0x5FC8d32690cc91D4c39d9d3abcBD16989F875707"}' http://localhost:3000/api/v1/fund/erc20
{
  "success": true,
  "transactionHash": "0x480eb9c7093f528c2fcd1c481e3dc90962c90fc5a2f2b28177ed1a21d72b3cf5"
}
```

> You can also fund ERC1155 tokens to the shielded account using the endpoint `/api/v1/fund/erc1155`. In this case, specify the ZSC token contract corresponding to the ERC-1155 token.

### Lookup zether balances

Query the zether balances for a shielded account.

```console
$ curl -H "Content-Type: application/json" http://localhost:3000/api/v1/accounts/0x088e0c151ce3828ac25245a6fd56977f986ae3bb3724651b4bf7b49ed45f1269,0x083c23057e981fc55494300414be6ddb4b42f9c6a4c8e6f989cf9419c2cff3c8/balance?zsc=0x5FC8d32690cc91D4c39d9d3abcBD16989F875707
{
  "balance": 100
}
```

### Transfer Zether

Transfer zether between shielded accounts.

```console
$ curl -H "Content-Type: application/json" -d '{"sender":"0x088e0c151ce3828ac25245a6fd56977f986ae3bb3724651b4bf7b49ed45f1269,0x083c23057e981fc55494300414be6ddb4b42f9c6a4c8e6f989cf9419c2cff3c8","receiver":"0x21c7312f9589a25e6ba90371046e95e511ae4ac71f1d75ebd32e0cb11454fee7,0x05607ed8af5a53bc9cf1955a377bb229e6d2a10e7a35e2a381021e641231d4b2","amount":10}' http://localhost:3000/api/v1/transfer
{
  "success": true,
  "transactionHash": "0x480eb9c7093f528c2fcd1c481e3dc90962c90fc5a2f2b28177ed1a21d72b3cf5"
}
```

### Withdraw Zether

Burn the zether tokens and get back equal amount in ERC20 tokens. This call burns the zether amount from the shielded account corresponding to the specified Ethereum account, and release equal amount of ERC20 tokens to the Ethereum account (which was being held by the Zether contract).

```console
$ curl -H "Content-Type: application/json" -d '{"ethAddress":"0x173Fdf2C4845D61dB0CB93B789154475aff30729","amount":10}' http://localhost:3000/api/v1/withdraw
{
  "success": true,
  "transactionHash": "0x480eb9c7093f528c2fcd1c481e3dc90962c90fc5a2f2b28177ed1a21d72b3cf5"
}
```

### Exchange an ERC-1155 token for an ERC-20 token (or vice-versa) through a DvP transaction:

The main feature added by this project is to implement a DvP transaction through Zether. For the complete flow of transactions, start two servers (e.g. on port 3000 and 3001, as described previously), with the correct .env files configured for each case, and follow the sequence of transactions in the Postman Collection `anonzether-client-DvP.postman_collection` available in the root folder. The 2 main functions involved are `startDvP`, to initiate the DvP process and generate the zero-knowledge proof, and `executeDvP`, to finalize the process, performing the atomic trade. Both users (buyer and seller) must execute these transactions.

```console
$ curl -H "Content-Type: application/json" -d '{"sender": "0x1d928a38961520e9811de0a16ccaf04b8b52c5a0edb1466a21ea28c01a7bb628,0x2daa93cb3205e54065db8405574ac909c9d0ab06ecf1836c35331148ceee0965", "receiver": "0x1302d6525fe11c00fa06f5e28b320d1549b695d6d1ea5fe378b18a1643cbfde8,0x2815bdb86c773743bb61886078a22f723b437228e9caa987189b3b2d584da11a", "amount": 1, "signer": "0x3b37A3A7016c708DAe69E229f87102Dc66957e72", "zsc": "0x0165878A594ca255338adfa4d48449f69242Eb8F"}' http://localhost:3000/api/v1/startDvP
{
  "success": true,
  "transactionHash": "0x84e273928598219e4d3e7b6d49516114ee4ebb6bd99744b84b51691fadef9252"
  "transactionSubmitter": "0x3b37A3A7016c708DAe69E229f87102Dc66957e72",
  "proof": "0x0fb64548826586d4424b0fd5604800c66be15a021e36abae6f1586026259779a208ca5a4c1e22f90e62a7cb6b55765141efecad6bcbb581b620942e7c78ee82e23cb727..."
}
```

```console
$ curl -H "Content-Type: application/json" -d '{"senderEthAddress":"0x3b37A3A7016c708DAe69E229f87102Dc66957e72","counterpartyEthAddress": "0xDe20B3d5fA8Cbe978161a5f7d2bA00184772F4E9","proof":"0x0fb64548826586d4424b0fd5604800c66be15a021e36abae6f1586026259779a208ca5a4c1e22f90e62a7cb6b55765141efecad6bcbb581b620942e7c78ee82e23cb727..."}' http://localhost:3000/api/v1/executeDvP
{
  "success": true,
  "transactionHash": "0xe40fb3ba9ae33f5cdabff464ec2b2cd3c6131a7b463df627a0f655fa9255fd05"
}
```

## Run Unit Tests

Make sure to delete the file `.env` from the root of the project, if you have created it. Otherwise it will interface with the tests and cause some tests to fail.

### Launch An Ethereum blockchain

There are tests that interacts with a live Ethereum blockchain. The easiest way to have such a test chain is by running a hardhat node. Refer to the instructions in the [Blockchain network](#blockchain-network) section.

### Deploy Contracts

The easiest way to deploy the Anonymous Zether contracts is by using the hardhat scripts as described in the [Deploy contracts](#deploy-contracts) section.

### Add test configurations

Add the following environment variables used by the tests, using the addresses of the `CashToken` and `ZSC` contracts resulted from the deployment above:

```console
export ERC20_ADDRESS_TEST=0x9C62Ce07E53FC2fE1C3fd834CD5B762f39c85440
export ZSC_ADDRESS_TEST=0x0C27D112B6E02BC662EeD8eF577449Effc04DFc5
```

### Run Tests

Then you can launch the test suite:

```console
npm test
```
