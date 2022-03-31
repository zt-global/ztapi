---
title: API Reference

search: true

code_clipboard: true

meta:

- name: description content: Documentation for the ZT API

---

# Introduction

This is the developer documentation, ZT provides a simple and easy-to-use API interface, through which you can obtain
market data, conduct transactions, and manage orders.

## Create API Key

After registering an account with **[ZT](https://www.ztb.im)**, the user needs to create a pair of API keys
in [API management]. After the creation, a set of randomly generated `Access key (hereinafter referred to as api key) `
and `Secret key` will be obtained. With this set of data, programmed transactions can be carried out. A single account
can create up to 5 keys.
<aside class="notice">
Please do not disclose the information of api key and secret key, so as to avoid asset loss. It is recommended that the user bind the IP address for the API. Each key is bound to a maximum of 5 IPS, separated by English commas.
</aside>

## Interface Type REST API

It provides functions of market query, balance query, currency transaction and order management. It is recommended that
users use REST API to query account balance, currency transaction and order management.

## The Server

ZT server runs in Tokyo. In order to minimize the API access delay, it is recommended to use a server with smooth
communication with Tokyo.

# Guide

## Access URL

Please make API requests with the following base urls:

* [https://www.ztb.im](https://www.ztb.im) (for overseas users)
* [https://www.ztbzh.net](https://www.ztbzh.net) (for mainland China users)

## Request Format

The APIs accept GET, or POST request, as will be described in later sections

* For a GET request, all parameters are path paremeters (query string)
* For a POST request, all parameters(includes `api_key` and `sign`) are to be put in request body and set `content-type`
  to `application/x-www-form-urlencoded`.
* All http requests' header must include field `X-SITE-ID`, and value is `1`.

<aside class="notice">
All http requests' header must include field `X-SITE-ID`, and value is `1`
</aside>

## Signature

Public APIs (for market APIs) and private APIs (for account and trade APIs) are provided.

Signature is not required for public APIs.

And for private APIs, signature is required and will be used for checking the validity of each request.

## Signature Parameters

Parameters related to request signature

| **Components**      | **Description**                                              |
| ------------------- | ------------------------------------------------------------ |
| param `api_key`     | access key from API key                                      |
| param `sign`        | calculated signature                                         |
| business parameters | business parameters required by each private API, only the business parameters of `POST` requests will be included in the signature calulation |

## Signature Method

> Go Example:

```go
func Sign(secretKey string, params url.Values) string {
keys := make([]string, 0, len(params))
for field := range params {
keys = append(keys, field)
}
sort.Strings(keys)
for index, field := range keys {
keys[index] = field + "=" + params.Get(field)
}
src := strings.Join(keys, "&") + "&secret_key=" + secretKey
sum := md5.Sum([]byte(src))
return strings.ToUpper(hex.EncodeToString(sum[:]))
}
```

1. Signature is not required for public APIs.

2. For private APIs, the signature rules are as follows:

  1) All business parameters need be formatted like `foo=bar`, and spliced in dictionary order with `&` interval, and
     finally the string like `foo=bar&foo1=bar1&foo2=bar2`.

  2) Append the secret key to the end of the string obtained in the previous step. Assuming your secret key
     is `bf58b336-0cf3-4d54-82a4-6577251813d8`, the spliced string should be
     like `foo=bar&foo1=bar1&foo2=bar2&secret_key=bf58b336-0cf3-4d54-82a4-6577251813d8`. If request without business
     parameters, the string should be like `&secret_key=bf58b336-0cf3-4d54-82a4-6577251813d8`.

  3) The string obtained in the previous step needs be hashed with `md5`, and output in `uppercase hexadecimal`. Finally
     the string like `D183FA41A05BBF692B319D6D08C80646`.

## Errors Code

| **Error Code** | **Introduce**                                | **Reason**               |
| :------------- | :------------------------------------------- | :----------------------- |
| 0              | Success                                      |                          |
| 1              | parameter is invalid                         |                          |
| 2              | Internal error                               |                          |
| 3              | service is not available                     |                          |
| 4              | Method not found                             |                          |
| 5              | Service timeout                              |                          |
| 10             | Insufficient amount                          |                          |
| 11             | The number of transactions is too small      |                          |
| 12             | Insufficient depth                           |                          |
| 10005          | Record not found                             | X-SITE-ID，Setting error |
| 10022          | User does not have a real name               |                          |
| 10051          | User prohibited transaction                  |                          |
| 10056          | Less than minimum amoun                      |                          |
| 10059          | No transaction has been opened for the asset |                          |
| 10060          | The transaction has not been opened          |                          |
| 10062          | Incorrect amount accuracy                    |                          |

# Market APIs

## Ticker Information

> Response:

```json
{
  "ticker": [
    {
      "buy": "28.3437",
      "change": "2.95",
      "high": "29.8265",
      "last": "28.8402",
      "low": "27.9001",
      "sell": "29.3233",
      "symbol": "ZEC_USDT",
      "vol": "18564.4392"
    },
    ...
  ],
  "timestamp": "1577680809298"
}
```

* **GET** `/api/v1/tickers`

<aside class="notice">
Frequency limit：20 times/s
</aside>

### Response Parameters

| **Field** | **Data type** | **Description**                  |
| -------------- | ------------- | -------------------------------- |
| buy            | string        | current bid price                |
| change         | string        | price change rate of this period |
| high           | string        | the highest price of this period |
| last           | string        | the latest deal price            |
| low            | string        | the lowest price of this period  |
| sell           | string        | current ask price                |
| symbol         | string        | symbol name                      |
| vol            | string        | deal total volume of this period |

## Market Depth

> Response:

