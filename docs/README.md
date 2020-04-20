doc

# 概览

欢迎查看 Abit API 文档。 我们提供完整的 REST、Websocket API 以满足您的程序交易需求。您可以在    上找到每个连接选项的示例代码。

# REST API

## 身份验证

API的身份验证通过Header上的参数进行加密验证。

具体加密算法为MD5加密。  待加密字段为"(Body)+(secret_key)+(ts)"，其中()+等符号不计入加密字符串。

Body为每个接口所要求的参数列表，转换为json字符串形式。

secret_key为向系统申请的个人账户加密私钥。

ts为utc时间戳，单位为毫秒。

以下为Header中的身份验证相关字段

|    参数名    | 参数类型 |          参数描述          |
| :----------: | :------: | :------------------------: |
| Ex-Accesskey |  String  |          加密公钥          |
|    EX-Ts     |   Long   |   utc时间戳，单位为毫秒    |
|   EX-Sign    |  String  | 通过加密算法得到的加密签名 |

## 合约公共信息

对于合约公共信息请求的EndPoint地址为：https://host.com/

### 获取指数k线数据-GET

请求Url: swap/indexkline

请求方式:GET

参数列表:

|   参数名   |  参数类型  |       参数示例        |                           参数描述                           |
| :--------: | :--------: | :-------------------: | :----------------------------------------------------------: |
|  indexID   |   String   |          11           |                            指数id                            |
| startTime  | 1533686400 | 获取k线数据的起始时间 |                                                              |
|  endTime   | 1533688400 | 获取k线数据的结束时间 |                                                              |
|    unit    |    Int     |           5           | 选择的时间跨度的长度，在示例中表示为5M，即5分钟为每根蜡烛图的时间跨度 |
| resolution |   String   |           M           |         表示获取K线数据的频率类型,M:分钟,H:小时;D:天         |

curl请求示例:

```curl
curl --location --request GET 'https://host.com/swap/indexkline?indexID=11&startTime=1533686400&endTime=1533688400&unit=5&resolution=M'
```

Response：

```
{
    "errno": "OK",
    "message": "Success",
    "data": [
        {
            "low": "130", // 最低价
            "high": "130", // 最高价
            "open": "130", // 开盘价
            "close": "130", // 收盘价
            "last_px": "130", // 最后一次交易价
            "avg_px": "130", // 均价
            "qty": "0", // 交易量
            "timestamp": 1532610000, // 时间戳,单位秒
            "change_rate": "0", // 涨跌幅比例
            "change_value": "0" // 涨跌幅值
        }
    ]
}
```

### 获取合约交易记录

请求Url: swap/trades

请求方式:GET

参数列表:

|    参数名    | 参数类型 | 参数示例 |                参数描述                |
| :----------: | :------: | :------: | :------------------------------------: |
| instrumentID |  String  |    11    | 合约id，具体可见获取合约交易对详情接口 |

curl请求示例:

```curl
curl --location --request GET 'https://host.com/swap/trades?instrumentID=1'
```

Response：

```
{
    "errno": "OK",
    "message": "Success",
    "data": {
        // 交易记录列表,按创建时间由近及远排序
        "trades": [
            {
                "oid": 10116361,        // taker order id taker订单的id
                "tid": 10116363,        // trade id
                "instrument_id": 1,     // 合约ID
                "px": "16",   					// 成交价
                "qty": "10",     				// 成交的合约张数
                "make_fee": "0.04",   	// make fee
                "take_fee": "0.12",   	// take fee
                "created_at": null,   	// 创建时间
                "side": 5,             	// 订单交易方向，字段表示两笔订单成交所得到的交易记录的性质，
                																	1 //买单为taker,两笔订单方向：开多买 开空卖
                																	2 //买单为taker,两笔订单方向：开多买 平多卖
                                                  3 //买单为taker,两笔订单方向：平空买 开空卖
                                                  4 //买单为taker,两笔订单方向：平空买 平多卖
                                                  5 //卖单为taker,两笔订单方向：开空卖 开多买
                                                  6 //卖单为taker,两笔订单方向：开空卖 平空买
                                                  7 //卖单为taker,两笔订单方向：平多卖 开多买
                                                  8 //卖单为taker,两笔订单方向：平多卖 平空买
                "change": "0"    				// 对行情的影响
                										如本次交易前的最新交易价是10,本次交易的交易价是11,则change为"1"
            }
        ]
    }

```



