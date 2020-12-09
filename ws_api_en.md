AEX Websocket API Protocol Documentation (V3)
---

# API Request URL
```
wss://api.aex.zone/wsv3
Currently the Websocket API is only available for the api.aex.zone domain name.
The server will respond if the connection is established successfully   {"cmd":0}  
```

# Websocket connection process
```
With the same account, only one connection can survive at the same time. The connection with successful authentication will disconnect other connections. In the case of unauthentication, the same IP allows more connections.
such as:
  1) IP1 connection on websocket
  2) IP1 initiates an auth request and succeeds
  3) IP2 connection on websocket
  4) IP2 initiates an auth request and succeeds
  5) The successful connection of auth on IP1 will be disconnected actively by the server.
```

# Table of Contents
+ [Heartbeat message](#Heartbeat message)
+ [Request instructions](#Request instructions)
+ [Request example](#Request example)
+ [Protocol command word](#Protocol command word)
+ [Error Code](#error-code)
+ [Protocol request/response structure (json)](#protocol-request-response-data-structure)
   + [CMD: 1, Latest public transaction data](#cmd-1-Latest public transaction data)
   + [CMD: 2, K line](#cmd-2-K line)
   + [CMD: 3, Deep Disk](#cmd-3-Deep Disk)
   + [CMD: 4, 24-hour market](#cmd-4-24-hour market)
   + [CMD: 6, Streamline the 24-hour market](#cmd-6-Streamline the 24-hour market)
   + [CMD: 99, Login](#cmd-99-Login)
   + [CMD: 100, Individual order changes](#cmd-100-Individual order changes)
   + [CMD: 101, Personal latest transaction](#cmd-101-Personal latest transaction)
+ [Link orders and trades through tags](#link-orders-and-trades-through-tags)
+ [Implementation of trading strategy](#implementation-of-trading-strategy)

# Heartbeat message
```
When a user's Websocket client is connected to AEX's Websocket server, a ping message is sent to AEX periodically (currently 20 seconds) :

ping

When the AEX Websocket client receives this heartbeat message, it returns a pong message:

pong

When the AEX server does not receive a 'ping' message twice in a row, the server will actively disconnect from this client.

When the user's Websocket client does not receive 'pong' messages twice in a row, try to disconnect and reconnect.
```

# Request instructions
```
Instruction format

{"cmd":"<cmd>","action":"<action>","parameter1":"value1","parameter2":"value2"}

Among them, cmd is the identification of various channels, and the values of action are sub--subscribe; unsub--unsubscribe; login--login; logout--logout

Successful response format
{"cmd":99,"action":"login","code":20000}
{"cmd":4,"action":"sub","code":20000}

Failure response format

{"cmd":99,"action":"login","code":100003}
{"cmd":4,"action":"sub","code":10002}

cmd is the identification of the request, and action is the requested action. The difference between success and failure is mainly in the code. Success depends mainly on
The first digit of code is parity, even number means success, odd number means failure; failure code has actual meaning in the code set below.

```

# Request example
```
subscription

{"cmd":4,"action":"sub","symbol":"btc_cnc"}

unsubscribe

{"cmd":4,"action":"unsub","symbol":"btc_cnc"}

log in
{"cmd":99,"action":"login","key":"5f81d6e3cd3e96efa89d5297c1cec0c7","time":1594348186,"md5":"93a336226be823df3c7c50944df3eeb5"}

log out
{"cmd":99,"action":"logout"}

```

# the data shows
```
Divided into personal data, public data; personal data requires login authentication.
The request parameter symbol supports multiple subscriptions, connected by English commas.

Public data includes: latest transaction, K-line, depth, 24-hour market quotation, streamlined 24-hour market quotation.
Personal data includes: personal order changes, personal latest transactions.

```

# Protocol command word

|cmd|Description|
|:----  |-----   |
|1     | Latest public transaction data   |
|2     |  K line  |
|3     |  depth  |
|4     |  24-hour market  |
|6     |  Streamline the 24-hour market  |
|99     |  Login authentication  |
|100 | Individual order changes |
|101 | Personal latest transaction |


# Error Code
 
code | description
----- | ---------
100001 |key, time, md5 have null values
100002 |time is not of type int
100003 |The difference between the request time and the server timestamp exceeds 30s
100004 |The key value is illegal or contains illegal characters
100005 |non-existent key
100006 |md5 verification failed
100007 |api verification ip address failed
100008 |No access rights
20000 | Success

# Protocol request response data structure

# cmd-1-Latest public transaction data
```
Description:
no

Subscription format:
{"cmd":1,"action":"sub","symbol":"btc_cnc"}
{"cmd":1,"action":"sub","symbol":"btc_cnc,eth_cnc"}

Unsubscribe format:
{"cmd":1,"action":"unsub","symbol":"btc_cnc"}
```

Format parameter description:

Parameter name | Description
-----  | ---------
cmd | 1 mark
action| sub Action ID 
symbol | Trading pair  


Normal response (json)
  
```
{"cmd":1,"action":"sub","code":20000,"symbol":"btc_cnc"}
```

Push data normally (json)

```
{
  "trade": [
    [
      1594361458,
      "5.00000000",
      "35.00000000",
      "buy",
      44957
    ]
  ],
  "cmd": 1,
  "symbol": "btc_cnc"
}
```

Data push instructions:

Parameter name | Type | Description
-----  | ---------| ---------
cmd | int| mark
symbol |string| Trading pair  
trade.0 | int|   unix timestamp second level
trade.1 | string | The number of transactions 
trade.2 | string | deal price
trade.3 | string | Transaction Type buy or sell
trade.4 | int | Unique ID of transaction id



# cmd-2-K line
```
Description:
Push the data of the last K line

Subscription format:
{"cmd":2,"action":"sub","symbol":"btc_cnc@1mon,btc_cnc@4hour"}

Unsubscribe format:
{"cmd":2,"action":"unsub","symbol":"btc_cnc@1mon"}
```

Format parameter description:

Parameter name | Description
----- | ---------
cmd | Business Identity
action| Action ID
symbol | trading pair @ time period You can subscribe to more than one, separated by commas

Optional time period (json)

```
 '1mon' =>'one month',
 '4hour' => '4hour',
 '1day' =>'one day',
 '15min' => '15 minutes',
 '1min' => '1min',
 '5day' => '5day',
 '5min' => '5 minutes',
 '1wek' => '1 week',
 '1hour' => '1hour',
 '30min' => '30 minutes',
```

Normal response (json)
  
```
{"cmd":2,"action":"sub","code":20000,"symbol":"btc_cnc@1mon"}
```

Push data normally (json)
  
```
{
  "kline": {
    "i": "1min",
    "t": 1594364040,
    "v": "68",
    "o": "35",
    "h": "35",
    "l": "30",
    "c": "35"
  },
  "cmd": 2,
  "symbol": "btc_cnc@1hour"
}
```

Data push instructions:
  
Parameter name | Type | Description
----- | ---------| ---------
cmd | int| logo
symbol |string| trading pair
i |string| period
t |int| K-line opening time
v |string| Number of transactions
o |string| Opening price
h |string|High price
l |string| Low price
c |string| The latest price or closing price



# cmd-3-Deep Disk
```
Description:
no

Subscription format:
{"cmd":3,"action":"sub","symbol":"btc_cnc"}

Unsubscribe format:
{"cmd":3,"action":"unsub","symbol":"btc_cnc"}
```

Format parameter description:

Parameter name | Description
----- | ---------
cmd | Business Identity
action| Action ID
symbol | trading pair

Normal response (json)
  
```
{"cmd":3,"action":"sub","code":20000,"symbol":"btc_cnc"}
```

Push data normally (json)
  
```
{
  "depth": {
    "bids": [
      [
        "21.42857100",
        "28"
      ],
      [
        "3.50000000",
        "25"
      ]
    ],
    "asks": [
      [
        "162",
        "35"
      ],
      [
        "12289",
        "36"
      ]
    ]
  },
  "cmd": 3,
  "symbol": "btc_cnc"
}
  ```

Data push instructions:

Parameter name | Type | Description
----- | ---------| ---------
cmd | int| logo
symbol |string| trading pair
depth.buy |list| Buying depth Price from high to low
depth.sell |list| Buying depth Price from low to high
buy(sell).0.0 |string| The quantity to buy (sell) a plate
buy(sell).0.1 |string| The price of buying (selling) a plate


# cmd-4-24-hour market
```
Description:
no

Subscription format:
{"cmd":4,"action":"sub","symbol":"btc_cnc"}

Unsubscribe format:
{"cmd":4,"action":"unsub","symbol":"btc_cnc"}
```

Format parameter description:
  
Parameter name | Description
----- | ---------
cmd | Business Identity
action| Action ID
symbol | trading pair, usage: ①btc_cnc is a subscription single transaction; ②btc_cnc,eth_cnc is a subscription to multiple trading pairs at once

Normal response (json, regardless of the subscription method, a response is data for only one transaction pair)
  
```
{"cmd":4,"action":"sub","code":20000,"symbol":"btc_cnc"}
```

Push data normally (json)
  
```
{
  "ticker":{
    "c":"73915",
    "o":"-0.001",
    "h":"74289",
    "l":"73000",
    "q":"328.50085959",
    "m":"24254181.7361749",
    "b":"73892",
    "s":"73952"
  },
  "cmd":4,
  "symbol":"btc_cnc"
}

```

Data push instructions:

Parameter name | Type | Description
----- | ---------| ---------
cmd | int| logo
symbol |string| trading pair
c |string| current price
o |string| 24-hour change
h |string| 24 hours highest price
l |string| 24 hours lowest price
q |string| 24 Transaction quantity
m |string| 24-hour transaction amount
b |string| buy a price
s |string| Sell one price


# cmd-6-Streamline the 24-hour market

```
{"cmd":6,"action":"sub","symbol":"btc_cnc"}
```

Push data normally (json)
  
  ```
{
  "ticker": {
    "c": "36.00000000",
    "o": "0.0606",
    "m": "4536.00000000",
  },
  "cmd": 6,
  "symbol": "btc_cnc"
}
  ```


# cmd-99-Login

Subscription format:
```
 {
   "cmd": 99,
   "action": "login",
   "key": "5f81d6e3cd3e96efa89d5297c1cec0c7",
   "time": 1594348186,
   "md5": "93a336226be823df3c7c50944df3eeb5"
 }
```

Format parameter description:

Parameter name | Description
----- | ---------
cmd | 99 logo
action| login action ID
key | api key
time| current time Unix time unit is second 30s validity period
md5 | md5=md5("key_user_id_skey_time")

Normal response (json)
```
{"cmd": 99,"action": "login","code": 20000}
```

  
# cmd-100-Individual order changes
```
Description:
no

Subscription format:
{"cmd":100,"action":"sub","symbol":"btc_cnc"}

Unsubscribe format:
{"cmd":100,"action":"unsub","symbol":"btc_cnc"}
```
    
Format parameter description:
    
Parameter name | Description
----- | ---------
cmd | Business Identity
action| Action ID
symbol | trading pair
    
Normal response (json)
    
```
{"cmd":100,"action":"sub","code":20000,"symbol":"btc_cnc"}
```
    
Push data normally (json)
    
```
{
  "orderUser": {
    "i": "38519467",
    "s": 2,
    "q": "9.00000000",
    "p": "42",
    "z": "500.00000000",
    "x": 2,
    "T":1600657893,
  },
  "cmd": 100,
  "symbol": "btc_cnc"
}
```
  
Data push instructions:

Parameter name | Type | Description
----- | ---------| ---------
cmd | int| logo
symbol |string| trading pair
i |string| Streaming id
s |int| Order type 1 buy 2 sell
q |string| Order quantity
p |string| price
z |string| remaining quantity
x |int|Status 4-Order cancelled successfully 3-Order returned 2-Order successfully placed
T |int|time

#### Error response (error code)
Refer to the response description and code meaning
  
  
# cmd-101-Personal latest transaction
```
Description:
no

Subscription format:
{"cmd":101,"action":"sub","symbol":"btc_cnc"}

Unsubscribe format:
{"cmd":101,"action":"unsub","symbol":"btc_cnc"}
```

Format parameter description:

Parameter name | Description
----- | ---------
cmd | Business Identity
action| Action ID
symbol | trading pair

Normal response (json)
```
{"cmd":101,"action":"sub","code": 20000,"symbol":"btc_cnc"}
```

Push data normally (json)
  
```
{
  "tradeUser": {
    "i": "182",
    "T": 1594369897,
    "q": "11.05",
    "p": "11.72",
    "s": "buy",
    "t": 6,
    "n": "0.12",
    "N": "CNC",
    "m": 1,
    "e": 1
  },
  "cmd": 101,
  "symbol": "btc_cnc"
}
```

Data push instructions:

Parameter name | Type | Description
----- | ---------| ---------
cmd | int| logo
symbol |string| trading pair
i |string| Streaming ID
T |int| time
q |string| quantity
p |string| price
s |string| Transaction type
t |int| transaction id
n |string| Handling fee
N |string| Transaction fee currency
m |int| Whether self-deal
e |int| liquidity direction


 
 # Link orders and trades through tags
 + In the current Aex RESTful interface, there is no correlation between orders and transaction records, which makes it difficult to analyze the status of pending orders and transactions, and also increases the difficulty of implementing trading strategies.
 + The problem of lack of association between order and transaction record in RESTful api is solved by the tag in the pending order in the websocket api. The order and the result of the transaction formed by the same pending order will record the tag in the pending order, so the order and the transaction record can be Associated by tags.
 
 
 # Implementation of trading strategy
 Since the tag in the pending request in the websocket api is completely user-defined, as long as tag > 0, the user can encode the tag to implement a certain custom trading strategy.