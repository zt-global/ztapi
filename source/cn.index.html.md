---
title: API Reference

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the ZT API
---

# 介绍

这是ZT交易所的开发者文档，提供了一系列简单易用的API接口，您可以通过它们获取行情数据、进行交易、管理订单等。

## 创建 API Key

用户在**[ZT](https://www.ztbzh.net)**注册账号后，需要在【API管理】中创建一对API密钥。创建后会得到一组随机生成的`Access key（以下简称api key）`和`Secret key`。有了这组数据，就可以进行程序化的交易。一个ZT帐号最多可以创建5对密钥。
<aside class="notice">
请不要泄露api key和secret key的信息，以免造成资产损失。建议用户绑定IP地址，每个key最多绑定5个IP，用英文逗号隔开。
</aside>

## 接口类型 REST API

提供行情查询、余额查询、货币交易、订单管理等功能。建议用户使用REST API查询账户余额、币种交易和订单管理。

## 服务器

ZT服务器在东京运行，为了尽量减少API访问延迟，建议使用与东京通信顺畅的服务器。

# 接入说明

## 接入 URLs

请使用以下域名发出API请求:

* [https://www.ztb.im](https://www.ztb.im)（海外域名）
* [https://www.ztbzh.net](https://www.ztbzh.net)（大陆域名）

## 请求格式
API接受GET或POST请求

* 对于 GET 请求，所有参数都是query参数。
* 对于 POST 请求，所有参数（包括 `api_key` 和 `sign`）都将放在请求body中，并将 `content-type` 设置为 `application/x-www-form-urlencoded`。
* 所有 http 请求的标头必须包含字段`X-SITE-ID`，值为`1`。

<aside class="notice">
所有 http 请求的标头必须包含字段`X-SITE-ID`，值为`1`。
</aside>

## 签名

HTTP 接口提供公共 API（用于市场）和私有 API（用于账户和交易）。

公共 API 不需要签名。

而对于私有 API，则需要签名，并将用于检查每个请求的有效性。

## 签名参数

请求签名相关参数

| **组成**      | **说明**                                              |
| ------------------- | ------------------------------------------------------------ |
| 参数 `api_key`     | API创建后显示的access key                                      |
| 参数 `sign`        | 签名                                         |
| 业务参数 | 每个私有API需要的业务参数，只有`POST`请求的业务参数会被包含在签名计算中 |

## 签名方式

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

1. 公共 API 不需要签名。

2. 对于私有 API，签名规则如下:

	1) 所有业务参数都需要格式化为`foo=bar`，并以`&`间隔按字典顺序拼接，最后拼接成`foo=bar&foo1=bar1&foo2=bar2`这样的字符串。

	2) 将secret key追加到上一步中获得的字符串的末尾。假设你的秘钥是`bf58b336-0cf3-4d54-82a4-6577251813d8`，那么拼接后的字符串应该是`foo=bar&foo1=bar1&foo2=bar2&secret_key=bf58b336-0cf3-4d52-8513d8`。如果请求没有业务参数，则字符串应为`&secret_key=bf58b336-0cf3-4d54-82a4-6577251813d8`。

	3) 上一步得到的字符串需要用`md5`散列，并以`大写十六进制`输出，最后得到像`D183FA41A05BBF692B319D6D08C80646`这样的字符串。

## 错误码

| **错误码** | **说明**                                | **Reason**               |
| :------------- | :------------------------------------------- | :----------------------- |
| 0              | 操作成功                                      |                          |
| 1              | 参数不合法                        |                          |
| 2              | 数据加载中                              |                          |
| 3              | 服务不可用                   |                          |
| 4              | 方法未找到                             |                          |
| 5              | 服务超时                              |                          |
| 10             | 金额不足                          |                          |
| 11             | 交易数量太小      |                          |
| 12             | 无可成交的委托单                           |                          |
| 10005          | 记录未找到                             | X-SITE-ID，Setting error |
| 10022          | 用户未实名认证               |                          |
| 10051          | 用户禁止交易                  |                          |
| 10056          | 价格不得低于最小价格                      |                          |
| 10059          | 该资产暂未开启交易 |                          |
| 10060          | 该交易对暂未开启交易          |                          |
| 10062          | 金额精度不正确                    |                          |

# 市场 APIs

## 行情信息

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
访问频率限制：20 次/秒
</aside>

### 响应参数