### 获取合约仓位的自动减仓排序表

请求Url: swap/pnls

请求方式:GET

参数列表:

|    参数名    | 参数类型 | 参数示例 |                参数描述                |
| :----------: | :------: | :------: | :------------------------------------: |
| instrumentID |  String  |    11    | 合约id，具体可见获取合约交易对详情接口 |

curl请求示例:

```curl
curl --location --request GET 'https://host.com/swap/pnls?instrumentID=1'
```

Response：

```
{
    "errno": "OK",
    "message": "Success",
    "data": {
// 合约自动减仓排序表
"pnls": [
{
"instrument_id": 1,
// 合约看多仓位自动减仓排序表
"long_pnls": [
    {
        // 该分位的仓位盈利范围min~max
        "min_pnl": "-0.0379107725788900979",
        "max_pnl": "-0.0103298131866111136",
        "quan_tile": 40 // 五分位值,分别有20,40,60,80,100,quan_tile值越大,说明仓位发生自动减仓的可能													性越大.可以用这些值表示自动减仓警示灯的个数,20:1个灯,
        																											40:2个灯,
        																											60:3个灯,
        																											80:4个灯,
        																											100:5个灯
    },
    {
        "min_pnl": "-0.0103298131866111136",
        "max_pnl": "0",
        "quan_tile": 60 // 五分位值
    }
],
 // 合约看空仓位自动减仓排序表
"short_pnls": [
    {
        "min_pnl": "-49962.3980439671407168788",
        "max_pnl": "0",
        "quan_tile": 20
    }
]
            }
        ]
    }
}
```

### 获取指数列表

请求Url: swap/indexes

请求方式:GET

参数列表:无

curl请求示例:

```curl
curl --location --request GET 'https://host.com/swap/indexes'
```

Response：

```
{
    "errno": "OK",
    "message": "Success",
    "data": [
        {
            "index_id": 1, // 指数ID
            "name": "BTC", // 指数名称
            "base_coin": "BTC", // 指数基础币
            "quote_coin": "USDT", // 指数计价币
            "brief_en": "", // 英文简称
            "brief_zh": "", // 中文简称
            "px_unit": "0.000001", // 价格精度
            "created_at": "2018-07-30T16:04:08Z",
            // 指数成员
            "members": [
                {
                    "index_id": 11,
                    "coin_code": "BTC",
                    "weight": "1",
                    "market": [
                        {
                            "market_name": "Bitstamp",
                            "weight": "1"
                        },
                        {
                            "market_name": "Coinbase.Pro",
                            "weight": "1"
                        }
                    ]
                }
            ],
            // 合约列表
            "instruments": [
                {
                    // 跟指数关联的合约对象
                    // 数据与获取合约接口的结构一致
                },
                {

                }
            ]
        }
    ]
}
```

### 获取单个币种合约深度

请求Url: swap/depth

请求方式:GET

参数列表:

|    参数名    | 参数类型 | 参数示例 |                参数描述                |
| :----------: | :------: | :------: | :------------------------------------: |
| instrumentID |  String  |    11    | 合约id，具体可见获取合约交易对详情接口 |
|    count     |   Int    |    10    |   合约深度档数大小，不传表示获取全部   |

curl请求示例:

```curl
curl --location --request GET 'https://host.com/swap/depth?instrumentID=1&count=10'
```

Response：

