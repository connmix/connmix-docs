# WebSocket 数字交易所推送系统

数字货币交易所近些年逐步成熟，和以前的蛮荒时代不同，技术上越来越先进，虽然区块链是去中心化的，但是交易所是中心化的系统，和互联网行业没有任何区别；和传统金融业的保守不同，WebSocket 在数字货币交易所被普遍使用在行情系统中，通常用于Web/App实时获取行情、指数、用户成交等交易相关信息；

[CONNMIX](https://connmix.com/) 是一个基于 go 与 lua 的分布式长连接引擎，可以在短时间内搭建一个高性能交易所推送系统。

## 要求

- [connmix](https://connmix.com/) >= v1.0.4

## 需求分析

通常交易所推送系统包括以下部分：

- `公有接口` 无需登录，可以随意获取市场行情数据，包括 orderbook, trades, mark_price, ticker, kline 等信息
- `私有接口` 需要发送消息登录，用来获取 balance、order、position、account 等个人信息的推送

## 模块设计

通过以下模块设计可见，CONNMIX 让需求设计变得更纯粹，程序员无需考虑用户连接处理，只需专注于业务逻辑。

| 模块名                         | 描述                                                                                                                               | 工具              |
|-----------------------------|----------------------------------------------------------------------------------------------------------------------------------|-----------------|
| 用户连接推送                      | 负责与用户连接，根据业务订阅某个通道，实现根据channel主动推送                                                                                               | CONNMIX         |
| 公有推送计算 orderbook            | 定时获取撮合系统的book快照，对比后得出变化的档位，将计算结果推送到 channel: depth_\<symbol>                                                                     | java,php,go ... |
| 公有推送计算 trades               | 通过UDP/TCP从撮合系统实时获取trades信息，降频合并后推送到 channel: trades_\<symbol>                                                                    | java,php,go ... |
| 公有推送计算 mark_price           | 标记价格推送通常还包括指数价格，预估结算价，资金费率等信息，这些需要根据产品的公式计算好后推送到 channel: markprice_\<symbol>                                                    | java,php,go ... |
| 公有推送计算 ticker               | 通常是记录24小时内开、高、低加上当前最新价格、量、额一起定时推送到 channel: ticker_\<symbol>                                                                     | java,php,go ... |
| 公有推送计算 kline                | 该推送通常是在前端的TradingView内调用，由后端程序记录某个时间间隔内的开、高、低、收，定时推送到 channel: kline_\<symbol>                                                   | java,php,go ... |
| 私有推送登录设计                    | 只需在现有系统中增加一个给ws做登录的接口，返回对应uid即可                                                                                                  | java,php,go ... |
| 私有推送 balance、order、position | balance、order、position 变化信息都由撮合系统获取，从信息中提取出uid，然后分别发送到 channels: user_\<uid>\_balance, user_\<uid>\_order, user_\<uid>\_position | java,php,go ... |
| 私有推送 account                | account 变化通常是rest触发，从信息中提取出uid，然后发送到 channel:  user_\<uid>_account                                                               | java,php,go ... |

## 交互协议设计

消息设计

| 功能     | json格式                                                 |
|--------|--------------------------------------------------------|
| 登录     | {"op":"auth","token":"***"}                            |
| 订阅通道   | {"op":"subscribe","channel":"BTCUSDT@depth"}           |
| 取消通道   | {"op":"unsubscribe","channel":"BTCUSDT@depth"}         |
| 用户通道消息 | {"event":"BTCUSDT@depth","data":{"bids":[],"asks":[]}} | 
| 成功     | {"result":true}                                        | 
| 错误     | {"code":1,"msg":"Error"}                               | 

通道设计

| 功能         | 协议通道                 | CONNMIX 内部通道格式        |
|------------|----------------------|-----------------------|
| orderbook  | `<symbol>@depth`     | `depth_<symbol>`      |
| trades     | `<symbol>@trades`    | `trades_<symbol>`     |
| mark_price | `<symbol>@markPrice` | `markprice_<symbol>`  |
| ticker     | `<symbol>@ticker`    | `ticker_<symbol>`     |
| kline      | `<symbol>@kline`     | `kline_<symbol>`      |
| balance    | `@balance`           | `user_<uid>_balance`  |
| order      | `@order`             | `user_<uid>_order`    |
| position   | `@position`          | `user_<uid>_position` |
| account    | `@account`           | `user_<uid>_account`  |