| **字段** | **数据类型** | **说明**                  |
| -------------- | ------------- | -------------------------------- |
| buy            | string        | 当前最高买入价                |
| change         | string        | 涨跌幅(UTC+8零点开始算) |
| high           | string        | 最高价(UTC+8零点开始算) |
| last           | string        | 最新成交价            |
| low            | string        | 最低价(UTC+8零点开始算)  |
| sell           | string        | 当前最低卖出价                |
| symbol         | string        | 交易对                     |
| vol            | string        | 交易量(UTC+8零点开始算) |

## 市场深度

> Response:

```json
{
    "asks":[
        [
            "48095.8344",
            "0.02899"
        ]
    ],
    "bids":[
        [
            "48070.1725",
            "0.07424"
        ]
    ]
}
```

* **GET** `/api/v1/depth`

<aside class="notice">
访问频率限制：20 次/秒
</aside>

### 请求参数

| **参数** | **说明**            |
| ---------- | ------------------------------ |
| symbol     | 交易对                    |
| size       | 深度数量 |

### 响应参数

| **字段** | **数据类型** | **说明**              |
| -------------- | ------------- | ---------------------------- |
| asks           | array         | 卖价深度, [价位, 挂单量] |
| bids           | array         | 买价深度, [价位, 挂单量] |

## 最新成交

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
访问频率限制：20 次/秒
</aside>

### 请求参数

| **参数** | **说明**            |
| ---------- | ------------------------------ |
| symbol     | 交易对                    |
| size       | 成交单数量 |

### 响应参数

| **字段** | **数据类型** | **说明**              |
| ---- | ---- | ---- |
| amount | string | 成交量 |
| price | string | 成交价 |
| side | string | 买单或卖单 |
| timestamp | string | 成交时间 |

## K线数据

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
访问频率限制：20 次/秒
</aside>

### 请求参数

| **参数** | **说明**            |
| ---------- | ------------------------------ |
| symbol     | 交易对                    |
| size       | 数据量 |
| type       | 分时参数，可选值为1min,5min,15min,30min,hour,day,week |

### 响应参数

| **字段** | **数据类型** | **说明**              |
| ---- | ---- | ---- |
| index 0 | number | 时间 |
| index 1 | string | 开盘价 |
| index 2 | string | 最高价 |
| index 3 | string | 最低价 |
| index 4 | string | 收盘价 |
| index 5 | string | 交易量 |

## 交易对信息

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
访问频率限制：20 次/秒
</aside>

### 响应参数

| **字段** | **数据类型** | **说明**              |
| ---- | ---- | ---- |
| baseAsset | string | 基础货币 |
| baseAssetPrecision | number | 基础货币最大精度 |
| quoteAsset | string | 计价货币 |
| quoteAssetPrecision | number | 计价货币最大精度 |
| status | string | 交易对状态 |
| symbol | string | 交易对 |
| min_quantity | string | 最小交易数量 |
| tick_size | string | 最小价格增幅 |

# 账户 APIs

所有私有API的请求都采用POST方式，参数以form的形式提交（必须传递api_key和sign）。

## 币币账户余额

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
访问频率限制：10 次/秒
</aside>

### 响应参数

| **字段** | **数据类型** | **说明**              |
| ---- | ---- | ---- |
| code | number | 错误码 |
| message | string | 错误信息 |
| <result> | map | 响应数据 |
| <asset> | object | 币种余额信息 |
| available | string | 可用余额 |
| freeze | string | 交易冻结 |
| other_freeze | string | 其他冻结 |
| recharge_status | number | 币种充值状态, 1 表示可充值 |
| trade_status | number | 币种交易状态, 1 表示可交易 |
| withdraw_fee | string | 币种提币手续费 |
| withdraw_max | string | 币种提币最大数量 |
| withdraw_min | string | 币种提币最小数量 |
| withdraw_status | number | 币种提币状态, 1 表示可提币 |

## 充值历史

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
访问频率限制：10 次/秒
</aside>

### 请求参数

| **参数** | **类型** | **是否必填** | **说明**    | **可选值**      |
| -------------- | -------- | ------------- | ------------------ | -------------- |
| asset         | string   | N             | 充值币种 | （eg: USDT） |
| status       | number  | N             | 充值状态, 1 成功, 6 区块确认中, 7 失败  | 1,6,7 |
| limit       | number  | Y             | 数据量 | 不超过 100 |
| offset       | number  | N          | 偏移量 |  |

### 响应参数

