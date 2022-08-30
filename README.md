---
title: BitMake Api Documentation

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

meta:
  - name: description
    content: Documentation for the bitmake API
  - name: keywords
    content: bitmake,API,Documentation

search: true

code_clipboard: true
---

# Introduction

Welcome to the BitMake API documentation. We offer complete REST, Websocket, and BitMake APIs to suit your algorithmic trading needs.
## Public definitions
### Terminology

These terms will be used throughout the documentation, so it is recommended especially for new users to read to help their understanding of the API.

* base asset refers to the asset that is the quantity of a symbol. For the symbol BTCUSDT, BTC would be the base asset.
* quote asset refers to the asset that is the price of a symbol. For the symbol BTCUSDT, USDT would be the quote asset.

### Enum definitions

#### Order side (side)

enum | description
----- | -------
BUY | Buy side
SELL | Sell side

#### Order type (orderType)

enum | description
----- | -------
LIMIT | Limit
MARKET | Market
LIMIT_MAKER | Limit Maker
CONDITION_MARKET | Condition Market
CONDITION_LIMIT | Condition Limit

#### Order status (orderStatus)

enum | description
----- | -------
NEW | NEW
PARTIALLY_FILLED | PARTIALLY_FILLED
FILLED | FILLED
CANCELLED | CANCELLED
PENDING_CANCEL | PENDING_CANCEL
REJECTED | REJECTED
PENDING_NEW | PENDING_NEW

#### Order cancel status (cancelStatus)

enum | description
----- | -------
SUCCESS | SUCCESS
ORDER_IN_CANCEL | ORDER_IN_CANCEL
ORDER_NOT_EXIST | ORDER_NOT_EXIST
ORDER_FILLED | ORDER_FILLED


#### Time in force (timeInForce)

enum | description
----- | -------
GTC | Good till cancel
FOK | Fill or kill
IOC | Immediate or cancel


#### Match type (matchType)

enum | description
----- | -------
NORMAL |  Normal
LIQUIDATION |  Liquidation
ADL |  ADL
SETTLEMENT |  Settlement

#### Open close (openClose)

enum | description
----- | -------
OPEN |  Open
CLOSE |  Close

## Rate limits

When a rate limit is exceeded, a status of 429 Too Many Requests will be returned.

* Public endpoints: We throttle public endpoints by IP: 10 requests per second. Some endpoints may have custom rate limits.
* Private endpoints: We throttle private endpoints by user ID: 10 requests per second. Some endpoints may have custom rate limits.

## Authentication

REST and WebSocket are the same authentication method.

### APIKey
* Create an APIKey on the website before signing any requests. After creating an APIKey, keep the following information safe:  
   * APIKey
   * SecretKey  
* Once API key is created, it is recommended to set IP restrictions on the key for security reasons.
* Never share your API key/secret key to ANYONE.

<aside class="notice">
    If the API keys were accidentally shared, please delete them immediately and create a new key.
</aside>

### Making Requests

For authenticated requests, the following headers should be sent with the request:

* `FM-API-KEY` Your API key.
* `FM-API-TIMESTAMP` Request timestamp (e.g. 1635157011351).
* `FM-API-SIGN` SHA256 HMAC of the following six strings, using your API secret, as a hex string:
    * Your API key, is the same as the `FM-API-KEY` header.
    * Request timestamp, is the same as the `FM-API-TIMESTAMP` header.
    * HTTP method in uppercase (e.g. GET or POST).
    * Request path, including leading slash and any URL parameters but not including the hostname (e.g. /account)
    * Request queryString, is the request url query string or omitted if there is no query parameters.
    * (POST only) Request body (JSON-encoded)

## HTTP return codes

* HTTP 4XX return codes are used for malformed requests; the issue is on the sender's side.
* HTTP 403 return code is used when the WAF Limit (Web Application Firewall) has been violated.
* HTTP 429 return code is used when breaking a request rate limit.
* HTTP 5XX return codes are used for internal errors; the issue is on Bitmake's side. It is important to NOT treat this as a failure operation; the execution status is UNKNOWN and could have been a success.  
    
# REST API

HTTP-based API with full trading and asset management functionality, with public orderbook and trades data as well as private account data and order management.

