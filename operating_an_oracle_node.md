# Operating an Oracle Node

It is recommended to use the official Acria-Oracle-Node-Qt as an Oracle Node client. Via the Acria-Oracle-Node Client it is possible to serve a single api to all compatible chains simultaniously.

## Setting up Accounts

Right After starting the Acria Qt Node you have to configure the accounts (keystore files) to use for each chain you would like to support. Alternatively it is also possible to generate a new keystore files which will be saved under ./keystore/&#60;chain&#62;/&#60;date&#62;--&#60;address&#62;

![alt text](/img/qt7.png)

## Configuring your Node

After setting up the accounts you have to configure the Oracle Node to supply all your data items. To do this you have to click on the „Config“ button.

![alt text](/img/qt1.png)

After this a menu opens where you can specify each item. You can add a single api item by clicking on the „Add Item“ button.

![alt text](/img/qt2.png ':size=50%x50%')

In the following dialog you can specify all the options of that data point.
1. The Name of the resource (e.g. GBP/USD)
2. The API url which gets used if no additional parameters are provided
3. The API url which gets used when an additional parameter gets provided. You should mark the place where the parameter should be placed with %data%.
4. The parameter type which indicates the way to handle the parameter provided. See the section parameter types for more information.

![alt text](/img/qt3.png ':size=50%x50%')

### Parameter Types

The parameter type defines the way to process the parameter provided. For example a user might provide a unix timestamp but your api-url actually needs a timestamp string such as "2021-18-03".

In such a case one would enter `timestamp => from_timestamp yyyy-MM-dd`. This command first converts the provided uint256 to a normal timestamp and afterwards to a timestamp with the format `yyyy-MM-dd`.

The available conversions/commands include:
- timestamp
- number
- string
- address
- from_timestamp \<format\>

?> **Tip** You can chain as many function as you like. It's not limited to two.

## Connect to a local blockchain

To connect to your local nodes you have to first click on the settings icon.

After this a menu opens where you can specify the RPC endpoint as well as the main account of your Ethereum, Polkadot and Binance Smart Chain node. In addition to this you should now also deploy your personal data contract.

![alt text](/img/qt4.png)

?> **Tip** The default RPC address of geth is http://127.0.0.1:8545

### Starting a geth node

To start a geth node on the goerli testnet run:

```cmd
./geth --goerli --http
```

Please refer to the official ethereum documentation for more information on how to start a geth node.

https://geth.ethereum.org/docs/getting-started

?> **Tip** Don't forget the --http option as the Oracle Node is otherwise unable to communicate with the geth node!

### Starting a Binance Smart Chain node

Please refer to the binance smart chain documentation on how to start a node.

https://docs.binance.org/smart-chain/developer/fullnode.html

## Verify the Node Operation

Click on the Incoming Transactions icon to view all unprocessed data requests. After a requests is completed it will be moved to the „completed requests“ tab.

![alt text](/img/qt5.png)

## Slashing

If a validator misbehaves by supplying malicious data or due to excessive downtime their delegated stake, as well as the stake of everyone staking on this node, will be completely or partially slashed.

The various reasons due to which slashing may occur are in detail explained below.

## Fees

An Oracle Node receives 80% of the all fees paid. The other 20% are getting distributed among the stakers.

?> **Tip** If there aren't any stakers active the 20% staking reward is also rewarded to the Node operator.

## Withdrawing

It is possible to withdraw the earned fees on the balances tab. To withdraw click one of the button on the lower right.

![alt text](/img/qt6.png)

?> **Tip** It is not neccessary to claim the reward every period as the staking reward can also be claimed in bulk for multiple periods at once.

?> **Tip** It is always possible to withdraw the fees although depending on the gas price it might be worth to wait in order to save some gas fees.

## Deploying a Node Contract

In order to operate an Oracle Node it is required to have a personal Oracle Contract deployed. A contract can be deployed via the settings tab.

![alt text](/img/qt8.png ':size=50%x50%')

?> **Tip** The name of an Oracle Node must be unique. The transaction will fail otherwise.