```
{
    "errno": "OK",
    "message": "Success",
    "data": {
         // 卖盘,按价格由小到大排序
        "asks": [
            [
                80000, // key
                "8",   // 价格
                "1",   // 量
                0      // 0:表示用户订单,1:表示包含系统订单
            ]...
        ],
        // 买盘,按价格由大到小排序
        "bids": [
            [
                74000, // key
                "7.4", // 价格
                "1",   // 量
                0
            ]...
        ]
    }
}
```

### 

### 获取合约信息

请求Url: swap/instruments

请求方式:GET

参数列表:

|    参数名    | 参数类型 | 参数示例 |                参数描述                |
| :----------: | :------: | :------: | :------------------------------------: |
| instrumentID |  String  |    11    | 合约id，具体可见获取合约交易对详情接口 |

curl请求示例:

```curl
curl --location --request GET 'https://host.com/swap/instruments?instrumentID=11'
```

Response：

```
{
    "errno": "OK",
    "message": "Success",
    "data": {
        "instruments": [
            {
                "instrument_id": 7,                         //合约id
                "index_id": 16,															//指数id
                "symbol": "BCHUSDT",												//合约名称
                "name_zh": "BCHUSDT永续合约",                //合约中文名称
                "name_en": "BCHUSDT SWAP",									//合约英文名称
                "base_coin": "BCH",													//基础币种
                "quote_coin": "USDT",												//计价币种
                "margin_coin": "USDT",											//保证金币种
                "is_reverse": false,                        //是否是反向合约，usdt合约为正向合																																约，币本位合约为反向合约
                "market_name": "",                          //
                "face_value": "0.001",                      //单张合约面值，以基础币种为单位
                "begin_at": "2018-09-29T04:00:00Z",         //
                "settle_at": "2019-09-08T12:00:00Z",        //
                "settlement_interval": 28800,               //
                "min_leverage": "1.0",                      //最小支持杠杆
                "max_leverage": "50.0",                     //最大支持杠杆
                "position_type": 3,                       	//支持的仓位类型,1:全仓,2:逐仓,3:都																																支持
                "px_unit": "0.01",                       		//价格精度
                "qty_unit": "1",                       			//数量精度
                "value_unit": "0.0001",                     //价值精度
                "min_qty": "1",                     			  //单笔订单最小数量
                "max_qty": "100000",                        //单笔订单最大数量
                "underweight_type": 1,                      //穿仓补偿方式 1:ADL方式,2:盈利均摊																																						方式
                
                
                "status": 3,                      				  //合约状态
                																								1  // 审批中,系统内部使用
																																2  // 测试中,系统内部使用
																																3  // 可用,正在撮合的合约
																																4  // 暂停,合约可见,撮合暂停
																																5  // 交割中,期货合约到期后,暂停																																			撮合,开始结算
																																6  // 交割完成,结算完成
																																7  // 下线,合约生命周期结束
                "area": 2,                       						//1:USDT区,2:主区,3:创新区,4:模拟区
                "created_at": "2018-09-29T10:02:42Z",       //创建时间
                "depth_round": "1.001",                     //深度边框系数
                "base_coin_zh": "比特现金",                  //基础币种中文名称
                "base_coin_en": "BCH",                      //基础币种英文缩写
                "max_funding_rate": "0",                    //最大资金费率
                "min_funding_rate": "0",                    //最小资金费率
                "risk_limit_base": "100000",                //风险限额基础
                "risk_limit_step": "50000",                 //风险限额补偿
                "mmr": "0.005",                             //基本维持保证金率
                "imr": "0.01",                              //基本开仓保证金率
                "maker_fee_ratio": "0.00025",               //maker费率
                "taker_fee_ratio": "0.00075",               //taker费率
                "settle_fee_ratio": "0",                    //交割手续费率
                "plan_order_price_min_scope": "0",          //计划委托最小价格范围
                "plan_order_price_max_scope": "0",          //计划委托最大价格范围,如果为0,表示该																																合约不支持计划委托
                "plan_order_max_count": 0,                  //单用户计划委托最大数量
                "plan_order_min_life_cycle": 0,             //计划委托最小生命周期
                "plan_order_max_life_cycle": 0              //计划委托最大生命周期
            }
        ]
    }
}


```