REST endpoint URL: https://api.bitmake.com/

Requests and responses use JSON.

## Markets
### Get index

> response example:

```javascript
[
    {
        "t": 1634897295001, // timestamp
        "s": "BTC-SWAP-USDT", // symbol name
        "i": "63283.815" // index price
    }, 
    {
        "t": 1634897296321, 
        "s": "BTC-SWAP", 
        "i": "63279.8"
    }
]
```

Get symbol's index price

#### Request path

`GET /t/v1/quote/index`

#### Request parameters

Parameter | Type | Required | Description
--------- | ------- | ------- | -----------
symbol | string | false | symbol name

### Get orderbook

> response example:

```javascript
{
    "s": "BTCUSDT", // symbol name
    "t": 1634892253479, // timestamp
    "vs": 24487496, // start version
    "ve": 24487558, // end version
    "b": [
        [
            "63502.45",  // bid price
            "0.047" // bid quantity
        ], 
        [
            "63497.85", 
            "0.051"
        ]
    ], 
    "a": [
        [
            "63502.47",  // ask price
            "0.008" // ask quantity
        ], 
        [
            "63507.26", 
            "0.051"
        ]
    ]
}
```

Get symbol's depth

#### Request path

`GET /t/v1/quote/depth`

#### Request parameters

Parameter | Type | Required | Description
--------- | ------- | ------- | -----------
symbol | string | true | symbol name
dumpScale | integer | false | merged price precision 
depthLevel | integer | false | max 100, default 20

### Get kline/candlestick data

> response example:

```javascript
[
    {
        "s": "BTCUSDT", // symbol name
        "t": 1634892720000,  // timestamp
        "c": "63566.27", // close price
        "h": "63582.45", // high price
        "l": "63554.92", // low price
        "o": "63559.58", // open price
        "v": "0.233", // trade quantity
        "qv": "14811.65" // trade amount
    },
    {
        "s": "BTCUSDT", 
        "t": 1634892660000, 
        "c": "63555.36", 
        "h": "63555.36", 
        "l": "63531.3", 
        "o": "63542.55", 
        "v": "0.159", 
        "qv": "10103.32"
    }
]
```

Get symbol's kline

#### Request path

`GET /t/v1/quote/klines`

#### Request parameters

Parameter | Type | Required | Description
--------- | ------- | ------- | -----------
symbol | string | true | symbol name
interval | string | true | kline type (1m,5m,15m,30m,1h,1d,1w,1M)
limit | integer | false | limit number, max is 1000,the default value is 1000,null means default
to | integer | false | the timestamp of last kline,null means default

### Get trades

> response example:

```javascript
[
    {
        "v": "2270426677249", // trade id
        "t": 1634893166526, // timestamp
        "p": "63566.18", // trade price
        "q": "0.006", // trade quantity
        "m": false // is buyer taker
    }, 
    {
        "v": "2270411292673", 
        "t": 1634893156723, 
        "p": "63566.34", 
        "q": "0.02", 
        "m": false
    }
]
```

Get symbol's last trades

#### Request path

`GET /t/v1/quote/trades`

#### Request parameters

Parameter | Type | Required | Description
--------- | ------- | ------- | -----------
symbol | string | true | symbol name
limit | integer | false | limit numbers. default 100,null means default

### Get ticker

> response example:

```javascript
[
    {
        "s": "BTCUSDT", // symbol name
        "t": 1634893200000, // timestamp
        "c": "63583.38", // close price
        "h": "65952.02", // high price
        "l": "62026.45", // low price
        "o": "65952.02", // open price
        "v": "790.209", // trade quanity
        "qv": "49752069.62", // trade amount
        "m": "-0.035914593669761744" // price change percent
    }
]
```

Get symbol's realtime ticker

#### Request path

`GET /t/v1/quote/ticker`

#### Request parameters

Parameter | Type | Required | Description
--------- | ------- | ------- | -----------
symbol | string | false | symbol name,if it is empty, all ticker data will be returned,multiple symbol are separated by ','

## Account

### Get balance

> response example:

