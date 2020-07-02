# CUSTODIAN OPENAPI

## 1.入门指引

API旨在帮助用户快速、高效地将热钱包功能整合到自己的应用当中。接口是提供服务的基础，API分为查询、提币两种类型。开发者可以根据自身需求建立不同权限的API，并利用API进行热钱包账户信息查询或者提现。



## 2.使用流程
步骤：开发者如需使用API ，请先在Custodian系统申请API key等信息,然后根据此文档详情进行开发交易，使用过程中如有问题或者建议请及时反馈

## 3.REST API
REST，即Representational State Transfer的缩写，是目前最流行的一种互联网软件架构。它结构清晰、符合标准、易于理解、扩展方便，正得到越来越多网站的采用。其优点如下：

在RESTful架构中，每一个URL代表一种资源；
客户端和服务器之间，传递这种资源的某种表现层；
客户端通过四个HTTP指令，对服务器端资源进行操作，实现“表现层状态转化”。

## 4.标准规范
时间戳
除非另外指定，API中的所有时间戳均以毫秒为单位返回时间戳，返回UTC时间.

例子

1573574713991

数字

为了保持跨平台时精度的完整性，十进制数字作为字符串返回。建议您在发起请求时也将数字转换为字符串以避免截断和精度错误。

整数（如交易编号和顺序）不加引号。


## 5.接口类型
接口类型的细节分以下两个方面：

公共接口

公共接口可用于获取API服务器系统时间。公共请求无需认证即可调用。

私有接口

私有接口可用于热钱包账户管理。每个私有请求必须使用规范的验证形式进行签名。

私有接口需要使用您的API key进行验证。

## 6.验证
验证的细节分以下六个方面：

-  生成API Key

-  发起请求

-  签名

-  时间戳

-  ip白名单

-  获取服务器时间

###### 6.1生成API Key


在对任何请求进行签名之前，您必须通过Custodian系统创建一个API Key。创建API Key后，您将获得3个必须记住的信息：

API Key
Secret
Passphrase
API Key和Secret将由Custodian系统随机生成和提供，Passphrase将由您提供以确保API访问的安全性。Custodian系统将存储Passphrase加密后的哈希值进行验证，但如果您忘记Passphrase，则无法恢复，请您通过Custodian系统重新生成新的API Key。

###### 6.2发起请求

所有REST请求头都必须包含以下内容：

AUTHORIZATION  认证信息：ApiKey:request time stamp:signature(英文以冒号拼接).

CUSTODIAN-ACCESS-PASSPHRASE 此密码为api secret的加密信息,由客户自己保管,在调用方法时，传入.

所有请求都应该含有application/json类型内容，并且是有效的JSON。

###### 6.3签名

signature 签名串是对timestamp + method +API Key+ requestPath+queryString + body字符串(转成map后,key通过ASCII排序后对key和value进行拼接)，以及secret，使用HMAC SHA256方法加密，通过BASE64编码输出而得到的。

例如：sign=CryptoJS.enc.Base64.Stringify(CryptoJS.HmacSHA256(timestamp + 'GET' +'14db63d7f3614664ad1c71dd134a21dc'+ '/v1/api/account'+"?"+"coinType=ETH", secret))

###### 6.4签名参数说明
timestamp 为UTC时区Unix时间戳格式，精确到毫秒.
时间戳和服务器时间前后相差30秒以上的请求将被系统视为过期并拒绝。如果您认为服务器和API服务器之间存在较大的时间偏差，那么我们建议您使用获取服务器时间的接口来查询API服务器时间

method是请求方法，字母全部大写：GET/POST。

requestPath是请求接口路径。例如：/v1/api/account

queryString是url中的参数。例如：page_num=1&page_size=10&coin_type=BTC

body是指请求主体的字符串，如果请求没有主体(通常为GET请求)则body可省略.

#### 如何构造待签名的请求body string：

请求 body 需要按照 `ASCII` 码的顺序对参数名进行排序，以  `=` 拼接 key 和 value，并以 `&` 分割多个 key-value，转换成字符串。

例如请求 `body` 为：