| **字段** | **数据类型** | **说明**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| < result > | array | business data |
| id | number |  |
| created_at | string | 充值记录创建时间 |
| tx_id | string | 哈希 |
| asset | string | 币种 |
| amount | string | 数量 |
| status | number | 1 成功, 6 区块确认中, 7 失败 |
| address | string | 充值地址 |
| chain_name | string | 链 |
| confirmation | string | 当前区块确认数 |
| min_confirmation | string | 可入账的最小区块确认数 |

## 提币历史

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
访问频率限制：10 次/秒
</aside>

### 请求参数

| **参数** | **类型** | **是否必填** | **说明**    | **可选值**      |
| -------------- | -------- | ------------- | ------------------ | -------------- |
| asset         | string   | N             | 提现币种 | （eg: USDT） |
| status       | number  | N             | 提币状态, 1 成功, 2 取消, 3 已提交, 4 初审, 5 二审, 6 区块确认中, 7 失败  | 1,2,3,4,5,6,7 |
| limit       | number  | Y             | 数据量 | 不超过 100 |
| offset       | number  | N          | 偏移量 |  |

### 响应参数

| **字段** | **数据类型** | **说明**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| < result > | array | business data |
| id | number |  |
| created_at | string | 提币记录创建时间 |
| tx_id | string | 哈希 |
| asset | string | 币种 |
| amount | string | 数量 |
| fee | string | 手续费 |
| fee_asset | string | 手续费币种，不一定跟提现币种相同，看是否启用了其他币种抵扣 |
| status | number |  |
| address | string | 提币地址 |
| chain_name | string | 链 |
| confirmation | string | 当前区块确认数 |

# 交易 APIs

## 限价挂单

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
访问频率限制：500 次/秒
</aside>

### 请求参数

| **参数** | **类型** | **是否必填** | **说明**    | **可选值**      |
| -------------- | -------- | ------------- | ------------------ | -------------- |
| market         | string   | Y             | 交易对 | （eg: BTC_USDT） |
| side           | number  | Y             | 买卖类型   | 1(ask), 2(bid) |
| amount         | string   | Y             |        数量            |                |
| price          | string   | Y             |         价格           |                |

### 响应参数

| **字段** | **数据类型** | **说明**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| <result> | map | business data |
| id | number |  |
| ctime | number |  |
| mtime | number |  |
| market | string |  |
| amount | string |  |
| price | string |  |
| side | number | ask 或 bid |
| type | number | 1 限价单, 2 市价单 |
| user | number | 用户id |
| maker_fee | string | maker手续费 |
| taker_fee | string | taker手续费 |
| deal_stock | string | base asset成交量 |
| deal_money | string | quote asset成交量 |
| deal_fee | string | 成交额手续费 |
| left | string | 未成交数量 |

## 市价挂单

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
访问频率限制：500 次/秒
</aside>

### 请求参数

| **参数** | **类型** | **是否必填** | **说明**    | **可选值**      |
| -------------- | -------- | ------------- | ------------------ | -------------- |
| market         | string   | Y             | 交易对 | （eg: BTC_USDT） |
| side           | number  | Y             | 买卖类型   | 1(ask), 2(bid) |
| amount         | string   | Y             |        数量            |                |

### 响应参数

| **字段** | **数据类型** | **说明**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| <result> | map | business data |
| id | number |  |
| ctime | number |  |
| mtime | number |  |
| market | string |  |
| amount | string |  |
| price | string |  |
| side | number | ask 或 bid |
| type | number | 1 限价单, 2 市价单 |
| user | number | 用户id |
| maker_fee | string | maker手续费 |
| taker_fee | string | taker手续费 |
| deal_stock | string | base asset成交量 |
| deal_money | string | quote asset成交量 |
| deal_fee | string | 成交额手续费 |
| left | string | 未成交数量 |

## 取消订单

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
访问频率限制：20 次/秒
</aside>

### 请求参数

| **参数** | **类型** | **是否必填** | **说明**    | **可选值**      |
| -------------- | -------- | ------------- | ------------------ | -------------- |
| market         | string   | Y             | 交易对 | （eg: BTC_USDT） |
| order_id       | number  | Y             | 订单id   |  |

### 响应参数

| **字段** | **数据类型** | **说明**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| <result> | map | business data |
| id | number |  |
| ctime | number |  |
| mtime | number |  |
| market | string |  |
| amount | string |  |
| price | string |  |
| side | number | ask 或 bid |
| type | number | 1 限价单, 2 市价单 |
| user | number | 用户id |
| maker_fee | string | maker手续费 |
| taker_fee | string | taker手续费 |
| deal_stock | string | base asset成交量 |
| deal_money | string | quote asset成交量 |
| deal_fee | string | 成交额手续费 |
| left | string | 未成交数量 |