```javascript
[
    {
        "token": "USDT", // Currency
        "available": "98647.95", // Available balance of the currency
        "total": "100007.95" // Total balance of the currency
    }, 
    {
        "token": "BTC", // Currency
        "available": "986.91", // Available balance of the currency
        "total": "986.91" // Total balance of the currency
    }
]
```

Get current account information.

#### Request path

`GET /f/v1/account/balance`

#### Request parameters


### Get margin info

> response example:

```javascript
{
  "symbol": "BTC_USD",
  "totalCollateral": "0",
  "freeCollateral": "0.000000000000000000",
  "maxLeverage": 10,
  "marginFraction": "0",
  "maintenanceMarginRequirement": "0.03",
  "shortMax": "0.000000000000000000",
  "shortMaxNonBorrow": "0",
  "longMax": "0.000000000000000000",
  "longMaxNonBorrow": "0",
  "baseAvailable": "0",
  "quoteAvailable": "0"
}
```

Get current account information.

#### Request path

`GET /f/v1/account/margininfo`

#### Request parameters

Parameter | Type | Required | Description
--------- | ------- | ------- | -----------
symbol | string | yes | symbol name

### Set leverage
> response example:

```javascript
{
    "leverage": 7
}
```

Set position leverage

#### Request path

`POST /f/v1/account/leverage`

#### Request parameters

Parameter | Type | Required | Description
--------- | ------- | ------- | -----------
symbol | string | yes | symbol name
leverage | int | yes | 

### Get positions
> response example:

```javascript
[
    {
        "symbol": "BTC-PERP",
        "quantiy": "-2000",
        "avgPrice": "45810.5",
        "realizedPnl": "-0.000071",
        "openValue": "-0.043658",
        "updateTime": 1631607593014
    }
]
```

Get user positions

#### Request path

`GET /f/v1/account/futures_positions`

#### Request parameters

Parameter | Type | Required | Description
--------- | ------- | ------- | -----------
symbol | string | no | If it is empty, all position data will be returned

## Order

### Place order

> response example:

```javascript
{
    "symbol": "BTCUSDT", 
    "orderId": 190937927663278200, 
    "clientOrderId": "clientTest001", 
    "createTime": 1630576601671
}
```
Place an order 

#### Request path

`POST /f/v1/order`

#### Request parameters

Parameter | Type | Required | Description
--------- | ------- | ------- | -----------
symbol | string | yes | symbol name |
clientOrderId | string | yes | custom order number (maximum length: 23 bytes)| 
price | string | yes | | 
quantity | string | yes | |
orderSide | string | yes | `BUY` or `SELL`|
orderType | string | yes | `LIMIT`,`MARKET`, `CONDITION_LIMIT`, `CONDITION_MARKET`|
timeInForce | string | yes | `FOK`, `GTC`, `IOC` |
triggerMorePrice | string | no | 
triggerLessPrice | string | no | 
executeMorePrice | string | no | 
executeLessPrice | string | no | 

### Cancel order

> response example:

```javascript
{
    "orderId": 783738728209001,
    "cancelStatus": "SUCCESS",
    "orderStatus": "CANCELLED",
    "executedQuantity": "0", 
    "executedAmount": "0"
}
```

Cancel an active order.

#### Request path

`DELETE /f/v1/order`

#### Request parameters

Parameter | Type | Required | Description
--------- | ------- | ------- | -----------
clientOrderId | string | no | custom order number
orderId | long | no | order id

### Batch cancel orders

> response example:

```javascript
[
    {
        "orderId": 783738728209001, 
        "cancelStatus": "SUCCESS"
    }, 
    {
        "orderId": 783738728209002, 
        "cancelStatus": "SUCCESS"
    }
]
```

Batch cancel an active order.

#### Request path

`POST /f/v1/order/batch_cancel`

#### Request parameters

<aside class="notice">
    When batch canceling orders, the request is a JSON array containing the following parameters.
</aside>

Parameter | Type | Required | Description
--------- | ------- | ------- | -----------
clientOrderId | string | no | custom order number
orderId | long | no | order id

### Get open orders
> response example:

