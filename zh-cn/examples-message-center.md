# WebSocket 用户消息中心

使用 WebSocket 做消息中心，通常做法是采用kafka、redis等中间件搭配实现，使用 [CONNMIX](https://connmix.com/) 则无需使用中间件，同时分布式集群能力也无需担忧用户量大增后带来的性能问题。

## 要求

- [connmix](https://connmix.com/) >= v1.0.4

## 设计思路

- 客户端在ws连接成功后发送消息执行登录，使用lua调用业务api接口解析登录token数据中的uid，然后将uid保存到连接的context中。
- 登录成功后发送消息订阅一个用户ID的通道 `user:<uid>`，该uid从context中取出。
- 在发送用户消息的接口中，调用connmix任意节点的 `/v1/mesh/publish` 接口往对应 `uid` 发送实时消息，所有订阅该通道的ws客户端都将会收到消息。
- 以上都是增量推送设计，全量通常都是在页面加载时通过一个全量api接口获取。

## 交互协议设计

- 必须登录后才能执行订阅、取消订阅
- 当用户发送 @user 我们在 lua 代码中就执行订阅 user:\<uid\> 通道。

| 功能     | json格式                                                     |
|--------|------------------------------------------------------------|
| 登录     | {"op":"auth","token":"***"}                                |
| 订阅用户消息 | {"op":"subscribe","channel":"@user"}                       |
| 取消用户消息 | {"op":"unsubscribe","channel":"@user"}                     |
| 用户消息事件 | {"event":"@user","data":{"uid":1001,"msg":"Hello,World!"}} | 
| 成功     | {"result":true}                                            | 
| 错误     | {"code":1,"msg":"Error"}                                   | 

## 安装引擎

- `connmix` [install-engine](zh-cn/install-engine)

## 修改配置

在 `connmix.yaml` 配置文件的 `options` 选项，修改websocket的url路径

```
options:
  - name: path
    value: /message-center
```

## CONNMIX 编码

修改 `entry.websocket.lua` 的 `on_message` 方法如下：

- 当消息为auth类型时，调用 `auth_url` 接口通过token获取到uid，并保存到context中
- 当消息为subscribe、unsubscribe时，从context取出uid，执行订阅/取消订阅对应的通道

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

    if op == "subscribe" and channel_type == "user" then
        if uid == nil then
            conn:send('{"code":1,"msg":"Not Auth"}')
            return
        end
        local err = conn:subscribe("user:" .. uid)
        if err then
            mix_log(mix_DEBUG, "subscribe error: " .. err)
            conn:close()
            return
        end
    end

    if op == "unsubscribe" and channel_type == "user" then
        if uid == nil then
            conn:send('{"code":1,"msg":"Not Auth"}')
            return
        end
        local err = conn:unsubscribe("user:" .. uid)
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

### 发送消息接口

在现有系统的框架中实现主动消息推送

- 可以在 spring、laravel 框架中写一个发送用户消息接口
- 该接口中验证完用户身份后，执行以下http请求完成推送
- 如果发送请求非常频繁，可以改用 [websocket-api推送](zh-cn/websocket-api?id=%e7%bd%91%e6%a0%bc%e5%8f%91%e5%b8%83%ef%bc%9a%e5%8f%af%e4%bb%a5%e5%8f%91%e9%80%81%e7%bb%99%e6%95%b4%e4%b8%aa%e7%bd%91%e6%a0%bc%e5%86%85%e6%89%80%e6%9c%89%e8%ae%a2%e9%98%85%e4%ba%86%e8%bf%99%e4%ba%9b%e9%a2%91%e9%81%93%e7%9a%84%e5%ae%a2%e6%88%b7%e7%ab%af%e8%bf%9e%e6%8e%a5-1) 提升性能

```
curl --request POST 'http://127.0.0.1:6789/v1/mesh/publish' \
--header 'Content-Type: application/json' \
--data-raw '{
    "c": "user:1001",
    "d": "{\"event\":\"@user\",\"data\":{\"uid\":1001,\"msg\":\"Hello,World!\"}}"
}'
```

## 测试

使用 [wstool](http://www.easyswoole.com/wstool.html) 进行测试

- 连接 `ws://127.0.0.1:6790/message-center`
- 发送 `{"op":"auth","token":"***"}`
- 接收到 `{"result":true}`
- 发送 `{"op":"subscribe","channel":"@user"}`
- 接收到 `{"result":true}`
- 执行 curl 主动推送
- 接收到 `{"event":"@user","data":{"uid":1001,"msg":"Hello,World!"}}`