## 批量取消订单

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
访问频率限制：10 次/秒
</aside>

### 请求参数

| **参数** | **类型** | **是否必填** | **说明**    | **可选值**      |
| -------------- | -------- | ------------- | ------------------ | -------------- |
| orders_json         | string   | Y             | 取消订单接口参数的数组，json序列化后的字符串 | eg: [{"market":"BTC_USDT", "order_id":456647},{"market":"ETH_USDT", "order_id":456648}] |

### 响应参数

| **字段** | **数据类型** | **说明**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| <result> | map | business data |
| market | string |  |
| order_id | number |  |
| result | boolean | true 表示取消成功，false取消失败 |

## 未成交订单

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
访问频率限制：20 次/秒
</aside>

### 请求参数

| **参数** | **类型** | **是否必填** | **说明**    | **可选值**      |
| -------------- | -------- | ------------- | ------------------ | -------------- |
| market         | string   | Y             | 交易对 | （eg: BTC_USDT） |
| offset       | number  | N             | Offset   |  |
| limit       | number  | Y             | Limit   | 不超过 100 |

### 响应参数

| **字段** | **数据类型** | **说明**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| <result> | map | business data |
| limit | number |  |
| offset | number |  |
| total | number |  |
| <records> | array |  |
| id | number |  |
| ctime | number |  |
| mtime | number |  |
| market | string |  |
| amount | string |  |
| price | string |  |
| side | number | ask 或 bid |
| type | number | 1 限价单，2 市价单 |
| user | number | 用户id |
| maker_fee | string | maker手续费 |
| taker_fee | string | taker手续费 |
| deal_stock | string | base asset成交量 |
| deal_money | string | quote asset成交量  |
| deal_fee | string | 成交额手续费 |
| left | string | 未成交数量 |

## 未成交订单明细

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
访问频率限制：20 次/秒
</aside>

### 请求参数

| **参数** | **类型** | **是否必填** | **说明**    | **可选值**      |
| -------------- | -------- | ------------- | ------------------ | -------------- |
| market         | string   | Y             | 交易对 | （eg: BTC_USDT） |
| order_id       | number  | Y             | 订单id   |  |

### 响应参数

| **字段** | **数据类型** | **说明**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| <result> | map | business data |
| id | number |  |
| ctime | number |  |
| mtime | number |  |
| market | string |  |
| amount | string |  |
| price | string |  |
| side | number | ask 或 bid |
| type | number | 1 限价单，2 市价单 |
| user | number | 用户id |
| maker_fee | string | maker手续费 |
| taker_fee | string | taker手续费 |
| deal_stock | string | base asset成交量 |
| deal_money | string | quote asset成交量  |
| deal_fee | string | 成交额手续费 |
| left | string | 未成交数量 |

## 已成交订单

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
访问频率限制：10 次/秒
</aside>

### 请求参数

| **参数** | **类型** | **是否必填** | **说明**    | **可选值**      |
| -------------- | -------- | ------------- | ------------------ | -------------- |
| market         | string   | Y             | 交易对 | （eg: BTC_USDT） |
| start_time       | number  | N             | 开始时间，秒级时间戳, 0为不限   |  |
| end_time       | number  | N             | 结束时间，秒级时间戳, 0为不限   |  |
| offset       | number  | N             | 偏移量   |  |
| limit       | number  | Y             | 数据量   | 不超过 100 |
| side       | number  | N             | 1 卖, 2 买, 0为不限   |  |

### 响应参数

| **字段** | **数据类型** | **说明**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| <result> | map | business data |
| limit | number |  |
| offset | number |  |
| <records> | array |  |
| id | number |  |
| ctime | number |  |
| ftime | number |  |
| market | string |  |
| price | string |  |
| side | number | ask 或 bid |
| type | number | 1 限价单，2 市价单 |
| user | number | 用户id |
| maker_fee | string | maker手续费 |
| taker_fee | string | taker手续费 |
| deal_stock | string | base asset成交量 |
| deal_money | string | quote asset成交量  |
| deal_fee | string | 成交额手续费 |
| left | string | 未成交数量 |

## 已成交订单明细

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
访问频率限制：10 次/秒
</aside>

### 请求参数