```javascript
[
    {
        "clientOrderId": "1630658683006", 
        "orderId": 3176692328818573300, 
        "symbol": "BTCUSDT", 
        "orderSide": "BUY", 
        "orderType": "LIMIT", 
        "orderStatus": "FILLED", 
        "price": "1", 
        "quantity": "10000", 
        "amount": "0", 
        "executedQuantity": "10000", 
        "executedAmount": "10000", 
        "triggerMorePrice": "0", 
        "triggerLessPrice": "0", 
        "executedMorePrice": "0", 
        "executedLessPrice": "0", 
        "createTime": 1630658686301, 
        "updateTime": 1630658686301
    }
]
```

Get open orders

#### Request path

`GET /f/v1/openOrders`

#### Request parameters

Parameter | Type | Required | Description
--------- | ------- | ------- | -----------
symbol | string | false | symbol name
orderSide | string | false | enum order side:BUY or SELL
orderType | string | false | enum order type:LIMIT,MARKET, CONDITION_LIMIT, CONDITION_MARKET
fromId | long | false | Query start data ID, default 0,null means default
limit | long | false | limit number, default 100,null means default

### Get order history
> response example:

```javascript
[
    {
        "clientOrderId": "1630658683006", 
        "orderId": 3176692328818573300,
        "historyOrderIndex": 101,
        "symbol": "BTCUSDT", 
        "orderSide": "BUY", 
        "orderType": "LIMIT", 
        "orderStatus": "FILLED", 
        "price": "1", 
        "quantity": "10000", 
        "amount": "0", 
        "executedQuantity": "10000", 
        "executedAmount": "10000", 
        "triggerMorePrice": "0", 
        "triggerLessPrice": "0", 
        "executedMorePrice": "0", 
        "executedLessPrice": "0", 
        "createTime": 1630658686301, 
        "updateTime": 1630658686301
    }
]
```

Get order history

#### Request path

`GET /f/v1/historyOrders`

#### Request parameters

Parameter | Type | Required | Description
--------- | ------- | ------- | -----------
symbol | string | false | symbol name
orderSide | string | false | enum order side:BUY or SELL
orderType | string | false | enum order type:LIMIT,MARKET, CONDITION_LIMIT, CONDITION_MARKET
fromIndex | long | false | Query start `historyOrderIndex`, default 0, null means default
limit | long | false | limit number, default 100, null means default

### Get order detail
> response example:

```javascript
{
    "clientOrderId": "1630658683006", 
    "orderId": 3176692328818573300, 
    "symbol": "BTCUSDT", 
    "orderSide": "BUY", 
    "orderType": "LIMIT", 
    "orderStatus": "FILLED", 
    "price": "1", 
    "quantity": "10000", 
    "amount": "0", 
    "executedQuantity": "10000", 
    "executedAmount": "10000", 
    "triggerMorePrice": "0", 
    "triggerLessPrice": "0", 
    "executedMorePrice": "0", 
    "executedLessPrice": "0", 
    "createTime": 1630658686301, 
    "updateTime": 1630658686301
}
```

Get order detail

#### Request path

`GET /f/v1/spot/order`

#### Request parameters

Parameter | Type | Required | Description
--------- | ------- | ------- | -----------
clientOrderId | string | false | Custom order number, length must be less than 23 characters
orderId | long | false | 

<aside class="notice">
    clientOrderId and orderId must have one
</aside>

### Get trade list
> response example:

```javascript
[
    {
        "tradeId": 3176692328818606, 
        "orderId": 31766923288185732, 
        "matchTime": 1630658686301, 
        "matchType": "NORMAL", 
        "symbol": "BTCUSDT", 
        "price": "1", 
        "quantity": "10000", 
        "tradeFee": "0", 
        "tradeFeeToken": "USDT", 
        "isMaker": false, 
        "isBuyer": false
    }
]
```

Get trades for current account

#### Request path

`GET /f/v1/trades`

#### Request parameters

Parameter | Type | Required | Description
--------- | ------- | ------- | -----------
symbol | string | false | symbol name
orderSide | string | false | enum order side:BUY or SELL
openClose | string | false | `OPEN` or `CLOSE`
matchType | string | false | `NORMAL`, `ADL`, `LIQUIDATION`, `SETTLEMENT`
fromId | long | false | Query start data ID, default 0,null means default
limit | long | false | limit number, default 100,null means default


