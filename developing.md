# Developing

Depending on the target blockchain the way to interact with the Acria Network might differ quite a bit. Please refer to the section explaining the process for the target blockchain.

## Ethereum

The Ethereum branch of the Acria Network operates via callbacks. Meaning that a request sent always produces an callback by the target Node.

To get started one should first select an Oracle Node offering the desired information. After this the information can be easily accessed via an external call to the Oracle Node contract.

### Requesting data

A complete example on how to achieve this can be seen below.

```js
// SPDX-License-Identifier: MIT

pragma solidity >=0.4.22 <0.9.0;


interface AcriaNode {
    function create_request(bytes8 _requestID, address callback, uint256 _expire, uint32 max_gas) external payable;
}

contract ClientExample {
    //The address of the targeted Oracle Node
    address payable node;

    //last most recent value received via a callback
    uint256 public lastValue;

    constructor(address payable _node) {
        node = _node;
    }

    function callOracle() public payable{
        //make a call to the Oracle Node and include the fee (all ETH provided)
        //first parameter is the item requested in this case the USD/GBP exchange rate
        //the second parameter is the callback address (this contract)
        //the third is the request expire date. It is expressed in blocks until the request should be dropped.
        //the fourth is the amount of gas the oracle node should provide for the callback. The higher the requested gas the higher should be the fee provided.
        AcriaNode(node).create_request{value: msg.value, gas: 100000}("USD/GBP", address(this), 10000, 50000);
    }

    //the function which gets called by the Oracle Node
    //it must be named value_callback with exactly one uint256 as parameter
    function value_callback(uint256 _value) public{
        //only the Oracle Node is allowed to call this function
        require(msg.sender == node);

        //update the value
        lastValue = _value;
    }
}
```

The actual call to the Oracle Node is here
```js
AcriaNode(node).create_request{value: msg.value, gas: 100000}("USD/GBP", address(this), 10000, 50000);
```

The parameters one has to provide are:
1. the item requested (in this case the USD/GBP exchange rate)
2. the callback address (this contract)
3. the request expire date. It is expressed in blocks until the request should be dropped. (here: 10000 blocks)
4. the amount of gas the oracle node should provide for the callback. The higher the requested gas the higher should be the fee provided.

In addition to this one also has to specify the fee which is in this the value provided by the call.

Please ensure that the function call is not reverting as the following parameter checks are done when creating a request.

```js
    require(_expire > 100);
    require(msg.value < 10**18);
    require(_expire < 1000000);
    require(max_gas < 500000);
```

?> **Tip** It is important to make sure that the fee provided `{value: msg.value}` covers the transaction fee of the oracle node which is roughly ~40000 gas plus the maximum gas need by the callback.

?> **Tip** If the fee is too low the request might not be processed immediately. In this regard the Arcia Network fee market works very much the same as the Ethereum gas market.

### Callback

After requesting data from an Oracle Node it responds via a special callback function. It is a special function which must be named value_callback with exactly one uint256 as parameter.

```js
    //the function which gets called by the Oracle Node
    //it must be named value_callback with exactly one uint256 as parameter
    function value_callback(uint256 _value) public{
        //only the Oracle Node is allowed to call this function
        require(msg.sender == node);

        //update the value
        lastValue = _value;
    }
```

?> **Tip** It is recommended to only allow the Oracle Node to call this function which we achieve with this line `require(msg.sender == node);`

?> **Tip** Don't forget to make the callback function `public` or `external` as it would be impossible for the Oracle Node to call it otherwise.

### Requesting Data with parameters

It is also possible to request data with an additional parameter. For example it is possible to request the exchange rate at an specific point in time (e.g. the BTC/USD exchange rate from yesterday 12am). 

A complete example on how to achieve this can be seen below.

```js
// SPDX-License-Identifier: MIT

pragma solidity >=0.4.22 <0.9.0;


interface AcriaNode {
    function create_request_with_data(bytes8 _requestID, address callback, uint256 _expire, uint32 max_gas, uint256 _data, uint256 _data_passthrough) external payable;
}

contract ClientExample {
    //The address of the targeted Oracle Node
    address payable node;

    //last most recent value received via a callback
    mapping(uint256 => uint256) public lastValue;
    
    uint256 counter;

    constructor(address payable _node) {
        node = _node;
    }

    function callOracle() public payable{
        //make a call to the Oracle Node and include the fee (all ETH provided)
        //1. parameter is the item requested in this case the USD/GBP exchange rate
        //2. parameter is the callback address (this contract)
        //3. parameter is the request expire date. It is expressed in blocks until the request should be dropped.
        //4. parameter is the amount of gas the oracle node should provide for the callback. The higher the requested gas the higher should be the fee provided.
        //5. parameter is the request data which further specifies the data needed (here: the current timestamp)
        //6. parameter is the passthrough data which will be passed through to the callback function (see third parameter of the callback)
        AcriaNode(node).create_request_with_data{value: msg.value, gas: 100000}("USD/GBP", address(this), 10000, 50000, block.timestamp, counter);
        
        counter++;
    }
    
    //the function which gets called by the Oracle Node
    //it must be named value_callback with exactly three uint256 as parameter
    function value_callback(uint256 _value, uint256 data, uint256 data_passthrough) public{
        //only the Oracle Node is allowed to call this function
        require(msg.sender == node);

        //update the value
        lastValue[data_passthrough] = _value;
    }
}
```

The actual call to the Oracle Node is here
```js
AcriaNode(node).create_request_with_data{value: msg.value, gas: 100000}("USD/GBP", address(this), 10000, 50000, block.timestamp, counter);
```

