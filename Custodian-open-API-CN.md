# ONCHAIN CUSTODIAN API 文档

## 1. 简介

本 API 文档旨在帮助开发者快速高效地将热钱包功能整合到自己的系统中。开放 API 分为查询和提币两种类型。开发者可以根据自身需求创建不同权限的 API，并利用 API 进行热钱包账户信息查询或者提现。

## 2. API KEY

开发者使用 API 前需先在 Onchain Custodian 平台申请 API key 等信息，再根据此文档详情进行开发。使用过程中如有问题或者建议请联系 Onchain Custodian 客户经理或通过邮箱 support@oncustodian.com 联系支持团队。

> **注意**: API key 生成后一直有效。您可以手动删除不需要的 API key。

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

**API key**字符串由 Custodian 账户管理员生成。

> 每个 API key 与唯一 1 个钱包关联，每个钱包最多可以关联 5 个 API key。API key 不会过期。

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
|  POST  | [/v1/api/account/balance](#716-根据币种获取钱包地址余额详情)                            | 根据币种获取钱包地址余额详情                                      |
|  PUT   | [v1/api/account/collect](#717-将子地址余额归集至主地址)                                | 当余额超过指定值时将子地址余额归集至主地址                                         |
|  PUT  | [v1/api/account/collect/auto](#718-更新自动归集配置)                               | 更新自动归集配置                                                 |
|  PUT  | [v1/api/account/txfee](#719-获取指定币种当前手续费)                               | 获取指定币种当前手续费                                                 |
|  PUT  | [v1/api/account/hd/collect](#7110-指定子地址金额归集)                               | 指定子地址金额归集                                                |
|   POST   | [/v1/api/list-trans](#721-获取交易列表)                                                      | 获取钱包交易历史，可通过参数筛选     |
|   GET    | [/v1/api/trans/{tx_id}](#722-根据交易单号查询交易详情)                                       | 通过交易单号获取交易详情             |
|   POST   | [/v1/api/trans/withdrawal](#731-发起出金请求)                                                | 发送出金请求                         |
|   POST   | [/v1/api/hd-address](#741-为主地址创建子地址)                                                | 为主地址创建子地址                   |
|   PUT    | [/v1/api/hd-address](#742-修改子地址名称)                                                    | 更改子地址名称                       |
|   POST   | [host:port/{notice-type}](#751-交易通知接口回调)                                             | 交易通知回调方法，可配置                     |
|   POST   | [host:port/{notice-type}](#752-币种地址绑定通知接口回调)                                             | 币种地址绑定通知回调方法，可配置                     |
|   POST   | [/v1/api/Hbar/HTSAddAddressOne](#761-单HBAR币种关联单地址)                                             | 单HBAR币种关联单地址                   |
|   POST   | [/v1/api/Hbar/addressAddHTS](#762-HBAR地址关联多个币种)                                             | HBAR地址关联多个币种                     |
|   POST   | [/v1/api/Hbar/HTSAddAddress](#763-HBAR币种关联多个地址)                                             | HBAR币种关联多个地址                   |
|   POST   | [/v1/nft/wallet](#764-创建NFT钱包)                                             | 创建NFT钱包                   |
|   GET    | [/v1/nft/wallet](#765-查询NFT钱包列表)                                             | 查询NFT钱包列表                   |
|   GET    | [/v1/nft/wallet/asset](#766-查询NFT资产列表)                                             | 查询NFT资产列表                   |
|   POST   | [/v1/nft/withdraw/fee](#767-查询NFT出金手续费)                                             | 查询NFT出金手续费                   |
|   POST   | [/v1/nft/withdraw](#768-发送NFT出金请求)                                             | 发送NFT出金请求                   |

### 参考

#### 交易类型

| 交易类型 | 说明       |
| :------: | ---------- |
|    1     | 提币       |
|    2     | 主地址入金 |
|    3     | 子地址入金 |
|    4     | 归集交易   |

#### 币种

> 所有币种的 coin_type 的名称和 coin_unique_name 相同。

|   币种   |              全称             |      协议       |
| :------------: | :---------------------------------: | :-----------------: |
|     BTC     |               Bitcoin               |       Bitcoin       |
|     BCH     |            Bitcoin Cash             |   Bitcoin（分叉）    |
|     ETH     |              Ethereum               |      Ethereum       |
|     PASS  |                Blockpass              |      Ethereum       |
| USDT-ERC20  |             Tether USD              |      Ethereum       |
|     USDC    |             USD Coin                |      Ethereum       |
|     CGT     |       Curio Governance Token        |      Ethereum       |
|     REN     |              Republic               |      Ethereum       |
|     FTM     |            Fantom Token             |      Ethereum       |
| XSGD-ERC20  |                XSGD                 |      Ethereum       |
|     DAI     |           Dai Stablecoin            |      Ethereum       |
|     FXF     |               Finxflo               |      Ethereum       |
|     SATT    | Smart Advertising Transaction Token |      Ethereum       |
|     SNX     |       Synthetix Network Token       |      Ethereum       |
| MATIC-ERC20 |             Matic Token             |      Ethereum       |
|     SHIB    |              Shiba Inu              |      Ethereum       |
|     1INCH   |             1Inch Token             |      Ethereum       |
|     SUSHI   |             Sushi Token             |      Ethereum       |
|     AVVE    |             Aave Token              |      Ethereum       |
|     LINK    |           ChainLink Token           |      Ethereum       |
|     CHZ     |               chiliZ                |      Ethereum       |
|     OKB     |                 OKB                 |      Ethereum       |
|     BUSD    |             Binance USD             |      Ethereum       |
|     YFI     |            yearn.finance            |      Ethereum       |
|     HT      |             Huobi Token             |      Ethereum       |
|     COMP    |              Compound               |      Ethereum       |
|     UNI     |               Uniswap               |      Ethereum       |
|     MKR     |                Maker                |      Ethereum       |
|     WAVES   |                Waves                |      Ethereum       |
|     WBTC    |             Wrapped BTC             |      Ethereum       |
|     FTT     |              FTX Token              |      Ethereum       |
|     TUSD    |               TrueUSD               |      Ethereum       |
|     CRO     |           Crypto.com Coin           |      Ethereum       |
|     TEL     |               Telcoin               |      Ethereum       |
|     CEL     |               Celsius               |      Ethereum       |
|     HBTC    |              Huobi BTC              |      Ethereum       |
|     LEO     |         Bitfinex Leo Token          |      Ethereum       |
|     GXT     |         Gem Exchange and Trading    |      Ethereum       |
|     OOE     |               OpenOcean             |      Ethereum       |
| XIDR-ERC20  |           XIDR Token                |      Ethereum       |
|     ZIL     |               Ziliqa                |       Ziliqa        |
| XSGD-ZRC2   |                XSGD                 |       Ziliqa        |
| XIDR-ZRC2   |           XIDR Token                |       Ziliqa        |
|    HBAR     |           HBAR Token                |   Hedera Hashgraph  |
| XSGD-HTS    |                XSGD                 |   Hedera Hashgraph  |
|     BNB     |           BNB Token                 |       Binance       |
| BUSD-BEP20  |           Binance USD               |       Binance       |
| USDC-BEP20  |           USD Coin                  |       Binance       |
| USDT-BEP20  |           Tether USD                |       Binance       |
| FIL-BEP20   |           Filecoin                  |       Binance       |
| GXT-BEP20   |       Gem Exchange and Trading      |       Binance       |
|     SAC     |              SAC                    |       Sanus         |
|     FIL     |           Filecoin                  |       Filecoin      |
|     TRX     |           TRON                      |       Tron          |
| USDT-TRC20  |           Tether USD                |       Tron          |
|     KLAY    |           Klaytn                    |       Klaytn        |
|     MATIC   |          Matic Token                |       Polygon       |
|  USDC-POS   |           USD Coin                  |       Polygon       |
|  XIDR-POS   |           XIDR Token                |       Polygon       |
|  XSGD-POS   |           XSGD                      |       Polygon       |
|  SOL        |           Solana                    |       Solana        |
|  AVAX       |           Avalanche                 |       Avalanche     |

> API 请求限速规则如下：
>
> - **一般请求：** 每 2 秒 80 次
> - **提币请求：** 每 2 秒 40 次

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

#### 7.1.6. 根据币种获取钱包地址余额详情

> 如果沒有传入币种则返回所有币种的余额

```json
{
  "URL": "/v1/api/account/balance",

  "Method": "POST",

  "Params": {
    "coin_type": ["BTC", "ETH"],
    "include_master_address": true
  },

  "Response": {
    "code": 0,
    "msg": "string",
    "result": {
      "balance": {
        "BTC": "10",
        "ETH": "5"
      },
      "include_master_address": true
    }
  }
}
```

##### 请求参数

|          参数          | 数据类型 | 说明                                     | 必要 |
| :--------------------: | :------: | :--------------------------------------- | :--: |
|       coin_type        |  string  | 币种                                     |  否  |
| include_master_address | boolean  | 是否在响应中包含主地址（每个币种均返回） |  否  |

##### 响应参数

|          参数          | 数据类型 | 说明                                   |
| :--------------------: | :------: | :------------------------------------- |
|        balance         |  string  | key: 币种在本系统中的代号，value: 余额 |
| include_master_address | boolean  | 请求值                                 |

#### 7.1.7. 将子地址余额归集至主地址

> 没有传入子地址时，将从余额超过指定阈值的所有地址收集资金。

```json
{
  "URL": "v1/api/account/collect",

  "Method": "PUT",

  "Params": {
    "coin_type": "ETH",
    "sub_address_list": [
      "0x59e29511e5049fef98c00b4ad6954de27cdfafa3",
      "0xef87123e420b174ba8afcefa45a22d06f4bf482b"
    ],
    "trigger_threshold": 0
  },

  "Response": {
    "code": 0,
    "msg": "string",
    "result": {}
  }
}
```

##### 请求参数

|       参数        |   数据类型   | 说明                                 | 必要 |
| :---------------: | :----------: | :----------------------------------- | :--: |
|     coin_type     |    string    | 币种                                 |  是  |
| sub_address_list  | string array | 用于余额归集的子地址                 |  否  |
| trigger_threshold |    number    | 触发归集的最小余额阈值（最小值为 0） |  是  |

#### 7.1.8. 更新自动归集配置

```json
{
  "URL": "v1/api/account/collect/auto",

  "Method": "PUT",

  "Params": {
    "coin_type": "BTC",
    "switch_collect": true,
    "threshold": 0
  },

  "Response": {
    "code": 0,
    "msg": "string",
    "result": {}
  }
}
```

##### 请求参数

|      参数      | 数据类型 | 说明                                 | 必要 |
| :------------: | :------: | :----------------------------------- | :--: |
|   coin_type    |  string  | 币种                                 |  是  |
| switch_collect | boolean  | 自动归集开关                         |  是  |
|   threshold    |  number  | 触发归集的最小余额阈值（最小值为 0） |  是  |

#### 7.1.9. 获取指定币种当前手续费

> 目前只支持 Ethereum。

```json
{
  "URL": "/v1/api/account/txfee?coin_type={}&from_address={}&to_address={}&amount={}",
  "Method": "GET",
  "Params": {},
  "Response": {
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "chain_name": "Ethereum",
        "coin_type": "USDT-ERC20",
        "list_fee_step": [
            {
                "chain_fee": "0.000053",
                "usd_fee": "0.13945728"
            },
            {
                "chain_fee": "0.0000636",
                "usd_fee": "0.16734874"
            },
            {
                "chain_fee": "0.0000795",
                "usd_fee": "0.20918592"
            }
        ]
    }
}
  }
}
```

##### 请求参数

|    参数     | 数据类型 | 说明                         | 必要 |
| :---------: | :------: | :--------------------------- | :--: |
|  coin_type  |  string  | 币种（[参考](#币种名称)）                        |  是  |
| from_address  |  String  | 转出地址 | 是   |
| to_address |  String  |   转入地址   | 否   |
| amount  |  String  | 金额 | 否   |

##### 响应参数

| 参数  | 数据类型 | 说明                   |
| :---: | :------: | :--------------------- |
| chain_name |  String  | 链名 |
| coin_type |  String  | 手续费币种 |
| eth_usd_price |  Number  | eth美元价格 |
| chain_fee_dto |  Array  | waas erc20手续费模型 |
| fee |  Number  | 上链手续费 |
| gas_limit |  Number  | GAS限制 |
| gas_price |  Number  | GAS单价 |
| list_fee_step |  Array  | 根据链获取出金手续费模型梯度 |
| chain_fee |  Number  | 出金币种费用 |
| usd_fee |  Number  | usdt折合费用 |

#### 7.1.10. 指定子地址金额归集

```json
{
  "URL": "/v1/api/account/hd/collect",
  "Method": "PUT",
  "Params":  {
    "coin_type": "BTC",
    "sub_address": "0x59e29511e5049fef98c00b4ad6954de27cdfafa3",
    "tx_amount": 2.0
  },
  "Response": {
    "code": 0,
    "msg": "string",
    "result": "20220304130128348281"
  }
}
```

##### 请求参数

|    参数     | 数据类型 | 说明                         | 必要 |
| :---------: | :------: | :--------------------------- | :--: |
|  coin_type  |  string  | 币种（[参考](#币种名称)）                        |  是  |
| sub_address  |  String  | 归集地址 | 是   |
| tx_amount |  String  |   归集金额   | 是   |

##### 响应参数

| 参数  | 数据类型 | 说明                   |
| :---: | :------: | :--------------------- |
| result |  string  | 子地址归集产生的交易单号 |

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

##### 响应参数

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
    "note": "",
    "fee": ""
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
|    fee    |  string  | 手续费金额                          | 否   |

> 只有 ERC-20 币种可以自定义手续费金额。默认手续费为当前平均值。金额越高则交易确认越快。

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
|  count  |     int      | 需要创建子地址的个数，ERC20和ZRC2类型的地址单次最大为 500 个，HTS类型的地址单次只允许创建1个，其他类型的地址单次最大为5个 | 是   |
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
|   notice-type   |  string  | 通知类型。 transaction-notice |
|     address     |  string  | 交易地址                              |
|     amount      |  string  | 交易金额                              |
|    coinType     |  string  | 交易币种名称（[参考](#币种名称)）     |
| confirmedBlocks |  string  | 区块确认数                            |
|       fee       |  string  | 手续费                                |
|      hash       |  string  | 交易哈希                              |
|     status      |  string  | 交易状态（0=等待，1=成功，2=失败，4=审核失败，5=手动取消，6=发起风控失败，7=风控不通过，8=发起出金失败）    |
|      txId       |  string  | 交易单号。同一交易可多次通知，ID 唯一 |
|     txType      |  string  | 交易类型（[参考](#交易类型)）         |

##### 响应参数

| 参数 | 数据类型 | 说明                                                                                              |
| :--: | :------: | :------------------------------------------------------------------------------------------------ |
| code |   long   | OpenAPI 收到 0 说明通知成功。如果收到其他返回码或没有收到返回，会继续发送通知直到达到最大通知次数 |
| msg  |  string  | 返回描述                                                                                         |

#### 7.5.2 币种地址绑定通知接口回调

> {notice-type}为通知类型
```json
{
  "URL": "host:port/{notice-type}",
  "Method": "POST",

  "Params": {
   "address":"0.0.16404315",
   "status":"0",
   "unique_name":"XSGD-HTS"
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
|   notice-type   |  string  | 通知类型:hts-notice |
|     address     |  string  | 绑定地址                              |
|    unique_name   |  string  | 绑定币种名称（[参考](#币种名称)）     |
|     status      |  string  | 0代表绑定成功，1代表绑定失败需要重新发起绑定         |

##### 响应参数

| 参数 | 数据类型 | 说明                                                                                              |
| :--: | :------: | :------------------------------------------------------------------------------------------------ |
| code |   long   | OpenAPI 收到 0 说明通知成功。如果收到其他返回码或没有收到返回，会继续发送通知直到达到最大通知次数 |
| msg  |  string  | 返回描述                                                                                         |

### 7.6 Hbar 接口调用

#### 7.6.1 单HBAR币种关联单地址
>这是一个异步接口，用来发起币种绑定地址的请求。绑定结果会通过回调接口(币种地址绑定通知接口回调)通知。请根据需要配置接收通知的noticeUrl
```json
{
  "URL": "/v1/api/Hbar/HTSAddAddressOne",

  "Method": "POST",

  "Params": { 
   "address":"0.0.11973290",
   "unique_name":"HTS",
   "notice_url":"http://127.0.0.1:8090"
  },

  "Response": {
    "code": 0,
    "msg": "SUCCESS"
  }
}
```

##### 请求参数

|      参数       | 数据类型 | 说明                                  |必要 |
| :-------------: | :------: | :------------------------------------ |:--- |
|     address     |   String | 币种关联地址   String类型                           | 是   |
|     unique_name  |   String | 币种名称                              | 是   |
|     notice_url  |   String |    接收所发请求的地址和币种是否绑定通知的Url                          |是|


##### 响应参数

| 参数 | 数据类型 | 说明                                                                                              |
| :--: | :------: | :------------------------------------------------------------------------------------------------ |
| code |   long   |  0说明成功绑定，1代表绑定失败|
| msg  |  string  | 返回描述                                                                                          |

#### 7.6.2  HBAR地址关联多个币种
>这是一个异步接口，用来发起币种绑定地址的请求。绑定结果会通过回调接口(币种地址绑定通知接口回调)通知。请根据需要配置接收通知的noticeUrl
```json
{
  "URL": "/v1/api/Hbar/addressAddHTS",

  "Method": "POST",

  "Params": {
   "address": "0.0.11973290",
   "unique_name": ["HTS1","HTS2"],
   "notice_url":"http://127.0.0.1:8090"

  },

  "Response": {
    "code": 0,
    "msg": "SUCCESS"
  }
}
```

##### 请求参数

|      参数       | 数据类型 | 说明                                  |必要 |
| :-------------: | :------: | :------------------------------------ |:--- |
|     address     |  String  | 币种关联地址                              |是|
|     unique_name  |  List  | 币种名称 String类型                            |是|
|     notice_url  |   String |    接收所发请求的地址和币种是否绑定通知的Url                          |是|


##### 响应参数

| 参数 | 数据类型 | 说明                                                                                              |
| :--: | :------: | :------------------------------------------------------------------------------------------------ |
| code |   long   | 0说明成功发起绑定申请，1代表发起绑定申请失败|
| msg  |  string  | 返回描述                                                                                                |


#### 7.6.3 HBAR币种关联多个地址
>这是一个异步接口，用来发起币种绑定地址的请求。绑定结果会通过回调接口(币种地址绑定通知接口回调)通知。请根据需要配置接收通知的noticeUrl
```json
{
  "URL": "/v1/api/Hbar/HTSAddAddress",

  "Method": "POST",

  "Params": { 
   "address":["0.0.11973290","0.0.11972392"],
   "unique_name":"HTS",
   "notice_url":"http://127.0.0.1:8090"

  },

  "Response": {
    "code": 0,
    "msg": "SUCCESS"
  }
}
```

##### 请求参数

|      参数       | 数据类型 | 说明                                  |必要 |
| :-------------: | :------: | :------------------------------------ |:--- |
|     address     |   List | 币种关联地址   String类型                           |是|
|     unique_name  |   String | 币种名称                              |是|
|     notice_url  |   String |    接收所发请求的地址和币种是否绑定通知的Url                          |是|


##### 响应参数

| 参数 | 数据类型 | 说明                                                                                              |
| :--: | :------: | :------------------------------------------------------------------------------------------------ |
| code |   long   |0说明成功发起绑定申请，1代表发起绑定申请失败|
| msg  |  string  | 返回描述                                                                                          |

#### 7.6.4 创建NFT钱包

```json
{
  "URL": "/v1/nft/wallet",

  "Method": "POST",

  "Params": { 
   "wallet_name":"钱包1",
   "chain":"Ethereum"
  },

  "Response": {
    "code": 0,
    "msg": "SUCCESS",
    "result": {
      "wallet_id": 1,
      "address": "0x29d912b930c2288a2a54ad0446b3b53dce2718d5"
    }
  }
}
```

##### 请求参数

|      参数       | 数据类型 | 说明                                  |必要 |
| :-------------: | :------: | :------------------------------------ |:--- |
|     wallet_name     |   String | 钱包名字                          |是|
|     chain  |   String | 当前只支持Ethereum，固定值Ethereum                        |是|


##### 响应参数

| 参数 | 数据类型 | 说明                                                                                              |
| :--: | :------: | :------------------------------------------------------------------------------------------------ |
| code |   long   |0说明成功发起绑定申请，1代表发起绑定申请失败|
| msg  |  String  | 返回描述                                                                                          |
| wallet_id  |  Integer  | 钱包id                                                                                          |
| address  |  String  | 钱包地址                                                                                          |


#### 7.6.5 查询NFT钱包


```json
{
  "URL": "/v1/nft/wallet",

  "Method": "GET",

  "Response": {
    "code": 0,
    "msg": "SUCCESS",
    "result": [{
      "wallet_id": 1,
      "wallet_name": ”钱包1“,
      "chain": ”Ethereum“,
      "address": "0x29d912b930c2288a2a54ad0446b3b53dce2718d5",
      "isFrozen": false,
      "create_time": 1662541288000
    },
    {
      "wallet_id": 2,
      "wallet_name": ”钱包2“,
      "chain": ”Ethereum“,
      "address": "0x29d912b930c2288a2a54ad0446b3b53dce2718f6",
      "is_frozen": true,
      "create_time": 1648453695000
    }]
  }
}
```

##### 响应参数

| 参数 | 数据类型 | 说明                                                                                              |
| :--: | :------: | :------------------------------------------------------------------------------------------------ |
| code |   long   |0说明成功发起绑定申请，1代表发起绑定申请失败|
| msg  |  String  | 返回描述                                                                                          |
| wallet_id  |  Integer  | 钱包id                                                                                          |
| wallet_name  |  String  | 钱包名称                                                                                          |
| chain  |   String | 当前只支持Ethereum，固定值Ethereum                 
| address  |  String  | 钱包地址                                                                                          |
| is_frozen  |  Boolean  | 是否冻结                                                                                          |
| create_time  |  Long  | 创建时间，时间戳格式                                                                                 |


#### 7.6.6 查询NFT资产


```json
{
  "URL": "/v1/nft/wallet/asset?wallet_id=1&page_index=1&page_offset=10",

  "Method": "GET",

  "Response": {
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "total": 2,
        "records": [
            {
                "wallet_id": "136",
                "address": "0xebbdb7d9916c142cb8d082077f2f671436ad4bec",
                "contract_address": "0xebe96a77e9a80595d1106babe57e652a9d8e2b0f",
                "token_id": "0"
            },
            {
                "wallet_id": "136",
                "address": "0xebbdb7d9916c142cb8d082077f2f671436ad4bec",
                "contract_address": "0xebe96a77e9a80595d1106babe57e652a9d8e2b0f",
                "token_id": "1"
            }
        ]
    }}
}
```

##### 请求参数

|      参数       | 数据类型 | 说明                                  |必要 |
| :-------------: | :------: | :------------------------------------ |:--- |
|     wallet_id     |   Integer | 钱包id                          |是|
|     page_index     |   Integer | 第几页                          |是|
|     page_offset     |   Integer | 每页条数                         |是|

##### 响应参数

| 参数 | 数据类型 | 说明                                                                                              |
| :--: | :------: | :------------------------------------------------------------------------------------------------ |
| code |   long   |0说明成功发起绑定申请，1代表发起绑定申请失败|
| msg  |  String  | 返回描述                                                                                          |
| wallet_id  |  Integer  | 钱包id                                                                                          |
| address  |  String  | 钱包地址                                                                                          |
| contract_address  |  String  | 合约地址                                                                                          |
| token_id  |  String  | token id                                                                                          |


#### 7.6.7 查询NFT出金手续费


```json
{
  "URL": "/v1/nft/withdraw/fee",

  "Method": "POST",

  "Params": {
    "wallet_id":136,
    "chain":"Ethereum",
    "to_address":"0xdc587bc9a15383f2267f93b5f4379f10f75918b7",
    "contract_address":"0x0714f49f3527f8160f39a6e4d5003aeedc1409f8",
    "token_id":"6"
},

  "Response": {
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "chain_name": "Ethereum",
        "coin_type": "NFTETH",
        "fee_step": {
            "chain_fee": "0.00004821",
            "usd_fee": "0.07815774"
        },
        "chain_fee_dto": {
            "fee": "0.00004821",
            "gas_price": "1.00000001",
            "gas_limit": "48211"
        },
        "eth_usd_price": "1621.159954"
    }
}
}
```

##### 请求参数

|      参数       | 数据类型 | 说明                                  |必要 |
| :-------------: | :------: | :------------------------------------ |:--- |
|     wallet_id     |   Integer | 钱包id                          |是|
|     chain  |   String | 当前只支持Ethereum，固定值Ethereum                        |是|
|     to_address  |   String | to地址                        |是|
|     contract_address  |   String | 合约地址                       |是|
|     token_id  |   String | token id                        |是|


##### 响应参数

| 参数 | 数据类型 | 说明                                                                                              |
| :--: | :------: | :------------------------------------------------------------------------------------------------ |
| code |   long   |0说明成功发起绑定申请，1代表发起绑定申请失败|
| msg  |  String  | 返回描述                                                                                          |


#### 7.6.8 发送NFT出金申请


```json
{
  "URL": "/v1/nft/withdraw",

  "Method": "POST",

  "Params": {
    "request_id":"17f4d2d95b444e60b777bc129e8c6e18",
    "wallet_id":136,
    "chain":"Ethereum",
    "to_address":"0xdc587bc9a15383f2267f93b5f4379f10f75918b7",
    "contract_address":"0x0714f49f3527f8160f39a6e4d5003aeedc1409f8",
    "token_id":"6",
    "note":"",
    "fee":"0.00006531"
},

  "Response": {
    "code": 0,
    "msg": "SUCCESS",
    "result": {
      "tx_id": "20220908092725348944"
    }
  }
}
```

##### 请求参数

|      参数       | 数据类型 | 说明                                  |必要 |
| :-------------: | :------: | :------------------------------------ |:--- |
|     request_id     |   String | 唯一请求id                          |是|
|     wallet_id     |   Integer | 钱包id                          |是|
|     chain  |   String | 当前只支持Ethereum，固定值Ethereum                        |是|
|     to_address  |   String | to地址                        |是|
|     contract_address  |   String | 合约地址                       |是|
|     token_id  |   String | token id                        |是|
|     note  |   String | 备注                        |否|
|     fee  |   String | 手续费                        |是|


##### 响应参数

| 参数 | 数据类型 | 说明                                                                                              |
| :--: | :------: | :------------------------------------------------------------------------------------------------ |
| code |   long   |0说明成功发起绑定申请，1代表发起绑定申请失败|
| msg  |  String  | 返回描述                                                                                          |
| tx_id  |  String  | 事务id                                                                                          |

## 8. 错误代码

| code  |          Description             |
| :--------: | :------------------------------ |
|   0    | SUCCESS                                                      |
| 106001 | The request parameter format is incorrect.                   |
| 106002 | Unauthorized API request.                                    |
| 106003 | The service is temporarily unavailable.                      |
| 106004 | The timestamp has expired.                                   |
| 106005 | The IP address is incorrect.                                 |
| 106006 | Signature verification error.                                |
| 106008 | Data encryption error.                                       |
| 106009 | Data decryption error.                                       |
| 106011 | Service call exception.                                      |
| 106012 | The passphrase is incorrect.                                 |
| 106013 | The timestamp format is incorrect.                           |
| 106015 | API key null.                                                |
| 106016 | The amount parameter is in the wrong format.                 |
| 106018 | This wallet is currently frozen.                             |
| 106019 | The withdrawal amount is incorrect.                          |
| 106020 | The requested withdrawal amount exceeds the decimal limit.   |
| 106021 | The requested amount exceeds the maximum withdrawal limit defined in the risk control strategy. |
| 106022 | Authorization parameter format is incorrect.                 |
| 106023 | The requested information under this respective wallet has not been found. |
| 106024 | Remarks only supports alphanumeric input with a limited length of 1-32 digits. |
| 106025 | HD address quantity exceeds the maximum limit.               |
| 106026 | The API call frequency has exceeded its limit.               |
| 106027 | The API is being requested too frequent.                     |
| 106028 | Unable to procceed due to duplicate API request.             |
| 106029 | Coin type could not be found.                                |
| 106030 | The requested data has not been found.                       |
| 106031 | You do not have permission for this action.                  |
| 106032 | Database operation error.                                    |
| 106033 | Address status is abnormal.                                  |
| 106034 | Unable to fetch coin price.                                  |
| 106035 | The current balance is below the network fee amount required for this transaction. |
| 106036 | The requested amount exceeds the current available balance.  |
| 106037 | Failed to query fee rate.                                    |
| 106038 | The transaction fee is insufficient.                         |
| 106039 | Error detected with related address data.                    |
| 106040 | The requested  amount and transaction fee exceeds the available balance.|
| 106041 | From_address parameter is missing.                  |
| 106042 | Address format is incorrect.                  |
| 106043 | Duplicate withdrawal request.                  |
| 106044 | The default withdrawal strategy settings are missing.                  |
| 106045 | Coin type not supported.                 |
| 106046 | Transaction fee is below the required threshold.                  |
| 106047 | Transaction fee exceeds the maximum threshold.                  |
| 106048 | The requested wallet does not exist.                  |
| 106049 | Insufficient balance.                 |
| 106050 | The transaction cannot be cancelled.                 |
| 106051 | The requested amount exceeds the risk control strategy settings transaction limit per transaction.                  |
| 106052 | The requested amount exceeds the risk control strategy settings hourly transaction limit.                  |
| 106053 | The requested amount exceeds the risk control strategy settings daily transaction limit.                  |
| 106054 | The requested warm wallet is frozen.                  |
| 106055 | The requested warm wallet does not exist.                 |
| 106056 | Warm wallet nodes are abnormal.                 |
| 106057 | Unable to detect any withdrawable balance and fee balance.                 |
| 106058 | Unable to proceed with this action due to unresolved transactions in this wallet .                  |
| 106059 | The transaction fee amount format is incorrect.                 |
| 106060 | The recipient address has not been whitelisted to receive the respective token.                  |
| 106061 | Associate warm wallet address error.                  |
| 106062 | Address could not be found.                  |
| 106063 | Coin type duplicate association error.                  |
| 106064 | The collective address is not associated to the HTS coin.                  |
| 106065 | HTS token has already been associated to this address.                  |
| 106066 | Input parameter data duplication error.                  |
| 106067 | The HTS coin has not been defined.               |
| 106068 | The number of HBAR addresses cannot exceed 1.               |
| 106069 | The corresponding address and currency are waiting to be connected.               |
| 106070 | The beneficiary address is the same as the originating address.               |
| 106071 | Param error.               |
| 106072 | The number of ETH addresses cannot exceed 500.               |
| 106073 | The number addresses cannot exceed 5.               |
| 106074 | Master address handling fee is insufficient to support full collection transactions.               |
| 106075 | There are no addresses need to collect.               |