```json
{
  "asks": [
    [
      "48095.8344",
      "0.02899"
    ]
  ],
  "bids": [
    [
      "48070.1725",
      "0.07424"
    ]
  ]
}
```

* **GET** `/api/v1/depth`

<aside class="notice">
Frequency limit：20 times/s
</aside>

### Request Parameters

| **Parameters** | **Description**            |
| ---------- | ------------------------------ |
| symbol     | symbol name                    |
| size       | number of depth to be returned |

### Response Parameters

| **Field** | **Data type** | **Description**              |
| -------------- | ------------- | ---------------------------- |
| asks           | array         | asks data, [price, quantity] |
| bids           | array         | bids data, [price, quantity] |

## Latest Deals

> Response:

```json
[
  {
    "amount": "0.01049",
    "price": "48162.6633",
    "side": "sell",
    "timestamp": "1631780839333"
  },
  {
    "amount": "0.01425",
    "price": "48162.9032",
    "side": "buy",
    "timestamp": "1631780839333"
  }
]
```

* **GET** `/api/v1/trades`

<aside class="notice">
Frequency limit：20 times/s
</aside>

### Request Parameters

| **Parameters** | **Description**            |
| ---------- | ------------------------------ |
| symbol     | symbol name                    |
| size       | number of returned records |

### Response Parameters

| **Field** | **Data type** | **Description**              |
| ---- | ---- | ---- |
| amount | string | deal volume |
| price | string | deal price |
| side | string | sell or buy |
| timestamp | string | deal time |

## K-Lines

> Response:

```json
[
  [
    1631635200000,
    "46740.3536",
    "48312.8932",
    "46171.1234",
    "48261.0853",
    "10138.38514"
  ],
  [
    1631721600000,
    "48262.55",
    "48507.1536",
    "47149.7977",
    "48187.1089",
    "7145.77305024"
  ]
]
```

* **GET** `/api/v1/kline`

<aside class="notice">
Frequency limit：20 times/s
</aside>

### Request Parameters

| **Parameters** | **Description**            |
| ---------- | ------------------------------ |
| symbol     | symbol name                    |
| size       | number of returned records |
| type       | time sharing parameter, which can be 1min, 5min, 15min, 30min, hour, day, week |

### Response Parameters

| **Field** | **Data type** | **Description**              |
| ---- | ---- | ---- |
| index 0 | number | time |
| index 1 | string | opening price |
| index 2 | string | highest price |
| index 3 | string | lowest price |
| index 4 | string | closing price |
| index 5 | string | volume |

## Symbol Information

> Response:

```json
[
  {
    "baseAsset": "XRP",
    "baseAssetPrecision": 2,
    "min_quantity": "0.01",
    "quoteAsset": "USDT",
    "quoteAssetPrecision": 6,
    "status": "trading",
    "symbol": "XRP_USDT",
    "tick_size": "0.000001"
  }
]
```

* **GET** `/api/v1/exchangeInfo`

<aside class="notice">
Frequency limit：20 times/s
</aside>

### Response Parameters

| **Field** | **Data type** | **Description**              |
| ---- | ---- | ---- |
| baseAsset | string | base asset |
| baseAssetPrecision | number | base asset's precision |
| quoteAsset | string | quote asset |
| quoteAssetPrecision | number | quote asset's precision |
| status | string | symbol's status |
| symbol | string | symbol name |
| min_quantity | string | minimum volume |
| tick_size | string | minimum price increase |

# Account APIs

All private APIs' request use the POST method, and parameters are submitted in the form of data (api_key and sign must
be passed).

## Balance

> Response:

```json
{
  "code": 0,
  "message": "success",
  "result": {
    "USDT": {
      "available": "10718.74453852",
      "freeze": "0.10999996",
      "other_freeze": "0",
      "recharge_status": 0,
      "trade_status": 1,
      "withdraw_fee": "0.1",
      "withdraw_max": "1000",
      "withdraw_min": "0.001",
      "withdraw_status": 1
    },
    "BTC": {
      "available": "46.20370336",
      "freeze": "0",
      "other_freeze": "0",
      "recharge_status": 0,
      "trade_status": 1,
      "withdraw_fee": "0.1",
      "withdraw_max": "100000",
      "withdraw_min": "1",
      "withdraw_status": 1
    }
  }
}
```

* **POST** `/api/v1/private/user`

<aside class="notice">
Frequency limit：10 times/s
</aside>

### Response Parameters

| **Field** | **Data type** | **Description**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| < result > | map | response data |
| < asset > | object | asset balance information |
| available | string | available balance |
| freeze | string | trading freeze balance |
| other_freeze | string | other freeze balance |
| recharge_status | number | asset recharge status, 1 means enabled |
| trade_status | number | asset trading status, 1 means enabled |
| withdraw_fee | string | asset withdrawal fee |
| withdraw_max | string | asset withdrawal max volume |
| withdraw_min | string | asset withdrawal min volume |
| withdraw_status | number | asset withdrawal status, 1 means enabled |

## Deposit History

> Response:

```json
{
  "code": 0,
  "message": "success",
  "result": [
    {
      "id": 123243243,
      "created_at": "2021-12-02 15:04:05",
      "tx_id": "78dbafb0079536564f1652aebebaeddcd2a3c628315a2817146f778708ba2c05",
      "asset": "USDT",
      "amount": "201",
      "status": 1,
      "address": "TLiSQNfa3RtP8m25YAmd1Dofr5ecNrgsfz",
      "chain_name": "TRC20",
      "confirmation": "1",
      "min_confirmation": "1"
    }
  ]
}
```

* **POST** `/api/v1/private/user/rechargeMoneyLog`