### 

### 

### 获取合约信息

请求Url    :swap/fundingrate

请求方式  :GET

参数列表  :

|    参数名    | 参数类型 | 参数示例 |                参数描述                |
| :----------: | :------: | :------: | :------------------------------------: |
| instrumentID |  String  |    11    | 合约id，具体可见获取合约交易对详情接口 |

curl请求示例:

```curl
curl --location --request GET 'https://host.com/swap/fundingrate?instrumentID=11'
```

Response：

```
{
    "errno": "OK",
    "message": "Success",
    "data": [
        {
            "rate": "-0.0060483184843194741", // 资金费率
            "timestamp": 1534320000 // 时间戳
        },
        {
            "rate": "0.1041766553400505803",
            "timestamp": 1534291200
        }
    ]
}
```

### 

### 获取合约k线数据

请求Url    :swap/kline

请求方式  :GET

参数列表  :

|   参数名   | 参数类型 |  参数示例  |                           参数描述                           |
| :--------: | :------: | :--------: | :----------------------------------------------------------: |
|  indexID   |  String  |     11     |                            合约id                            |
| startTime  |    ts    | 1533686400 |                    获取k线数据的起始时间                     |
|  endTime   |    ts    | 1533688400 |                    获取k线数据的结束时间                     |
|    unit    |   Int    |     5      | 选择的时间跨度的长度，在示例中表示为5M，即5分钟为每根蜡烛图的时间跨度 |
| resolution |  String  |     M      |         表示获取K线数据的频率类型,M:分钟,H:小时;D:天         |



curl请求示例:

```curl
https://host.com/swap/kline?instrumentID=1&startTime=1532610072&endTime=1532656524&unit=5&resolution=M

```

Response：

```
ENVIRONMENT LAYOUT 
LANGUAGE 
Public
{
    "errno": "OK",
    "message": "Success",
    "data": [
        {
            "low": "130", 				// 最低价
            "high": "130", 				// 最高价
            "open": "130", 				// 开盘价
            "close": "130", 			// 收盘价
            "last_px": "130", 		// 最后一次交易价
            "avg_px": "130", 			// 均价
            "qty": "10", 					// 交易量,单位张数
            "base_coin_qty":"163.9", 				// 基础币的交易量(币值对前面的币为基础币)
            "quote_coin_qty":"34555.3812", 	// 计价币的交易量(币值对后面的币为计价币)
            "timestamp": 1532610000, 				// 时间戳,单位秒
            "change_rate": "0", 						// 涨跌幅比例
            "change_value": "0" 						// 涨跌幅值
        }
    ]
}
```

### 

### 获取合约ticker数据

请求Url    :swap/tickers

请求方式  :GET

参数列表  :

|    参数名    | 参数类型 | 参数示例 | 参数描述 |
| :----------: | :------: | :------: | :------: |
| instrumentID |  String  |    11    |  合约id  |



curl请求示例:

```curl
curl --location --request GET 'https://host.com/swap/tickers?instrumentID=1'
```

Response：

