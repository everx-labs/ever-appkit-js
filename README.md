# Free TON JS Application Kit

This library is a part of Free TON SDK for JavaScript.

AppKit is built over the [@tonclient/core](https://github.com/tonlabs/ton-client-js) package and purposed to simplify writing applications on Free TON.

Full API reference: [https://tonlabs.github.io/appkit-js/](https://tonlabs.github.io/appkit-js/)

If this package helped you, please give it a star:)

**Have a question? Get quick help in our channel:**

[![Chat on Telegram](https://img.shields.io/badge/chat-on%20telegram-9cf.svg)](https://t.me/ton\_sdk)

## Table of Contents

  - [Useful links](#useful-links)
  - [Before You Start](#before-you-start)
  - [Installation](#installation)
  - [Setup Client Library](#setup-client-library)
    - [NodeJs:](#nodejs)
    - [Web:](#web)
    - [React Native:](#react-native)
  - [Create Client Instance](#create-client-instance)
  - [A Few Words about the Code](#a-few-words-about-the-code)
  - [Use Account Object](#use-account-object)
    - [Sample source code](#sample-source-code)
  - [Subscribe for Changes](#subscribe-for-changes)
  - [Executing Contract on TVM](#executing-contract-on-tvm)
  - [Interacting with Core SDK](#interacting-with-core-sdk)

## Useful links

* [Full API reference](https://tonlabs.github.io/appkit-js/)
* [Appkit samples](https://github.com/tonlabs/sdk-samples/tree/master/appkit-examples)
* [SDK guides](https://tonlabs.gitbook.io/ton-sdk/guides/quick\_start) - to get a deeper understanding dive into our sdk guides where you can find extensive explanations and descriptions of each step of DApp development on Free TON.

## Before You Start

We strongly recommend installing [TONDEV](https://github.com/tonlabs/tondev) utility before you start playing with TON AppKit. This utility will help you manage your tools for Free TON development.

## Installation

```shell
# Install core package
npm i --save @tonclient/core

# Install lib-node bridge if you write node js application
npm i --save @tonclient/lib-node

# Or install lib-web bridge if you write web/browser application
npm i --save @tonclient/lib-web

# Or install lib-react-native if you write react-native mobile application
npm i --save @tonclient/lib-react-native

# And finally install appkit itself
npm i --save @tonclient/appkit
```

## Setup Client Library

You must initialize the core library before the first use. The best place to do it is in the initialization code of your application.

### NodeJs:

```javascript
const { TonClient } = require("@tonclient/core");
const { libNode } = require("@tonclient/lib-node");

// Application initialization

TonClient.useBinaryLibrary(libNode)
```

### Web:

```javascript
import { TonClient } from "@tonclient/core";
import { libWeb } from "@tonclient/lib-web";

// Application initialization

TonClient.useBinaryLibrary(libWeb);
```

By default, the library loads wasm module from relative URL `/tonclient.wasm`.

You can specify alternative URL if you want to place (or rename) wasm module.

```javascript
import { TonClient } from "@tonclient/core";
import { libWeb, libWebSetup } from "@tonclient/lib-web";

// Application initialization.

// You have to setup libWeb if the `tonclient.wasm`
// isn't located at root of your web site.
// Otherwise you havn't to call `libWebSetup`.
libWebSetup({
    binaryURL: "/assets/tonclient_1_2_3.wasm",
});

TonClient.useBinaryLibrary(libWeb);
```

### React Native:

```javascript
import { TonClient } from "@tonclient/core";
import { libReactNative } from "@tonclient/lib-react-native";

// Application initialization

TonClient.useBinaryLibrary(libReactNative);
```

## Create Client Instance

TON AppKit is built over [core JS library](https://github.com/tonlabs/ton-client-js/tree/master/packages/core). So you have to create an instance of `TonClient` to use it later with TON AppKit objects.

```javascript
const client = new TonClient({
    network: { endpoints: ["http://localhost"] }
});
```

In this sample we create a client instance configured to use local blockchain [TON OS SE](https://github.com/tonlabs/tonos-se) instance.

If you want to work with Developer Network or Free TON network, please use the [list of endpoints, listed here](https://tonlabs.gitbook.io/ton-sdk/reference/ton-os-api/networks). **Attention** You must specify all the endpoints as an array in `endpoints` parameter, because each endpoint does not guarantee its availability, but we guarantee that at least one endpoint is operational at the moment.

## A Few Words about the Code

Below we use a code snippets to illustrate `AppKit` usage.\
In this code we omit an initialization part because it is the same.\
We suppose that we are using lib-node bridge (NodeJs) to write examples. Also, we use the library to deal with local [TON OS SE](https://github.com/tonlabs/tonos-se) instance.

So the full code of each example can look like this:

```javascript
const { TonClient } = require("@tonclient/core");
const { libNode } = require("@tonclient/lib-node");
const { Account } = require("@tonclient/appkit");

TonClient.useBinaryLibrary(libNode);

(async () => {
    const endpoint = process.env.TON_NETWORK_ADDRESS || "http://localhost";
    const client = new TonClient({ network: { endpoints: [endpoint] } });
    try {
        await main(client);
    } catch (err) {
        console.error(err);
    } finally {
        client.close();
    }
})();

async function main(client) {
    // Snippet code is here
}
```

## Use Account Object

At the moment the key point of `AppKit` is an Account object (class). Application uses an Account instance to deal with specific blockchain account using specific owner ( signer in terms of TonClient library).

Each Account instance must use an ABI compliant contract. So we have to define the `Contract` object with an ABI and optionally tvc fields. This object must be provided to the Account constructor.

In the example below we use predefined giver already included in AppKit and predeployed in TONOS SE.

```javascript
// Define Contract object.
const AccContract = {
    abi: { /* ABI declarations */ },
    tvc: "... base64 encoded string ...",
};

// Generate new keys pair for new account.
const keys = await client.crypto.generate_random_sign_keys();

// Create owner (signer) instance for new account.
const signer = signerKeys(keys);

// Construct Account instance.
//
// Note that this account is not deployed in the blockchain yet.
// We just create an object to deal with this account.
const acc = new Account(AccContract, { signer, client });

// We can determine the future addres of the account 
// and print it to the user before deploying.
console.log(`New account future address: ${await acc.getAddress()}`);

// Deploy account to the blockchain.
// Here we use TONOS SE giver to create a positive balance
// before deploying.
await acc.deploy({ useGiver: true });

// Send external inbound message to our new account
// and receives result from external outboud message.
const response = await acc.run("someFunction", { someParam: 1 });

// Print decoded response message
console.log("Account has responded to someFunction with", response.decoded.output);

// Print current balance.
// Note that balance returned as a string in decimal representation.
// This is because of a value measure is a nano.
// So its value may not be representable using JS Number.
console.log("Account balance now is", await acc.getBalance());
```

In the example above we demonstrated typical basic usage of the Account object.

### Sample source code

Find the sample that demonstrates AppKit usage source code here https://github.com/tonlabs/sdk-samples/tree/master/demo/hello-wallet

## Subscribe for Changes

Sometimes it is required to listen for events related to an account in realtime.

It is easy: just call one of the `subscribe` methods of an account instance.

For example, if we need to track all changes in the account state on the blockchain we can use `subscribeAccount`:

```ts
const hello = new Account(Hello, { signer });
await hello.deploy();

await hello.subscribeAccount("balance", (acc) => {
    // This callback triggers every time the account data 
    // is changed on the blockchain 
    console.log("Account has updated. Current balance is ", parseInt(acc.balance));
});

await hello.subscribeMessages("boc", async (msg) => {
    // This callback triggers every time the message related to this account 
    // is appeared on the blockchain.
    // Releated messages include inbound and outbound messages.  
    console.log("Message is appeared ", msg);
});

// ...... do something with hello account ...........

// In addition to other cleanup stuff the `free` method 
// unsubscribes all active subscriptions for this account instance.
await hello.free();
```

## Executing Contract on TVM

There are some situations where running the contract on the blockchain is not acceptable:

* Writing a tests for developing contract.
* Emulating execution for an existing account to detect failure reason or to calculate estimated fees.
* Getting information from an existing account by running its get methods.

In these cases we can play with an account on the TVM included in TON SDK client library:

```ts
const hello = new Account(Hello, { signer });

// We don't deploy contract on real network.
// We just emulate it. After this call the hello instance
// will have an account boc that can be used in consequent 
// calls.
await hello.deployLocal();

// We execute contract locally.
// But exactly the same way as it executes on the real blockchain.
const result = await hello.runLocal("touch", {});
console.log('Touch output', result);
```

We can call get method on accounts in the blockchain:

```ts
const acc = new Account(MyAccount, { address: someAddress });

// Contracts code and data will be downloaded from the blockchain
// and used to execute on the local TVM.
// Without any fees.
const lastBid = (await acc.runLocal("getLastBid", {})).decoded.output.lastBid;
console.log('Last bid is', lastBid);

// As laways we need to cleanup resources associated with insdtance.
await acc.free();
```

There are some situations where running the contract on the blockchain is not acceptable:

* Writing a tests for developing contract.
* Emulating execution for an existing account to detect failure reason or to calculate estimated fees.
* Getting information from an existing account by running its get methods.

In these cases we can play with an account on the TVM included in TON SDK client library:

```ts
const hello = new Account(Hello, { signer });

// We don't deploy contract on real network.
// We just emulate it. After this call the hello instance
// will have an account boc that can be used in consequent 
// calls.
await hello.deployLocal();

// We execute contract locally.
// But exactly the same way as it executes on the real blockchain.
const result = await hello.runLocal("touch", {});
console.log('Touch output', result);
```

We can call get method on accounts in the blockchain:

```ts
const acc = new Account(MyAccount, { address: someAddress });

// Contracts code and data will be downloaded from the blockchain
// and used to execute on the local TVM.
// Without any fees.
const lastBid = (await acc.runLocal("getLastBid", {})).decoded.output.lastBid;
console.log('Last bid is', lastBid);

// As laways we need to cleanup resources associated with insdtance.
await acc.free();
```

## Interacting with Core SDK

AppKit is a convenient library built over the TON-SDK core library. Of course the AppKit doesn't cover a lot of tasks the core SDK do.

So you have to easily interact with low level SDK functions. It is really easy.

Each `Account` instance has a reference to the TON SDK client instance. So you can use this reference.

```ts
async function test(wallet: Account) {
    const rnd = (await wallet.client.crypto.generate_random_bytes({
        length: 100
    })).bytes;
}
```

Each account instance has an `abi` field with ABI this account belongs to.

```ts
async function test(wallet: Account) {
    const decoded = (await wallet.client.abi.decode_message_body({
        abi: wallet.abi,
        body: someMessageBody,
        is_internal: false,
    })).value;
}
```