<aside class="notice">
Frequency limit：10 times/s
</aside>

### Request Parameters

| **Parameters** | **Type** | **Mandatory** | **Description**    | **Value**      |
| -------------- | -------- | ------------- | ------------------ | -------------- |
| asset         | string   | N             | deposit coin | （eg: USDT） |
| status       | number  | N             | deposit status, 1 means success, 6 means confirming, 7 means fail  | 1,6,7 |
| limit       | number  | Y             | data size | no more than 100 |
| offset       | number  | N          | data offset |  |

### Response Parameters

| **Field** | **Data type** | **Description**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| < result > | array | response data |
| id | number | deposit record id |
| created_at | string | deposit record created time |
| tx_id | string | hash |
| asset | string | coin |
| amount | string | coin's quantity |
| status | number | 1 success, 6 processing, 7 fail |
| address | string | deposit address |
| chain_name | string |  |
| confirmation | string | current confirmation quantity |
| min_confirmation | string | minimum confirmation quantity required for deposit |

## Withdrawal History

> Response:

```json
{
  "code": 0,
  "message": "success",
  "result": [
    {
      "id": 123243243,
      "created_at": "2021-12-02 15:04:05",
      "tx_id": "78dbafb0079536564f1652aebebaeddcd2a3c628315a2817146f778708ba2c05",
      "asset": "USDT",
      "amount": "201",
      "fee": "0.1",
      "fee_asset": "USDT",
      "status": 1,
      "address": "TLiSQNfa3RtP8m25YAmd1Dofr5ecNrgsfz",
      "chain_name": "TRC20",
      "confirmation": "1"
    }
  ]
}
```

* **POST** `/api/v1/private/user/withdrawMoneyLog`

<aside class="notice">
Frequency limit：10 times/s
</aside>

### Request Parameters

| **Parameters** | **Type** | **Mandatory** | **Description**    | **Value**      |
| -------------- | -------- | ------------- | ------------------ | -------------- |
| asset         | string   | N             | withdrawal coin | （eg: USDT） |
| status       | number  | N             | withdrawal status, 1 means success, 2 means canceled, 3 means submitted, 4 means first verify, 5 means second verify, 6 means confirming, 7 means fail  | 1,2,3,4,5,6,7 |
| limit       | number  | Y             | data size | no more than 100 |
| offset       | number  | N          | data offset |  |

### Response Parameters

| **Field** | **Data type** | **Description**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| < result > | array | response data |
| id | number | deposit record id |
| created_at | string | withdrawal record created time |
| tx_id | string | hash |
| asset | string | coin |
| amount | string | coin's quantity |
| fee | string | withdrawal fee |
| fee_asset | string | withdrawal fee's coin, maybe different from asset |
| status | number |  |
| address | string | withdrawal address |
| chain_name | string |  |
| confirmation | string | current confirmation quantity |

## All Coins' Information

> Response:

```json
{
  "code": 0,
  "message": "success",
  "result": {
      "BTC": {
		  "asset_code": "BTC",
		  "asset_name": "Bitcoin",
		  "trade_status": 1, 
		  "withdraw_deduct_status": 1,
		  "withdraw_deduct_asset": "ZTB",
		  "withdraw_deduct_rate": "0.8",
		  "withdraw_deduct_fixed_amount": "10",
		  "chain": [{
			  "coin_id": "9794ac2a-7725-47c0-8f10-a7118df9b2f4",
			  "chain_name": "Bitcoin",
			  "regex": "^[13][a-km-zA-HJ-NP-Z1-9]{25,34}$",
			  "recharge_status": 1,
			  "withdraw_status": 1,
			  "withdraw_min": "100",
			  "withdraw_max": "100000",
			  "withdraw_fee": "1",
			  "withdraw_fee_percent": "0",
			  "recharge_min": "10",
			  "recharge_min_confirmation": 1
		  }]
	  }
  }
}
```

* **POST** `/api/v1/private/asset`

<aside class="notice">
Frequency limit：10 times/s
</aside>

### Response Parameters

| **Field** | **Data type** | **Description**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| < result > | object | business data |
| < coin > | object | coin's information |
| asset_code | string | coin's code，the asset parameters involved in APIs such as withdrawals are all this field |
| asset_name | string | coin's full name |
| trade_status | number | trading status，1 means tradable |
| withdraw_deduct_status | number | can other coins be used to deduct the withdrawal fee, 1 means can |
| withdraw_deduct_asset | string | coin that can be used for withdrawal fee deduction, such as ZTB |
| withdraw_deduct_rate | string | discounts for withdrawal fee deductions |
| withdraw_deduct_fixed_amount | string | A fixed number of coin that can be used for withdrawal fee deduction. When this field is > 0, no matter how much amount for withdrawal, only this fixed value of withdrawal fee needs to be paid for deduction. |
| < chain > | array | network |
| chain_name | string | network name |
| coin_id | string | id of the network |
| regex | string | regex for address |
| withdraw_status | number | withdrawal status，1 means can withdraw |
| withdraw_min | string | minimum quantity for a single withdrawal |
| withdraw_max | string | maximum quantity for a single withdrawal |
| withdraw_fee | string | withdrawal base fee |
| withdraw_fee_percent | string | extra fee for withdrawal |
| recharge_status | number | recharge status，1 means can recharge |
| recharge_min | string | minimum quantity for a single recharge |
| recharge_min_confirmation | number | the minimum number of block confirmations for recharge to account |

## Withdraw

> Response:

```json
{
  "code": 0,
  "message": "success",
  "result": {
	  "id": 10086
  }
}
```

* **POST** `/api/v1/private/user/withdraws`

