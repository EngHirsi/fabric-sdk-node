# fabric-sdk-node

Applications that use Hyperledger Fabric need a way to connect with a network identity, submit transactions, and query ledger data. fabric-sdk-node provides JavaScript client APIs for these tasks, plus clients for Fabric CA and lower-level network access.

> **Compatibility note:** The upstream SDK API was deprecated as of Fabric v2.5. For applications targeting Fabric v2.4 and later, use the [Fabric Gateway client API](https://hyperledger.github.io/fabric-gateway/) instead.

## Features

- Submit transactions and query ledger data through the higher-level `fabric-network` API.
- Enroll application users and network identities with Fabric CA through `fabric-ca-client`.
- Send lower-level requests to a Fabric network through `fabric-common`.
- Access Fabric protocol buffer definitions for gRPC communication through `fabric-protos`.
- Use Hardware Security Modules through the SDK's PKCS #11 support.

## Installation

Install Node.js and npm. Check the package metadata for the Node.js versions supported by the SDK release you choose.

For an application that uses the higher-level network API:

```sh
mkdir fabric-app
cd fabric-app
npm init -y
npm install fabric-network
```

If your application also needs to enroll identities through Fabric CA:

```sh
npm install fabric-ca-client
```

These are the SDK's package names; `fabric-sdk-node` is the project name, not a command-line executable.

## Usage

You need a running Fabric network, a JSON connection profile, and a filesystem wallet with an enrolled identity. The identity must have permission to access the target channel and chaincode. This example does not create the network or enroll a user.

Save the following as `query.js`:

```js
const fs = require('fs');
const { Gateway, Wallets } = require('fabric-network');

async function main() {
  const [profilePath, walletPath, identity, channel, chaincode, transaction, ...args] =
    process.argv.slice(2);

  if (!transaction) {
    throw new Error(
      'Usage: node query.js <connection.json> <wallet> <identity> <channel> <chaincode> <transaction> [args...]'
    );
  }

  const connectionProfile = JSON.parse(fs.readFileSync(profilePath, 'utf8'));
  const wallet = await Wallets.newFileSystemWallet(walletPath);
  const gateway = new Gateway();

  try {
    await gateway.connect(connectionProfile, {
      wallet,
      identity,
      discovery: { enabled: true, asLocalhost: true }
    });

    const network = await gateway.getNetwork(channel);
    const contract = network.getContract(chaincode);
    const result = await contract.evaluateTransaction(transaction, ...args);
    console.log(result.toString());
  } finally {
    gateway.disconnect();
  }
}

main().catch((error) => {
  console.error(error.message);
  process.exitCode = 1;
});
```

For a local Fabric test network with the asset-transfer basic chaincode deployed, query an existing asset:

```sh
node query.js ./connection.json ./wallet appUser mychannel basic ReadAsset asset1
```

Replace the paths, identity label, channel, and chaincode arguments with values from your network. The discovery setting `asLocalhost: true` assumes network endpoints are exposed on the local machine; use `false` when the discovered hostnames are reachable directly.

To submit a ledger update, call `contract.submitTransaction()` instead of `contract.evaluateTransaction()`, with the transaction name and arguments defined by your chaincode.

The [upstream SDK documentation](https://hyperledger.github.io/fabric-sdk-node/) covers the package APIs and configuration options.

## License

Apache-2.0. This project is based on [hyperledger/fabric-sdk-node](https://github.com/hyperledger/fabric-sdk-node). Retain the upstream copyright notices, license text, and any applicable NOTICE file when redistributing the code.
