# Deploy with AppKit

Find out how to deploy a contract to Free TON Blockchain with SDK

- [About Deploy](#about-deploy)
- [Deploy steps](#deploy-steps)
- [Initialize Account object](#initialize-account-object)
- [Calculate the future address of the contract](#calculate-the-future-address-of-the-contract)
- [Define deploy parameters](#define-deploy-parameters)
- [Specify Giver](#specify-giver)
- [Deploy](#deploy)
- [Sample source code](#sample-source-code)


# About Deploy

Deploy operation means that you upload contract code and initial data to the blockchain. 

The address of the contract can be calculated before deploy and it depends on code and data.  

To deploy a contract you need to sponsor its address, so that deploy fee will be paid out of these funds.

# Deploy steps

Below is the sequence of steps you need to complete to deploy a contract:

1. Initialize Account object with Contract data and a pair of keys that will be used later to interact with it. 
2. Calculate future address of the contract - In this case - just for information. 
3. Deploy

Let's take a look at every step.

# Initialize Account object

To initialize an account, we need:

- Contract object, that we import from `HelloContract.js`. We prepared this file when [adding contract to our app](https://github.com/tonlabs/TON-SDK/blob/master/guides/installation/3_add_contract_to_your_app.md).

- Account options, that are passed as the following structure:

```
    export type AccountOptions = {
    /**
     * Initial contract data, that is stored to contract data (along with pubkey) during deploy, that will affect account address. You can pass here data defined as public in the contract
     */
    initData?: object,
    /**
     * Default is `signerNone`
     Defines signature-provider that can be empty (no signature), can be a pair of keys,
     or a signing box interface that performs signing outside sdk. Affects account address. Read more about Signer type here https://github.com/tonlabs/TON-SDK/blob/master/docs/mod_abi.md#Signer
     */
    signer?: Signer,
    /**
     * If not specified will be calculated from initData, tvc and pubkey.
     */
    address?: string,
    /**
     * If not specified the Account.getDefaultClient() will be used.
     */
    client?: TonClient,
    }
```

We will generate a pair of keys with `generate_random_sign_keys` function and use them as Signer in this sample.  

We do not have any `initData` here, we do not know the address yet, and we will use the default client, so the only field we define in the second parameter `AccountOptions` is Signer. 

    // Generate an ed25519 key pair for new account
    const helloAcc = new Account(HelloContract, {
        signer: signerKeys(await TonClient.default.crypto.generate_random_sign_keys()),
    });

Or you can use a predefined set of keys.

    const keys  = {
			secret: "00232a8e5c0b156cbfe4fd41f9ca9194168fd441c18101526eec2567d36f22db"
            public: "f36f1bbfd0768ff4461e5743152180b9767519c40fe697cca0c1bfdfb622e11d",         
        }


# Calculate the future address of the contract

TON blockchain requires every contract to have a positive token balance before deployment. The contract pays for the initial deployment message reducing account balance. To sponsor account before deploy we need to calculate its address. 

Under the hood this calculation is performed on tvc, `initData` and `pubkey`.

    const address = await helloAcc.getAddress();
    console.log(`Future address of the contract will be: ${address}`);

# Define deploy parameters

To deploy a contract we need to pass `AccountDeployOptions` structure into `deploy` method:

    export type AccountDeployOptions = {
    /**
     * Function name that will be run on deploy. Special values:
     * - `undefined` (omitted): library will use `constructor` as a function name.
     * - `null`: no init function invocation.
     */
    initFunctionName?: string | null,
    /**
     * Parameters of init function.
     * Note: library ignores this parameter if `initFunctionName` is `null`.
     */
    initInput?: object,
    /**
     * Giver to be used to send amount of value to deploying address
     * before deploying.
     *
     * If `true` then Account.getDefaultGiver() will be used.
     * If omitted then application must prepay address
     * using own logic.
     */
    useGiver?: true | AccountGiver,
    }

# Specify Giver

By default `Account.getDefaultGiver()` is [TON OS SE giver](https://github.com/tonlabs/tonos-se/tree/master/contracts). It is integrated into Account module. We will use it.  But you can always re-define it with method `Account.giver(newGiver: AccountGiver)` with the following signature:

    /**
     * Object that can be used to send some value to an address 
     */
    export type AccountGiver = (address: string, value: number) => Promise<void>;


# Deploy

Because our contract has a constructor function as init function without parameters,  we can omit the other 2 deploy parameters:

    // Request contract deployment funds form a local TON OS SE giver
        // not suitable for other networks.
        // Deploy `hello` contract.
        await helloAcc.deploy({ useGiver: true });
        console.log(`Hello contract was deployed at address: ${address}`);

# Sample source code

https://github.com/tonlabs/sdk-samples/blob/master/demo/hello-wallet/index.js


Check out [core api documentation](https://github.com/tonlabs/TON-SDK/blob/master/guides/work_with_contracts/1_deploy.md) for more information.
