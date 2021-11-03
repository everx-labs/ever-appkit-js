# Run ABI Get Method

* [About ABI Get Method](run\_abi\_get\_method.md#about-abi-get-method)
* [Run ABI get method](run\_abi\_get\_method.md#run-abi-get-method)
* [Sample source code](run\_abi\_get\_method.md#sample-source-code)

## About ABI Get Method

Get method is a method that is executed locally and it does not change account state. Get methods are used to retrieve contract data locally for free.

ABI compatible contract - a contract which has an ABI interface.

## Run ABI get method

AppKit provides `runLocal` method for running get methods of ABI-compatible contracts.

```
const dePoolAddress = "0:a07c4668a8ac1801b5ea77c86e317ca027d76c288c6da4d29d7d1fd716aff40a";

const dePoolAcc = new Account(DePoolContract, {
    address: dePoolAddress,
    client,
    signer: signerNone(), 
});

response = await dePoolAcc.runLocal("getDePoolInfo", {});
console.log(`DePool ${dePoolAddress} Info:`, response.decoded.output);
const validatorWallet = response.decoded.output.validatorWallet;

response = await dePoolAcc.runLocal("getParticipantInfo", { "addr": validatorWallet });
console.log(`\nValidator Wallet ${validatorWallet} Stake Info:`, response.decoded.output);

response = await dePoolAcc.runLocal("getDePoolBalance", {});
console.log(`\nDePool Balance Nano Crystal:`, response.decoded.output.value0);
```

## Sample source code

Observe the full sample: [https://github.com/tonlabs/sdk-samples/tree/master/appkit-examples/depool-statistics](https://github.com/tonlabs/sdk-samples/tree/master/appkit-examples/depool-statistics)

Check out [core api documentation](https://tonlabs.gitbook.io/ton-sdk/guides/work\_with\_contracts/run\_abi\_get\_method) for more information about running get methods.