```javascript
{
	"mnt_nd":"ZPG",
	"amount":190,
	"to_address":"AUol16ghiT9AtxRDtNeq3ovhWJ5iaY6iyd"
}
```

转换后为：

```text
amount=190&mnt_nd=ZPG&to_address=AUol16ghiT9AtxRDtNeq3ovhWJ5iaY6iyd
```
secret为用户申请API Key时所生成。例如：22582BD0CFF14C41EDBF1AB98506286D

###### 6.5 ip白名单验证
api接口在创建时必须设置了ip白名单，在后续的接口调用中，来源必须是白名单ip.
###### 6.6获取服务时间

获取API服务器的时间。此接口为公共接口，不需要身份验证。

HTTP请求

GET /v1/api/general/time

限速规则：20次/2s

返回参数

参数名	描述
timestamp	UTC时间戳的十进制秒数格式

- 示例


{

"timestamp": 1573574713991

}

###### 6.7调用API接口的测试例子
https://github.com/aixingjuele/custodian-sdk-java


## 7.API LIST

### 7.1Account

#### 7.1.1Query Account Summary

> 

```json
{

    "URL": "/v1/api/account",

    "Method": "GET",

    "Params": {},

    "Response": 
    {
        "code": 0,
        "msg": "SUCCESS",
        "result": 
        [
			{
            "address": "tb1q07r35czmvuhl28r93qgv02h6t030gcheqd34rs",
            "address_name": "Hot-Wallet-16-BTC",
            "coin_unique_name": "BTC",
            "coin_full_name": "Bitcoin",
            "coin_decimal": 8,
            "deposit_allowed": 1,
            "withdrawal_allowed": 1,
            "current_balance": "0.084800000000000000",
            "fee_coin": "BTC",
            "estimated_fee": "0.0001",
            "upper_limit": "10.000000000000000000",
            "lower_limit": "0.010000000000000000"
        }
          ]
   }
}
```

| Parameter  |  Type  |             Description             |
| :--------: | :----: | :------------------------------ |
|   address   | String | 地址  |
|   address_name   |  String   |  钱包名称  |
|   coin_unique_name |  String   |币种代号 |
|   coin_full_name |  String  |   币种名全称         |
|   coin_decimal |  Integer  |      币种余额小数点位数          |
|   deposit_allowed |  Integer  | 是否可以充值,0=不允许,1允许 |
|   withdrawal_allowed |  Integer  |是否可以提现,0=不允许,1允许  |
|   current_balance |  String  |当前余额 |
|   fee_coin |  String  | 手续费币种   |
|   estimated_fee |  String  | 预估手续费 |
|   upper_limit |  String  |   警报上限        |
|   lower_limit |  String  |  警报下限         |

#### 7.1.2 Query Account Detail

> 

```json
{

    "URL": "/v1/api/account/{coinType}",

    "Method": "GET",

    "Params": {},

    "Response": 
    {
        "code": 0,
        "msg": "SUCCESS",
        "result":
		{
        "address": "tb1q07r35czmvuhl28r93qgv02h6t030gcheqd34rs",
        "address_name": "Hot-Wallet-16-BTC",
        "coin_unique_name": "BTC",
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
        "hour_limit_amount": "1.000000000000000000"
    }
   }
}
```

| Parameter  |  Type  |             Description             |
| :--------: | :----: | :------------------------------ |
| coin_type | String | coin type(BTC,ETH,USDT-ERC20,USDC,CGT,WTC,REN,FTM,MATIC,XSGD-ERC20,BCH) |
|   address   | String | 钱包地址  |
|   address_name   | String | 钱包名称   |
|   coin_unique_name   | String | 币种代号   |
|   coin_full_name |  String  |   币种名称全称         |
|   coin_decimal |  Integer  |      币种余额小数点位数          |
|   deposit_allowed |  Integer  | 是否可以充值,0=不允许,1允许 |
|   withdrawal_allowed |  Integer  |是否可以提现,0=不允许,1允许  |
|   current_balance |  String  |当前余额 |
|   fee_coin |  String  | 手续费币种   |
|   estimated_fee |  String  | 预估手续费 |
|   upper_limit |  String  |   警报上限        |
|   lower_limit |  String  |  警报下限         |
|   limit_per_deal |  String   | 单笔交易限额 |
|   day_limit_amount |  String   |日交易剩余额度|
|   hour_limit_amount |  String   | 小时内交易剩余额度|