# WebSocket API

Streaming API with the most up-to-date market and account order data. With this API, you can send messages to a server and receive event-driven responses without having to poll the server for a reply.

## Public Channels

### Overview

#### Websocket endpoint URL

wss://api.bitmake.com/t/v1/ws

#### URL parameters

Parameter | Type | Required | Description
--------- | ------- | ------- | -----------
binary | string | no | Enable GZIP for push data(true/false)

Requests and responses use JSON.

> ping/pong example:

```javascript
{
    "ping": 1535975085052 // timestamp
}
{
    "pong": 1535975085052 // timestamp
}
```

#### Heartbeat

* The websocket server will send a ping frame every 3 minutes. If the websocket server does not receive a pong frame back from the connection within a 6 minute period, the connection will be disconnected. 
* After the client connects to the server, it needs to periodically send a ping frame to the server, and the server will reply to the client with a pong frame after receiving the message.

#### Subscription format

| Parameter   | Type           | Description |e.g |
| ----- | -------------------------------------- | ----------- | ----------- |
| e  |  string | request event    |"sub" |
| t  | string | subscribe topic    | "realtimes" |
| ps | json object  | topic parameters        | {"symbol": "LTCBTC"}

> Subscribe request/response example:

```javascript
// request
{
    "e": "sub",
    "t": "realtimes",
    "ps": {
        "symbol": "LTCBTC"
    }
}
// response
{
    "e": "sub",
    "t": "realtimes",
    "ps": {
        "symbol": "LTCBTC"
    },
    "co": 0
}
```

#### Request event

| event  | description  |
| ------------ | ------------ |
| sub  | subscribe topic  |
| cancel  |  unsubscribe topic  |
| cancel_all  |  unsubscribe all topics |

#### Subscribe topic 

| topic  | description  |
| ------------ | ------------ |
| trade  | symbol trade data  |
| realtimes  |  symbol realtime market data |
| kline  |  symbol kline data |
| index  |  index data |
| diffMergedDepth  |  symbol depth data |
| bookTicker  | symbol best price |

#### Error
> Error message example

```javascript
{
    "co": "-100010", // error code
    "m": "Invalid Symbols!" // error message
}
```
If client's subscribe request message contains error, Websocket server will responsd this JSON message:

Field | Type | Description
| --------| -------- | -----
co | string  | error code
m | string | error message

### Order book

> Subscribe request

```javascript
{
    "e": "sub",
    "t": "diffMergedDepth",
    "ps": {
        "symbol": "BTCUSDT",
        "dumpScale": 2, 
        "depthLevel": 4
    }
}
```
> Subscribe response

```javascript
{
    "t": "diffMergedDepth", 
    "e": "sub", 
    "ps": {
        "dumpScale": 2, 
        "depthLevel": 4, 
        "symbol": "BTCUSDT"
    }, 
    "co": 0, 
    "m": "success"
}
```
> Push data

```javascript
{
    "s": "BTCUSDT", 
    "t": 1634294382984, 
    "vs": 87093, 
    "ve": 87624, 
    "f": false, 
    "b": [
        [
            "59052.4", 
            "0.001083"
        ], 
        [
            "59050.17", 
            "0.000957"
        ]
    ], 
    "a": [
        [
            "59063.38", 
            "0.00467"
        ], 
        [
            "59063.52", 
            "0.005177"
        ]
    ]
}
```

#### Subscribe topic

diffMergedDepth

#### Subscribe parameters

Parameter | Type | Required | Description
--------- | ------- | ------- | -----------
symbol | string | yes | symbol name
dumpScale | integer | no | Merge precision, the default is the symbol price precision
depthLevel | integer | no | depth level:1-five 2-ten 3-twenty 4-fifty 5-one hundred

#### Push data

Field | Type | Description
| --------| -------- | -----
t | long  | timestamp
s | string | symbol
vs | long | start version id
ve | long | end version id
a | array | ask list [price, quantity]
b | array | bid list [price, quantity]
f | bool | is full data: true/false

<aside class="notice">
     will push the full data after subscribing. Incremental data will be pushed tick by tick, i.e. whenever there is change in order book.
</aside>

