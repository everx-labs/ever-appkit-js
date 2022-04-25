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

If you are working with local blockchain [Evernode SE](https://github.com/tonlabs/evernode-se), specify [http://localhost](http://localhost) in the `endpoints`.

Check the full list of [supported network endpoints](https://docs.everos.dev/ever-sdk/reference/ever-os-api/networks).

You can find reference guide to `TonClient` here: [SDK API Documentation](https://docs.everos.dev/ever-sdk/reference/types-and-methods/mod\_client).