#### 7.1.3 Query Deposit Address

> 

```json
{

    "URL": "/v1/api/account/deposit-address/{coinType}",

    "Method": "GET",

    "Params": {},

    "Response": 
    {

        "code": 0,

        "msg": "SUCCESS",

        "result":
        {
            "coin_unique_name":"",
            
            "deposit_address":""
        }
   }
}
```
| Parameter  |  Type  |             Description             |
| :--------: | :----: | :------------------------------ |
|   coin_unique_name   | String | 币种代号  |
|   deposit_address   | String | 充值地址   |


#### 7.1.4 Verify Deposit Address

> 

```json
{

    "URL": "/v1/api/account/verify-deposit-address/{coinType}/{address}",

    "Method": "GET",

    "Params": {},

    "Response": 
    {

        "code": 0,

        "msg": "SUCCESS",

        "result": true
   }
}
```
| Parameter  |  Type  |             Description             |
| :--------: | :----: | :------------------------------ |
|   result   | Boolean | true为成功,false为失败  |







#### 7.1.5 根据主地址获取热钱包账户HD地址信息

> 

```json
{

    "URL": "/v1/api/account/list-hdaddress",

    "Method": "POST",

    "Params": {
		  "master_address": "0x29ff770a1edaa03736336dc84440e0d42a77ee5c",
		  "page_num": 1,
		  "page_size": 10
		},

    "Response": 
    {
	  "code": 0,
	  "msg": "string",
	  "result": {
		"list": [
		  {
			"address": "string",
			"address_name": "string",
			"coin_decimal": 0,
			"coin_full_name": "string",
			"coin_unique_name": "string",
			"current_balance": "string",
			"deposit_allowed": 0,
			"estimated_fee": "string",
			"fee_coin": "string", 
			"withdrawal_allowed": 0,
			"lower_limit": "string",
			"upper_limit": "string"
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

######   request
| Parameter  |  Type  |             Description             |
| :--------: | :----: | :------------------------------ |
|   master_address   | String | 热钱包主地址  |
|   page_num   | String | 页数   |
|   page_size   | String | 页大小   |

######   response
| Parameter  |  Type  |             Description             |
| :--------: | :----: | :------------------------------ |
|   address   | String | hd地址  |
|   address_name   | String | hd地址名   |
|   coin_decimal   | Integer | 小数点位数   |
|   coin_full_name   | String | 币种全称   |
|   coin_unique_name   | String |币种代号   |
|   current_balance   | String | 余额   |
|   deposit_allowed   | Integer | 是否可以充值,0=不允许,1允许   |
|   estimated_fee   | String | 预估手续费   |
|   fee_coin   | String | 手续费币种   |
|   withdrawal_allowed   | Integer | 是否可以提现,0=不允许,1允许   |
|   lower_limit   | String | 警报下限   |
|   upper_limit   | String | 警报上限   |


### 7.2 Transaction Info

#### 7.2.1 Query Transaction List

> defualt limit: 10

```json
{

    "URL": "/v1/api/list-trans",

    "Method": "POST",

    "Params": {
        "tx_type": "1",
        "tx_id": "123",
        "coin_type": "BTC",
        "page_num":1,
        "page_size":10，
    },

    "Response": 
        {

            "code": 0,

            "msg": "SUCCESS",

            "result":{

                "total":120,

                "records":[
                {
                "wallet_name":"",
                "coin_unique_name":"",
                "coin_full_name":"",
                "coin_decimal":"",
                "address":"",
                "source_address":"",
                "tx_type":"交易类型（1=提币,2=主地址入金,3=子地址入金,4=归集交易）",
                "amount":100,
                "tx_id":"",
                "tx_hash":"",
                "tx_status":"",
                "create_time":"",
                "confirm_time":"",
                "fee_coin":"",
                "fee":""
            }
            ]
        }

        }
    

}
```
######   request
| Parameter  |  Type  |             Description             |
| :--------: | :----: | :------------------------------ |
|   tx_type   | String | 1=出金，2=入金 |
|   coin_type   | String | 币种代号  |
|   tx_id   | String | 交易单号   |
|   page_num   | int | 页数   |
|   page_size   | int | 页大小 |

######   response
| Parameter  |  Type  |             Description             |
| :--------: | :----: | :------------------------------ |
|   wallet_name   | String | 钱包名字  |
|   coin_unique_name   | String |币种代号   |
|   coin_full_name   | String | 币种全称   |
|   coin_decimal   | Integer | 小数点位数   |
|   address   | String | 地址 |
|   source_address   | String | 源地址 |
|   tx_type   | String | 交易类型（1=提币,2=主地址入金,3=子地址入金,4=归集交易） |
|   amount   | String | 交易金额|
|   tx_id   | String | 交易ID |
|   tx_hash   | String | 区块链交易hash |
|   tx_status   | String | 交易状态 0待确认、1成功、2失败,3审核|
|   create_time   | String | 交易创建时间 |
|   confirm_time   | String | 交易成功(失败)时间 |
|   fee_coin   | String | 手续费币种   |
|   fee   | String | 手续费  |

#### 7.2.2 Query Transaction Detail

> 

```json
{

    "URL": "/v1/api/trans/{tx_id}",

    "Method": "GET",

    "Params": {},

    "Response": 
    {
        "code": 0,
        "msg": "SUCCESS",
        "result": 
			 {
					"wallet_name":"",
					"coin_unique_name":"",
					"coin_full_name":"",
					"coin_decimal":1,
					"address":"",
					"source_address":"",
					"tx_type":"交易类型（1=提币,2=主地址入金,3=子地址入金,4=归集交易）",
					"amount":"100",
					"tx_id":"",
					"tx_hash":"",
					"tx_status":"",
					"create_time":"",
					"confirm_time":"",
					"fee_coin":"",
					"fee":""
				}
    }
}
```

| Parameter  |  Type  |             Description             |
| :--------: | :----: | :------------------------------ |
|   wallet_name   | String | 钱包名字  |
|   coin_unique_name   | String |币种代号   |
|   coin_full_name   | String | 币种全称   |
|   coin_decimal   | Integer | 小数点位数   |
|   address   | String | 地址 |
|   source_address   | String | 源地址 |
|   tx_type   | String | 交易类型（1=提币,2=主地址入金,3=子地址入金,4=归集交易） |
|   amount   | String | 交易金额|
|   tx_id   | String | 交易ID |
|   tx_hash   | String | 区块链交易hash |
|   tx_status   | String | 交易状态 0待确认、1成功、2失败,3审核|
|   create_time   | String | 交易创建时间 |
|   confirm_time   | String | 交易成功(失败)时间 |
|   fee_coin   | String | 手续费币种   |
|   fee   | String | 手续费  |


### 7.3 Withdrawal

#### 7.3.1 Submit Withdrawal Application

> 

```json
{

    "URL": "/v1/api/trans/withdrawal",

    "Method": "POST",

    "Params": {
        "request_id": "",
        "coin_type": "",
        "to_address": "",
        "tx_amount":99.99,
        "note":""，
   },

    "Response": {

        "code": 0,

        "msg": "SUCCESS",

        "result": {

            "tx_id":""

        }

    }

}
```


######   request
| Parameter  |  Type  |             Description             |
| :--------: | :----: | :------------------------------ |
|   request_id   | String | 单次请求id，每次请求不得重复  |
|   coin_type   | String | 币种代号  |
|   to_address   | String | 入金地址   |
|   tx_amount   | String | 提取金额   |
|   note   | String | 备注 |

######   response
| Parameter  |  Type  |             Description             |
| :--------: | :----: | :------------------------------ |
|   tx_id   | String | 请求成功则返回交易单号  |

### 7.4 HD子地址管理

#### 7.4.1 HD子地址创建

> 

```json
{

    "URL": "/v1/api/hd-address",

    "Method": "POST",

    "Params": {
	  "address": "0x29ff770a1edaa03736336dc84440e0d42a77ee5c", 
	  "count": 2,
	  "remarks": [
		"hd-name-1",
		"hd-name-2"
	  ]
	},

    "Response": {

        "code": 0,

        "msg": "SUCCESS",

        "result": {
            "0x0797bb11a5bbe3692a042cfa57873a38b21ab96a":"hd-name-1",#返回格式为 地址:地址名
			"0x1673332f5a8b894e834e1934657c44d2f47443b6":"hd-name-2",

        }

    }

}
```

######   request
| Parameter  |  Type  |             Description             |
| :--------: | :----: | :------------------------------ |
|   address   | String | hd主地址 |
|   count   | Integer | 需要创建hd地址的个数，单次最大为5个  |
|   remarks   | String[] | 子地址名称，count为5，就需要传入5个hd子地址名称   |


######   response
| Parameter  |  Type  |             Description             |
| :--------: | :----: | :------------------------------ |
|   body   | map | 格式为 地址:地址名  |


#### 7.4.2 HD子地址名称修改

> 

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

######   request
| Parameter  |  Type  |             Description             |
| :--------: | :----: | :------------------------------ |
|   address   | String | hd主地址 |
|   remark   | String | 修改后的子地址名称 |


### 7.5 通知回调接口


#### 7.5.1 出入金通知回调接口
######url参数{notice-type}为通知类型，目前transaction-notice为交易类通知


```json
{

    "URL": "host:port/{notice-type}",

    "Method": "POST",

    "Params":
			{
			"address": "0x29d912b930c2288a2a54ad0446b3b53dce2718d5",
			"amount": "0.099000000000001000",
			"coinType": "ETH",
			"confirmedBlocks": "6",
			"fee": "0.000999999999999",
			"hash": "0xc24b4b8ecec4de14aed9523591de867fff965517d02d66a3f1d110ef5c4dcbd4",
			"status": "1",
			"txId": "20200103025302304697",
			"txType": "Withdraw" 
			}
    "Response": {
        "code": 0,

        "msg": "SUCCESS"

    }

}
```

######   request

| Parameter  |  Type  |             Description             |
| :--------: | :----: | :------------------------------ |
| notice-type | String | 通知类型: transaction-notice |
|   address   | String | 交易地址 |
|   amount   | String | 交易金额  |
|   coinType   | String | 交易币种  |
|   confirmedBlocks   | String| 区块确认数   |
|   fee   | String | 手续费  |
|   hash   | String | 链上hash |
|   status   | String | 交易状态:0=等待,1=成功,2失败   |
|  txId| String | 交易id,同一交易多次通知,id唯一 |
|   txType   | String | 交易类型 Withdraw=出金,Deposit=入金  |


######   response
| Parameter  |  Type  |             Description             |
| :--------: | :----: | :------------------------------ |
|   code   | long | OpenAPI期望收到0为通知成功，如果收到其他返回码或没有任何返回会继续通知，直到通知次数到最大通知次数  |
|   msg   | String | 返回描述  |

## 8.限速规则（apiKey+请求路径）
-  查询类API  20次/2s 
-  提币类API  10次/2s




## 9.Error Code


| code  |          Description             |
| :--------: | :------------------------------ |
|   0   |  SUCCESS|
|   106001   |The request parameter error|
|   106002   |  Unauthorized|
|   106003   |  The service is temporarily unavailable|
|   106004   |  Interface Timeout|
|   106005   |  Ip address check error|
|   106006   |  Signature verification error|
|   106008   |  Data encryption error|
|   106009   |  Data decryption error|
|   106011   |  Service call exception|
|   106012   |  Passphrase error|
|   106013   |  Timestamp error|
|   106015   |  ApiKey error|
|   106016   |  The amount is in the wrong format.|
|   106018   |  The hot wallet is frozen|
|   106019   |  The amount of withdrawal is wrong|
|   106020   |  The amount of withdrawal is too long in decimal places|
|   106021   |  The transaction amount exceeds the upper limit of the single pen|
|   106022   |  Authorization format error|
|   106023   |  Address error|
|   106024  |  Address notes are 1-64 in length and only alphanumeric|
|   106025   |  HD address quantity exceeds|
|   106026   |  The frequency of the call exceeded the limit.|
|   106027   |  The request is too fast |
|   106028   |  The Same request received|
|   106029   |  Cointype not support.|




