[![Latest Version](https://img.shields.io/github/release/jaggedsoft/node-idex-api.svg?style=flat-square)](https://github.com/jaggedsoft/node-idex-api/releases) 
[![GitHub last commit](https://img.shields.io/github/last-commit/jaggedsoft/node-idex-api.svg?maxAge=2400)](#)
[![npm downloads](https://img.shields.io/npm/dt/node-idex-api.svg?maxAge=7200)](https://www.npmjs.com/package/node-idex-api)

[![NPM](https://nodei.co/npm/node-idex-api.png?compact=true)](https://npmjs.org/package/node-idex-api)

# Node IDEX API
This project is designed to help you make your own projects that interact with the [IDEX API](https://github.com/AuroraDAO/idex-api-docss). This project seeks to have complete API coverage including WebSockets.

#### Installation
```
npm install node-idex-api
```

#### Getting started
```js
(async () => {
    const idex = require("node-idex-api");
    let currencies = await idex.returnCurrencies();
    console.log(currencies);

    let volume = await idex.return24Volume();
    console.log(volume);
})();
```

> ### Example: Top 10 highest volume symbols
```js
let ticker = await idex.returnTicker();
//last high low lowestAsk highestBid percentChange baseVolume quoteVolume
let sorted = idex.obj_to_array(ticker).sort(function(a, b) {
    return b.baseVolume - a.baseVolume;
});
console.log(sorted.slice(0, 10));
```

> ### Examples/improvements are coming soon
```js
// Documentation
    returnTicker(symbol)
    return24Volume()
    returnOpenOrders(address)
    returnOrderBook(symbol)
    returnTradeHistory()
    returnCurrencies()
    returnBalances()
    returnCompleteBalances()
    returnDepositsWithdrawals()
    returnOrderTrades()
    returnNextNonce()
    returnContractAddress()
    order()
    trade()
    cancel()
    withdraw()
    websockets(ticker)
```

### returnTicker

Designed to behave similar to the API call of the same name provided by the Poloniex HTTP API, with the addition of highs and lows. Returns all necessary 24 hr data.

Possible JSON encoded parameters:

* market (string) - If included, this shall be the base market followed by an underscore, followed by the trade market. If omitted, will return an object of all markets

Sample code:

```js
console.log(await idex.returnTicker('ETH_SAN'));
```

Output for sample code would be of the following structure:

```js
{ last: '0.000981',
  high: '0.0010763',
  low: '0.0009777',
  lowestAsk: '0.00098151',
  highestBid: '0.0007853',
  percentChange: '-1.83619353',
  baseVolume: '7.3922603247161',
  quoteVolume: '7462.998433' }
```

To get data across all possible markets, use the same endpoint but omit the `market` parameter in the JSON payload. The object returned will be of the following structure:

```js
{
  ETH_SAN: 
   { last: '0.000981',
     high: '0.0010763',
     low: '0.0009777',
     lowestAsk: '0.00098151',
     highestBid: '0.0007853',
     percentChange: '-1.83619353',
     baseVolume: '7.3922603247161',
     quoteVolume: '7462.998433' },
  ETH_LINK: 
   { last: '0.001',
     high: '0.0014',
     low: '0.001',
     lowestAsk: '0.002',
     highestBid: '0.001',
     percentChange: '-28.57142857',
     baseVolume: '13.651606265667369466',
     quoteVolume: '9765.891979953083752189' }
  // all possible markets follow ...
}
```

Please note: If any field is unavailable due to a lack of trade history or a lack of 24hr data, the field will be set to `'N/A'`. `percentChange`, `baseVolume`, and `quoteVolume` will never be `'N/A'` but may be 0.

### return24Volume
Returns the 24-hour volume for all markets, plus totals for primary currencies. Sample JSON output:

* This function takes no JSON arguments

```js
{ ETH_REP: { ETH: '1.3429046745', REP: '105.29046745' },
  ETH_DVIP: { ETH: '4', DVIP: '4' },
  totalETH: '5.3429046745' }
```

### returnOrderBook

Returns the order book for a given market.

* market (string) - Required. E.g. `ETH_AURA`

Each order book has `asks` and `bids` properties that contain all the sell and buy orders sorted by best price. Order objects will contain a `price`, `amount`, `total`, and `orderHash` properties, as well as a `params` property which contains additional data about the order - this is useful for verifying the order's authenticity and to fill it. See the `trade` API call below for details on how to fill orders.

Sample response:

```js
{
    "asks": [
        {
            "price": "0.00031199",
            "amount": "9370.06502015067623067",
            "total": "2.923366585636809477",
            "orderHash": "0x22a9ba7f8dd37ed24ae327b14a8a941b0eb072d60e54bcf24640c2af819fc7ec",
            "params": {
                "tokenBuy": "0x0000000000000000000000000000000000000000",
                "buySymbol": "ETH",
                "buyPrecision": 18,
                "amountBuy": "2923366585636809477",
                "tokenSell": "0xcdcfc0f66c522fd086a1b725ea3c0eeb9f9e8814",
                "sellSymbol": "AURA",
                "sellPrecision": 18,
                "amountSell": "9370065020150676230670",
                "expires": 190000,
                "nonce": 42,
                "user": "0xcdabe4960cb700816a9b3e287b10b7c9ce39a127"
            }
        },
        ...
    ],
    "bids": [
        {
            "price": "0.000303401295904014",
            "amount": "3131.164000000000131072",
            "total": "0.94999921528799616",
            "orderHash": "0xdb5d8e8f07e97cebb8b670fd8338e5db81160523e663505c5ef2c59e6a8adcec",
            "params": {
                "tokenBuy": "0xcdcfc0f66c522fd086a1b725ea3c0eeb9f9e8814",
                "buySymbol": "AURA",
                "buyPrecision": 18,
                "amountBuy": "3131164000000000131072",
                "tokenSell": "0x0000000000000000000000000000000000000000",
                "sellSymbol": "ETH",
                "sellPrecision": 18,
                "amountSell": "949999215287996160",
                "expires": 10000,
                "nonce": 11532128096,
                "user": "0xaac6a264554eae80fc6335cf5b7b56bcad3055b0"
            }
        },
        ...
    ]
}
```

### returnOpenOrders

Returns the open orders for a given market or address.

* address (address string) - Returns all open orders placed by this address
* market (string) - Returns all open orders for a market. E.g. `ETH_AURA`

Either `address` or `market` is required.

The response is similar to the response of `returnOrderBook` except that orders are sorted chronologically (newest first) instead of by price. The data format of an order is the same, and there are some additional data points. See the sample below.

Sample response:

```js
[
    {
        "timestamp": 1516415000,
        "orderHash": "0x32858bf5d73f538d83893af18970ebd9bef4ace2df759d2a5ce8c9dbd5fff43f",
        "market": "ETH_AURA",
        "type": "sell",
        "orderNumber": 2228127,
        "params": {
            "tokenBuy": "0x0000000000000000000000000000000000000000",
            "buySymbol": "ETH",
            "buyPrecision": 18,
            "amountBuy": "5000000000000000000",
            "tokenSell": "0xcdcfc0f66c522fd086a1b725ea3c0eeb9f9e8814",
            "sellSymbol": "AURA",
            "sellPrecision": 18,
            "amountSell": "500000000000000000000",
            "expires": 190000,
            "nonce": 6,
            "user": "0x2db094ffeb85cb09de753846425d4c415cb2c07a"
        },
        "price": "0.01",
        "amount": "500",
        "total": "5"
    },
    ...
]
```

### returnTradeHistory

Returns the past 200 trades for a given market, or up to 5,000 trades between a range specified in UNIX timetsamps by the "start" and "end" properties of your JSON input.

Possible properties of JSON input:

* market (string) - If specified, will return an array of trade objects for the market, if omitted, will return an object of arrays of trade objects keyed by each market
* address (address string) - If specified, return value will only include trades that involve the address as the maker or taker. Note: if specified the `type` property of the trade objects will refer to the action on the market taken relative to the user, not relative to the market. This behavior is designed to mimic the My Trades section of the IDEX appication, also to mimic the behavior of the private `returnTradeHistory` API call on Poloniex
* start (number) - The inclusive UNIX timestamp (seconds since epoch, not ms) marking the earliest trade that will be returned in the response, if omitted will default to 0
* end (number) - The inclusive UNIX timestamp marking the latest trade that will be returned in the response. If omitted will default to the current timestamp

Sample output:

```js
{ ETH_REP: 
   [ { date: '2017-10-11 21:41:15',
       amount: '0.3',
       type: 'buy',
       total: '0.09',
       price: '0.3',
       orderHash: '0x600c405c44d30086771ac0bd9b455de08813127ff0c56017202c95df190169ae',
       uuid: 'e8719a10-aecc-11e7-9535-3b8451fd4699',
       transactionHash: '0x28b945b586a5929c69337929533e04794d488c2d6e1122b7b915705d0dff8bb6' } ] }
```

### returnCurrencies

Returns an object of token data indexed by symbol

This API call takes no input..

Sample output:

```js
{ ETH:
   { decimals: 18,
     address: '0x0000000000000000000000000000000000000000',
     name: 'Ether' },
  REP:
   { decimals: 8,
     address: '0xc853ba17650d32daba343294998ea4e33e7a48b9',
     name: 'Reputation' },
  DVIP:
   { decimals: 8,
     address: '0xf59fad2879fb8380ffa6049a48abf9c9959b3b5c',
     name: 'Aurora' } }
```

### returnBalances

Returns your available balances (total deposited minus amount in open orders) indexed by token symbol.

* address (address string) - Address to query balances of

Sample output

```js
{ REP: '25.55306545',
  DVIP: '200000000.31012358' }
```

### returnCompleteBalances

Returns your available balances along with the amount you have in open orders for each token, indexed by token symbol.

* address (address string) - Address to query balances of

Sample output

```js
{ REP: { available: '25.55306545', onOrders: '0' },
  DVIP: { available: '200000000.31012358', onOrders: '0' } }
```

### returnDepositsWithdrawals

Returns your deposit and withdrawal history within a range, specified by the "start" and "end" properties of the JSON input, both of which must be UNIX timestamps. Withdrawals can be marked as "PENDING" if they are queued for dispatch, "PROCESSING" if the transaction has been dispatched, and "COMPLETE" if the transaction has been mined.

* address (address string) - Address to query deposit/withdrawal history for
* start (number) - Inclusive starting UNIX timestamp of returned results, defaults to 0
* end (number) - Inclusive ending UNIX timestamp of returned results, defaults to current timestamp

Sample output:

```js
{ deposits:
   [ { depositNumber: 265,
       currency: 'ETH',
       amount: '4.5',
       timestamp: 1506550595,
       transactionHash: '0x52897291dba0a7b255ee7a27a8ca44a9e8d6919ca14f917616444bf974c48897' } ],
  withdrawals:
   [ { withdrawalNumber: 174,
       currency: 'ETH',
       amount: '4.5',
       timestamp: 1506552152,
       transactionHash: '0xe52e9c569fe659556d1e56d8cca2084db0b452cd889f55ec3b4e2f3af61faa57',
       status: 'COMPLETE' } ] }
```

### returnOrderTrades

Returns all trades involving a given order hash, specified by the `orderHash` property of the JSON input.

* orderHash (256-bit hex string) - The order hash to query for associated trades

Sample output:

```js
[ { date: '2017-10-11 21:41:15',
    amount: '0.3',
    type: 'buy',
    total: '0.09',
    price: '0.3',
    uuid: 'e8719a10-aecc-11e7-9535-3b8451fd4699',
    transactionHash: '0x28b945b586a5929c69337929533e04794d488c2d6e1122b7b915705d0dff8bb6' } ]
```

### returnNextNonce

Returns the lowest nonce that you can use from the given address in one of the trade functions (see below)

* address (address string) - The address to query for the next nonce to use

Sample output:

```js
{ nonce: 2650 }
```

### returnContractAddress

Returns the contract address used for depositing, withdrawing, and posting orders

* No JSON parameters

Sample output:

```js
{ address: '0x2a0c0dbecc7e4d658f48e01e3fa353f44050c208' }
```

## Contract-backed trade functions

This section will explain the required process for placing an order, filling a trade, cancelling an order, or making a withdrawal. Each function requires a signature using the private key associated with the address you want to interact with. When hashing input to be signed, you will always hash all the arguments using sha3 (keccak256), prefix the ASCII encoded message `\x19Ethereum Signed Message:\n32` to the hash, then hash the resulting data once more with sha3. The final hash is ready to be signed and sent to the server to be dispatched to the contract. All signatures must be in the `{ v, r, s }` form where `v` is a Number type between 27 and 28, and `r` and `s` are both hex strings with the standard 0x prefix.

The types of data to be hashed will always be `address`, `uint256`, or `bytes32`. Address types will always be left-padded to 160-bits, and number types will always be left-padded to `uint256`. bytes32 types will use the intuitive width of 256-bits. For easy-to-use hash functions we recommend using web3-utils for its `soliditySha3` function ([docs here](https://web3js.readthedocs.io/en/1.0/web3-utils.html#soliditysha3)), and for automatically salting your hash you can make use of ethereumjs-util for its `hashPersonalMessage` function ([docs here](https://github.com/ethereumjs/ethereumjs-util/blob/master/docs/index.md)). As long as the instructions are followed you can use any standard sha3 library for any language.

All hashes which are to be signed require a `nonce` parameter to derive, which must be a numeric value that increases by 1 with every message that requires a nonce. Naturally, a nonce is only associated with the address it is used from, so you can use 0 for a nonce if you were to interact with the server from a different address. If you need to find out what the next available nonce you can use in a signature is for a given address, use the `returnNextNonce` API call. A nonce must increase by 1 for every API call that requires one, i.e. you cannot use the same nonce for an order and then a withdrawal.

Some hashes to be signed require the contract address as the first argument to the hash function, you can retrieve the current contract address using the `returnContractAddress` API call. 

Both the `order` and `withdraw` API call require token addresses as input parameters. If you are interacting with ETH, you must supply the null address for it, i.e. 0x0000000000000000000000000000000000000000

There are no floating point values in the trade functions, you must adjust your number values which represent token quantities to the level of precision that the token uses. i.e. If you are filling an order with 1 of a token that uses 8 decimal places for precision, you must supply the value 100000000. To find out what amount of precision is used by the desired token, use the `returnCurrencies` API call.

If the type `uint256` is given as a parameter for any property of the JSON payloads, it is recommended you use a string to represent it.

If you are having issues producing the correct signature, review the `Exchange.sol` file for the actual Solidity code which validates them.

Tips for success:

1. Read this part of the documentation very carefully. **If you do not understand what is written here, you risk losing your funds.**
2. Always fully understand what you are signing, if something is unclear, reach out to our team before you send your signature off.
3. If you are handling data within a programming environment, favor arbitrary-precision arithmetic to avoid issues with precision with big numbers, especially when using JavaScript which exclusively uses 64-bit floating point values for its `Number` type.
4. Review relevant parts of the contract included in this repo to fully understand how the contract uses your data and associated signature.
5. Keep track of your nonce as you make use of the signed functions to ensure that they are always increasing as every request reaches the server, mind the note given below regarding trades that fill multiple orders, when you are filling many orders in one trade make sure the nonce increases with every item in the list, as the server will process them front-to-back

### order

Places a limit order on IDEX. The JSON object passed as input to this API call must include the following properties:

* tokenBuy (address string) - The address of the token you will receive as a result of the trade
* amountBuy (uint256) - The amount of the token you will receive when the order is fully filled
* tokenSell (address string) - The address of the token you will lose as a result of the trade
* amountSell (uint256) - The amount of the token you will give up when the order is fully filled
* address (address string) - The address you are posting the order from
* nonce (uint256) - One time number associated with the limit order
* expires (uint256) - *DEPRECATED* this property has no effect on your limit order but is still REQUIRED to submit a limit order as it is one of the parameters that is hashed. It must be a numeric type
* v - ...
* r - ... 
* s - (v r and s are the values produced by your private key signature, see above for details)

Example request body:

```js
{
  "tokenBuy": "0xd6e8a328c5c9b6cc4c917a50ecbe0aeb663c666e",
  "amountBuy": "1000000000000000000",
  "tokenSell": "0x0000000000000000000000000000000000000000",
  "amountSell": "20354156573527349",
  "address": "0x2dbdcec64db33e673140fbd0ceef610a273b84db",
  "nonce": 1544,
  "expires": 100000,
  "v": 28,
  "r": "0xc6ddcbdf69d0e20fe879d2405b40ee417773c8a177a5d7f4461f2310565ac3d1",
  "s": "0x497cdfedfde3308bb9d9e80ea2eabff43c7a15fef0eb164c265e3855a1bd9073"
}
```

To produce the required signature to place a limit order, you must hash the following parameters in this order

1. contract address
2. tokenBuy
3. amountBuy
4. tokenSell
5. amountSell
6. expires
7. nonce
8. address

Example code to hash and sign:

```js
const { soliditySha3 } = require('web3-utils');
const {
  hashPersonalMessage,
  bufferToHex,
  toBuffer,
  ecsign
} = require('ethereumjs-util')
const { mapValues } = require('lodash');
const raw = soliditySha3({
  t: 'address',
  v: contractAddress
}, {
  t: 'address',
  v: tokenBuy
}, {
  t: 'uint256',
  v: amountBuy
}, {
  t: 'address',
  v: tokenSell
}, {
  t: 'uint256',
  v: amountSell
}, {
  t: 'uint256',
  v: expires
}, {
  t: 'uint256',
  v: nonce
}, {
  t: 'address',
  v: address
});
const salted = hashPersonalMessage(toBuffer(raw))
const {
  v,
  r,
  s
} = mapValues(ecsign(salted, privateKeyBuffer), (value, key) => key === 'v' ? value : bufferToHex(value));
// send v, r, s values in payload
```

The response from the server will be a new order object if successful, or have an `error` property if unsuccessful

Sample output:
```js
{ orderNumber: 2101,
  orderHash: '0x3fe808be7b5df3747e5534056e9ff45ead5b1fcace430d7b4092e5fcd7161e21',
  price: '0.000129032258064516',
  amount: '3100',
  total: '0.4',
  type: 'buy',
  params: 
   { tokenBuy: '0x7c5a0ce9267ed19b22f8cae653f198e3e8daf098',
     buyPrecision: 18,
     amountBuy: '3100000000000000000000',
     tokenSell: '0x0000000000000000000000000000000000000000',
     sellPrecision: 18,
     amountSell: '400000000000000000',
     expires: 100000,
     nonce: 1,
     user: '0x57b080554ebafc8b17f4a6fd090c18fc8c9188a0' } }
```

### trade

Making a trade on IDEX actually involves signing a message for each order you wish to fill across and passing in an array of trades. For trades that fill a single order, the usual array with 1 object, or the object alone. The benefit of passing in multiple objects to fill across is that your action is atomic. All trades either succeed or none succeed.

Properties of each trade object in the trade you submit:

* orderHash (256-bit hex string) - This is the unsalted hash of the order you are filling. See `raw` in the example code given with in the section that describes the `order` API call. The orderHash property of an order can be retrieved from the API calls which return orders, but for higher security we recommend you derive the hash yourself from the order parameters.
* amount (uint256) - This is the amount of the order you are filling, the raw value not adjusted for precision *IMPORTANT: THIS PROPERTY IS IN TERMS OF THE ORDER'S amountBuy PROPERTY. This is NOT the amount of `tokenSell` you are receiving, but the amount of `tokenBuy` you are filling the order with. Do not trade unless you fully understand this idea. The amount of the token you will receive as a result of the trade is proportional to the ratio between `amountSell` and `amountBuy`*
* nonce (uint256) - One time numeric value associated with the trade. *Note: if filling multiple orders in one trade, every nonce in the list of trades must be greater than the one in the previous item*
* address (address string) - The address you are transacting from
* v - ...
* r - ...
* s - v, r, and s refer to the values produced by signing the message

To derive the hash you must sign for each order you are filling across, you must hash the following values in this order

1. orderHash
2. amount
3. address
4. nonce

Apply the salt and hash the result as usual, then sign your salted hash.

NOTE: Currently, all orders being filled in a trade must be for the same tokenBuy/tokenSell pair, and must all be signed from the same address

Sample output:

```js
[ { amount: '0.07',
    date: '2017-10-13 16:25:36',
    total: '0.49',
    market: 'ETH_DVIP',
    type: 'buy',
    price: '7',
    orderHash: '0xcfe4018c59e50e0e1964c979e6213ce5eb8c751cbc98a44251eb48a0985adc52',
    uuid: '250d51a0-b033-11e7-9984-a9ab79bb8f35' } ]
```

### cancel

Cancels an order associated with the address. JSON input must include the following properties

* orderHash (256-bit hex string) - The raw hash of the order you are cancelling
* nonce (uint256) - One time number associated with the address
* address (address string) - The address you are sending the cancel from, must own the order
* v - ...
* r - ...
* s - v, r, and s values derived from signing the hash of the message

To derive the signature for this API call, hash the following parameters in this order

1. orderHash
2. nonce

Salt and sign the hash as usual to prepare your payload

Sample output:
```js
{ success: 1 }
```

### withdraw

Withdraws funds associated with the address. You cannot withdraw funds that are tied up in open orders. JSON payload must include the following properties:

* address (address string) - The address you are transacting from
* amount (uint256) - The raw amount you are withdrawing, not adjusted for token precision
* token (address string) - The address of the token you are withdrawing from, see earlier notes for ETH
* nonce (uint256) - One time numeric value associated with your address
* v - ...
* r - ...
* s - v, r, and s values obtained from signing message hash

To derive the signature for this API call, hash the following parameters in this order

1. contract address
2. token
3. amount
4. address
5. nonce

Salt the hash as described earlier and sign it to produce your signature triplet.

Useful response upon withdrawal success is in the works, for now simply test that there is no `error` property in the result object to confirm your withdrawal has succeeded.

## Direct contract calls

To deposit into the contract, you must use the public `depositToken(address,uint256)` contract call for ERC-20 tokens (following calling the token's `approve(address,uint256)` function for that amount using the contract address as the first argument). The first argument to `depositToken` is the address of the token you are depositing, and the second argument is the raw amount you are depositing.

To deposit ETH into the contract, simply call the `deposit()` contract function with the desired ETH value.

## Errors

If an error is returned from the API, it will be in the form of a simple object containing an `error` property whose value is the error message.

Example:

```js
{ error: 'Market ETH_BTC not found' }
```

## WebSocket API

The IDEX WebSocket API offers a simple mechanism to receive push updates about order book changes and new trades.

**Note that this API is under active development and that the API offered today is going to change with the upcoming v2 API rollout.** To highlight some of the upcoming changes, we will require you to subscribe to individual markets in the near future, and the data points returned for each event as well as the respective message formats are going to change. Consider the current API an experimental offering that allows you to explore the potential of future integrations.

### Handshake

All connections require completion of a handshake. To begin, open a WebSocket connection to `wss://v1.idex.market`. As soon as the connection is established, send a handshake request in the form of a JSON-encoded message:

```js
{
  "method": "handshake",
  "payload": {
    "type": "client",
    "version": "2.0",
    "key": "17paIsICur8sA0OBqG6dH5G1rmrHNMwt4oNk4iX9"
  }
}
```

The server will then respond by sending a message that contains `"method": "handshake"`. You are now connected and ready to receive messages.

Note that the `key` property is currently a static string as shown in the example and is likely to change in the future.

### Message Types

All messages are sent as JSON documents and have `method` and `payload` properties.

#### New Trade

May contain multiple trades.

```js
{
  "method": "notifyTradesInserted",
  "payload": [
    {
      "id": 1885452,
      "price": "0.00001846",
      "amountPrecision": "21216.22909795735940685",
      "totalPrecision": "0.391651589148292854",
      "date": "2018-07-18 22:55:15",
      "timestamp": 1531954515,
      "sellerFee": "0.000783303178296586",
      "buyerFee": "21.216229097957359407",
      "type": "sell",
      "tokenBuy": "0xebbdf302c940c6bfd49c6b165f457fdb324649bc",
      "amountBuy": "28375118824500956446371",
      "tokenSell": "0x0000000000000000000000000000000000000000",
      "amountSell": "523804693500287656",
      "feeMake": "1000000000000000",
      "feeTake": "4170296313232015",
      "gasFee": "2170296313232015",
      "buy": "0x51a6e2ca4c4cc2f86b0062e1b5000327c3e0dff4",
      "v": 27,
      "r": "0xc048e8a1e2b4f96a6eb3a4cfb30f5911fe6995e4aa4be89e0c368538c22c51e9",
      "s": "0x4a72b6056cfa6def96bf8e1086ae5700f7f05d7c541845ce9c884e48e143b27d",
      "user": "0x51a6e2ca4c4cc2f86b0062e1b5000327c3e0dff4",
      "sell": "0x66b9dfaea3ddef53b98da82a224f70842c817703",
      "hash": "0x997d3478478e4262fa4348781554213074a8f2e41d08820babe368f3e699789d",
      "nonce": 6782959,
      "amount": "21216229097957359406850",
      "usdValue": "274.1561124038049978",
      "gasFeeAdjusted": "0.00085",
      "uuid": "a29f6240-8add-11e8-9e20-3198b95d073e",
      "updatedAt": "2018-07-18T22:55:15.000Z",
      "createdAt": "2018-07-18T22:55:15.000Z"
    }
  ]
}
```

#### New Order

```js
{
  "method": "notifyOrderInserted",
  "payload": {
    "complete": false,
    "id": 36630055,
    "tokenBuy": "0x0000000000000000000000000000000000000000",
    "amountBuy": "422158261046066630",
    "tokenSell": "0x423b5f62b328d0d6d44870f4eee316befa0b2df5",
    "amountSell": "277737525277183800000",
    "expires": 190000,
    "nonce": 1346058,
    "user": "0xd2c3477f7fbf39ccbc29d5fabf901557b489b27b",
    "v": "27",
    "r": "0x29d5d52c5ffe0803c6f94b6dde4e1e5a97d0ff3f45e8ac0bf25195ffaee23d44",
    "s": "0x291089501f25c00e8c5c397bc2394814ad4a4c6f6773437d394386f8c1dc8e50",
    "hash": "0x094d676543ccaf0564741b9afb43307324730281434cc7ea763d3440a7ddef1a",
    "feeDiscount": "0",
    "rewardsMultiple": "100",
    "updatedAt": "2018-07-18T21:58:29.000Z",
    "createdAt": "2018-07-18T21:58:29.000Z"
  }
}
```

#### Order Cancellation

Cancellation of a single order:

```js
{
  "method": "pushCancel",
  "payload": {
    "id": 37585076,
    "hash": "0xab8242e6a1f4124875d61bc242f029f38038a3630eb99194c0d8bd9f65ab11d1",
    "user": "0x4706f0844ad8b8d27b4bb2d6f6ae0c0a08cf76bd",
    "v": 27,
    "r": "0x6b981fb497f3ffc5fe9dc71e651635f97e0d93fdd2b1d43a8c088bbf6733d159",
    "s": "0x4a1e0d2e11deca6430d7642802f6d0c1c3726bbde42749e8592cfa95aafb52d5",
    "updatedAt": "2018-07-18T22:54:43.000Z",
    "createdAt": "2018-07-18T22:54:43.000Z"
  }
}
```

Cancellation of multiple orders:

``` js
{
  "method": "pushCancels",
  "payload": [
    {
      "id": 37585211,
      "hash": "0x997d3478478e4262fa4348781554213074a8f2e41d08820babe368f3e699789d",
      "updatedAt": "2018-07-18T22:55:15.000Z",
      "createdAt": "2018-07-18T22:55:15.000Z"
    }
  ]
}
```