| **参数** | **类型** | **是否必填** | **说明**    | **可选值**      |
| -------------- | -------- | ------------- | ------------------ | -------------- |
| order_id         | number   | Y             | 订单id |  |

### 响应参数

| **字段** | **数据类型** | **说明**              |
| ---- | ---- | ---- |
| code | number | error code |
| message | string | error message |
| <result> | map | business data |
| id | number |  |
| ctime | number |  |
| ftime | number |  |
| market | string |  |
| amount | string |  |
| price | string |  |
| side | number | ask 或 bid |
| type | number | 1 限价单，2 市价单 |
| user | number | 用户id |
| maker_fee | string | maker手续费 |
| taker_fee | string | taker手续费 |
| deal_stock | string | base asset成交量 |
| deal_money | string | quote asset成交量  |
| deal_fee | string | 成交额手续费 |
| left | string | 未成交数量 |

# Websocket APIs

## 接入介绍

### 接入URL

* wss://www.ztb.im/ws (海外域名)
* wss://www.ztbzh.net/ws (大陆域名)

### 请求格式

* method: string
* params: array
* id:     integer, 自定义随机数

### 响应格式

* result: json object，无数据返回null
* error:  json object，成功返回 null, 失败返回非 null

	* code:    error code 
	* message: error message 

* id:     integer

### 通知格式

* method: string
* params: array
* id:     null

### 错误码

* 1: 参数不合法
* 2: 内部错误
* 3: 服务不可用
* 4: 方法未找到
* 5: 服务超时
* 6: 需要鉴权

### 订阅主题

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

与websocket服务端建立连接成功后，websocket客户端发送如下请求订阅特定主题：

`{"method":"method to sub","params":[request params],"id":id generated by client}`

### 取消订阅

> Example:

```json
{
  "method": "kline.unsubscribe"
}
```

websocket订阅特定主题后，如果需要退订，websocket客户端发送如下请求退订：

`{"method":"method to unsubscribe"}`

### 查询

> Example:

```json
{
  "method": "kline.query",
  "params": [
    "BTC_USDT",
    1575561600,
    1575648000,
    300
  ],
  "id": 10086
}
```

websocket 服务也支持客户端拉取数据。

请求格式如下：

`{"method":"method to query","params":[request params],"id":id generated by client}`

## 市场状态

本主题推送市场最新行情。

### 订阅

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

| 参数 | 类型   | **是否必填** | 说明 | **可选值**    |
| --------- | ------ | ------------------- | ----------- | ------------ |
| market    | string | Y                |             | eg: BTC_USDT |

### 取消订阅

`
{"method":"state.unsubscribe"}
`

### 查询

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

一次性获取过去特定时间的行情数据

`
{"method":"state.query","params":[$market$,$period$],"id":10086}
`

| 参数 | 类型   | 是否必填 | 说明 | **可选值**   |
| --------- | ------ | --------------- | ----------- | ----------- |
| market    | string | Y            |             | eg:BTC_USDT |
| period    | int    | Y            |             | Eg: 86400   |

## 今日市场状态

本主题推送今日市场状态。

### 订阅

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

| 参数 | 类型   | **是否必填** | 说明 | **可选值**    |
| --------- | ------ | ------------------- | ----------- | ------------ |
| market    | string | Y                |             | eg: BTC_USDT |

### 取消订阅

`
{"method":"today.unsubscribe"}
`

### 查询

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

可按要求一次性获取今日市场状态数据。

`
{"method":"today.query","params":[$market$,$period$],"id":10086}
`

| 参数 | 类型   | 是否必填 | 说明 | **可选值**   |
| --------- | ------ | --------------- | ----------- | ----------- |
| market    | string | Y            |             | eg:BTC_USDT |

## K-Line Data

本主题推送最新K线数据。

### 订阅

> Request:

```json
{
  "method": "kline.subscribe",
  "params": [
    "BTC_USDT", 	market
    300   			interval
  ],
  "id": 10086		id
}
```

> Response:

```json
{
  "id": null,
  "method": "kline.update",
  "params": [
    [
      1575705900,           time
      "7542.8082",  		open
      "7534.9152", 			close
      "7547.0765", 			high
      "7530.8753", 			low
      "70.7463", 			amount
      "533428.87370982",	deal_money
      "BTC_USDT" 			market
    ]
  ]
}
```

`
{"method":"kline.subscribe","params":[$market$,$interval$],"id":10086}
`

