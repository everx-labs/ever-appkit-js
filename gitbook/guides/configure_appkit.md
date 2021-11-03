# Configure AppKit

### Using default config in AppKit

In `AppKit` you can specify the default config. This will allow you to omit passing `client` object into every `AppKit` function later. If a function works with another client (another network), you need to create a client object for that network separately, like we did in the previous step, and pass it as a parameter.

```graphql
TonClient.defaultConfig = {
network: {
    // Local node URL here
    endpoints: ['net.ton.dev']
  },
};
```

If you are working with [local blockchain TON OS SE](https://github.com/tonlabs/tonos-se), specify [http://localhost](http://localhost) in the `endpoints`.

Check the full list of [supported network endpoints](https://tonlabs.gitbook.io/ton-sdk/reference/ton-os-api/networks).

You can find reference guide to `TonClient` here: [TON-SDK API Documentation](https://tonlabs.gitbook.io/ton-sdk/reference/types-and-methods/mod_client).