<aside class="notice">
Frequency limit：10 times/s
</aside>

### Request Parameters

| **Parameters** | **Type** | **Mandatory** | **Description**    | **Value**      |
| -------------- | -------- | ------------- | ------------------ | -------------- |
| asset         | string   | Y             | coin | （eg: USDT） |
| coin_id         | string   | N             | id of the network, recommended to be required | （eg: d16c9104-e058-4004-8b91-3999c37fbceb） |
| amount       | string  | Y             |  |  |
| address       | string  | Y             |  |  |
| memo       | string  | N          | tag, secondary address identifier for coins like XRP,XMR etc. |  |
| deduction       | boolean  | N          | whether the withdrawal fee is deducted by ZTB | (eg: true or false) |

### Response Parameters

| **Field** | **Data type** | **Description**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| < result > | array | business data |
| id | number | id of the withdrawal record |

## Withdrawal Cancellation

> Response:

```json
{
  "code": 0,
  "message": "success",
  "result": {
	  "id": 10086,
	  "asset": "USDT",
	  "amount": "100",
	  "coin_id": "d16c9104-e058-4004-8b91-3999c37fbceb",
	  "status": "CANCEL"
  }
}
```

* **POST** `/api/v1/private/user/withdraws/cancel`

<aside class="notice">
Frequency limit：10 times/s
</aside>

### Request Parameters

| **Parameters** | **Type** | **Mandatory** | **Description**    | **Value**      |
| -------------- | -------- | ------------- | ------------------ | -------------- |
| id         | number   | Y             | id of the withdrawal record |  |

### Response Parameters

| **Field** | **Data type** | **Description**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| < result > | array | business data |
| id | number | id of the withdrawal record |
| asset | string | coin |
| amount | string |  |
| coin_id | string | id of the network |
| status | string |  |

# Trading APIs

## Limit Order

> Response:

```json
{
  "code": 0,
  "message": "success",
  "result": {
    "amount": "1",
    "ctime": 1535537926.246487,
    "deal_fee": "0",
    "deal_money": "0",
    "deal_stock": "0",
    "id": 32865,
    "left": "1",
    "maker_fee": "0.001",
    "market": "BTC_USDT",
    "mtime": 1535537926.246487,
    "price": "10",
    "side": 2,
    "source": "web,1",
    "taker_fee": "0.001",
    "type": 1,
    "user": 670865
  }
}
```

* **POST** `/api/v1/private/trade/limit`

<aside class="notice">
Frequency limit：500 times/s
</aside>

### Request Parameters

| **Parameters** | **Type** | **Mandatory** | **Description**    | **Value**      |
| -------------- | -------- | ------------- | ------------------ | -------------- |
| market         | string   | Y             | The trading symbol | （eg: BTC_USDT） |
| side           | number  | Y             | The trading side   | 1(ask), 2(bid) |
| amount         | string   | Y             |                    |                |
| price          | string   | Y             |                    |                |

### Response Parameters

| **Field** | **Data type** | **Description**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| < result > | map | response data |
| id | number |  |
| ctime | number |  |
| mtime | number |  |
| market | string |  |
| amount | string |  |
| price | string |  |
| side | number | ask or bid |
| type | number | 1 is limit order, 2 is market order |
| user | number | user id |
| maker_fee | string | the trading fee of maker |
| taker_fee | string | the trading fee of taker |
| deal_stock | string | the base asset deal volume |
| deal_money | string | the quote asset deal volume |
| deal_fee | string | the deal fee |
| left | string | the untraded amount |

## Market Order

> Response:

```json
{
  "code": 0,
  "message": "success",
  "result": {
    "amount": "1",
    "ctime": 1535537926.246487,
    "deal_fee": "0",
    "deal_money": "0",
    "deal_stock": "0",
    "id": 32865,
    "left": "1",
    "maker_fee": "0.001",
    "market": "BTC_USDT",
    "mtime": 1535537926.246487,
    "price": "10",
    "side": 2,
    "source": "web,1",
    "taker_fee": "0.001",
    "type": 1,
    "user": 670865
  }
}
```

* **POST** `/api/v1/private/trade/market`

<aside class="notice">
Frequency limit：500 times/s
</aside>

### Request Parameters

| **Parameters** | **Type** | **Mandatory** | **Description**    | **Value**      |
| -------------- | -------- | ------------- | ------------------ | -------------- |
| market         | string   | Y             | The trading symbol | （eg: BTC_USDT） |
| side           | number  | Y             | The trading side   | 1(ask), 2(bid) |
| amount         | string   | Y             |                    |                |

### Response Parameters

| **Field** | **Data type** | **Description**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| < result > | map | response data |
| id | number |  |
| ctime | number |  |
| mtime | number |  |
| market | string |  |
| amount | string |  |
| price | string |  |
| side | number | ask or bid |
| type | number | 1 is limit order, 2 is market order |
| user | number | user id |
| maker_fee | string | the trading fee of maker |
| taker_fee | string | the trading fee of taker |
| deal_stock | string | the base asset deal volume |
| deal_money | string | the quote asset deal volume |
| deal_fee | string | the deal fee |
| left | string | the untraded amount |

## Cancel Order

> Response:

```json
{
  "code": 0,
  "message": "success",
  "result": {
    "amount": "1",
    "ctime": 1535537926.246487,
    "deal_fee": "0",
    "deal_money": "0",
    "deal_stock": "0",
    "id": 32865,
    "left": "1",
    "maker_fee": "0.001",
    "market": "BTC_USDT",
    "mtime": 1535537926.246487,
    "price": "10",
    "side": 2,
    "source": "web,1",
    "taker_fee": "0.001",
    "type": 1,
    "user": 670865
  }
}
```