#### How to manage a local order book correctly
* Get a depth snapshot, first compare the price:
    * If there is no current price in the local full data, write the record to the correct location.
    * If there is a current price in the local full data, if `quantity` greater than 0 then update, if `quantity` equal to 0 then delete.
* Check whether it is continuous data
    * If the start version number(`vs`) of the subsequent data is not equal to the end version number(`ve`) of the local full data,then need to re-subscribe.
    
### Kline/Candlestick

> Subscribe request

```javascript
{
    "e": "sub",
    "t": "kline",
    "ps": {
        "symbol": "LTCBTC",
        "klineType": "15m"
    }
}
```
> Subscribe response

```javascript
{
    "e": "sub",
    "t": "kline",
    "ps": {
        "symbol": "LTCBTC",
        "klineType": "15m" // K线间隔
    },
    "co": 0, 
    "m": "success"
}
```
> Push data

```javascript
{
    "t": "kline", 
    "e": "sub", 
    "ps": {
        "symbol": "BTCUSDT", 
        "klineType": "1h"
    }, 
    "d": [
        {
            "s": "TBTC_USDC", 
            "t": 1634439600000, 
            "c": "61173.79", 
            "h": "61347.96", 
            "l": "61038.69", 
            "o": "61092.16", 
            "v": "2805.027318", 
            "qv": "171707186.93"
        }
    ], 
    "f": false
}
```
#### Subscribe topic

kline

#### Subscribe parameters

Parameter | Type | Required | Description
--------- | ------- | ------- | -----------
symbol | string | yes | symbol name
klineType | string | yes | 1m、5m、15m、30m、1h、2h、4h、6h、12h、1d、1w、1M

#### Push data

Field | Type | Description
| --------| -------- | -----
t | long  | timestamp
s | string | symbol
c | string | close price
h | string | high price
l | string | low price
o | string | open price
v | string | trading volume
qv | string | trading amount

### Ticker

> Subscribe request

```javascript
{
    "e": "sub",
    "t": "realtimes",
    "ps": {
        "symbol": "LTCBTC"
    }
}
```
> Subscribe response

```javascript
{
    "e": "sub",
    "t": "realtimes",
    "ps": {
        "symbol": "LTCBTC"
    },
    "co": 0
}
```
> Push data

```javascript
{
    "t": "realtimes", 
    "e": "sub", 
    "ps": {
        "symbol": "BTCUSDT"
    }, 
    "d": [
        {
            "s": "BTCUSDT", 
            "t": 1634289300000, 
            "c": "59093.73", 
            "h": "59402.09", 
            "l": "58742.02", 
            "o": "59390.24", 
            "v": "11.274047", 
            "qv": "664457.21"
        }
    ], 
    "f": false
}
```
#### Subscribe topic

realtimes

#### Subscribe parameters

Parameter | Type | Required | Description
--------- | ------- | ------- | -----------
symbol | string | yes | symbol name

#### Push data

Field | Type | Description
| --------| -------- | -----
t | long  | timestamp
s | string | symbol
c | string | close price
h | string | high price
l | string | low price
o | string | open price
v | string | trading volume
qv | string | trading amount

### Index

> Subscribe request

```javascript
{
    "e": "sub",
    "t": "index",
    "ps": {
        "symbol": "BTC-SWAP-USDT"
    }
}
```
> Subscribe response

```javascript
{
    "e": "sub",
    "t": "index",
    "ps": {
        "symbol": "BTC-SWAP-USDT"
    },
    "co": 0, 
    "m": "success"    
}
```
> Push data

```javascript
{
    "t": "index", 
    "e": "sub", 
    "ps": {
        "symbol": "BTC-SWAP-USDT"
    }, 
    "d": [
        {
            "t": 1634443347001, 
            "s": "BTC-SWAP-USDT", 
            "i": "61077.265"
        }
    ], 
    "f": false
}
```

#### Subscribe topic

index

#### Subscribe parameters

Parameter | Type | Required | Description
--------- | ------- | ------- | -----------
symbol | string | yes | symbol name

#### Push data

Field | Type | Description
| --------| -------- | -----
t | long  | timestamp
s | string | symbol
i | string | index price

### Trade

