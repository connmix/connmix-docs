# WebSocket 数字交易所行情系统

数字货币交易所近些年逐步成熟，和以前的蛮荒时代不同，技术上越来越先进，虽然区块链是去中心化的，但是交易所是中心化的系统，和互联网行业没有任何区别；和传统金融业的保守不同，WebSocket 在数字货币交易所被普遍使用在行情系统中，通常用于Web/App实时获取行情、指数、用户成交等交易相关信息；

[CONNMIX](https://connmix.com/) 是一个基于 go 与 lua 的分布式长连接引擎，可以在短时间内搭建一个高性能交易所行情推送系统。

## 要求

- [connmix](https://connmix.com/) >= v1.0.4

## 需求分析

通常交易所推送系统包括以下部分：

- `公有接口` 无需登录，可以随意获取市场行情数据，包括 orderbook, trades, mark_price, ticker, kline 等信息
- `私有接口` 需要发送消息登录，用来获取 balance、order、position、account 等个人信息的推送

## 模块设计

通过以下模块设计可见，CONNMIX 让需求设计变得更纯粹，程序员无需考虑用户连接处理，只需专注于业务逻辑。

| 模块名                         | 描述                                                                                                      | 工具              |
|-----------------------------|---------------------------------------------------------------------------------------------------------|-----------------|
| 用户连接推送                      | 负责与用户连接，根据业务订阅某个通道，实现根据channel主动推送                                                                      | CONNMIX         |
| 公有推送计算 orderbook            | 定时获取撮合系统的book快照，对比后得出变化的档位，将计算结果推送到 `<symbol>@depth`                                                    | java,php,go ... |
| 公有推送计算 trades               | 通过UDP/TCP从撮合系统实时获取trades信息，降频合并后推送到 `<symbol>@trades`                                                   | java,php,go ... |
| 公有推送计算 mark_price           | 标记价格推送通常还包括指数价格，预估结算价，资金费率等信息，这些需要根据产品的公式计算好后推送到 `<symbol>@markprice`                                   | java,php,go ... |
| 公有推送计算 ticker               | 通常是记录24小时内开、高、低加上当前最新价格、量、额一起定时推送到 `<symbol>@ticker`                                                    | java,php,go ... |
| 公有推送计算 kline                | 该推送通常是在前端的TradingView内调用，由后端程序记录某个时间间隔内的开、高、低、收，定时推送到 `<symbol>@kline`                                  | java,php,go ... |
| 私有推送登录设计                    | 只需在现有系统中增加一个给ws做登录的接口，返回对应uid即可                                                                         | java,php,go ... |
| 私有推送 balance、order、position | balance、order、position 变化信息都由撮合系统获取，从信息中提取出uid，然后分别发送到 `<uid>@balance`, `<uid>@order`, `<uid>@position` | java,php,go ... |
| 私有推送 account                | account 变化通常是rest触发，从信息中提取出uid，然后发送到 `<uid>@account`                                                    | java,php,go ... |

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

| 功能         | Channel              | CONNMIX 内部通道格式       |
|------------|----------------------|----------------------|
| orderbook  | `<symbol>@depth`     | `<symbol>@depth`     |
| trades     | `<symbol>@trades`    | `<symbol>@trades`    |
| mark_price | `<symbol>@markprice` | `<symbol>@markprice` |
| ticker     | `<symbol>@ticker`    | `<symbol>@ticker`    |
| kline      | `<symbol>@kline`     | `<symbol>@kline`     |
| balance    | `@balance`           | `<uid>@balance`      |
| order      | `@order`             | `<uid>@order`        |
| position   | `@position`          | `<uid>@position`     |
| account    | `@account`           | `<uid>@account`      |

## 安装引擎

- `connmix` [install-engine](zh-cn/install-engine)

## 修改配置

在 `connmix.yaml` 配置文件的 `options` 选项，修改websocket的url路径

```
options:
  - name: path
    value: /exchange-stream
```

## CONNMIX 编码

修改 `entry.websocket.lua` 的 `on_message` 方法如下：

- 当消息为auth类型时，调用 `auth_url` 接口通过token获取到uid，并保存到context中
- 当消息为subscribe、unsubscribe时，从context取出uid拼接到内部通道，执行订阅/取消订阅对应的通道

```lua
function on_message(msg)
    --print(msg)
    if msg["type"] ~= "text" then
        conn:close()
        return
    end

    local auth_url = "http://127.0.0.1:8000/websocket_auth" --填写解析token的api接口地址
    local conn = mix.websocket()

    local data, err = mix.json_decode(msg["data"])
    if err then
        mix_log(mix_DEBUG, "json_decode error: " .. err)
        conn:close()
        return
    end

    local op = data["op"]
    if op == nil then
        mix_log(mix_DEBUG, "op is nil")
        conn:close()
        return
    end
    if op == "auth" then
        local token = data["token"]
        if token == nil then
            token = ""
        end
        local resp, err = mix.http.request("POST", auth_url, {
            body = '{"token:"' .. token .. '"}'
        })
        if err then
            mix_log(mix_DEBUG, "http.request error: " .. err)
            conn:close()
            return
        end
        if resp.status_code ~= 200 then
            mix_log(mix_DEBUG, "http.request status_code: " .. resp["status_code"])
            conn:close()
            return
        end
        local body_table, err = mix.json_decode(resp["body"])
        if err then
            mix_log(mix_DEBUG, "json_decode error: " .. err)
            conn:close()
            return
        end
        conn:set_context_value("uid", body_table["uid"])
        return
    end

    local uid = conn:context_value("uid")
    local channel_raw = data["channel"]
    if channel_raw == nil then
        channel_raw = ""
    end
    local channel_table = mix.str_split(channel_raw, "@")
    if table.getn(channel_table) ~= 2 then
        mix_log(mix_DEBUG, "invalid channel: " .. channel_raw)
        conn:close()
        return
    end
    local channel_type = channel_table[2]
    local inner_channel = ""
    if channel_type == "balance" or channel_type == "order" or channel_type == "position" or channel_type == "account" then
        --需要登录才可订阅
        if uid == nil then
            conn:send('{"code":1,"msg":"Not Auth"}')
            return
        end
        inner_channel = string.format("%d@%s", uid, channel_type)
    else
        inner_channel = channel_raw
    end
    
    --为了安全而设定前缀
    local prefix = "ex:"

    if op == "subscribe" then
        local err = conn:subscribe(prefix .. inner_channel)
        if err then
            mix_log(mix_DEBUG, "subscribe error: " .. err)
            conn:close()
            return
        end
    end

    if op == "unsubscribe" then
        local err = conn:unsubscribe(prefix .. inner_channel)
        if err then
            mix_log(mix_DEBUG, "unsubscribe error: " .. err)
            conn:close()
            return
        end
    end

    conn:send('{"result":true}')
end
```

## API 编码

### WebSocket登录接口

在现有系统的框架中编写一个登录信息验证接口 `/websocket_auth`，用于ws登录获取用户uid

- 接口入参：token

```json
{"token":"***"}
```

- 接口出参：uid

```json
{"uid":1001}
```

### 内部消息推送

在现有系统的框架中完成对应频道的主动消息推送

- 可以在 spring、laravel 框架写的常驻程序中执行以下http请求完成推送
- 如果发送请求非常频繁，可以改用 [websocket-api推送](zh-cn/websocket-api?id=%e7%bd%91%e6%a0%bc%e5%8f%91%e5%b8%83%ef%bc%9a%e5%8f%af%e4%bb%a5%e5%8f%91%e9%80%81%e7%bb%99%e6%95%b4%e4%b8%aa%e7%bd%91%e6%a0%bc%e5%86%85%e6%89%80%e6%9c%89%e8%ae%a2%e9%98%85%e4%ba%86%e8%bf%99%e4%ba%9b%e9%a2%91%e9%81%93%e7%9a%84%e5%ae%a2%e6%88%b7%e7%ab%af%e8%bf%9e%e6%8e%a5-1) 提升性能

```
curl --request POST 'http://127.0.0.1:6789/v1/mesh/publish' \
--header 'Content-Type: application/json' \
--data-raw '{
    "c": "ex:1001@balance",
    "d": "{\"event\":\"@balance\",\"data\":{\"uid\":1001,\"balance\":[{\"currency\":\"BTC\",\"available\":100,\"unavailable\":100}]}}"
}'
```

## 测试

使用 [wstool](http://www.easyswoole.com/wstool.html) 进行测试

- 连接 `ws://127.0.0.1:6790/exchange-stream`
- 发送 `{"op":"auth","token":"***"}`
- 接收到 `{"result":true}`
- 发送 `{"op":"subscribe","channel":"@balance"}`
- 接收到 `{"result":true}`
- 执行 curl 主动推送
- 接收到 `{"event":"@balance","data":{"uid":1001,"balance":[{"currency":"BTC","available":100,"unavailable":100}]}}`