* **POST** `/api/v1/private/trade/cancel`

<aside class="notice">
Frequency limit：20 times/s
</aside>

### Request Parameters

| **Parameters** | **Type** | **Mandatory** | **Description**    | **Value**      |
| -------------- | -------- | ------------- | ------------------ | -------------- |
| market         | string   | Y             | The trading symbol | （eg: BTC_USDT） |
| order_id       | number  | Y             | The Order id   |  |

### Response Parameters

| **Field** | **Data type** | **Description**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| < result > | map | response data |
| id | number |  |
| ctime | number |  |
| mtime | number |  |
| market | string |  |
| amount | string |  |
| price | string |  |
| side | number | ask or bid |
| type | number | 1 is limit order, 2 is market order |
| user | number | user id |
| maker_fee | string | the trading fee of maker |
| taker_fee | string | the trading fee of taker |
| deal_stock | string | the base asset deal volume |
| deal_money | string | the quote asset deal volume |
| deal_fee | string | the deal fee |
| left | string | the untraded amount |

## Cancel Orders In Batch

> Response:

```json
{
  "code": 0,
  "message": "success",
  "result": [
    {
      "market": "BTC_USDT",
      "order_id": 456647,
      "result": true
    },
    {
      "market": "ETH_USDT",
      "order_id": 456648,
      "result": true
    }
  ]
}
```

* **POST** `/api/v1/private/trade/cancel_batch`

<aside class="notice">
Frequency limit：10 times/s
</aside>

### Request Parameters

| **Parameters** | **Type** | **Mandatory** | **Description**    | **Value**      |
| -------------- | -------- | ------------- | ------------------ | -------------- |
| orders_json         | string   | Y             | Orders json string | eg: [{"market":"BTC_USDT", "order_id":456647},{"market":"ETH_USDT", "order_id":456648}] |

### Response Parameters

| **Field** | **Data type** | **Description**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| < result > | map | response data |
| market | string |  |
| order_id | number |  |
| result | boolean | true means cancel successfully |

## Pending Orders

> Response:

```json
{
  "code": 0,
  "message": "success",
  "result": {
    "limit": 10,
    "offset": 0,
    "records": [
      {
        "amount": "1",
        "ctime": 1535544362.168106,
        "deal_fee": "0",
        "deal_money": "0",
        "deal_stock": "0",
        "id": 32871,
        "left": "1",
        "maker_fee": "0.001",
        "market": "BTC_USDT",
        "mtime": 1535544362.168106,
        "price": "5.1",
        "side": 2,
        "source": "web,1",
        "status": 1,
        "taker_fee": "0.001",
        "type": 1,
        "user": 670865
      }
    ],
    "total": 1
  }
}
```

* **POST** `/api/v1/private/order/pending`

<aside class="notice">
Frequency limit：20 times/s
</aside>

### Request Parameters

| **Parameters** | **Type** | **Mandatory** | **Description**    | **Value**      |
| -------------- | -------- | ------------- | ------------------ | -------------- |
| market         | string   | Y             | The trading symbol | （eg: BTC_USDT） |
| offset       | number  | N             | Offset   |  |
| limit       | number  | Y             | Limit   | no more than 100 |

### Response Parameters

| **Field** | **Data type** | **Description**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| < result > | map | response data |
| limit | number |  |
| offset | number |  |
| total | number |  |
| < records > | array |  |
| id | number |  |
| ctime | number |  |
| mtime | number |  |
| market | string |  |
| amount | string |  |
| price | string |  |
| side | number | ask or bid |
| type | number | 1 is limit order, 2 is market order |
| user | number | user id |
| maker_fee | string | the trading fee of maker |
| taker_fee | string | the trading fee of taker |
| deal_stock | string | the base asset deal volume |
| deal_money | string | the quote asset deal volume |
| deal_fee | string | the deal fee |
| left | string | the untraded amount |

## Pending Order Detail

> Response:

```json
{
  "code": 0,
  "message": "success",
  "result": {
    "amount": "1",
    "ctime": 1535537926.246487,
    "deal_fee": "0",
    "deal_money": "0",
    "deal_stock": "0",
    "id": 32865,
    "left": "1",
    "maker_fee": "0.001",
    "market": "BTC_USDT",
    "mtime": 1535537926.246487,
    "price": "10",
    "side": 2,
    "source": "web,1",
    "taker_fee": "0.001",
    "type": 1,
    "user": 670865
  }
}
```

* **POST** `/api/v1/private/order/pending/detail`

<aside class="notice">
Frequency limit：20 times/s
</aside>

### Request Parameters

| **Parameters** | **Type** | **Mandatory** | **Description**    | **Value**      |
| -------------- | -------- | ------------- | ------------------ | -------------- |
| market         | string   | Y             | The trading symbol | （eg: BTC_USDT） |
| order_id       | number  | Y             | The Order id   |  |

### Response Parameters

| **Field** | **Data type** | **Description**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| < result > | map | response data |
| id | number |  |
| ctime | number |  |
| mtime | number |  |
| market | string |  |
| amount | string |  |
| price | string |  |
| side | number | ask or bid |
| type | number | 1 is limit order, 2 is market order |
| user | number | user id |
| maker_fee | string | the trading fee of maker |
| taker_fee | string | the trading fee of taker |
| deal_stock | string | the base asset deal volume |
| deal_money | string | the quote asset deal volume |
| deal_fee | string | the deal fee |
| left | string | the untraded amount |

## Finished Orders

> Response:

