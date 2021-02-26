# ONCHAIN CUSTODIAN API 文档

## 1. 简介

本 API 文档旨在帮助开发者快速高效地将热钱包功能整合到自己的系统中。开放 API 分为查询和提币两种类型。开发者可以根据自身需求创建不同权限的 API，并利用 API 进行热钱包账户信息查询或者提现。

## 2. API KEY

开发者使用 API 前需先在 Onchain Custodian 平台申请 API key 等信息，再根据此文档详情进行开发。使用过程中如有问题或者建议请联系 Onchain Custodian 客户经理或通过邮箱 support@oncustodian.com 联系支持团队。

> **注意**: API key 有效期为 **90** 天。请在过期前删除旧的 API key，并生成新的 API key 继续使用。

## 3. REST API

REST，即 Representational State Transfer 的缩写，是目前最流行的一种互联网软件架构。它因结构清晰，扩展方便而被越来越多的程序员使用。

## 4.标准规范

### 时间戳

除非另外说明，API 返回的所有时间戳均以毫秒为单位，使用 UTC 时间。例如：**1573574713991**。

### 数字

为了保证跨平台的精确度和完整性，数字会作为字符串返回。建议您在发起请求时也将数字转换为字符串，以避免截断和精度错误。返回的整数（如交易编号和序号）不加引号。

## 5. API 接口类型

API 接口分以下两种类型：

### 公共 API 接口

公共接口可用于获取 API 服务器系统时间。

发送请求前无需验证。

### 私有 API 接口

私有 API 接口用于 Onchain Custodian 热钱包。每个私有 API 请求必须使用规范的形式签名验证。

私有 API 接口需要使用 API key 进行验证。

## 6. 验证

验证的细节分以下四个方面：

- 生成 API Key

- 发起请求

- IP 白名单

- 服务器时间

### 6.1 生成 API KEY 和签名

发起任何请求之前，您必须通平台创建一个 API key。创建 API key 后，您将获得 2 个重要信息：

#### API KEY

**API key**字符串由 Custodian 平台生成。

#### 签名

签名字符串是对以下多种因素使用 HMAC SHA256 加密方式生成的。

`sign=CryptoJS.enc.Base64.Stringify(CryptoJS.HmacSHA256(timestamp + 'GET' +'14db63d7f3614664ad1c71dd134a21dc'+ '/v1/api/account'+"?"+"coinType=ETH", secret))`

这里面的 timestamp、method、API key、request path、query string、body string（转成 map 后 key 由 ASCII 码排序再进行拼接）和 secret 是通过 BASE64 编码输出得到的。以上因素细节如下：

- **时间戳（timestamp）：** 使用 UTC 时间 Unix 格式，精确到毫秒。

> 时间戳和服务器时间前后**相差 30 秒以上**的请求将被系统视为过期并拒绝。如果您的服务器和 API 服务器之间存在较大的时间偏差，建议您使用获取服务器时间的 API 来获取服务器时间。

- **请求方法（method）：** GET 或 POST，全部大写。
- **请求接口路径（request path）：** 例如：/v1/api/account。
- **query string：** URL 中的参数，例如：`page_num=1&page_size=10&coin_type=BTC`
- **body string：** POST 请求的请求主体参数。参数名需要按照 ASCII 码进行排序，以 **=** 拼接 key 和 value，并以 **&** 分割多个 key-value。

例如 body 为：

```json
{
  "mnt_nd": "ZPG",
  "amount": 190,
  "to_address": "AUol16ghiT9AtxRDtNeq3ovhWJ5iaY6iyd"
}
```

以上 body 转换后生成的字符串为：

```text
amount=190&mnt_nd=ZPG&to_address=AUol16ghiT9AtxRDtNeq3ovhWJ5iaY6iyd
```

- **Secret：** 用户申请 API key 时生成。Custodian 会储存 secret 使用 passphrase 加密的哈希值。例如：22582BD0CFF14C41EDBF1AB98506286D。

> Passphrase 是 secret 的验证方式，由客户自定并提供给 Custodian 平台以确保 API 访问的安全性。如客户忘记 Passphrase，Custodian 平台无法进行恢复，客户需生成新的 API key。

### 6.2 发起 API 请求

所有**请求头**都必须包含以下内容：

