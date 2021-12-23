# Implementing Custom Giver

By default `Account.getDefaultGiver()` is [TON OS SE giver](https://github.com/tonlabs/tonos-se/tree/master/contracts). It is integrated into Account module. We will use it. But you can always re-define it with method `Account.giver(newGiver: AccountGiver)` with the following signature:

```
/**
 * Object that can be used to send some value to an address 
 */
export type AccountGiver = (address: string, value: number) => Promise<void>;
```

This guide will help you to implement and configure your custom giver. In the example implementation we will use TON OS SE giver with its address, keys and ABI, but you can simply substitute them with your own.

First of all, let's declare our Giver's address, keys and ABI:

```javascript
// Address of the Giver
const giverAddress = "0:ece57bcc6c530283becbbd8a3b24d3c5987cdddc3c8b7b33be6e4a6312490415";
// Keypair for the Giver
const giverKeys = signerKeys({
    public: "2ada2e65ab8eeab09490e3521415f45b6e42df9c760a639bcf53957550b25a16",
    secret: "172af540e43a524763dd53b26a066d472a97c4de37d5498170564510608250c3",
});

const giverContract = {
    abi: {
        "ABI version": 2,
        header: ["time", "expire"],
        functions: [
            {
                name: "sendTransaction",
                inputs: [
                    {
                        "name": "dest",
                        "type": "address",
                    },
                    {
                        "name": "value",
                        "type": "uint128",
                    },
                    {
                        "name": "bounce",
                        "type": "bool",
                    },
                ],
                outputs: [],
            },
            {
                name: "getMessages",
                inputs: [],
                outputs: [
                    {
                        components: [
                            {
                                name: "hash",
                                type: "uint256",
                            },
                            {
                                name: "expireAt",
                                type: "uint64",
                            },
                        ],
                        name: "messages",
                        type: "tuple[]",
                    },
                ],
            },
            {
                name: "upgrade",
                inputs: [
                    {
                        name: "newcode",
                        type: "cell",
                    },
                ],
                outputs: [],
            },
            {
                name: "constructor",
                inputs: [],
                outputs: [],
            },
        ],
        data: [],
        events: [],
    },
};
```

Next create Giver's account object, using the values from the previous step:

```javascript
// Create Giver's account
const giverAccount = new Account(giverContract, {
    client,
    address: giverAddress,
    signer: giverKeys,
});
```

Now you can implement `AccountGiver` interface:

```javascript
const giverSendTo = async (address, value) => {
    // Run method `sendTransaction` for the Giver. You can use your custom account,
    // in this case, method name and arguments might vary:
    return await giverAccount.run("sendTransaction", {
        dest: address,
        value,
        bounce: false,
    });
};

// In order to implement giver's logics, we must implement `AccountGiver` interface
const giver = {
    address: giverAddress,
    sendTo: async (address, value) => await giverSendTo(address, value),
};
```

In this example, we created separated `giverSendTo` function in order to be able get result of `sendTransaction` execution, because `AccountGiver.sendTo` doesn't return any value.

Configure AppKit to use our Giver:

```javascript
// Set Giver for a client
Account.setGiverForClient(client, giver);
```

Thats it. Now any contracts deployment using this client will lead to using your configured Giver.

## Usage examples:

### Example 1. Contract deployment

In this example we will deploy test `HelloWallet` contract.

```javascript
// Generate an ed25519 key pair for new account
const helloAccKeys = await TonClient.default.crypto.generate_random_sign_keys();

// Create test contract
const helloAcc = new Account(HelloWallet, {
    signer: signerKeys(helloAccKeys),
    client,
});

// Get the future address of the contract:
const address = await helloAcc.getAddress();

// Request contract deployment funds form the Giver and deploy `HelloWallet` contract.
await helloAcc.deploy({ useGiver: true });
console.log(`HelloWallet contract was deployed at address: ${address}`);
```

### Example 2. Sending some funds from the Giver to a random address

In this example we will send 10 tokens from our Giver to a random address and check messages and transactions, which were created during the operation.

```javascript
// Generate new random keypair:
const helloAcc2Keys = await TonClient.default.crypto.generate_random_sign_keys();

// Create account object without client's connection. We don't need to deploy it, we just want to obtain it's future address:
const helloAcc2 = new Account(HelloWallet, {
    signer: signerKeys(helloAcc2Keys),
});

// Get address for the contract:
const address2 = await helloAcc2.getAddress();

console.log("Sending funds to address:", address2);

// Send funds using our separated function in order to obtain result message `id`:
const result = await giverSendTo(address2, 10000000000);

// Wait and get all messages and transactions, which were created during the whole operation:
const transaction_tree = await client.net.query_transaction_tree({
    in_msg: result.transaction.in_msg,
});

// Do some checks in order to assure, that funds are received:

assert.equal(transaction_tree.messages.length, 2, "There are must be 2 messages");
assert(!transaction_tree.messages[1].bounce, "Expected 2nd message to be not-bounceable");
assert.equal(
    transaction_tree.messages[1].value,
    '0x2540be400' /*10 000 000 000*/,
    "2nd message must have a value of 10 000 000 000",
);
assert.equal(transaction_tree.messages[1].dst, address2, "2nd message's destination must be " + address2);

assert.equal(transaction_tree.transactions.length, 2, "There are must be 2 transactions");
assert.equal(
    transaction_tree.transactions[1].account_addr,
    address2,
    "2nd transaction's account address must be " + address2,
);
assert(
    transaction_tree.transactions[1].aborted,
    "2nd transaction must be aborted because of uninitialized account",
);

console.log("Funds transferred.");
```