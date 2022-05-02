---
title: Custom Integrations
---

As of now, World ID only includes a [Javascript package](/docs/js) for use in web projects. However, creating a custom integration to use World ID in other type of projects (e.g. mobile apps) is fairly straightforward. Below you will find details on how to do this.

## Overview

Creating a custom integration involves the following process:

**On the frontend side:**

1. Establishing a connection to the Worldcoin app through [WalletConnect v1.0](https://docs.walletconnect.com).
2. Sending a `wld_worldIDVerification` JSON-RPC 2.0 request with the required proof input.
3. Receiving the proof results through WalletConnect.
4. Handling errors received from the Worldcoin app appropiately.

**On the blockchain side:**

5. Deploy your own smart contract that validates the ZKP and executes whatever action you want.
6. Submit a regular ETH wallet transaction containing the ZKP params.

:::note
Support for regular non-crypto applications will be possible soon. Stay tuned for more details on how to integrate with your own backend instead of the blockchain.
:::

## Details

:::tip
To test your integration or if you haven't gone to a Worldcoin orb you can test using the [Test network](/docs/about/test-network). You can easily launch a mock version of the app at [https://mock-app.id.worldcoin.org](https://mock-app.id.worldcoin.org) so you can test your flow end to end.
:::

### 1. Connecting to the Worldcoin app

Follow the [WalletConnect docs](https://docs.walletconnect.com/quick-start/dapps/client) to connect your app to the Worldcoin app. We strongly recommend you use the same UI/UX patterns used in the JS package.

<div className="text--center">
<img src="/img/world-id-js-modal.png" alt="Screenshot for World ID modal" width="500px" />
</div>

**Example:**

```js
import WalletConnect from "@walletconnect/client";

// Create a connector
const connector = new WalletConnect({
  bridge: "https://bridge.walletconnect.org",
});

// Check if connection is already established
if (!connector.connected) {
  // create new session
  await connector.createSession();
  // display the URI below in the modal's QR code
  console.log(connector.uri);
}
```

:::warning
Avoid using the default WalletConnect modal. A large number of users will not be familiar with this pattern and will not know to connect their Worldcoin app.
:::

### 2. Send verification request

The Worldcoin app will automatically accept the connection request after the user scans the QR code with their app. Upon the connection being established, send the verification request as a JSON-RPC 2.0 message with the following structure.

```js
const request = {
  id: 1000, // use a different ID every time
  jsonrpc: "2.0", // always `2.0`
  method: "wld_worldIDVerification", // always `wld_worldIDVerification`
  params: [
    {
      proofSignal:
        "0x0000000000000000000000004976fb03c32e5b8cfe2b6ccb31c09ba78ebaba41", // example; send relevant signal here
      externalNullifier:
        "0x0000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000f63616e64792d64726f702d323032320000000000000000000000000000000000", // example; send relevant external nullifier here
    },
  ],
};
```

**Example (partial, see #4 for complete snippet):**

```js
await connector.sendCustomRequest(request);
```

:::note
Please be mindful that the `proofSignal` and `externalNullifier` must be ABI-encoded. Please review [Parameter encoding](/docs/js/reference#parameter-encoding) for details.
:::

### 3. Receive the proof

When the Worldcoin app receives the verification request, it'll show a prompt to the user to confirm they want to verify their request. Once the user approves this request, you will receive the proof results. The proof result will be a JSON object with 3 ABI-encoded parameters, which your smart contract should expect. Read more about the received response [here](/docs/js/reference#response).

**Example (partial, see #4 for complete snippet):**

```js
const { merkleRoot, nullifierHash, proof } = await connector.sendCustomRequest(
  request
);
```

### 4. Handle errors appropiately

You should properly handle errors when interacting with the Worlcoin app. There are some error cases that in particular should be expected. For example, if the user declines the verification request in the Worldcoin app. Check out the list of [error codes](/docs/js/error-handling#error-codes) for details on possible error codes.

**Example:**

```js
try {
  const { merkleRoot, nullifierHash, proof } =
    await connector.sendCustomRequest(request);
} catch (errorResult) {
  const { code, detail } = errorResult;
}
```

### 5. Blockchain integration

Upon completing the integration steps above the only thing that remains is integrating the blockchain side. This process is the same as if you were using the [Javascript integration](/docs/js). We recommend you check out Step 4 and onwards from the [Quick start](/docs/quick-start) guide.