| Key                         | Value                               | 说明                                                                   |
| --------------------------- | ----------------------------------- | ---------------------------------------------------------------------- |
| Authorization 认证信息      | ApiKey:Request time stamp:Signature | API key, UNIX 请求时间戳（精确到毫秒）, 和签名通过英文冒号拼接 (**:**) |
| Custodian-access-passphrase | 随机字符串                          | 用于加密 secret 的密码                                                 |

> **注意：** 所有请求都应该含有 application/json 类型内容，并且是有效的 JSON。

### 6.3 IP 白名单

API 接口在创建时必须设置 IP 白名单。在后续的接口调用中，只有白名单内的 IP 才能正常调用。

### 6.4 获取服务器时间

此公共接口用于获取 API 服务器的时间，不需要身份验证。

**请求方法:** GET

**请求 URL:** /v1/api/general/time

> 限速规则：每 2 秒 20 次

**返回示例：**

```json
{
  "timestamp": 1573574713991
}
```

### 6.5 API 接口调用示例

请参考此 GitHub 链接：https://github.com/aixingjuele/custodian-sdk-java

## 7.API 列表

| 请求方法 | URL                                                                                          | 描述                                 |
| :------: | -------------------------------------------------------------------------------------------- | ------------------------------------ |
|   GET    | [/v1/api/account](#711-查询钱包内全部资产详情)                                               | 获取钱包中各种类资产的详情           |
|   GET    | [/v1/api/account/{coinType}](#712-根据币种查询钱包详情)                                      | 获取钱包中某一种资产的详情           |
|   GET    | [/v1/api/account/deposit-address/{coinType}](#713-根据币种查询钱包地址)                      | 获取钱包中一种资产的充值地址         |
|   GET    | [/v1/api/account/verify-deposit-address/{coinType}/{address}](#714-验证钱包充值地址是否正确) | 验证一个地址是否是某个资产的充值地址 |
|   GET    | [/v1/api/account/list-hdaddress](#715-获取主地址和子地址详情)                                | 通过主地址获取子地址                 |
|   POST   | [/v1/api/list-trans](#721-获取交易列表)                                                      | 获取钱包交易历史，可通过参数筛选     |
|   GET    | [/v1/api/trans/{tx_id}](#722-根据交易单号查询交易详情)                                       | 通过交易单号获取交易详情             |
|   POST   | [/v1/api/trans/withdrawal](#731-发起出金请求)                                                | 发送出金请求                         |
|   POST   | [/v1/api/hd-address](#741-为主地址创建子地址)                                                | 为主地址创建子地址                   |
|   PUT    | [/v1/api/hd-address](#742-修改子地址名称)                                                    | 更改子地址名称                       |
|   POST   | [host:port/{notice-type}](#751-交易通知接口回调)                                             | 回调方法，可配置                     |

### 参考

#### 交易类型

| 交易类型 | 说明       |
| :------: | ---------- |
|    1     | 提币       |
|    2     | 主地址入金 |
|    3     | 子地址入金 |
|    4     | 归集交易   |

#### 币种名称

> 所有币种的 coin_type 的名称和 coin_unique_name 相同。

|  币种种类  | 说明                 |
| :--------: | -------------------- |
|    BTC     | 比特币               |
|    ETH     | 以太币               |
| USDT-ERC20 | 基于以太坊的 USDT    |
|    USDC    | USDC                 |
|    CGT     | Cache Gold 币        |
|    WTC     | Waltonchain 币       |
|    REN     | REN 币               |
|    FTM     | Fantom 币            |
|   MATIC    | Matic 币             |
| XSGD-ERC20 | 基于以太坊的 XSGD    |
|    BCH     | 比特币现金           |
|    ZIL     | Zilliqa 币           |
| XSGD-ZRC2  | 基于 Zilliqa 的 XSGD |

> API 请求限速规则如下：
>
> - **一般请求：** 每 2 秒 20 次
> - **提币请求：** 每 2 秒 10 次

### 7.1 账号

#### 7.1.1 查询钱包内全部资产详情

>

```json
{
  "URL": "/v1/api/account",

  "Method": "GET",

  "Params": {},

  "Response": {
    "code": 0,
    "msg": "SUCCESS",
    "result": [
      {
        "address": "tb1q07r35czmvuhl28r93qgv02h6t030gcheqd34rs",
        "address_name": "Hot-Wallet-16-BTC",
        "coin_unique_name": "BTC",
        "coin_symbol": "BTC",
        "coin_full_name": "Bitcoin",
        "coin_decimal": 8,
        "deposit_allowed": 1,
        "withdrawal_allowed": 1,
        "current_balance": "0.084800000000000000",
        "fee_coin": "BTC",
        "estimated_fee": "0.0001",
        "upper_limit": "10.000000000000000000",
        "lower_limit": "0.010000000000000000",
        "alarm_frequency": 0
      }
    ]
  }
}
```

|        参数        | 数据类型 | 说明                                              |
| :----------------: | :------: | :------------------------------------------------ |
|      address       |  string  | 钱包地址                                          |
|    address_name    |  string  | 钱包名称                                          |
|  coin_unique_name  |  string  | 币种在本系统中的代号                              |
|    coin_symbol     |  string  | 币种缩写                                          |
|   coin_full_name   |  string  | 币种全称                                          |
|    coin_decimal    |   int    | 余额值精度                                        |
|  deposit_allowed   |   int    | 可否充值（0=不允许，1=允许）                      |
| withdrawal_allowed |   int    | 可否提现（0=不允许，1=允许）                      |
|  current_balance   |  string  | 当前余额                                          |
|      fee_coin      |  string  | 手续费所用币种                                    |
|   estimated_fee    |  string  | 预估手续费数额                                    |
|    upper_limit     |  string  | 触发提示邮件的账户余额上限                        |
|    lower_limit     |  string  | 触发提示邮件的账户余额下限                        |
|  alarm_frequency   |   int    | 提示邮件频率设置（0=每笔交易，1=每小时， 2=每天） |

#### 7.1.2 根据币种查询钱包详情

> coin_type 参数传入的字符串必须是所需查询币种的系统代号。

```json
{
  "URL": "/v1/api/account/{coinType}",

  "Method": "GET",

  "Params": {},

  "Response": {
    "code": 0,
    "msg": "SUCCESS",
    "result": {
      "address": "tb1q07r35czmvuhl28r93qgv02h6t030gcheqd34rs",
      "address_name": "Hot-Wallet-16-BTC",
      "coin_unique_name": "BTC",
      "coin_symbol": "BTC",
      "coin_full_name": "Bitcoin",
      "coin_decimal": 8,
      "deposit_allowed": 1,
      "withdrawal_allowed": 1,
      "current_balance": "0.084800000000000000",
      "fee_coin": "BTC",
      "estimated_fee": "0.0001",
      "upper_limit": "10.000000000000000000",
      "lower_limit": "0.010000000000000000",
      "limit_per_deal": "1.000000000000000000",
      "day_limit_amount": "1.000000000000000000",
      "hour_limit_amount": "1.000000000000000000",
      "alarm_frequency": 0
    }
  }
}
```

##### 请求参数

|   参数    | 数据类型 | 说明                      | 必要 |
| :-------: | :------: | :------------------------ | :--- |
| coin_type |  string  | 币种（[参考](#币种名称)） | 是   |

##### 响应参数

|        参数        | 数据类型 | 说明                                              |
| :----------------: | :------: | :------------------------------------------------ |
|      address       |  string  | 钱包地址                                          |
|    address_name    |  string  | 钱包名称                                          |
|  coin_unique_name  |  string  | 币种在本系统中的代号                              |
|   coin_full_name   |  string  | 币种全称                                          |
|    coin_symbol     |  string  | 币种缩写                                          |
|    coin_decimal    |   int    | 余额值精度                                        |
|  deposit_allowed   |   int    | 可否充值（0=不允许，1=允许）                      |
| withdrawal_allowed |   int    | 可否提现（0=不允许，1=允许）                      |
|  current_balance   |  string  | 当前余额                                          |
|      fee_coin      |  string  | 手续费所用币种                                    |
|   estimated_fee    |  string  | 预估手续费                                        |
|    upper_limit     |  string  | 触发提示邮件的账户余额上限                        |
|    lower_limit     |  string  | 触发提示邮件的账户余额下限                        |
|   limit_per_deal   |  string  | 单笔交易限额                                      |
|  day_limit_amount  |  string  | 当日剩余可交易额度                                |
| hour_limit_amount  |  string  | 当前小时剩余可交易额度                            |
|  alarm_frequency   |   int    | 提示邮件频率设置（0=每笔交易，1=每小时， 2=每天） |

#### 7.1.3 根据币种查询钱包地址

```json
{
  "URL": "/v1/api/account/deposit-address/{coinType}",

  "Method": "GET",

  "Params": {},

  "Response": {
    "code": 0,
    "msg": "SUCCESS",
    "result": {
      "coin_unique_name": "",
      "deposit_address": ""
    }
  }
}
```

##### 请求参数

|   参数    | 数据类型 | 说明                      | 必要 |
| :-------: | :------: | :------------------------ | :--- |
| coin_type |  string  | 币种（[参考](#币种名称)） | 是   |

##### 响应参数

|       参数       | 数据类型 | 说明                 |
| :--------------: | :------: | :------------------- |
| coin_unique_name |  string  | 币种在本系统中的代号 |
| deposit_address  |  string  | 充值地址             |

#### 7.1.4 验证钱包充值地址是否正确

> 此接口只能验证主地址。传入任何子地址的都会返回“FALSE”。

```json
{
  "URL": "/v1/api/account/verify-deposit-address/{coinType}/{address}",

  "Method": "GET",

  "Params": {},

  "Response": {
    "code": 0,
    "msg": "SUCCESS",
    "result": true
  }
}
```

##### 请求参数

|   参数    | 数据类型 | 说明                      | 必要 |
| :-------: | :------: | :------------------------ | :--- |
| coin_type |  string  | 币种（[参考](#币种名称)） | 是   |
|  address  |  string  | 主地址                    | 是   |

##### 响应参数

|  参数  | 数据类型 | 说明                          |
| :----: | :------: | :---------------------------- |
| result | Boolean  | true = success , false = fail |

#### 7.1.5 获取主地址和子地址详情

> 查询子地址详情需要传入相应的主地址

```json
{
  "URL": "/v1/api/account/list-hdaddress",

  "Method": "POST",

  "Params": {
    "master_address": "0x29ff770a1edaa03736336dc84440e0d42a77ee5c",
    "hd_address": [
      "0x01cf1d31f3ad0fddc0194c0406fb0b75d85b859e",
      "0x01c93e442561decd970c68d51e9fa86d7c7ccd75"
    ],
    "coin_type": "USDT-ERC20",
    "start_balance": "0",
    "end_balance": "100",
    "page_num": 1,
    "page_size": 10
  },

  "Response": {
    "code": 0,
    "msg": "string",
    "result": {
      "list": [
        {
          "address": "string",
          "address_name": "string",
          "coin_decimal": 0,
          "coin_full_name": "string",
          "coin_symbol": "string",
          "coin_unique_name": "string",
          "current_balance": "string",
          "deposit_allowed": 0,
          "estimated_fee": "string",
          "fee_coin": "string",
          "withdrawal_allowed": 0,
          "lower_limit": "string",
          "upper_limit": "string",
          "alarm_frequency": 0
        }
      ],
      "page_num": 0,
      "page_size": 0,
      "pages": 0,
      "total": 0
    }
  }
}
```

##### 请求参数

|      参数      | 数据类型 | 说明                        | 必要 |
| :------------: | :------: | :-------------------------- | :--- |
| master_address |  string  | 热钱包主地址                | 是   |
|   hd_address   |  array   | 子地址。数组参数最大 100 个 | 否   |
|   coin_type    |  string  | 币种                        | 否   |
| start_balance  |  string  | 所查询地址的最小余额        | 否   |
|  end_balance   |  string  | 所查询地址的最大余额        | 否   |
|    page_num    |   int    | 页码                        | 否   |
|   page_size    |   int    | 每页数据条数                | 否   |

##### 响应参数

|        参数        | 数据类型 | 说明                                              |
| :----------------: | :------: | :------------------------------------------------ |
|      address       |  string  | 子地址                                            |
|    address_name    |  string  | 子地址名称                                        |
|    coin_decimal    |   int    | 余额值精度                                        |
|   coin_full_name   |  string  | 币种全称                                          |
|    coin_symbol     |  string  | 币种缩写                                          |
|  coin_unique_name  |  string  | 币种在本系统中的代号                              |
|  current_balance   |  string  | 当前余额                                          |
|  deposit_allowed   |   int    | 可否充值,0=不允许,1 允许                          |
|   estimated_fee    |  string  | 预估手续费                                        |
|      fee_coin      |  string  | 手续费所用币种                                    |
| withdrawal_allowed |   int    | 可否提现,0=不允许,1 允许                          |
|    lower_limit     |  string  | 触发提示邮件的账户余额下限                        |
|    upper_limit     |  string  | 触发提示邮件的账户余额上限                        |
|  alarm_frequency   |   int    | 提示邮件频率设置（0=每笔交易，1=每小时， 2=每天） |
|      page_num      |   int    | 当前页码                                          |
|     page_size      |   int    | 每页数据条数                                      |
|       total        |   int    | 总数据条数                                        |
|       pages        |   int    | 总页数                                            |

### 7.2 交易详情

#### 7.2.1 获取交易列表

> 每页数据条数默认为 10

```json
{
  "URL": "/v1/api/list-trans",

  "Method": "POST",

  "Params": {
    "tx_type": "1",
    "tx_id": "123",
    "coin_type": "BTC",
    "address": "tb1q07r35czmvuhl28r93qgv02h6t030gcheqd34rs",
    "start_tx_amount": "0",
    "end_tx_amount": "100",
    "page_num": 1,
    "page_size": 10
  },

  "Response": {
    "code": 0,
    "msg": "SUCCESS",
    "result": {
      "page_num": 1,
      "page_size": 10,
      "total": 1882,
      "pages": 189,
      "list": [
        {
          "wallet_name": "XSGD_test",
          "coin_unique_name": "ETH",
          "coin_symbol": "ETH",
          "coin_full_name": "Ethereum",
          "coin_decimal": 18,
          "address": "0x99dc5fe06a7c6ec548dd283e3b67c28dea289d4e",
          "source_address": "0x975e71b7b89183bb9f86b098d0bb9af3baedff79",
          "tx_type": "2",
          "amount": "0.009000000000000000",
          "tx_id": "20210226030405202339",
          "tx_hash": "0x1439a2634cea40e4673498fc3aad600a9360a37531df919f9ec94413ef692a75",
          "tx_status": "1",
          "create_time": 1614308646000,
          "confirm_time": 1614308646000,
          "confirm_threshold": 6,
          "confirm_threshold_count": 6,
          "fee_coin": "ETH",
          "fee": "0",
          "fee_coin_decimal": 18
        }
      ]
    }
  }
}
```

##### 请求参数

|      参数       | 数据类型 | 说明                          | 必要 |
| :-------------: | :------: | :---------------------------- | :--- |
|     tx_type     |  string  | 交易类型（[参考](#交易类型)） | 否   |
|    coin_type    |  string  | 币种（[参考](#币种名称)）     | 否   |
|     address     |  string  | 交易地址                      | 否   |
|      tx_id      |  string  | 交易单号                      | 否   |
| start_tx_Amount |  string  | 所查询交易的最小交易金额      | 否   |
|  end_tx_amount  |  string  | 所查询交易的最大交易金额      | 否   |
|    page_num     |   int    | 页码                          | 否   |
|    page_size    |   int    | 每页数据条数                  | 否   |

##### 响应参数

|          参数           | 数据类型 | 说明                                          |
| :---------------------: | :------: | :-------------------------------------------- |
|        page_num         |   int    | 当前页码                                      |
|        page_size        |   int    | 每页数据条数                                  |
|          total          |   int    | 总数据条数                                    |
|          pages          |   int    | 总页数                                        |
|       wallet_name       |  string  | 钱包名称                                      |
|    coin_unique_name     |  string  | 币种在本系统中的代号                          |
|       coin_symbol       |  string  | 币种缩写                                      |
|     coin_full_name      |  string  | 币种全称                                      |
|      coin_decimal       |   int    | 余额值精度                                    |
|         address         |  string  | 钱包地址                                      |
|     source_address      |  string  | 交易对方地址                                  |
|         tx_type         |  string  | 交易类型（[参考](#交易类型)）                 |
|         amount          |  string  | 交易金额                                      |
|          tx_id          |  string  | 交易单号                                      |
|         tx_hash         |  string  | 交易哈希                                      |
|        tx_status        |  string  | 交易状态（0=待确认，1=成功，2=失败,3=审核中） |
|       create_time       |   date    | 交易创建时间                                  |
|      confirm_time       |   date    | 交易状态（成功或失败)确定时间                 |
|    confirm_threshold    |   int    | 交易确认数（区块数）                          |
| confirm_threshold_count |   int    | 交易完成后已确认区块数                        |
|        fee_coin         |  string  | 手续费所用币种                                |
|           fee           |  string  | 手续费金额                                    |
|    fee_coin_decimal     |   int    | 手续费所用币种精度                            |

#### 7.2.2 根据交易单号查询交易详情

```json
{
  "URL": "/v1/api/trans/{tx_id}",

  "Method": "GET",

  "Params": {},

  "Response": {
    "code": 0,
    "msg": "SUCCESS",
    "result": {
      "wallet_name": "",
      "coin_unique_name": "",
      "coin_full_name": "",
      "coin_decimal": "",
      "address": "",
      "source_address": "",
      "tx_type": "1-withdrawal,2-master address deposit,3-hd address deposit,4-collect transaction",
      "amount": 100,
      "tx_id": "",
      "tx_hash": "",
      "tx_status": "",
      "create_time": "",
      "confirm_time": "",
      "fee_coin": "",
      "fee": ""
    }
  }
}
```

|       参数       | 数据类型 | 说明                                          |
| :--------------: | :------: | :-------------------------------------------- |
|   wallet_name    |  string  | 钱包名称                                      |
| coin_unique_name |  string  | 币种在本系统中的代号                          |
|  coin_full_name  |  string  | 币种全称                                      |
|   coin_decimal   |   int    | 余额值精度                                    |
|     address      |  string  | 钱包地址                                      |
|  source_address  |  string  | 交易对方地址                                  |
|     tx_type      |  string  | 交易类型[参考](#交易类型)）                   |
|      amount      |   int    | 交易金额                                      |
|      tx_id       |  string  | 交易单号                                      |
|     tx_hash      |  string  | 交易哈希                                      |
|    tx_status     |  string  | 交易状态（0=待确认，1=成功，2=失败,3=审核中） |
|   create_time    |  date  | 交易创建时间                                  |
|   confirm_time   |  date  | 交易状态（成功或失败)确定时间                 |
|     fee_coin     |  string  | 手续费所用币种                                |
|       fee        |  string  | 手续费金额                                    |

### 7.3 出金

#### 7.3.1 发起出金请求

```json
{
  "URL": "/v1/api/trans/withdrawal",

  "Method": "POST",

  "Params": {
    "request_id": 10,
    "coin_type": "",
    "to_address": "",
    "tx_amount": "",
    "note": ""
  },

  "Response": {
    "code": 0,
    "msg": "SUCCESS",
    "result": {
      "tx_id": ""
    }
  }
}
```

##### 请求参数

|    参数    | 数据类型 | 说明                          | 必要 |
| :--------: | :------: | :---------------------------- | :--- |
| request_id |   int    | 单次请求 ID，每次请求不得重复 | 是   |
| coin_type  |  string  | 币种（[参考](#币种名称)）     | 是   |
| to_address |  string  | 入金地址                      | 是   |
| tx_amount  |  string  | 出金金额                      | 是   |
|    note    |  string  | 备注                          | 否   |

##### 响应参数

| 参数  | 数据类型 | 说明                   |
| :---: | :------: | :--------------------- |
| tx_id |  string  | 请求成功则返回交易单号 |

### 7.4 子地址管理

#### 7.4.1 为主地址创建子地址

```json
{
  "URL": "/v1/api/hd-address",

  "Method": "POST",

  "Params": {
    "address": "0x29ff770a1edaa03736336dc84440e0d42a77ee5c",
    "count": 2,
    "remarks": ["hd-name-1", "hd-name-2"]
  },

  "Response": {
    "code": 0,
    "msg": "SUCCESS",
    "result": {
      "0x0797bb11a5bbe3692a042cfa57873a38b21ab96a": "hd-name-1",
      "0x1673332f5a8b894e834e1934657c44d2f47443b6": "hd-name-2"
    }
  }
}
```

##### 请求参数

|  参数   |   数据类型   | 说明                                  | 必要 |
| :-----: | :----------: | :------------------------------------ | :--- |
| address |    string    | 子地址的主地址                        | 是   |
|  count  |     int      | 需要创建子地址的个数，单次最大为 5 个 | 是   |
| remarks | string array | 子地址名称。传入个数与 count 数值一致 | 是   |

##### 响应参数

| 参数 | 数据类型 | 说明                              |
| :--: | :------: | :-------------------------------- |
| body |   map    | 格式为：子地址的主地址:子地址名称 |

#### 7.4.2 修改子地址名称

```json
{
  "URL": "/v1/api/hd-address",

  "Method": "PUT",

  "Params": {
    "address": "0x0797bb11a5bbe3692a042cfa57873a38b21ab96a",
    "remark": "hd-name-6"
  },

  "Response": {
    "code": 0,
    "msg": "SUCCESS"
  }
}
```

##### 请求参数

|  参数   | 数据类型 | 说明               | 必要 |
| :-----: | :------: | :----------------- | :--- |
| address |  string  | 子地址的主地址     | 是   |
| remark  |  string  | 修改后的子地址名称 | 是   |

### 7.5 接口回调

#### 7.5.1 交易通知接口回调

> {notice-type}为通知类型

```json
{
  "URL": "host:port/{notice-type}",

  "Method": "POST",

  "Params": {
    "address": "0x29d912b930c2288a2a54ad0446b3b53dce2718d5",
    "amount": "0.099000000000001000",
    "coinType": "ETH",
    "confirmedBlocks": "6",
    "fee": "0.000999999999999",
    "hash": "0xc24b4b8ecec4de14aed9523591de867fff965517d02d66a3f1d110ef5c4dcbd4",
    "status": "1",
    "txId": "20200103025302304697",
    "txType": "Withdraw"
  },

  "Response": {
    "code": 0,
    "msg": "SUCCESS"
  }
}
```

##### 请求参数

|      参数       | 数据类型 | 说明                                  |
| :-------------: | :------: | :------------------------------------ |
|   notice-type   |  string  | 通知类型。目前只有 transaction-notice |
|     address     |  string  | 交易地址                              |
|     amount      |  string  | 交易金额                              |
|    coinType     |  string  | 交易币种名称（[参考](#币种名称)）     |
| confirmedBlocks |  string  | 区块确认数                            |
|       fee       |  string  | 手续费                                |
|      hash       |  string  | 交易哈希                              |
|     status      |  string  | 交易状态（0=等待，1=成功，2=失败）    |
|      txId       |  string  | 交易单号。同一交易可多次通知，ID 唯一 |
|     txType      |  string  | 交易类型（[参考](#交易类型)）         |

##### 响应参数

| 参数 | 数据类型 | 说明                                                                                              |
| :--: | :------: | :------------------------------------------------------------------------------------------------ |
| code |   long   | OpenAPI 收到 0 说明通知成功。如果收到其他返回码或没有收到返回，会继续发送通知直到达到最大通知次数 |
| msg  |  string  | 返回描述                                                                                          |

## 8. 错误代码

|  code  | Description                                                      |
| :----: | :--------------------------------------------------------------- |
|   0    | SUCCESS                                                          |
| 106001 | The request parameter error                                      |
| 106002 | Unauthorized                                                     |
| 106003 | The service is temporarily unavailable                           |
| 106004 | Interface Timeout                                                |
| 106005 | Ip address check error                                           |
| 106006 | Signature verification error                                     |
| 106008 | Data encryption error                                            |
| 106009 | Data decryption error                                            |
| 106011 | Service call exception                                           |
| 106012 | Passphrase error                                                 |
| 106013 | Timestamp error                                                  |
| 106015 | ApiKey error                                                     |
| 106016 | The amount is in the wrong format.                               |
| 106018 | The hot wallet is frozen                                         |
| 106019 | The amount of withdrawal is wrong                                |
| 106020 | The amount of withdrawal is too long in decimal places           |
| 106021 | The transaction amount exceeds the upper limit of the single pen |
| 106022 | Authorization format error                                       |
| 106023 | Address error                                                    |
| 106024 | Address notes are 1-64 in length and only alphanumeric           |
| 106025 | HD address quantity exceeds                                      |
| 106026 | The frequency of the call exceeded the limit.                    |
| 106027 | The request is too fast                                          |
| 106028 | The Same request received                                        |
| 106029 | Cointype not support.                                            |
