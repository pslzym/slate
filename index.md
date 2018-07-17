---
title: Excore RPC API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - python



toc_footers:
  - <a href='https://gateio.io/'>Sign Up for Gate.IO</a>

search: true
---


# Introduction

Build at `version_tag_string`

Welcome to the Excore RPC API! You can use our API to access excore service.

We have language bindings in Shell, and Python! You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.

<aside class="notice">
In this document, we use <code>http://Website/</code> to represent the server url. You should replace it according to actual condition.
</aside>


# API Overview

The API is based on [JSON RPC](http://json-rpc.org/wiki/specification) of Websocket protocol. Repeated subscription will be cancelled for the same data type.


## Request

parameter | type | required | description
--------- | ---- |---------  | -----------
`method`| String |  yes  | method of request
`params`| Array |   yes  |   detail parameters
`id`| Integer |  yes  | request ID

**Authenticaion**

```python
# coding: utf-8 -*-

import hmac
import hashlib
import json
import requests

api_key = 'prearranged api key'
secret_key = 'prearranged secret key'

def get_sign(data, secret_key):
    sign = hmac.new(bytes(secret_key, encoding='utf-8'),
                    bytes(data, encoding='utf-8'),
                    hashlib.sha512).hexdigest()
    return sign


url = 'http://host:port/v3sign/'
payload = {"id": 1, "method":"market.last", "params":["BTC_USDT"]}
data = json.dumps(payload)
sign = get_sign(data, secret_key)

r = requests.post(url, headers={'KEY': api_key, 'SIGN': sign}, data=data)
print(r.text)
```
Authentication for each internal RPC interface is a must. The request data is signatured by sha512 hash algorithm.
There is a python demo at the right side.

## Response

field | type  | description
--------- | ---- | ----------
`result`| JSON Object | null for failure
`error`| JSON Object | null for success<br>non-null for failure: `code`: error code, `message`: error message
`id`| Integer | request ID


## Error
All errors follow general HTTP status code principles. Included in the body of any error response (e.g. non-200 status code) will be an error object of the form:
> JSON example:

```shell

  
  {
      "error": {
          "code": 23,
          "message": "order not found"
      },
      "result": null,
      "id": 1515752473250
  }
  
```


code      | classification | message  
--------- | ---- |--------------
`1`| General error|invalid argument
`2`|General error |internal error
`3`|General error |service unavailable
`4`| General error|method not found
`5`| General error|service timeout
`6`|General error| invalid user_id
`7`| General error| exist yet
`10`| Balance error|repeat update
`11`| Balance error|balance not enough
`12`|Balance error|asset not found
`13`|Balance error| invalid asset
`21`| Order error | amount too small
`22`| Order error | no enough trader
`23`| Order error | order not found
`24`| Order error | user not match
`25`|Order error |  market not found
`26`| Order error | invalid side
`27`| Order error | source too long
`27`| Order error | limit too big
`41`| Market error| asset not found
`42`| Market error| invalid precision
`43`| Market error| market not found


# Asset API

##Asset inquiry

```shell

curl -d \
'{"id": 1515752473250, "method": "balance.query", "params": [1,"coin2"]}'\
http://Website/


```

> The above command returns JSON structured like this:

```json
{
  "error": null,
  "result": {
    "BTC": {
      "available": "213833174.077851",
      "freeze": "30715.848"
    }
  },
  "id": 1515752473250
}
```
### method

`balance.query`

### Request

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`user_id`| Integer |  yes | user_id
`asset`| String | no | asset name, if null returns all of the asset information

### Response

field | type  | description
--------- | ---- | ----------
`available`| String | available
`freeze`| String | freeze



## Asset change


```shell

curl -d \
'{"id": 1515752473250, "method": "balance.update", "params": [1, "USTD", "deposit", 100, "1.2344", {}, "force"]}' \
http://Website/
 

```

> The above command returns JSON structured like this:

```json

{
  "error": null,
  "result": {
    "status": "success"
  },
  "id": 1515752473250
}

```

### method
`balance.update`


### Request

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`user_id`| Integer |  yes | user_id
`asset`| String | yes | asset name
`business`| String |  yes | business type
`business_id`| Integer | yes | must ensure the uniqueness of each operation of user_id, asset, business, business_id, or you will think that the update is repeated
`change`| String |  yes | balance change, Positive is plus, negative is minus
`detail`| JSON Object | yes | additional information
`force`| String | no | Optional, support setting negtaive balance, the value must be string `force`.

### Response



**Errors**

code  |  message
---------   | -----------
`10`|   repeat update  
`11`|   balance not enough
`12`|   asset not found

##Asset history

```shell
curl -d \
'{"id": 1515752473250, "method": "balance.history", "params": [1, "BTC", "", 0, 0, 0, 100]}' \
http://Website/

```

> The above command returns JSON structured like this:

```json
{
  "error": null,
  "result": {
    "offset": 0,
    "limit": 2,
    "records": [
      {
        "time": 1518074034.9090531,
        "asset": "BTC",
        "business": "trade",
        "change": "-0.00611473",
        "balance": "10106854.1032393852063269",
        "detail": {
          "a": "243.32395341",
          "id": 3065562,
          "m": "INK_BTC",
          "p": "0.00002513"
        }
      },
      {
        "time": 1518074034.9090531,
        "asset": "BTC",
        "business": "trade",
        "change": "-0.00001223",
        "balance": "10106854.1093541161555202",
        "detail": {
          "a": "243.32395341",
          "f": "0.002",
          "id": 4841326,
          "m": "INK_BTC",
          "p": "0.00002513"
        }
      }
    ]
  },
  "id": 1515752473250
}
```

### method
`balance.history`


### Request

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`user_id`| Integer |  yes | user_id
`asset`| String | no | asset name, if null returns all of the business information
`business`| String |  no | business type,if null returns all of the business information
`start_time`| Integer |  yes | start time, 0 for unlimited
`end_time`| Integer | yes | end time, 0 for unlimited
`offset`| Integer |  yes | offset position
`limit`| Integer |  yes | count limit


### Response

field | type  | description
--------- | ---- | ----------
`offset`| Integer |   offset
`limit`| Integer |   limit
`records`| JSON Object | records


**records**

field | type  | description
--------- | ---- | ----------
`time`| Integer |   time
`asset`| String |   asset
`business`| String | business
`change`| String |   change
`balance`| String |   balance
`detail`| JSON Object | detail


field  |  description
---------  | -----------
`a`|  amount
`p`| price
`id`|  order_id
`m`|  market
`f`|  fee_rate

## Asset list

```shell
curl -d \
'{"id": 1515752473250, "method": "asset.list", "params": [0,10]}'\
 http://Website/
```

> The above command returns JSON structured like this:

```json
{
    "result": {
        "limit": 2,
        "offset": 0,
        "records": [
            {
                "name": "EOS",
                "prec": 12
            },
            {
                "name": "BOT",
                "prec": 12
            }
        ]
    },
    "error": null,
    "id": 1515752473250
}
```

### method

`asset.list`


### Request

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`offset`| Integer |  yes | offset position
`limit`| Integer | yes | count limit


### Response

field | type  | description
--------- | ---- | ----------
`limit`| Integer |  limit
`offset`| Integer |  offset
`records`| JSON Object |  records


**records**

field | type  | description
--------- | ---- | ----------
`name`| String  | name
`prec`| Integer  | prec


## Asset summary


```shell
curl -d\
 '{"id": 1515752473250, "method": "asset.summary", "params": ["coin3"]}'\
 http://Website/

```

> The above command returns JSON structured like this:

```json
{
  "error": null,
  "result": [
    {
      "name": "BCH",
      "total_balance": "200000017554.982918264899",
      "available_count": 1000,
      "available_balance": "199999643377.321244264899",
      "freeze_count": 32,
      "freeze_balance": "374177.661674"
    },
    {
      "name": "BTC",
      "total_balance": "100384450213.044873",
      "available_count": 1000,
      "available_balance": "100384240238.467873",
      "freeze_count": 50,
      "freeze_balance": "209974.577"
    }
  ],
  "id": 1515752473250
}
```

### method
`asset.summary`


### Request

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`asset list`| List |  no | asset list, if null returns all of the asset information


### Response

field | type  | description
--------- | ---- | ----------
`name`| String |   name
`total_balance`| String |   total_balance
`available_count`| Integer | available_count
`available_balance`| String |   available_balance
`freeze_count`| Integer |   freeze_count
`freeze_balance`| String | freeze_balance


## Asset add

```shell
curl -d \
'{"id": 1515752473250, "method": "asset.add", "params": ["BOS",20,10,"coin2"]}' \
http://Website/
```

> The above command returns JSON structured like this:

```json
{
  "error": null,
  "result": {
    "status": "success"
  },
  "id": 1515752473250
}
```

### method

`asset.add`


### Request


parameter | type | required | description
--------- | ---- | ---------  |  -----------
`name`| String |  yes | name
`prec_save`| Integer |  yes | prec_save, precision saved, Integer
`prec_show`| Integer |  yes | prec_show, precision shown, Integer
`notes`| String |  yes | notes

### Response


## Asset rename

```shell
curl -d \
'{"id": 1515752473250, "method": "asset.rename", "params": ["BOS","BTC"]}' \
http://Website/
```

> The above command returns JSON structured like this:

```json
{
  "error": null,
  "result": {
    "status": "success"
  },
  "id": 1515752473250
}
```

### method
`asset.rename`


### Request


parameter | type | required | description
--------- | ---- | ---------  |  -----------
`origin_name`| String |  yes | origin_name
`new_name`| String |  yes | new_name


### Response

**Errors**

code  |  message
---------   | -----------
`7`|   exist yet 
`12`|   asset not found



# Trade API

## Place limit order

```shell
curl -d \
'{"id": 1515752473250, "method": "order.put_limit", "params": [1, "coin3", 1, "10", "8000", "0.002", "0.001", "api.v1"]}' \
http://Website/

```

> The above command returns JSON structured like this:

```json
{
  "error": null,
  "result": {
    "id": 2287706,
    "market": "BTCBCH",
    "source": "api.v1",
    "type": 1,
    "side": 1,
    "user": 1,
    "ctime": 1515760692.8521471,
    "mtime": 1515760692.8521471,
    "price": "8000",
    "amount": "10",
    "taker_fee": "0.002",
    "maker_fee": "0.001",
    "left": "10",
    "deal_stock": "0",
    "deal_money": "0",
    "deal_fee": "0",
    "ref_user": 100,
    "ref_rate": "0.003"
  },
  "id": 1515752473250
}
```

### method

`order.put_limit`



### Request

***Request1(normal mode)***

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`user`| Integer |  yes | user
`market`| String | yes | market name
`side`| Integer |  yes | 1: sell, 2: buy
`amount`| String |  yes | count
`price`| String | yes | price
`taker_fee`| String |  yes | taker fee
`maker_fee`| String |  yes | maker fee
`source`| String | yes | source, up to 30 bytes

***Request2(reference mode)***

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`user`| Integer |  yes | user
`market`| String | yes | market name
`side`| Integer |  yes | 1: sell, 2: buy
`amount`| String |  yes | count
`price`| String | yes | price
`taker_fee`| String |  yes | taker fee
`maker_fee`| String |  yes | maker fee
`ref_user`| Integer | yes | ref_user id
`ref_rate`| String | yes | e.g. "0.3" means 30%
`source`| String | yes | source, up to 30 bytes

### Response

field  | type |  description
--------- | ----  | -----------
`id`| Integer   | ID 
`market`| String   | market name
`source`| String   | source
`type`| Integer | type
`side`| Integer | 1: sell, 2: buy
`user`| Integer | user
`ctime`| Integer | ctime
`mtime`| Integer | mtime
`price`| String |   price
`amount`| String |   amount
`taker_fee`| String | taker fee
`maker_fee`| String |  maker fee
`left`| String |   left
`deal_stock`| String | deal_stock
`deal_money`| String | deal_money
`deal_fee`| String |  deal_fee
`ref_user`| Integer |  ref_user
`ref_rate`| String |  e.g. "0.3" means 30%



**Errors**

code  |  message
---------   | -----------
`11`|   balance not enough



## Place market order

```shell
curl -d \
'{"id": 1515752473250, "method": "order.put_market", "params": [1, "BTCBCH", 1, "10", "0.002", "api.v1"]}' \
http://Website/
```

> The above command returns JSON structured like this:

```json
{
  "error": null,
  "result": {
    "id": 2287709,
    "market": "BTCBCH",
    "source": "api.v1",
    "type": 2,
    "side": 1,
    "user": 1,
    "ctime": 1515761179.1297419,
    "mtime": 1515761179.1314609,
    "price": "0",
    "amount": "10",
    "taker_fee": "0.002",
    "maker_fee": "0",
    "left": "0",
    "deal_stock": "10",
    "deal_money": "11.7433988",
    "deal_fee": "0.0234867976",
    "ref_user": 100,
    "ref_rate": "0.003"
  },
  "id": 1515752473250
}
```


### method
`order.put_market`



### Request


parameter | type | required | description
--------- | ---- | ---------  |  -----------
`user_id`| Integer |  yes | user_id
`market`| String | yes | market name
`side`| Integer |  yes | 1: sell, 2: buy
`amount`| String |  yes | amount
`taker_fee_rate`| String |  yes | taker fee
`source`| String | yes | source, up to 30 bytes


### Response


field  | type |  description
--------- | ----  | -----------
`id`| Integer   | user ID 
`market`| String   | market name
`source`| String |  source, up to 30 bytes
`type`| Integer | type
`side`| Integer | 1: sell, 2: buy
`user`| Integer   | user 
`ctime`| Float   | ctime , e.g. 1515761179.1297419
`mtime`| Float   | mtime , e.g. 1515761179.1297419
`price`| String | price
`amount`| String |   amount
`taker_fee`| String | taker fee
`maker_fee`| String |  maker fee
`left`| String |  left
`deal_stock`| String |  deal_stock
`deal_money`| String |  deal_money
`deal_fee`| String |  deal_fee
`ref_user`| Integer |  ref_user
`ref_rate`| String |  ref_rate


**Errors**

code  |  message
---------   | -----------
`11`|   balance not enough






## Cancel order


```shell
curl -d \
'{"id": 1515752473250, "method": "order.cancel", "params": [1, "BTCBCH", 2287710]}' \
http://Website/

```

> The above command returns JSON structured like this:

```json
{
  "error": null,
  "result": {
    "status": "success"
  },
  "id": 1515752473250
}
```


### method
`order.cancel`



### Request


parameter | type | required | description
--------- | ---- | ---------  |  -----------
`user_id`| Integer |  yes | user_id
`market`| String | yes | market name
`order_id`| Integer |  no |  order ID, optional parameters, all orders will be cancelled without this parameter



### Response

**Errors**

code  |  message
---------   | -----------
`23`|  order not found
`24`|   user not match





## Order details

```shell
curl -d \
'{"id": 1515752473250, "method": "order.deals", "params": [342915, 0, 2]}' \
http://Website/

```

> The above command returns JSON structured like this:

```json
{
  "error": null,
  "result": {
    "offset": 0,
    "limit": 2,
    "records": [
      {
        "id": 6012385,
        "time": 1523465898,
        "user": 10,
        "role": 1,
        "amount": "1.9887",
        "price": "2.445",
        "deal": "0",
        "fee": "0.002",
        "deal_order_id": 342915
      }
    ]
  },
  "id": 1515752473250
}
```


### method
`order.deals`


### Request

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`order_id`| Integer |  yes | order id
`offset`| Integer | yes | offset position
`limit`| Integer |  yes | count limit

### Response

field  | type |  description
--------- | ----  | -----------
`offset`| Integer   | offset 
`limit`| Integer   | limit
`records`| JSON Object   | records 

**records**

field  | type |  description
--------- | ----  | -----------
`id`| Integer   | executed ID 
`time`| Integer   | time 
`user`| Integer   | user id
`role`| Integer   | role, 1：maker, 2: taker
`price`| String | price
`amount`| String | amount
`deal`| String |  deal
`fee`| String | fee
`deal_order_id`| Integer | counterpart transaction ID

## Order list

```shell
curl -d \
'{"id": 1515752473250, "method": "order.book", "params": ["BTCBCH", 1, 0, 2]}'\
 http://Website/
```

> The above command returns JSON structured like this:


```json
{
  "error": null,
  "result": {
    "offset": 0,
    "limit": 2,
    "total": 138577,
    "orders": [
      {
        "id": 999202,
        "market": "BTCBCH",
        "type": 1,
        "side": 1,
        "user": 9,
        "ctime": 1515684147.9900739,
        "mtime": 1515684147.9905679,
        "price": "1.5158",
        "amount": "1.158",
        "taker_fee": "0.002",
        "maker_fee": "0.001",
        "left": "1.151",
        "deal_stock": "0.007",
        "deal_money": "0.0106106",
        "deal_fee": "0.0000106106",
        "ref_user": 100,
        "ref_rate": "0.0003"
      }
    ]
  },
  "id": 1515752473250
}
```


### method
`order.book`


### Request


parameter | type | required | description
--------- | ---- | ---------  |  -----------
`market`| String |  yes | market name
`side`| Integer | yes | side, 1: sell, 2: buy
`offset`| Integer |  yes | offset position
`limit`| Integer |  yes |  count limit

### Response

field  | type |  description
--------- | ----  | -----------
`offset`| Integer  | offset position
`limit`| Integer |  count limit
`total`| Integer   | total 
`orders`| JSON Object   | orders

**order**

field  | type |  description
--------- | ----  | -----------
`id`| Integer   | ID 
`market`| String   | market name
`type`| Integer | type
`side`| Integer | 1: sell, 2: buy
`user`| Integer   | user id
`ctime`| Float   | ctime , e.g. 1515761179.1297419
`mtime`| Float   | mtime , e.g. 1515761179.1297419
`price`| String | price
`amount`| String |   count
`taker_fee`| String | taker fee
`maker_fee`| String |  maker fee
`left`| String |  left
`deal_stock`| String |  deal_stock
`deal_money`| String |  deal_money
`deal_fee`| String |  deal_fee
`ref_user`| Integer |  ref_user
`ref_rate`| String |  ref_rate


## Order depth

```shell
curl -d \
'{"id": 1515752473250, "method": "order.depth", "params": ["BTCBCH", 2, "0.01"]}'\
 http://Website/
```


> The above command returns JSON structured like this:

```json
{
    "error": null,
    "result": {
        "asks": [
            [
                "1.52",  price
                "1.151"  count
            ],
            [
                "1.53",
                "1.218"
            ]
        ],
        "bids": [
            [
                "1.17",   price
                "201.863" count
            ],
            [
                "1.16",
                "725.464"
            ]
        ]
    },
    "id": 1515752473250
}
```


### method
`order.depth`




### Request

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`market`| String |  yes | market name
`limit`| Integer | yes | count limit
`interval`| String |  yes | interval

### Response


field  | type |  description
--------- | ----  | -----------
`asks`| Json List   | asks 
`bids`| Json List   | bids

**asks**

field  | type |  description
--------- | ----  | -----------
`price`| String   | price 
`amount`| String   | amount

**bids**

field  | type |  description
--------- | ----  | -----------
`price`| String   | price 
`amount`| String   | amount


## Market list of uncompleted orders 


```shell

curl -d \
'{"id": 1515752473250, "method": "order.pending_list", "params": [1,  0, 10]}'\
 http://Website/
 
```

> The above command returns JSON structured like this:

```json
{
  "error": null,
  "result": {
    "limit": 10,
    "offset": 2,
    "markets": [
      {
        "name": "TNT_ETH",
        "total": 15245         
      },
      {
        "name": "XRP_BTC",
        "total": 38465
      }
    ]
  },
  "id": 1515752473250
}

```


### method
`order.pending_list`

 
### Request

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`user_id`| Integer |  yes | user_id
`offset`| Integer | yes | offset
`limit`| Integer |  yes | limit


### Response

field | type  | description
--------- | ---- | ----------
`limit`| Integer  | limit
`offset`| Integer  | offset
`markets`| JSON Object  | markets

**markets**

field | type  | description
--------- | ---- | ----------
`name`| String  | name
`total`| Integer  | total


## Inquire unexecuted orders

```shell
curl -d \
'{"id": 1515752473250, "method": "order.pending", "params": [1, "BTC_BCH", 0, 1]}'\
 http://Website/
```

> The above command returns JSON structured like this:

```json
{
  "error": null,
  "result": {
    "limit": 1,
    "offset": 0,
    "total": 20286,
    "records": [
      {
        "id": 2287706,
        "market": "BTCBCH",
        "type": 1,
        "side": 1,
        "user": 1,
        "ctime": 1515760692.8521471,
        "mtime": 1515760692.8521471,
        "price": "8000",
        "amount": "10",
        "taker_fee": "0.002",
        "maker_fee": "0.001",
        "left": "10",
        "deal_stock": "0",
        "deal_money": "06",
        "deal_fee": "0",
        "ref_user": 100,
        "ref_rate": "0.003"
      }
    ]
  },
  "id": 1515752473250
}
```


### method
`order.pending`


### Request

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`user_id`| Integer |  yes | user_id
`market`| String | yes | market name
`offset`| Integer |  yes | offset
`limit`| Integer |  yes | limit, maximum 100

### Response

field  | type |  description
--------- | ----  | -----------
`offset`| Integer   | offset 
`limit`| Integer   | limit
`total`| Integer   | total
`records`| JSON Object   | record object list

**record**

field  | type |  description
--------- | ----  | -----------
`id`| Integer   | ID 
`market`| String   | market name
`type`| Integer | type
`side`| Integer | 1: sell, 2: buy
`user`| Integer   | user id
`ctime`| Float   | ctime , e.g. 1515761179.1297419
`mtime`| Float   | mtime , e.g. 1515761179.1297419
`price`| String | price
`amount`| String |   count
`taker_fee`| String | taker fee
`maker_fee`| String |  maker fee
`left`| String |  left
`deal_stock`| String |  deal_stock
`deal_money`| String |  deal_money
`deal_fee`| String |  deal_fee
`ref_user`| Integer |  ref_user
`ref_rate`| String |  ref_rate


## Unexecuted order details

```shell
curl -d \
'{"id": 1515752473250, "method": "order.pending_detail", "params": ["BTCBCH", 997201]}'\
 http://Website/
```

> The above command returns JSON structured like this:

```json
{
    "error": null,
    "result": {
        "id": 997201,
        "market": "BTCBCH",
        "type": 1,
        "side": 1,
        "user": 1,
        "ctime": 1515684147.9555719,
        "mtime": 1515684147.9555719,
        "price": "1.749",
        "amount": "1.49",
        "taker_fee": "0.002",
        "maker_fee": "0.001",
        "left": "1.49",
        "deal_stock": "0",
        "deal_money": "0",
        "deal_fee": "0"
        "ref_user": 100,
        "ref_rate": "0.003"
    },
    "id": 1515752473250
}
```


### method
`order.pending_detail`




### Request

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`market`| String |  yes | market name
`order_id`| Integer | yes | order ID



### Response

field  | type |  description
--------- | ----  | -----------
`id`| Integer   | ID 
`market`| String   | market name
`type`| Integer | type
`side`| Integer | 1: sell, 2: buy
`user`| Integer   | user id
`ctime`| Float   | ctime , e.g. 1515761179.1297419
`mtime`| Float   | mtime , e.g. 1515761179.1297419
`price`| String | price
`amount`| String |   amount
`taker_fee`| String | taker fee
`maker_fee`| String |  maker fee
`left`| String |  left
`deal_stock`| String |  deal_stock
`deal_money`| String |  deal_money
`deal_fee`| String |  deal_fee
`ref_user`| Integer |  ref_user
`ref_rate`| String |  ref_rate




## Inquire executed orders

```shell
curl -d \
'{"id": 1515752473250, "method": "order.finished", "params": [1, "BTCBCH", 0, 0, 0, 1, 0]}'\ 
http://Website/
```

> The above command returns JSON structured like this:

```json
{
  "error": null,
  "result": {
    "offset": 0,
    "limit": 1,
    "records": [
      {
        "id": 2287711,
        "ctime": 1515761485.7350211,
        "ftime": 1515761485.7361529,
        "user": 1,
        "market": "BTCBCH",
        "source": "api.v1",
        "type": 2,
        "side": 1,
        "price": "0",
        "amount": "10",
        "taker_fee": "0.002",
        "maker_fee": "0",
        "deal_stock": "10",
        "deal_money": "11.743",
        "deal_fee": "0.023486",
        "ref_user": 100,
        "ref_rate": "0.003"
      }
    ]
  },
  "id": 1515752473250
}
```

### method
`order.finished`



### Request

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`user_id`| Integer |  yes | user_id
`market`| String | yes | market name
`start_time`| Integer |  yes | start time, 0 for unlimited
`end_time`| Integer | yes | end time, 0 for unlimited
`offset`| Integer |  yes | offset 
`limit`| Integer |  yes |  limit
`side`| Integer |  yes |   side, 0 for no limit, 1 for sell, 2 for buy


### Response

field  | type |  description
--------- | ----  | -----------
`offset`| Integer   | offset 
`limit`| Integer   | limit
`records`| JSON Object   | records 

**records** 

field  | type |  description
--------- | ----  | -----------
`id`| Integer   | ID 
`market`| String   | market name
`type`| Integer | type
`side`| Integer | 1: sell, 2: buy
`user`| Integer   | user id
`ctime`| Float   | ctime , e.g. 1515761179.1297419
`ftime`| Float   | ftime , e.g. 1515761179.1297419
`price`| String | price
`amount`| String |  amount
`taker_fee`| String | taker fee
`maker_fee`| String |  maker fee
`source`| String |  source
`deal_stock`| String |  deal_stock
`deal_money`| String |  deal_money
`deal_fee`| String |  deal_fee
`ref_user`| Integer |  ref_user
`ref_rate`| String |  ref_rate




## Executed order details

```shell
curl -d \
'{"id": 1515752473250, "method": "order.finished_detail", "params": [999094]}' \
http://Website/

```

> The above command returns JSON structured like this:

```json
{
  "error": null,
  "result": {
    "id": 999094,
    "ctime": 1515683644.665751,
    "ftime": 1515683644.6661899,
    "user": 1,
    "market": "BTCBCH",
    "source": "hello",
    "type": 1,
    "side": 1,
    "price": "1.362",
    "amount": "1.62",
    "taker_fee": "0.002",
    "maker_fee": "0.001",
    "deal_stock": "1.62",
    "deal_money": "2.2066317",
    "deal_fee": "0.0022435974",
    "ref_user": 100,
    "ref_rate": "0.003"
  },
  "id": 1515752473250
}
```

### method
`order.finished_detail`


### Request

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`order_id`| Integer |  yes | order_id

### Response


field  | type |  description
--------- | ----  | -----------
`id`| Integer   | ID 
`market`| String   | market name
`type`| Integer | type
`side`| Integer | 1: sell, 2: buy
`user`| Integer   | user id
`ctime`| Float   | ctime , e.g. 1515761179.1297419
`ftime`| Float   | ftime , e.g. 1515761179.1297419
`price`| String | price
`amount`| String |  amount
`taker_fee`| String | taker fee
`maker_fee`| String |  maker fee
`deal_stock`| String |  deal_stock
`deal_money`| String |  deal_money
`deal_fee`| String |  deal_fee
`source`| String |  source
`ref_user`| Integer |  ref_user
`ref_rate`| String |  ref_rate


# Market API

## Market price

```shell
curl -d \
'{"id": 1515752473250, "method": "market.last", "params": ["coin1_coin2"]}'\
 http://Website/
```

> The above command returns JSON structured like this:

```json
{
  "error": null,
  "result": {
    "price": "1.51580000",
    "change": "-12.34"
  },
  "id": 1515752473250
}
```


### method

`market.last`




### Request

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`market`| String |  yes | market



### Response

field | type  | description
--------- | ---- | ----------
`price`| String |   price
`change`| String |   change






## Executed history

```shell
curl -d \
'{"id": 1515752473250, "method": "market.deals", "params": ["coin1_coin2", 2, 0]}'\ 
http://Website/
```

> The above command returns JSON structured like this:

```json
{
  "error": null,
  "result": [
    {
      "id": 702002,
      "time": 1515683644.721833,
      "price": "1.5158",
      "amount": "0.007",
      "type": "buy"
    },
    {
      "id": 702001,
      "time": 1515683644.7216251,
      "price": "1.5014",
      "amount": "1.014",
      "type": "buy"
    }
  ],
  "id": 1515752473250
}
```


### method

`market.deals`


### Request

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`market`| String |  yes | market name
`limit`| Integer | yes | count, no more than 1000
`last_id`| Integer |  yes | the last trade id



### Response

field  | type |  description
--------- | ----  | -----------
`id`| Integer   | id
`time`| Integer   | time 
`price`| String | price
`amount`| String |   count
`type`| String |  type


## Trade history

```shell
curl -d \
'{"id": 1515752473250, "method": "market.deal_history", "params": ["coin1_coin2", 2, 1191086]}'\ 
http://Website/
```

> The above command returns JSON structured like this:

```json
{
    "error": null,
    "result": {
        "records": [
            {
                "time": 1521710930.0876901,
                "id": 1191087,
                "side": 2,
                "price": "1.349",
                "amount": "0.924",
                "market": "coin1_coin2"
            },
            {
                "time": 1521710930.088294,
                "id": 1191088,
                "side": 2,
                "price": "1.349",
                "amount": "0.108",
                "market": "coin1_coin2"
            }
        ]
    },
    "id": 1515752473250
}
```


### method

`market.deal_history`


### Request

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`market`| String |  yes | market name
`limit`| Integer | yes | count, no more than 1000
`last_id`| Integer |  no | the last trade id, e.g. if `last_id`=1000, `limit`=100, then return trades with id from 1001 to 1100



### Response

field  | type |  description
--------- | ----  | -----------
`time`| Float   | time 
`id`| Integer   | id
`side`| Integer | 1: sell, 2: buy
`price`| String | price
`amount`| String |  amount
`market`| String |  market name




## User Executed history

```shell
curl -d \
'{"id": 1515752473250, "method": "market.user_deals", "params": [1, "coin1_coin2", 0, 2]}'\ 
http://Website/
```

> The above command returns JSON structured like this:

```json
{
  "error": null,
  "result": {
    "offset": 0,
    "limit": 2,
    "records": [
      {
        "time": 1518098569.2785561,
        "user": 1,
        "id": 2386562,
        "market": "DPY_USDT",
        "side": 2,
        "role": 1,
        "price": "1.411",
        "amount": "107.48675095",
        "deal": "151.66380559045",
        "fee": "0.2149735019",
        "deal_order_id": 5470216
      }
    ]
  },
  "id": 1515752473250
}
```


### method

`market.user_deals`



### Request

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`user_id`| Integer |  yes | user_id
`market`| String |  yes | market name,if null returns all of the market
`offset`| Integer | yes | offset
`limit`| Integer | yes | limit


### Response

field  | type |  description
--------- | ----  | -----------
`offset`| Integer   | offset 
`limit`| Integer   | limit
`records`| JSON Object   | records 


**records**

field  | type |  description
--------- | ----  | -----------
`time`| Integer   | time 
`user`| Integer   | user id
`id`| Integer   | ID 
`market`| String   | market name
`side`| Integer | 1: sell, 2: buy
`role`| Integer   | role
`price`| String | price
`amount`| String | amount
`deal`| String | deal
`fee`| String | fee
`deal_order_id`| Integer |  deal_order_id


## KLine

```shell
curl -d \
'{"id": 1515752473250, "method": "market.kline", "params": ["coin1_coin2", 1515719030, 1515759030, 600]}'\
 http://Website/
```

> The above command returns JSON structured like this:

```json
{
  "error": null,
  "result": [
    [
      1515718800,    time
      "1.5158",      open
      "1.5158",      close
      "1.5158",      highest
      "1.5158",      lowest
      "0",           volume
      "0",           amount
      "BTCBCH"       market name
    ],
    [
      1515719400,
      "1.5158",
      "1.5158",
      "1.5158",
      "1.5158",
      "0",
      "0",
      "BTCBCH"
    ],
    ...
  ],
  "id": 1515752473250
}
```



### method

`market.kline`


### Request

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`market`| String |  yes | market name,if null returns all of the market
`start`| Integer | yes | start
`end`| Integer | yes | end
`interval`| Integer | yes | interval

### Response


field | type  | description
--------- | ---- | ----------
`time`| Integer |   time
`open`| String |   open
`close`| String | close
`highest`| String |   highest
`lowest`| String | lowest
`volume`| String |   volume
`amount`| String | amount
`market`| String | market name





## Market status

```shell
curl -d \
'{"id": 1515752473250, "method": "market.status", "params": ["coin1_coin2", 86400]}'\
 http://Website/
```

> The above command returns JSON structured like this:

```json
{
  "error": null,
  "result": {
    "period": 3600,
    "last": "1.5158",
    "open": "0",
    "close": "0",
    "high": "0",
    "low": "0",
    "volume": "0",
    "deal": "0"
  },
  "id": 1515752473250
}
```
### method

`market.status`

### Request

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`market`| String |  yes | market name
`period`| Integer | yes | cycle period, e.g. 86400 for last 24 hours



### Response

field | type  | description
--------- | ---- | ----------
`period`| Integer |   period
`last`| String |   last
`open`| String |   open
`close`| String | close
`high`| String |   high
`low`| String | low
`volume`| String |   volume
`deal`| String | deal




## Market status today

```shell
curl -d \
'{"id": 1515752473250, "method": "market.status_today", "params": ["coin1_coin2"]}'\
 http://Website/
```

> The above command returns JSON structured like this:

```json
{
  "error": null,
  "result": {
    "open": "1.5158",
    "last": "1.5158",
    "high": "1.5158",
    "low": "1.5158",
    "volume": "137644.352",
    "deal": "192825.661284256"
  },
  "id": 1515752473250
}
```
### method

`market.status_today`


### Request

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`market`| String |  yes | market name


### Response

field | type  | description
--------- | ---- | ----------
`open`| String |   open
`last`| String |   last
`high`| String |   high
`low`| String | low
`volume`| String |   volume
`deal`| String | deal



## Market list

```shell
curl -d \
'{"id": 1515752473250, "method": "market.list", "params": [0,20]}' \
http://Website/
```

> The above command returns JSON structured like this:

```json
{
    "result": {
        "limit": 2,
        "offset": 0,
        "records": [
            {
                "name": "COIN1_COIN2",
                "stock": "COIN1",
                "money_prec": 8,
                "stock_prec": 8,
                "min_amount": "0.001",
                "money": "COIN2"
            }
        ]
    },
    "error": null,
    "id": 1515752473250
}
```


### method
`market.list`



### Request

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`offset`| Integer |  yes | offset position
`limit`| Integer |  yes | count limit

### Response


field  | type |  description
--------- | ----  | -----------
`offset`| Integer   | offset 
`limit`| Integer   | limit
`records`| JSON Object   | records 

**records**

field  | type |  description
--------- | ----  | -----------
`offset`| Integer   | offset 
`limit`| Integer   | limit
`records`| JSON Object   | records 

**records**

field | type  | description
--------- | ---- | ----------
`name`| String |   name
`stock`| String |   stock
`money_prec`| Integer |   money_prec
`stock_prec`| Integer | stock_prec
`min_amount`| String |   min_amount
`money`| String | money




## Market summary

```shell
curl -d \
'{"id": 1515752473250, "method": "market.summary", "params": ["coin1_coin2"]}'\ 
http://Website/
```

> The above command returns JSON structured like this:

```json
{
  "error": null,
  "result": [
    {
      "name": "BTCBCH",
      "ask_count": 138575,
      "bid_count": 31146
    }
  ],
  "id": 1515752473250
}
```




### method

`market.summary`



### Request

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`market list`| String |  yes | market list




### Response


field | type  | description
--------- | ---- | ----------
`name`| String |   name
`ask_count`| Integer |   ask count
`bid_count`| Integer |   bid count


## Market add

```shell
curl -d \
'{"id": 1515752473250, "method": "market.add", "params": ["BTC_coin1", "BTC", 8, "coin1", 8, "0.001", "BTC_coin1"]}' \
http://Website/
```

> The above command returns JSON structured like this:

```json
{
  "error": null,
  "result": {
    "status": "success"
  },
  "id": 1515752473250
}
```



### method
`market.add`



### Request
parameter | type | required | description
--------- | ---- | ---------  |  -----------
`market_name`| String |  yes | market_name
`stock_name`| String | yes | stock_name 
`stock_prec`| Integer |  yes | stock_prec
`money_name`| String |  yes | money_name
`money_prec`| Integer | yes | money_prec 
`min_amount`| String |  yes | min_amount
`notes`| String |  yes | notes


### Response


**Errors**

code  |  message
---------   | -----------
`41`|   asset not found 
`42`|   invalid precision






## Market rename

```shell
curl -d \
'{"id": 1515752473250, "method": "market.rename", "params": ["BTC_coin1","BTC_coin1 new"]}' \
http://Website/
```

> The above command returns JSON structured like this:

```json
{
  "error": null,
  "result": {
    "status": "success"
  },
  "id": 1515752473250
}
```


### method
`market.rename`



### Request

parameter | type | required | description
--------- | ---- | ---------  |  -----------
`origin_name`| String |  yes | origin_name
`new_name`| String | yes | new_name



### Response


**Errors**

code  |  message
---------   | -----------
`7`|    exist yet
`43`|   market not found