> Subscribe request

```javascript
{
    "t": "trade", 
    "e": "sub", 
    "ps": {
        "symbol": "BTC-SWAP-USDT"
    }
}
```
> Subscribe response

```javascript
{
    "t": "trade", 
    "e": "sub", 
    "ps": {
        "symbol": "BTC-SWAP-USDT"
    },
    "co": 0, 
    "m": "success"        
}
```
> Push data

```javascript
{
    "t": "trade", 
    "e": "sub", 
    "ps": {
        "symbol": "BTC-SWAP-USDT"
    }, 
    "d": [
        {
            "v": "399209283585", 
            "t": 1634443789436, 
            "p": "61113.08", 
            "q": "16.241479", 
            "m": false
        }
    ], 
    "f": false
}
```

#### Subscribe topic

trade

#### Subscribe parameters

Parameter | Type | Required | Description
--------- | ------- | ------- | -----------
symbol | string | yes | symbol name

#### Push data

Field | Type | Description
| --------| -------- | -----
t | long  | timestamp
s | string | symbol
q | string | quantity
m | bool | is maker
v | string | trade id

### Book ticker

> Subscribe request

```javascript
{
    "t": "bookTicker", 
    "e": "sub", 
    "ps": {
        "symbol": "ETHUSDT"
    }
}
```
> Subscribe response

```javascript
{
    "t": "bookTicker", 
    "e": "sub", 
    "ps": {
        "symbol": "ETHUSDT"
    },
    "co": 0, 
    "m": "success"        
}
```
> Push data

```javascript
{
    "t": "bookTicker", 
    "e": "sub", 
    "ps": {
        "symbol": "ETHUSDT"
    }, 
    "d": {
        "s": "ethusdt1001", 
        "b": "3156.06",
        "B": "0.051", 
        "a": "3156.09", 
        "A": "0.014", 
        "t": 1632707440724
    }, 
    "f": false
}
```

#### Subscribe topic

bookTicker

#### Subscribe parameters

Parameter | Type | Required | Description
--------- | ------- | ------- | -----------
symbol | string | yes | symbol name

#### Push data

Field | Type | Description
| --------| -------- | -----
t | long  | timestamp
s | string | symbol
b | string | best bid price
B | string | best bid quantity
a | string | best ask price
A | string | best ask quantity

## Private Channels

### Overview

#### Websocket endpoint URL

wss://api.bitmake.com/t/v1/ws

#### URL parameters

Parameter | Type | Required | Description
--------- | ------- | ------- | -----------
binary | string | no | Enable GZIP for push data(true/false)

Requests and responses use JSON.

> ping/pong example:

```javascript
{
    "ping": 1535975085052 // timestamp
}
{
    "pong": 1535975085052 // timestamp
}
```

#### Heartbeat

* The websocket server will send a ping frame every 3 minutes. If the websocket server does not receive a pong frame back from the connection within a 6 minute period, the connection will be disconnected. 
* After the client connects to the server, it needs to periodically send a ping frame to the server, and the server will reply to the client with a pong frame after receiving the message.

#### Subscription format

| Parameter   | Type           | Description |e.g |
| ----- | -------------------------------------- | ----------- | ----------- |
| e  |  string | request event type    |  |
| E  | long | event timestamp    | 1535975085052 |
| s  | integer | push sequence number    |  |
| d | json object  | push data      | 

#### Request event type

| event type  | description  |
| ------------ | ------------ |
| Balance  | full asset data push  |
| BalanceUpdate  |  incremental asset data push  |
| Order  |  full orders push |
| OrderUpdate  |  incremental orders push |
| FuturesPosition  |  full futures position data push |
| FuturesPositionUpdate  | incremental futures position data push |
| TradeUpdate  |  incremental trade data push |

### Balance

> Push data example:

```javascript
// Balance
{
    "e": "Balance", 
    "E": 1634638881029, 
    "s": 1, 
    "d": [
        {
            "token": "BTC", 
            "available": "10000", 
            "total": "10000"
        }
    ]
}
// BalanceUpdate
{
    "e": "BalanceUpdate", 
    "E": 1634639084790, 
    "s": 3, 
    "d": {
        "token": "USDT", 
        "available": "3776.743", 
        "total": "10000"
    }
}
```