```json
{
  "code": 0,
  "message": "success",
  "result": {
    "limit": 2,
    "offset": 0,
    "records": [
      {
        "amount": "1",
        "ctime": 1535538409.189721,
        "deal_fee": "0.00019607843",
        "deal_money": "0.999999993",
        "deal_stock": "0.19607843",
        "ftime": 1535538409.189735,
        "id": 32868,
        "maker_fee": "0",
        "market": "BTC_USDT",
        "price": "0",
        "side": 2,
        "source": "web,1",
        "taker_fee": "0.001",
        "type": 2,
        "user": 670865
      },
      {
        "amount": "10",
        "ctime": 1535538403.233823,
        "deal_fee": "0.001109999955",
        "deal_money": "1.109999955",
        "deal_stock": "0.21764705",
        "ftime": 1535538409.189735,
        "id": 32867,
        "maker_fee": "0.001",
        "market": "BTC_USDT",
        "price": "5.1",
        "side": 1,
        "source": "web,1",
        "taker_fee": "0.001",
        "type": 1,
        "user": 670865
      }
    ]
  }
}
```

* **POST** `/api/v1/private/order/finished`

<aside class="notice">
Frequency limit：10 times/s
</aside>

### Request Parameters

| **Parameters** | **Type** | **Mandatory** | **Description**    | **Value**      |
| -------------- | -------- | ------------- | ------------------ | -------------- |
| market         | string   | Y             | The trading symbol | （eg: BTC_USDT） |
| start_time       | number  | N             | time stamp in seconds, unlimited to 0   |  |
| end_time       | number  | N             | time stamp in seconds, unlimited to 0   |  |
| offset       | number  | N             | Offset   |  |
| limit       | number  | Y             | Limit   | no more than 100 |
| side       | number  | N             | 1 is ask, 2 is bid, unlimited to 0   |  |

### Response Parameters

| **Field** | **Data type** | **Description**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| < result > | map | response data |
| limit | number |  |
| offset | number |  |
| < records > | array |  |
| id | number |  |
| ctime | number |  |
| ftime | number |  |
| market | string |  |
| price | string |  |
| side | number | ask or bid |
| type | number | 1 is limit order, 2 is market order |
| user | number | user id |
| maker_fee | string | the trading fee of maker |
| taker_fee | string | the trading fee of taker |
| deal_stock | string | the base asset deal volume |
| deal_money | string | the quote asset deal volume |
| deal_fee | string | the deal fee |

## Finished Order Detail

> Response:

```json
{
  "code": 0,
  "message": "success",
  "result": {
    "amount": "10",
    "ctime": 1565681925.295415,
    "deal_fee": "0",
    "deal_money": "19.5",
    "deal_stock": "10",
    "ftime": 1565681925.295421,
    "id": 1081,
    "maker_fee": "0",
    "market": "BTC_USDT",
    "price": "2",
    "side": 2,
    "source": "web,127",
    "taker_fee": "0",
    "type": 1,
    "user": 2
  }
}
```

* **POST** `/api/v1/private/order/finished/detail`

<aside class="notice">
Frequency limit：10 times/s
</aside>

### Request Parameters

| **Parameters** | **Type** | **Mandatory** | **Description**    | **Value**      |
| -------------- | -------- | ------------- | ------------------ | -------------- |
| order_id         | number   | Y             | The Order ID |  |

### Response Parameters

| **Field** | **Data type** | **Description**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| < result > | map | response data |
| id | number |  |
| ctime | number |  |
| ftime | number |  |
| market | string |  |
| amount | string |  |
| price | string |  |
| side | number | ask or bid |
| type | number | 1 is limit order, 2 is market order |
| user | number | user id |
| maker_fee | string | the trading fee of maker |
| taker_fee | string | the trading fee of taker |
| deal_stock | string | the base asset deal volume |
| deal_money | string | the quote asset deal volume |
| deal_fee | string | the deal fee |

# Websocket APIs

## Introduce

### Access URL

* wss://www.ztb.im/ws (for overseas users)
* wss://www.ztbzh.net/ws (for mainland China users)

### Request format

* method: string
* params: array
* id:     integer, customed random number

### Response format

* result: json object，If not returned; otherwise null
* error:  json object，Successfully returned null, If unsuccessful, return No null

  * code:    error code
  * message: error message

* id:     integer

### Notice format

* method: string
* params: array
* id:     null

### General error code

* 1: illegal parameter
* 2: internal error
* 3: service not available
* 4: method not found
* 5: service timeout
* 6: authorization required

### Subscribe topics

> Example:

```json
{
  "method": "kline.subscribe",
  "params": [
    "BTC_USDT",
    300
  ],
  "id": 10086
}
```

After successfully establishing a connection with the websocket server, the websocket client sends the following request
to subscribe to a specific topic:

`{"method":"method to sub","params":[request params],"id":id generated by client}`

### Unsubscribe

> Example:

```json
{
  "method": "kline.unsubscribe"
}
```

After the websocket subscribes to a specific topic, if you need to unsubscribe, the websocket client sends the following
request to unsubscribe:

`{"method":"method to unsubscribe"}`

### Query

> Example:

```json
{
  "method": "kline.qurey",
  "params": [
    "BTC_USDT",
    1575561600,
    1575648000,
    300
  ],
  "id": 10086
}
```

The websocket server also supports pull.

Request format is as follows：

`{"method":"method to sub","params":[request params],"id":id generated by client}`

## Market Status

This topic sends the latest market status of the market.

### Subscribe

> Request:

```json
{
  "method": "state.subscribe",
  "params": [
    "BTC_USDT"
  ],
  "id": 10086
}
```

> Response:

