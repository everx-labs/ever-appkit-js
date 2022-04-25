# Run On-chain

Learn how to run methods of a contract on-chain

* [About Run](run\_onchain.md#about-run)
* [Run on-chain](run\_onchain.md#run-on-chain)
* [Sample source code](run\_onchain.md#sample-source-code)

## About Run

Run operation means that you execute the contract's function on-chain by sending it an external inbound message. Run steps

Run operation consists of few steps:

1. Creating a message;
2. Sending a message;
3. Receiving the message completion transaction;
4. Receiving external messages created by `return` function;
5. Decoding the messages bodies according to the ABI.

## Run on-chain

For running methods of ABI-compatible contracts AppKit provides `run` method:

```
 // Call `touch` function
let response = await helloAcc.run("touch", {});
```

## Sample source code

Observe the full sample: [https://github.com/tonlabs/sdk-samples/tree/master/demo/hello-wallet](https://github.com/tonlabs/sdk-samples/tree/master/demo/hello-wallet)

Check out [core api documentation](https://docs.everos.dev/ever-sdk/guides/work\_with\_contracts/run\_onchain) for more information about running a contract.