#### Push data

Field | Type | Description
| --------| -------- | -----
token | string  | token name
available | string | available balance
total | string | total balance

### Order

> Push data example:

```javascript
// Order
{
    "e": "Order", 
    "E": 1634638881029, 
    "s": 1, 
    "d": [
        {
            "clientOrderId": "1634639083840", 
            "orderId": 1842828247041, 
            "symbol": "BTCUSDT", 
            "orderSide": "BUY", 
            "orderType": "LIMIT", 
            "timeInForce": "GTC", 
            "orderStatus": "FILLED", 
            "price": "62232.57", 
            "quantity": "0.1", 
            "amount": "6223.257", 
            "executedQuantity": "0.044", 
            "executedAmount": "2738.23308", 
             "createTime": 1634639084928, 
             "updateTime": 1634639084928
        }
    ]
}
// OrderUpdate
{
    "e": "OrderUpdate", 
    "E": 1634639084791, 
    "s": 1, 
    "d": {
        "clientOrderId": "1634639083840", 
        "orderId": 1842828247041, 
        "symbol": "BTCUSDT", 
        "orderSide": "BUY", 
        "orderType": "LIMIT", 
        "timeInForce": "GTC", 
        "orderStatus": "NEW", 
        "price": "62232.57", 
        "quantity": "0.1", 
        "amount": "6223.257", 
        "executedQuantity": "0.044", 
        "executedAmount": "2738.23308", 
        "createTime": 1634639084928, 
        "updateTime": 1634639084928
    }
}
```

#### Push data

Field | Type | Description
| --------| -------- | -----
clientOrderId | string  | client order id
orderId | long | order id
symbol | string | symbol
orderSide | integer | 
orderType | integer | 
timeInForce | integer | 
orderStatus | integer | 
price | string | 
quantity | string | 
amount | string | 
executedQuantity | string | 
executedAmount | string | 
createTime | long | order create time
updateTime | long | order update time

### Futures position

> Push data example:

```javascript
// FuturesPosition
{
    "e": "FuturesPosition", 
    "E": 1634692989265, 
    "s": 2, 
    "d": [
        {
            "symbol": "BTC-PERP",
            "quantity": "-1000", 
            "avgPrice": "64179", 
            "liquidationPrice": "53749.9", 
            "realizedPnl": "0",
            "openValue": "-0.015581", 
            "updateTime": 1634691420863
        }
    ]
}
// FuturesPositionUpdate
{
    "e": "FuturesPositionUpdate", 
    "E": 1634693038941, 
    "s": 4, 
    "d": {
        "symbol": "BTC-PERP",
        "quantity": "-990", 
        "avgPrice": "64179", 
        "liquidationPrice": "53749.9", 
        "realizedPnl": "-0.000000",
        "openValue": "-0.015425", 
        "updateTime": 1634693038973
    }
}
```

#### Push data

Field | Type | Description
| --------| -------- | -----
symbol | string  | symbol
quantity | string | 
avgPrice | string | 
liquidationPrice | string | 
realizedPnl | string | 
openValue | string |
updateTime | long |

### Trade

> Push data example:

```javascript
{
    "e": "TradeUpdate", 
    "E": 1634639084791, 
    "s": 1, 
    "d": {
        "tradeId": 1842828279809, 
        "orderId": 1842828247041, 
        "matchTime": 1634639084928, 
        "matchType": "NORMAL", 
        "symbol": "BTCUSDT", 
        "price": "62232.57", 
        "quantity": "0.044", 
        "tradeFee": "0.0000264", 
        "tradeFeeToken": "BTC", 
        "isBuyer": true, 
        "isMaker": false
    }
}
```

#### Push data

Field | Type | Description
| --------| -------- | -----
tradeId | string  | 
orderId | long | 
matchTime | string | 
matchType | string | 
symbol | string | 
price | string | 
quantity | string | 
tradeFee | string | 
tradeFeeToken | string | 
isBuyer | string | 
isMaker | string | 

# Error Codes

Errors consist of two parts: an error code and a message. Codes are universal, but messages can vary.

## REST

## WebSocket