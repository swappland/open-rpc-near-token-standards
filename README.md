# Open-RPC Near Token Stanndards Specifications 

[View the ft spec][ft-standard]
[View the nft spec][nft-standard]

Near token contract standards are a collection of OpenRPC specifications that describe
Near smart contract interfaces for tokens and their adjacent extensions such as storage management.

This interface allows for tooling and infrastructure to connect to Near smart contracts in a typesafe
way with minimal effort from the specification. 

It also allows for extending the specifications, to have documentation and client generation for your 
custom contracts that adhere to the Near token standards

## Usage
There are many ways to use this package. This can be used to build your own documentation and specs see [swappland][swappland-market]
or you can use the generated clients to just connect with standards compliant NFT contracts.



### Use the standard clients
```
npm install @swappland/ft-standards-client
npm install @swappland/nft-standards-client
npm install @swappland/simple-nft-client
npm install @swappland/simple-ft-client

```

```ts

import os from 'os'
import { WalletConnection, keyStores, connect, Near } from "near-api-js";
import {Account, NftToken} from "near-api-js";
import NftClient from "@swappland/nft-standards-client";

// You can use the same credentials you're already familiar with and just pass the 
// account object into the generated client construct along with contractId

const doFunClientStuff = async () => {

  // Near Setup Things
  const accountId: string = "some_account_you_have_in_credentials.near"
  const keystore = new keyStores.UnencryptedFileSystemKeyStore(`${os.homedir()}/.near-credentials`)
  const near = new Near({
      networkId: "mainnet",
      nodeUrl: "https://rpc.mainnet.near.org",
      walletUrl: "https://wallet.mainnet.near.org",
      explorerUrl: "https://explorer.mainnet.near.org", 
      deps: {keyStore}, headers:{}})

  // The only thing the client really needs 
  const account:Account = await near.account(accountId);
  const contractId: string = "some_nft_token.near";

  // The actual client setup
  const client = new NftClient({account, contractId});
  const token: NftToken = await client.nft_token("sometoken_id");
  console.log(JSON.stringify(token,null,2))
}

doFunClientStuff()
```

### Use standard clients to jumpstart creating your own client

See example [Swappland Market Client][swappland-market]. We recommend using the [client generator][generator] to get started 
building out your custom code that uses the standards. We have a suggested process that we use with our contract to make it
easy to manage and add your own spec data as needed, feel free to modify dir structure to suit your own needs.

#### Install the following
```
npm install @shipsgold/open-rpc-near-client-generator
npm install @swappland/open-rpc-token-standards-specs
npm install @xops.net/openrpc-cli
```
#### Add these to your scripts in package.json to jump start the standards process
```json
  scripts: {
    "get:standards": "mkdir -p standards && cp -r node_modules/@swappland/open-rpc-token-standards-specs/generated/specs/** standards/",
    "bundle:open-rpc": "mkdir -p build && openrpc-cli bundle -s openrpc.json > build/openrpc.json",
    "gen:client": "open-rpc-generator generate -c open-rpc-generator-config.json",
    "build": "npm run bundle:open && npm run gen:client"
  }
```

#### Create a generator client config
echo '{
{
  "openrpcDocument": "./openrpc.json",
  "outDir": "./generated-client",
  "components": [
      {
          "type": "custom",
          "name": "test-contract-client",
          "language": "typescript",
          "customComponent":  "@shipsgold/open-rpc-near-client-generator",
          "customType": "client"
      } 
  ]
' > open-rpc-generator-config.json
```sh

# simply take the standards spec and place it into standards location that you can then refer to 
npm run get:standards  
# Take the documents that you've generated and bundle them into a single openrpc.json
npm run bundle:open-rpc
# Generate client from the bundled open rpc data 
npm run gen:client 
# or use the now that was easy button that combines the bundle and generation steps
npm run build
```

This outputs a build directory and generated client directories that you can package however you like 
and use in web or node context


#### Why the bundling 
The standards break things into components and schemas into smaller JSONSchema based files. This allows us to 
have more portable standards, and allow you the developer to cherry pick the items you need and not repeat yourself.

This however, would mean loading remote assets, so much like webpack we bundle these files into a single document for
portablity and current usability schemes. To use this however we package the schemas into the same modular setting so you get 
high resusability and modularity.

### General Pattern
All generated clients use the same pattern of only requiring an Account Object as described by the near js api sdk, and a contractId.

```ts
const client = new Client({account, contractId});
```

## Organization 

The specification is split into multiple packages for standard generated clients, and the specifications themselves are 
broken down into `methods`, `schemas`, and `tags`.
├── methods
│   ├── ft
│   ├── nft
│   ├── simple-ft
│   ├── simple-nft
│   └── storage
├── packages
│   ├── ft-standard
│   ├── nft-standard
│   ├── simple-ft
│   ├── simple-nft
│   └── standards-specs
├── schemas
│   ├── simple-market
│   └── simple-nft
└── tags

### Packages 
Packages are the standard clients pre bundled and generated from the specifications, we've included the generated assets in build
for ease and accessibility. They also include the `open-rpc.json` file that specifies the actual contract. These combine the references from 
`methods`, `tags`, and `schemas`. These are then compiled into a single dereffed json file that can be used as a single all inclusive document to 
represent a contract.

### Methods
Methods are all the methods that are used by the standards, it combines `tags` and `schemas` to describe the interfaces for the smart contracts
that adhere to the standard.

### Schemas
Schemas are the individual components that represent the JSON Schema for many of the objects used and referred to by the specification.
The subdirs are simple extensions `simple-market` and `simple-nft` that don't really belong in the standard but were useful to include. They will eventually be deprecated.

### Tags
Are the standard tags that are within the spec. These include the special `change` and `view` tags that allow you to avoid specifiying
the specific details of amount and gas parameters. These are filled in by the client generator


## Build
Specs can be compiled into a single document for each type of contract as follows:

```console
$ npm install
$ npm run build --workspaces
Build successful.

# Example
# cat packages/nft-standard/build/openrpc.json
# cat packages/*/build/openrpc.json
```
This will outputs a`openrpc.json` in the root of the project. This file resolves the `#ref`s .

## Multi Token Standard
Coming soon, once it moves past Review. [Nep-245](https://github.com/near/NEPs/blob/master/neps/nep-0245.md)

## Contributing

The specification is written in [OpenRPC][openrpc]. Refer to the
OpenRPC specification and the JSON schema specification to get started.


[generator]: https://github.com/shipsgold/open-rpc-near-client-generator
[swappland-market]: https://github.com/swappland/swappland-market-contract-spec
[nft-standard]: https://playground.open-rpc.org/?schemaUrl=https://raw.githubusercontent.com/swappland/open-rpc-near-token-standards/main/packages/nft-standard/generated-client/custom/typescript/src/openrpc.json&uiSchema[appBar][ui:splitView]=false&uiSchema[appBar][ui:input]=false&uiSchema[appBar][ui:examplesDropdown]=false
[ft-standard]: https://playground.open-rpc.org/?schemaUrl=https://raw.githubusercontent.com/swappland/open-rpc-near-token-standards/main/packages/ft-standard/generated-client/custom/typescript/src/openrpc.json&uiSchema[appBar][ui:splitView]=false&uiSchema[appBar][ui:input]=false&uiSchema[appBar][ui:examplesDropdown]=false
[openrpc]: https://open-rpc.org
