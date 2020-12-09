AEX Websocket API 协议说明文档 （V3）
---


# 简介

### API 接入URL

```
wss://api.aex.zone/wsv3  
目前 Websocket API 只针对 api.aex.zone 域名提供。
建立连接成功后服务端会响应  {"cmd":0}  
```

### Websocket 连接处理过程

```
同一个账号，同一时间只能有一个连接存活，后面认证成功的连接会把其他连接断开。在未认证的情况下，同一个IP允许多有个连接，
比如：
  1）IP1 连接上websocket
  2）IP1 发起auth请求且成功
  3）IP2 连接上websocket
  4）IP2 发起auth请求且成功
  5）IP1 上已auth成功的连接会被服务端主动断开连接
```

# 目录
+ [心跳消息](#心跳消息)
+ [请求说明](#请求说明)
+ [请求示例](#请求示例)
+ [协议命令字](#协议命令字)
+ [错误码](#错误码)
+ [协议请求/应答数据结构(json)](#协议请求应答数据结构)
   + [CMD: 1，最新公共成交数据](#cmd-1-最新公共成交数据)
   + [CMD: 2, K线](#cmd-2-K线)
   + [CMD: 3, 深度盘面](#cmd-3-深度盘面)
   + [CMD: 4, 24小时市场行情](#cmd-4-24小时市场行情)
   + [CMD: 6, 精简24小时行情](#cmd-6-精简24小时行情)
   + [CMD: 99, 登录](#cmd-99-登录)
   + [CMD: 100, 个人订单变化](#cmd-100-个人订单变化)
   + [CMD: 101, 个人最新成交](#cmd-101-个人最新成交)
+ [订单和成交记录的关联](#订单和成交记录的关联)
+ [交易策略的实现](#交易策略的实现)


# 心跳消息

```
当用户的Websocket客户端连接到AEX的Websocket服务器后，需定期（当前为20秒）向AEX发送ping消息：

ping

当AEX的Websocket客户端接收到此心跳消息后，会返回pong消息：

pong

当AEX服务器连续两次没有收到`ping`消息，服务器将主动断开与此客户端的连接。
当用户的Websocket客户端连续两次没有收到`pong`消息，应尝试断开重连。
```

# 请求说明

```
指令格式

{"cmd":"<cmd>","action":"<action>","parameter1":"value1","parameter2":"value2"}

其中cmd是各种频道的标识，action的取值有 sub--订阅；unsub--取消订阅；login--登录；logout--退出登录

成功响应格式
{"cmd":99,"action":"login","code":20000}
{"cmd":4,"action":"sub","code":20000}

失败响应格式

{"cmd":99,"action":"login","code":100003}
{"cmd":4,"action":"sub","code":10002}

cmd是请求的标识，action是请求的动作，成功和失败的区别主要在code上面。是否成功主要看
code的第一位是奇偶性，偶数为成功，奇数为失败；失败code在下面code集合有实际意思说明。
```

# 请求示例

```
订阅

{"cmd":4,"action":"sub","symbol":"btc_cnc"}

取消订阅

{"cmd":4,"action":"unsub","symbol":"btc_cnc"}

登录
{"cmd":99,"action":"login","key":"5f81d6e3cd3e96efa89d5297c1cec0c7","time":1594348186,"md5":"93a336226be823df3c7c50944df3eeb5"}

退出
{"cmd":99,"action":"logout"}

```

### 数据说明
```
分为个人数据，公共数据；个人数据需要登录认证。
请求参数symbol都支持多个订阅，用英文逗号连接。

公共数据包括：最新成交，K线，深度，24小时市场行情，精简24小时市场行情。
个人数据包括：个人订单变化，个人最新成交。

```

# 协议命令字
cmd|说明
----  |-----   
1     | 最新公共成交数据 
2     |  K线
3     |  深度盘面
4     |  24小时市场行情
6     |  精简24小时行情
99     |  登录认证
100     | 个人订单变化
101     | 个人最新成交


# 错误码
code  | 说明
-----  | ---------
100001 |key，time，md5 中有空值
100002 |time不是int类型
100003 |请求时间和服务器时间戳相差超过30s
100004 |key值不合法,或含有非法字符
100005 |不存在的key
100006 |md5校验失败
100007 |api校验ip地址失败
100008 |没有访问权限
20000  |成功

# cmd-1-最新公共成交数据
```
说明：
无

订阅格式：
{"cmd":1,"action":"sub","symbol":"btc_cnc"}  
{"cmd":1,"action":"sub","symbol":"btc_cnc,eth_cnc"}

取消订阅格式：
{"cmd":1,"action":"unsub","symbol":"btc_cnc"}
```

格式参数说明:   

参数名  | 说明
-----  | ---------
cmd | 1 标识
action| sub 动作标识 
symbol | 交易对  
 
正常应答（json）
```
{"cmd":1,"action":"sub","code":20000,"symbol":"btc_cnc"}
```

正常推送数据（json）
  
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

- 数据推送说明:   
 
参数名  |类型| 说明
-----  | ---------| ---------
cmd | int| 标识
symbol |string| 交易对  
trade.0 | int|   unix时间戳 秒级别
trade.1 | string | 成交数量 
trade.2 | string | 成交价格
trade.3 | string | 交易类型 buy or sell
trade.4 | int | 成交id 唯一标识


# cmd-2-K线

```
推送最近一根K线的数据
说明：

订阅格式：
{"cmd":2,"action":"sub","symbol":"btc_cnc@1mon,btc_cnc@4hour"}

取消订阅格式：
{"cmd":2,"action":"unsub","symbol":"btc_cnc@1mon"}

```

- 格式参数说明:   
  
参数名  | 说明
-----  | ---------
cmd |业务标识
action|  动作标识 
symbol | 交易对@时间周期  可以订阅多个 用逗号分隔连接
  
时间周期可选（json）
```
'1mon' => '一个月',
'4hour' => '4小时',
'1day' => '一天',
'15min' => '15分钟',
'1min' => '1分钟',
'5day' => '5天',
'5min' => '5分钟',
'1wek' => '1周',
'1hour' => '1小时',
'30min' => '30分钟',
```
  
正常应答（json）
```
{"cmd":2,"action":"sub","code":20000,"symbol":"btc_cnc@1mon"}
```

正常推送数据（json）
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

- 数据推送说明:   

参数名  |类型| 说明
-----  | ---------| ---------
cmd | int| 标识
symbol |string| 交易对  
i |string| 周期 
t |int| K线开盘时间  
v |string| 成交数量  
o |string| 开盘价格  
h |string|高点价格  
l |string| 低点价格  
c |string| 最新价格或者收盘价格


# cmd-3-深度盘面
```
说明：
无

订阅格式：
{"cmd":3,"action":"sub","symbol":"btc_cnc"}

取消订阅格式：
{"cmd":3,"action":"unsub","symbol":"btc_cnc"}
```

格式参数说明:   

参数名  | 说明
-----  | ---------
cmd |业务标识
action|  动作标识 
symbol | 交易对

正常应答（json）
```
{"cmd":3,"action":"sub","code":20000,"symbol":"btc_cnc"}
```

- 正常推送数据（json）
  
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

- 数据推送说明:   

参数名  |类型| 说明
-----  | ---------| ---------
cmd | int| 标识
symbol |string| 交易对  
depth.buy |list| 买盘深度   价格从高到低
depth.sell |list| 买盘深度   价格从低到高
buy(sell).0.0 |string| 买（卖）一盘的数量
buy(sell).0.1 |string| 买（卖）一盘的价格


# cmd-4-24小时市场行情
```
说明：
无

订阅格式：
{"cmd":4,"action":"sub","symbol":"btc_cnc"}

取消订阅格式：
{"cmd":4,"action":"unsub","symbol":"btc_cnc"}
```

格式参数说明:   

参数名  | 说明
-----  | ---------
cmd |业务标识
action|  动作标识 
symbol | 交易对，用法：①btc_cnc是订阅单交易；②btc_cnc,eth_cnc是一次订阅多个交易对
正常应答（json，无论订阅方式，一次应答都是只有一个交易对的数据）
  
```
{"cmd":4,"action":"sub","code":20000,"symbol":"btc_cnc"}
```

- 正常推送数据（json）
  
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

- 数据推送说明:   
  
参数名  |类型| 说明
-----  | ---------| ---------
cmd | int| 标识
symbol |string| 交易对  
c |string| 当前价格  
o |string| 24小时涨跌幅  
h |string| 24小时最高价  
l |string| 24小时最低价  
q |string| 24成交数量  
m |string| 24小时成交金额  
b |string| 买一价格
s |string| 卖一价格  

# cmd-6-精简24小时行情

```
{"cmd":6,"action":"sub","symbol":"btc_cnc"}
```

- 正常推送数据（json）
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

# cmd-99-登录

- 订阅格式：
  
```
 {
   "cmd": 99,
   "action": "login",
   "key": "5f81d6e3cd3e96efa89d5297c1cec0c7",
   "time": 1594348186,
   "md5": "93a336226be823df3c7c50944df3eeb5"
 }
```

格式参数说明:   

参数名  | 说明
-----  | ---------
cmd | 99 标识
action| login 动作标识
key | api key
time| 当前时间 Unix time 单位是秒 30s有效期
md5 | md5=md5("key_user_id_skey_time")  

正常应答（json）
```
{"cmd": 99,"action": "login","code": 20000}
```

  
# cmd-100-个人订单变化
```
说明：
无

订阅格式：
{"cmd":100,"action":"sub","symbol":"btc_cnc"}
    
取消订阅格式：
{"cmd":100,"action":"unsub","symbol":"btc_cnc"}
```
    
格式参数说明:   
    
参数名  | 说明
-----  | ---------
cmd |业务标识
action|  动作标识 
symbol | 交易对

```   
正常应答（json）
{"cmd":100,"action":"sub","code":20000,"symbol":"btc_cnc"}
  
正常推送数据（json）
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
  
数据推送说明:   

参数名  |类型| 说明
-----  | ---------| ---------
cmd | int| 标识
symbol |string| 交易对  
i |string| 流水id  
s |int| 订单类型 1买 2卖  
q |string| 下单数量  
p |string| 价格  
z |string| 剩余数量
x |int|状态  4-撤单成功 3-退单 2-下单成功 
T |int|时间
  
# cmd-101-个人最新成交
```
说明：
无

订阅格式：
{"cmd":101,"action":"sub","symbol":"btc_cnc"}

取消订阅格式：
{"cmd":101,"action":"unsub","symbol":"btc_cnc"}
```

格式参数说明:   

参数名  | 说明
-----  | ---------
cmd |业务标识
action|  动作标识 
symbol | 交易对

正常应答（json）
```
{"cmd":101,"action":"sub","code": 20000,"symbol":"btc_cnc"}
```
正常推送数据（json）
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

数据推送说明:   

参数名  |类型| 说明
-----  | ---------| ---------
cmd | int| 标识
symbol |string| 交易对  
i |string| 流水ID  
T |int| 时间  
q |string| 数量  
p |string| 价格  
s |string| 成交类型  
t |int| 成交id  
n |string| 手续费  
N |string| 手续费币种  
m |int| 是否自成交  
e |int| 流动性方向  

  
# 订单和成交记录的关联
+ 在当前 Aex 的 RESTful 接口中，订单和成交记录之间是没有任何关联的，这导致很难对挂单和成交的情况进行分析，也增加了交易策略的实现难度。
+ RESTful api 中订单和成交记录缺乏关联的问题，在 websocket api 中通过挂单请求中的tag来解决，由同一挂单请求形成的订单和成交结果都会记录挂单请求中的tag，因此订单和成交记录可以通过tag关联起来。


# 交易策略的实现
由于 websocket api 中挂单请求里的tag完全是用户自定义的，只要 tag > 0 就行，用户可以自行对tag进行编码，实现某种自定义的交易策略。