The first four parameters are the same as before but in addition to those there are now two additional parameters.
- the request data which further specifies the data needed
- the pass through data which will be unchanged passed through to the callback function

?> **Tip** Keep in mind that requests with parameters cost more gas (+40000). On the other hand the fee required might be slightly lower as more data can be deleted (15000 gas gets refunded per uint256) by the Oracle Node.

### Requesting Data with parameters - Callback

The callback function is pretty much the same as before with the only exception that it now features three parameters.

```js
    //the function which gets called by the Oracle Node
    //it must be named value_callback with exactly three uint256 as parameter
    function value_callback(uint256 _value, uint256 data, uint256 data_passthrough) public{
        //only the Oracle Node is allowed to call this function
        require(msg.sender == node);

        //update the value
        lastValue[data_passthrough] = _value;
    }
```

### Parsing Packed Node Response

Sometimes Acria-Oracle-Nodes respond with `uint32`, `uint64` or `uint128` variables merged together in a single `uint256` argument. To extract those values with Solidity you can use the following code.

```js
//2x uint128
var1 = uint128(_value);
var2 = uint128(_value >> 128);

//4x uint64
var1 = uint64(_value);
var2 = uint64(_value >> 64);
var3 = uint64(_value >> 128);
var4 = uint64(_value >> 192);
```

?> **Tip** For Oracle Node Operator see [Parsing the response with a Script](http://127.0.0.1:3000/#/operating_an_oracle_node#parsing-the-response-with-a-script)

### Fees

The Fee provided must cover the gas required (~40000 + whatever gas is required by the callback). Otherwise a request might be stuck till the Ethereum network fees are low enough for the request to be profitable for the Oracle Operator.

Oracle Operators generally only process transactions where the fee is 30% higher than the gas cost required.

The fee can be increased afterwards by calling the `pump_fee(uint256)` function of the Oracle Node Contract. The `uint256` parameter is the id of the request.

### Local Blockchain

It is possible to test smart contracts on a local Ethereum blockchain such as ganache (https://github.com/trufflesuite/ganache/releases). Although, in this case it is required to run a Oracle Node to supply data to the local node.

To deploy the contracts it is recommended to use truffle. After installing truffle run the following command to deploy the contracts.

```cmd
git clone https://github.com/Acria-Network/Acria-Contracts
cd Acria-Contracts
truffle develop
migrate
```

Alternatively it is also possible to use brownie (https://github.com/eth-brownie/brownie).

```cmd
brownie console
> run("deploy")
```

## Binance Smart Chain

The Binance Smart Chain is a fork of Ethereum. Therefore, please refer to the Ethereum section as the Binance Smart Chain works the same way as Ethereum.

?> **Tip** Don't forget that the contract address' might differ.

## Polkadot

## Run the chain as standalone node

Use Rust's native `cargo` command to build and launch the Acria node:

```sh
cargo run --release -- --dev --tmp
```
### Build the chain

The `cargo run` command will perform an initial build. Use the following command to build the node
without launching it:

```sh
cargo build --release
```

### User Interface

For debugging and testing the node, you can use this (web user interface)[https://ipfs.io/ipns/dotapps.io/].  
You should select the connection to your node on top left, for example if your node is installed in the same machine you are connecting from,
it will be `ws://127.0.0.1`   or `ws://localhost`.


### Api Interface

The node offers the following application programming interfaces, accessible from the user interface above:

 - acria.newOracle(oracleid,oracledata), a function to create a new ORACLE, the oracleid is an integer (u32) not already used and in the oracledata is a json structure with the following fields:  
    - shortdescription - a short description not longer than 64 bytes  
	- description  - a long description not longer than 6144 bytes  
    - apiurl  - an https address as reference for the API, explaining the possible parameters if any.  
    - fees - amount of fees applied to the requester.  
    example: {"shortdescription":"xxxxxxxxxxxxxxxxxx","description":"xxxxxxxxxxxxxxxxxxxxxxxxx","apiurl":"https://api.supplier.com/documentation","fees":0.0000001}  
 
 - acria.removeOracle(oracleid), a function to remove an ORACLE, only the original creator can remove it.  
 
 - acria.requestOracleUpdate(oracleaccount,oracleid), is the function used to request a data update to the Acria Oracle Node.  
 
 - acria.oracleUpdate(oracleid,oracledata), is the internal function used from the Oracle, to update the data on the blockchain.  

 - acria.oracle(AccountId,Oracleid), allows to query the data written from the Oracle matching the AccountId and Oracleid. From the user interface you should select "Chain State","Acria", "Oracle".


 For testing you should:  
 1. start the Blockchain node,  
 2. open the user interface (web user interface)[https://ipfs.io/ipns/dotapps.io/],  
 3. click on "Developer", "Extrinsics", "acria" and "newOracle",  
 4. select "Alice" account that will be the owner of the Oracle  
 5. insert "1" in the "oracleid" field  
 6. insert: 
 ```
 {"shortdescription":"Coingecko - Price BTC/USD","description":"Coingecko collect in real time the transaction from >20 exchanges and calculcate the average price every 60 seconds.","apiurl":"https://www.coingecko.com/","fees":0.1}  
```
in the field "oracledata",  
7. Click on "Submit Transaction" and check for the events shown.  
You should have stored a public Oracle in your blockchain.

8. now start the (Acria Oracle Node)[./oracle-node/README.md] in another screen  
9. from the web user interface select "acria" and "requestOracleUpdate",  
10. insert "1" in the field "oracleid" and "na" in the field "parameters" that could used to send some data to the API, in this case "na" = not applicable.  
11. click on "Submit Transaction",  
12. check the log of the AON and you should see that it has received the update request from the events generated and it has update the blockchain with its own data.  