| 参数 | 类型   | **是否必填** | 说明 | **可选值**                        |
| --------- | ------ | ------------------- | ----------- | -------------------------------- |
| market    | string | Y                |             | BTC_USDT, ETH_USDT...            |
| interval  | string | Y                |             | 60,300,900,1800,3600,7200,14400… |

### 取消订阅

`
{"method":"kline.unsubscribe"}
`

### 查询

> Request:

```json
{
  "method": "kline.query",
  "params": [
    "BTC_USDT",		// market
    1575561600,		// start
    1575648000,		// end
    300			    	// interval
  ],
  "id": 10086		id
}
```

> Response:

```json
{
  "error": null,
  "result": [
    [
      1575561600,		// time
      "7340.949",  	// open
      "7345.5655", 	// close
      "7357.0065", 	// high
      "7332.0522", 	// low
      "77.4528",		// amount
      "568870.6347857", // deal money
      "BTC_USDT"		// market
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

一次性获取K线数据

`
{"method":"kline.query","params":[$market$,$start$,$end$,$interval$],"id":10086}
`

| 参数 | 类型   | 是否必填 | 说明 | **可选值**   |
| --------- | ------ | --------------- | ----------- | ----------- |
| start     | integer | false           |             | eg: 1575561600 |
| end       | integer | false           |             | eg: 1575648000 |

## 市场深度

本主题推送最新的市场深度数据。

### 订阅

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
    true,  // true represents the full depth list，false represents update
    {
      "bids": [
        [
          "7457.1469"   price
          "0.0026"    	amount
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

| 参数 | 类型   | **是否必填** | 说明 | **可选值**                        |
| --------- | ------ | ------------------- | ----------- | -------------------------------- |
| market    | string | Y            |             | eg: BTC_USDT                                                 |
| limite    | int    | Y            |             | 1, 5, 10, 20, 30, 50, 100                                    |
| interval  | string | Y            | merge depth | "0", "0.00000001", "0.0000001", "0.000001", "0.00001", "0.0001", "0.001", "0.01", "0.1" |

### 取消订阅

`
{"method":"depth.unsubscribe"}
`

### 查询

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

`
{"method":"depth.query","params":[$market$,$limit$,$interval$],"id":10086}
`

| 参数 | 类型   | 是否必填 | 说明 | **可选值**   |
| --------- | ------ | --------------- | ----------- | ------------------------------------------------------------ |
| market    | string | Y            |             | eg: BTC_USDT                                                 |
| limit    | int    | Y            |             | 1, 5, 10, 20, 30, 50, 100                                    |
| interval  | string | Y            | 深度合并 | "0", "0.00000001", "0.0000001", "0.000001", "0.00001", "0.0001", "0.001", "0.01", "0.1" |

## 最新成交价

本主题推送最新成交价

### 订阅

> Request:

```json
{
  "method": "price.subscribe",
  "params": [
    "BTC_USDT",
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

| 参数 | 类型   | **是否必填** | 说明 | **可选值**                        |
| --------- | ------ | ------------------- | ----------- | -------------------------------- |
| market    | string | Y            |             | eg: BTC_USDT |

### 取消订阅

`
{"method":"price.unsubscribe"}
`

### 查询

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

`
{"method":"price.query","params":[$market$],"id":10086}
`

| 参数 | 类型   | 是否必填 | 说明 | **可选值**   |
| --------- | ------ | --------------- | ----------- | ------------------------------------------------------------ |
| market    | string | Y            |             | eg: BTC_USDT |

## 最新成交明细

本主题推送最新成交订单。

### 订阅

> Request:

```json
{
  "method": "deals.subscribe",
  "params": [
    "BTC_USDT",
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

| 参数 | 类型   | **是否必填** | 说明 | **可选值**                        |
| --------- | ------ | ------------------- | ----------- | -------------------------------- |
| market    | string | Y            |             | eg: BTC_USDT |

### 取消订阅

`
{"method":"deals.unsubscribe"}
`

### 查询

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

一次性获取最近成交订单。

`
{"method":"deals.query","params":[$market$,$limit$,$last_id$],"id":10086}
`

| 参数 | 类型   | 是否必填 | 说明 | **可选值**   |
| --------- | ------ | --------------- | ----------- | ------------------------------------------------------------ |
| market    | string | Y            |                                    | eg: BTC_USDT,             |
| limit     | int    | Y            |                                    | 1, 5, 10, 20, 30, 50, 100 |
| last_id   | int    | Y            | 上次返回结果的最大id | eg: 597967944             |