```json
{
  "method": "state.update",
  "params": [
    "BTC_USDT",
    {
      "last": "7461.1526",
      "volume": "15864.0388",
      "deal": "119049215.80982165",
      "period": 86400,
      "high": "7553.5791",
      "open": "7421.5379",
      "low": "7414.7222",
      "close": "7461.1526"
    }
  ],
  "id": null
}
```

`
{"method":"state.subscribe","params":[$market$],"id":10086}
`

| Parameter | Type   | **Required or not** | Description | **Value**    |
| --------- | ------ | ------------------- | ----------- | ------------ |
| market    | string | true                |             | eg: BTC_USDT |

### Unsubscribe

`
{"method":"state.unsubscribe"}
`

### Query

> Request:

```json
{
  "method": "state.query",
  "params": [
    "BTC_USDT",
    86400
  ],
  "id": 10086
}
```

> Response:

```json
{
  "error": null,
  "result": {
    "volume": "15952.03100501",
    "period": 86400,
    "deal": "119721963.749190401504",
    "last": "7467.2656",
    "open": "7431.0264",
    "low": "7428.725",
    "close": "7467.2656",
    "high": "7553.5791"
  },
  "id": 10086
}
```

Obtain the market status data of the past specific time in one time by request

`
{"method":"state.query","params":[$market$,$period$],"id":10086}
`

| Parameter | Type   | Required or not | Description | **Value**   |
| --------- | ------ | --------------- | ----------- | ----------- |
| market    | string | true            |             | eg:BTC_USDT |
| period    | int    | true            |             | Eg: 86400   |

## Market Status Today

This topic is sent to market today.

### Subscribe

> Request:

```json
{
  "method": "today.subscribe",
  "params": [
    "BTC_USDT"
  ],
  "id": 10086
}
```

> Response:

```json
{
  "method": "today.update",
  "params": [
    "BTC_USDT",
    {
      "last": "7461.1526",
      "volume": "15864.0388",
      "deal": "119049215.80982165",
      "period": 86400,
      "high": "7553.5791",
      "open": "7421.5379",
      "low": "7414.7222",
      "close": "7461.1526"
    }
  ],
  "id": null
}
```

`
{"method":"today.subscribe","params":[$market$],"id":10086}
`

| Parameter | Type   | **Required or not** | Description | **Value**    |
| --------- | ------ | ------------------- | ----------- | ------------ |
| market    | string | true                |             | eg: BTC_USDT |

### Unsubscribe

`
{"method":"today.unsubscribe"}
`

### Query

> Request:

```json
{
  "method": "today.query",
  "params": [
    "BTC_USDT",
    86400
  ],
  "id": 10086
}
```

> Response:

```json
{
  "error": null,
  "result": {
    "open": "7525.3908",
    "deal": "119600164.325722971504",
    "last": "7466.2622",
    "high": "7541.1691",
    "low": "7444.7897",
    "volume": "15935.75120501"
  },
  "id": 10086
}
```

Obtain today's market status data at one time by request.

`
{"method":"today.query","params":[$market$,$period$],"id":10086}
`

| Parameter | Type   | Required or not | Description | **Value**   |
| --------- | ------ | --------------- | ----------- | ----------- |
| market    | string | true            |             | eg:BTC_USDT |

## K-Line Data

This topic sends the latest K-line data.

### Subscribe

> Request:

```json
{
  "method": "kline.subscribe",
  "params": [
    "BTC_USDT",
    market
    300
    interval
  ],
  "id": 10086
  id
}
```

> Response:

```json
{
  "id": null,
  "method": "kline.update",
  "params": [
    [
      1575705900,
      time
      "7542.8082",
      open
      "7534.9152",
      close
      "7547.0765",
      high
      "7530.8753",
      low
      "70.7463",
      amount
      "533428.87370982",
      deal_money
      "BTC_USDT"
      market
    ]
  ]
}
```

`
{"method":"kline.subscribe","params":[$market$,$interval$],"id":10086}
`

| Parameter | Type   | **Required or not** | Description | **Value**                        |
| --------- | ------ | ------------------- | ----------- | -------------------------------- |
| market    | string | true                |             | BTC_USDT, ETH_USDT...            |
| interval  | string | true                |             | 60,300,900,1800,3600,7200,14400… |

### Unsubscribe

`
{"method":"kline.unsubscribe"}
`

### Query

> Request:

```json
{
  "method": "kline.query",
  "params": [
    "BTC_USDT",
    // market
    1575561600,
    // start
    1575648000,
    // end
    300
    // interval
  ],
  "id": 10086
  id
}
```

> Response:

```json
{
  "error": null,
  "result": [
    [
      1575561600,
      // time
      "7340.949",
      // open
      "7345.5655",
      // close
      "7357.0065",
      // high
      "7332.0522",
      // low
      "77.4528",
      // amount
      "568870.6347857",
      // deal money
      "BTC_USDT"
      // market
    ],
    [
      1575561900,
      "7346.2494",
      "7333.1595",
      "7350.2274",
      "7329.8995",
      "72.2238",
      "530009.12444915",
      "BTC_USDT"
    ]
    ...
    }
```

Obtain K-line data in one time by request, the following additional parameters

`
{"method":"kline.query","params":[$market$,$start$,$end$,$interval$],"id":10086}
`

| Parameter | Type   | Required or not | Description | **Value**   |
| --------- | ------ | --------------- | ----------- | ----------- |
| start     | integer | false           |             | eg: 1575561600 |
| end       | integer | false           |             | eg: 1575648000 |

## Market Depth

This topic sends the latest in-depth market data.

### Subscribe

> Request:

```json
{
  "method": "depth.subscribe",
  "params": [
    "BTC_USDT",
    50,
    "0.0001"
  ],
  "id": 10086
}
```