```
{
    "errno": "OK",
    "message": "Success",
    "data": {
        "tickers": [
            {
                "last_px": "6277.5", 					// 最后成交价
                "open": "6074.5", 						// 当天的开盘价
                "close": "6277.5", 						// 当前的收盘价
                "low": "6261.3", 							// 当天的最低价
                "high": "6279", 							// 当天的最高价
                "avg_px": "6274.67",					// 均价
                "last_qty": "20000", 					// 最后一笔交易量,单位张数,如果要显示btc的交易量,																											需要last_qty乘以合约的大小
                "qty24": "45462", 						// 24小时总量,单位张数
                "base_coin_qty":"163.9", 			// 基础币的交易量(币值对前面的币为基础币)
                "quote_coin_qty":"34555.381", // 计价币的交易量(币值对后面的币为计价币)
                "timestamp": 1534315695, 			// 时间戳
                "change_rate": "0.033", 			// 涨幅比例
                "change_value": "203", 				// 涨幅值
                "instrument_id": 1, 					// 合约id
                "position_size": "374266", 		// 未平仓位量
                "qty_day": "45270", 					// 当天交易量
                "amount24":"28520.77363", 		// 24小时交易额
                "index_px": "6406.53", 				// 指数价格
                "fair_basis": "0.000000690", 	// 基差率
                "fair_value": "0.00442673", 	// 基差
                "fair_px": "6406.5344267", 		// 标记价
                "rate": {
                    "quote_rate": "0.0003", 	// 计价币借贷利率
                    "base_rate": "0.0006", 		// 基础币借贷利率
                    "interest_rate": "-0.00009999" // 利率
                },
                "premium_index": "-0.0309530479798782534", 	// 溢价指数
                "funding_rate": "0.0001", 									// 当前资金费率
                "next_funding_rate": "-0.0304530", 					// 下一个预计资金费率
                "next_funding_at": "2018-08-15T08:00:01Z" 	// 下一个结算时间(UTC时间)
            }
        ]
    }
}
```

## 合约交易接口

在此类别中，所有接口都需要进行身份验证。

### 撤销合约订单

请求Url    :swap/cancelOrders

请求方式  :POST

参数列表  :

|    参数名    | 参数类型 | 参数示例  | 参数描述 |
| :----------: | :------: | :-------: | :------: |
| instrumentID |  String  |    11     |  合约id  |
|    Orders    | 整型列表 | [1,2,3,4] |          |

POST

curl请求示例:

```curl
curl --location --request GET 'https://host.com/swap/tickers?instrumentID=1'
```

Response：

```
{
    "errno": "OK",
    "message": "Success",
    "data": {
        "tickers": [
            {
                "last_px": "6277.5", 					// 最后成交价
                "open": "6074.5", 						// 当天的开盘价
                "close": "6277.5", 						// 当前的收盘价
                "low": "6261.3", 							// 当天的最低价
                "high": "6279", 							// 当天的最高价
                "avg_px": "6274.67",					// 均价
                "last_qty": "20000", 					// 最后一笔交易量,单位张数,如果要显示btc的交易量,																											需要last_qty乘以合约的大小
                "qty24": "45462", 						// 24小时总量,单位张数
                "base_coin_qty":"163.9", 			// 基础币的交易量(币值对前面的币为基础币)
                "quote_coin_qty":"34555.381", // 计价币的交易量(币值对后面的币为计价币)
                "timestamp": 1534315695, 			// 时间戳
                "change_rate": "0.033", 			// 涨幅比例
                "change_value": "203", 				// 涨幅值
                "instrument_id": 1, 					// 合约id
                "position_size": "374266", 		// 未平仓位量
                "qty_day": "45270", 					// 当天交易量
                "amount24":"28520.77363", 		// 24小时交易额
                "index_px": "6406.53", 				// 指数价格
                "fair_basis": "0.000000690", 	// 基差率
                "fair_value": "0.00442673", 	// 基差
                "fair_px": "6406.5344267", 		// 标记价
                "rate": {
                    "quote_rate": "0.0003", 	// 计价币借贷利率
                    "base_rate": "0.0006", 		// 基础币借贷利率
                    "interest_rate": "-0.00009999" // 利率
                },
                "premium_index": "-0.0309530479798782534", 	// 溢价指数
                "funding_rate": "0.0001", 									// 当前资金费率
                "next_funding_rate": "-0.0304530", 					// 下一个预计资金费率
                "next_funding_at": "2018-08-15T08:00:01Z" 	// 下一个结算时间(UTC时间)
            }
        ]
    }
}
```

## 



## 现货交易