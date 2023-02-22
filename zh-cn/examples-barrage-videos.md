# WebSocket 视频弹幕

使用 WebSocket 制作一个单机版弹幕系统非常简单，但是当单机性能达到瓶颈，需要扩展为集群部署时就会面临很多分布式问题，使用 [CONNMIX](https://connmix.com/) 则无需担心这些问题，很少的代码即可完成一个高性能分布式WebSocket集群。

## 前置条件

- [安装引擎](/zh-cn/install-engine.md)
- [快速入门](/zh-cn/start-debug.md)
- [Pubsub](/zh-cn/pubsub.md)

## 设计思路

- 客户端在ws连接成功后，订阅一个视频ID的通道 `video:<videoid>`
- 在发送弹幕的接口中，调用connmix任意节点的 `/v1/mesh/publish` 接口往对应 `video_id` 发送实时弹幕，所有订阅该通道的ws客户端都将会收到消息。
- 以上都是增量推送设计，全量通常都是在页面加载时通过一个全量api接口获取。

## 交互协议设计

- 当用户发送 100001@video 我们在 lua 代码中就执行订阅 video:100001 通道。
- 前端切换视频时，可先发送取消弹幕消息，然后发送新的订阅弹幕消息。

| 功能   | json格式                                                                        |
|------|-------------------------------------------------------------------------------|
| 订阅弹幕 | {"op":"subscribe","channel":"100001@video"}                                   |
| 取消弹幕 | {"op":"unsubscribe","channel":"100001@video"}                                 |
| 弹幕事件 | {"event":"@video","data":{"uid":1001,"video_id":100001,"msg":"Hello,World!"}} | 
| 成功   | {"result":true}                                                               | 
| 错误   | {"code":1,"msg":"Error"}                                                      | 

## 修改配置

在 `connmix.yaml` 配置文件的 `options` 选项，修改websocket的url路径

```
options:
  - name: path
    value: /barrage-videos
```

## CONNMIX 编码

修改 `entry.websocket.lua` 的 `on_message` 方法如下：

- 当接受到subscribe、unsubscribe消息时，执行订阅/取消订阅对应的通道

```lua
function on_message(msg)
    --print(msg)
    if msg["type"] ~= "text" then
        conn:close()
        return
    end

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
    local video_id = channel_table[1] --Lua的table索引默认从1开始
    local channel_type = channel_table[2]
    
    if op == "subscribe" and channel_type == "video" then
        local err = conn:subscribe("video:" .. video_id)
        if err then
            mix_log(mix_DEBUG, "subscribe error: " .. err)
            conn:close()
            return
        end
    end
    
    if op == "unsubscribe" and channel_type == "video" then
        local err = conn:unsubscribe("video:" .. video_id)
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

接下来在现有系统的框架中实现主动弹幕推送

- 可以在 spring、laravel 框架中写一个发送弹幕接口
- 该接口中验证完用户身份后，执行以下http请求完成推送
- 如果发送请求非常频繁，可以改用 [websocket-api推送](zh-cn/websocket-api?id=%e7%bd%91%e6%a0%bc%e5%8f%91%e5%b8%83%ef%bc%9a%e5%8f%af%e4%bb%a5%e5%8f%91%e9%80%81%e7%bb%99%e6%95%b4%e4%b8%aa%e7%bd%91%e6%a0%bc%e5%86%85%e6%89%80%e6%9c%89%e8%ae%a2%e9%98%85%e4%ba%86%e8%bf%99%e4%ba%9b%e9%a2%91%e9%81%93%e7%9a%84%e5%ae%a2%e6%88%b7%e7%ab%af%e8%bf%9e%e6%8e%a5-1) 提升性能

```
curl --request POST 'http://127.0.0.1:6789/v1/mesh/publish' \
--header 'Content-Type: application/json' \
--data-raw '{
    "c": "video:100001",
    "d": "{\"event\":\"@video\",\"data\":{\"uid\":1001,\"video_id\":100001,\"msg\":\"Hello,World!\"}}"
}'
```

## 测试

使用 [wstool](http://www.easyswoole.com/wstool.html) 进行测试

- 连接 `ws://127.0.0.1:6790/barrage-videos`
- 发送 `{"op":"subscribe","channel":"100001@video"}`
- 接收到 `{"result":true}`
- 执行 curl 主动推送
- 接收到 `{"event":"@video","data":{"uid":1001,"video_id":100001,"msg":"Hello,World!"}}`
