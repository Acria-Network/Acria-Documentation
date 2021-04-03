# Operating an Oracle Node

It is recommended to use the official Acria-Oracle-Node-Qt as an Oracle Node client. Via the Acria-Oracle-Node Client it is possible to serve a single API to all compatible chains simultaneously.

## Setting up Accounts

Right After starting the Acria Qt Node you have to configure the accounts (keystore files) to use for each chain you would like to support.

![alt text](/img/qt19.png)

Alternatively it is also possible to create a new keystore file. To do this simply click on the "Create Wallet" button followed by providing the password for the new keystore file.

![alt text](/img/qt20.png ':size=50%x50%')

!> **Important** All nonces are managed within the application. Therefore, do not attempt to make a transaction with an unlocked account outside of the application.

?> **Tip** Keystore files are saved under `./keystore/<chain>/<date>–<address>`

## Configuring your Node

After setting up the accounts you have to configure the Oracle Node to supply all your data items. To do this you have to click on the „Config“ button.

![alt text](/img/qt14.png)

After this a menu opens where you can specify each item. You can add a single API item by clicking on the „Add Item“ button.

![alt text](/img/qt13.png ':size=80%x80%')

### Creating a Resource Item

In the following dialog you can specify all the options of that resource.

1. The Name of the resource (e.g. GBP/USD)
2. The API URL which gets used if no additional parameters are provided
3. The API URL which gets used when an additional parameter gets provided. You should mark the place where the parameter should be placed with %data%.
4. The parameter type which indicates the way to handle the parameter provided. See the section parameter types for more information.

You must specify how the API response should be parsed. The options are:
- Json: You can specify the location of the value via Json keys.
- Regex: If the response is not a Json you can use a regular expression.

In addition to this you can also write a short description of the resource which is useful for everyone who intends to implement this oracle node.

![alt text](/img/qt11.png ':size=80%x80%')

?> **Tip** To Test all this you can make an example request to the API and then parse the response to see if everything works as expected.

### Editing a Resource Item

In addition it is also possible to edit an existing item. To edit an item simply double click it in the table view. After this, a window opens where you can edit it.

![alt text](/img/qt21.png ':size=80%x80%')

?> **Tip** You can also click on the "Save as Copy" button to save a copy of the item. This might be useful to create a lot of similar items.

### Parameter Types

The parameter type defines the way to process the parameter provided. For example a user might provide a unix-timestamp but your API URL actually needs a timestamp string such as "2021-18-03".

In such a case one would enter `timestamp => from_timestamp yyyy-MM-dd`. This command first converts the provided uint256 to a normal timestamp and afterwards to a timestamp with the format `yyyy-MM-dd`.

The available conversions/commands include:
- timestamp
- number
- string
- address
- from_timestamp \<format\>

?> **Tip** You can chain as many function as you like. It's not limited to two.

### Signing/Sharing Config File

For Developers to know which resource items are accessible it is important to give them access to the config file. To make this task simpler you can just click on the "Sign Config" button. 

After this a window opens with the configuration file (as Json) and the signed message for each chain (only if the private key is available). You can then click on the "Upload to Oracle-Marketplace" button to easily upload it to the official Acria-Oracle-Marketplace.

![alt text](/img/qt15.png ':size=80%x80%')

Developer can then on the Acria-Oracle-Marketplace see a detailed overview of the endpoints available.

![alt text](/img/s2.png)

?> **Tip** Signing the config file with the private key is required to prove that this is the real file. Otherwise anyone would be able to upload a config file in your name.

## Connect to a local blockchain

To connect to your local nodes you have to first click on the settings icon.

After this a menu opens where you can specify the RPC endpoint as well as the main account of your Ethereum, Polkadot and Binance Smart Chain node. In addition to this you should now also deploy your personal data contract.

![alt text](/img/qt17.png)

?> **Tip** The default RPC address of geth is http://127.0.0.1:8545

?> **Tip** It is also possible to specify a custom chain id. This can be useful for tools such as Ganache (when using a non-default chain id) as it's known to sometimes report the wrong chain id.

### Starting a geth node

To start a geth node on the goerli testnet run:

```cmd
./geth --goerli --http
```

Please refer to the official Ethereum documentation for more information on how to start a geth node.

https://geth.ethereum.org/docs/getting-started

?> **Tip** Don't forget the --http option as the Oracle Node is otherwise unable to communicate with the geth node!

### Starting a Binance Smart Chain node

Please refer to the Binance Smart Chain documentation on how to start a node.

https://docs.binance.org/smart-chain/developer/fullnode.html

## Verify the Node Operation

Click on the Incoming Transactions icon to view all unprocessed data requests. After a requests is completed it will be moved to the „completed requests“ tab.

![alt text](/img/qt5.png)

### View Transaction Information

You can double click on a row to view the details of the transaction.

![alt text](/img/qt18.png ':size=50%x50%')

## Slashing

If a validator misbehaves by supplying malicious data or due to excessive downtime their delegated stake, as well as the stake of everyone staking on this node, will be completely or partially slashed.

The various reasons due to which slashing may occur are in detail explained below.

## Fees

An Oracle Node receives 80% of the all fees paid. The other 20% are getting distributed among the Stakers.

?> **Tip** If there aren't any Stakers active the 20% staking reward is also rewarded to the Node operator.

## Withdrawing

It is possible to withdraw the earned fees on the balances tab. To withdraw click one of the button on the lower right.

![alt text](/img/qt16.png)

?> **Tip** It is not necessary to claim the reward every period as the reward can also be claimed in bulk for multiple periods at once.

?> **Tip** It is always possible to withdraw the fees although depending on the gas price it might be worth to wait in order to save some gas fees.

## Deploying a Node Contract

In order to operate an Oracle Node it is required to have a personal Oracle Contract deployed. A contract can be deployed via the settings tab.

![alt text](/img/qt8.png ':size=50%x50%')

?> **Tip** The name of an Oracle Node must be unique. The transaction will fail otherwise.