> Response:

```json
{
  "id": null,
  "method": "depth.update",
  "params": [
    true,
    // true represents the full depth list，false represents update
    {
      "bids": [
        [
          "7457.1469"
          price
          "0.0026"
          amount
        ],
        [
          "7457.137",
          "0.0028"
        ],
        ...
      ],
      "asks": [
        [
          "7550.6256",
          "0.2271"
        ],
        [
          "7550.9482",
          "0.0022"
        ],
        ...
      ]
    },
    "BTC_USDT"
  ]
}
```

`
{"method":"depth.subscribe","params":[$market$,$limit$,$interval$],"id":10086}
`

| Parameter | Type   | **Required or not** | Description | **Value**                        |
| --------- | ------ | ------------------- | ----------- | -------------------------------- |
| market    | string | true            |             | eg: BTC_USDT                                                 |
| limite    | int    | true            |             | 1, 5, 10, 20, 30, 50, 100                                    |
| interval  | string | true            | merge depth | "0", "0.00000001", "0.0000001", "0.000001", "0.00001", "0.0001", "0.001", "0.01", "0.1" |

### Unsubscribe

`
{"method":"depth.unsubscribe"}
`

### Query

> Request:

```json
{
  "method": "depth.query",
  "params": [
    "BTC_USDT",
    10,
    "0.0001"
  ],
  "id": 10086
}
```

> Response:

```json
{
  "id": 10086,
  "error": null,
  "result": {
    "asks": [
      [
        "7562.2075",
        "0.0228"
      ],
      [
        "7577.5392",
        "0.001"
      ],
      ...
    ],
    "bids": [
      [
        "7477.9723",
        "0.2047"
      ],
      [
        "7477.9225",
        "0.3294"
      ],
      ...
    ]
  }
}
```

Obtain depth in one time by request

`
{"method":"depth.query","params":[$market$,$limit$,$interval$],"id":10086}
`

| Parameter | Type   | Required or not | Description | **Value**   |
| --------- | ------ | --------------- | ----------- | ------------------------------------------------------------ |
| market    | string | true            |             | eg: BTC_USDT                                                 |
| limite    | int    | true            |             | 1, 5, 10, 20, 30, 50, 100                                    |
| interval  | string | true            | merge depth | "0", "0.00000001", "0.0000001", "0.000001", "0.00001", "0.0001", "0.001", "0.01", "0.1" |

## Latest Price

This topic sends the latest price.

### Subscribe

> Request:

```json
{
  "method": "price.subscribe",
  "params": [
    "BTC_USDT"
  ],
  "id": 10086
}
```

> Response:

```json
{
  "method": "price.update",
  "params": [
    "BTC_USDT",
    "7514.2520"
  ],
  "id": null
}
```

`
{"method":"price.subscribe","params":[$market$],"id":10086}
`

| Parameter | Type   | **Required or not** | Description | **Value**                        |
| --------- | ------ | ------------------- | ----------- | -------------------------------- |
| market    | string | true            |             | eg: BTC_USDT |

### Unsubscribe

`
{"method":"price.unsubscribe"}
`

### Query

> Request:

```json
{
  "method": "price.query",
  "params": [
    "BTC_USDT"
  ],
  "id": 10086
}
```

> Response:

```json
{
  "error": null,
  "result": "7482.0109",
  "id": 10086
}
```

Get the latest price in one time by request

`
{"method":"price.query","params":[$market$],"id":10086}
`

| Parameter | Type   | Required or not | Description | **Value**   |
| --------- | ------ | --------------- | ----------- | ------------------------------------------------------------ |
| market    | string | true            |             | eg: BTC_USDT |

## Latest Trade History

This topic sends the latest finished order.

### Subscribe

> Request:

```json
{
  "method": "deals.subscribe",
  "params": [
    "BTC_USDT"
  ],
  "id": 10086
}
```

> Response:

```json
{
  "method": "deals.update",
  "params": [
    "BTC_USDT",
    [
      {
        "id": 597933730,
        "time": 1575876545.1941223,
        "type": "sell",
        "price": "7477.6154",
        "amount": "0.1416"
      }
    ]
  ],
  "id": null
}
```

`
{"method":"deals.subscribe","params":[$market$],"id":10086}
`

| Parameter | Type   | **Required or not** | Description | **Value**                        |
| --------- | ------ | ------------------- | ----------- | -------------------------------- |
| market    | string | true            |             | eg: BTC_USDT |

### Unsubscribe

`
{"method":"deals.unsubscribe"}
`

### Query

> Request:

```json
{
  "method": "deals.query",
  "params": [
    "BTC_USDT",
    10,
    598129296
  ],
  "id": 10086
}
```

> Response:

```json
{
  "error": null,
  "result": [
    {
      "id": 598136190,
      "type": "sell",
      "time": 1575881302.0342646,
      "price": "7459.6875",
      "amount": "0.1781"
    },
    {
      "id": 598136185,
      "type": "sell",
      "time": 1575881301.876456,
      "price": "7463.8087",
      "amount": "0.2333"
    },
    ...s
  ],
  "id": 10086
}
```

Obtain Latest finished order in one time by request.

`
{"method":"deals.query","params":[$market$,$limit$,$last_id$],"id":10086}
`

| Parameter | Type   | Required or not | Description | **Value**   |
| --------- | ------ | --------------- | ----------- | ------------------------------------------------------------ |
| market    | string | true            |                                    | eg: BTC_USDT,             |
| limit     | int    | true            |                                    | 1, 5, 10, 20, 30, 50, 100 |
| last_id   | int    | true            | Maximum ID of last returned result | eg: 597967944             |
