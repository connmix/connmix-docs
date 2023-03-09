# 快速入门 - 几行代码搞定websocket实时通讯

## 前置条件

- [安装网关](/zh-cn/install-engine.md)
- [Pubsub](/zh-cn/pubsub.md)

## 启动网关

在开发阶段，我们使用 `dev` 模式，该模式会把 Center、Engine 两种节点一同启动，同时**无需授权**也拥有2CPU的执行能力。

```
% bin/connmix dev -f conf/connmix.yaml 

 _________________________________________________________________________________________________         
  ______________________________________________________________________________/\\\_______________      
   _____/\\\\\\\\_____/\\\\\_____/\\/\\\\\\____/\\/\\\\\\______/\\\\\__/\\\\\___\///___/\\\____/\\\_     
    ___/\\\//////____/\\\///\\\__\/\\\////\\\__\/\\\////\\\___/\\\///\\\\\///\\\__/\\\_\///\\\/\\\/__    
     __/\\\__________/\\\__\//\\\_\/\\\__\//\\\_\/\\\__\//\\\_\/\\\_\//\\\__\/\\\_\/\\\___\///\\\/____   
      _\//\\\________\//\\\__/\\\__\/\\\___\/\\\_\/\\\___\/\\\_\/\\\__\/\\\__\/\\\_\/\\\____/\\\/\\\___  
       __\///\\\\\\\\__\///\\\\\/___\/\\\___\/\\\_\/\\\___\/\\\_\/\\\__\/\\\__\/\\\_\/\\\__/\\\/\///\\\_ 
        ____\////////_____\/////_____\///____\///__\///____\///__\///___\///___\///__\///__\///____\///__
        
        connmix1.0.3, go1.17.5, lua5.1+bitop, darwin, arm64

2022-08-05 16:19:31.780199      WARN    commands/welcome.go:30  cpu underutilized, max_procs: 2, device_cpus: 8
2022-08-05 16:19:31.780432      INFO    center/server.go:46     start the center server 0.0.0.0:6787
2022-08-05 16:19:31.780510      INFO    registry/server.go:42   start the registry server (0.0.0.0:6786)
2022-08-05 16:19:31.780571      INFO    apiserver/server.go:58  start the api server (0.0.0.0:6789)
2022-08-05 16:19:31.780643      INFO    mesh/server.go:42       start the mesh point (0.0.0.0:6788)
2022-08-05 16:19:31.781903      INFO    lua/registrycli.go:36   connect to center registry localhost:6786 success
2022-08-05 16:19:31.781925      INFO    lua/registrycli.go:80   register engine node_id vsemvqedsc
2022-08-05 16:19:31.782274      INFO    registry/registry.go:118        register node 192.168.1.16 node_id vsemvqedsc
2022-08-05 16:19:31.782376      INFO    lua/wsserver.go:60      start the websocket server /Users/liujian/Documents/mycode/connmix/lua/entry.websocket.lua (0.0.0.0:6790)
```

我们的入口文件 `lua/entry.websocket.lua` 执行的服务在 `6790` 端口，采用的是 `websocket` 协议。

## 编写服务端逻辑

示范一个聊天室Demo，修改 `entry.websocket.lua` 文件 `on_message` 方法的内容如下：

- 当用户发送 `{"op":"join","room_id":1002}` 就给该连接订阅 `room:1002` 通道。
- 当用户发送 `{"op":"send","msg":"Hello,World!"}` 就给当前通道发送消息。
- 当用户发送 `{"op":"quit"}` 就给该连接取消订阅当前通道。

> 通过 [Lua API](/zh-cn/lua-api) 我们可以编写各种复杂的业务逻辑

```lua
function on_message(msg)
    --参数解析
    local tb, err = mix.json_decode(msg["data"])
    if err then
        mix_log(mix_DEBUG, "json_decode error: " .. err)
        return
    end
    local conn = mix.websocket()
    local op = tb["op"]
    local room_id = tb["room_id"]
    local msg = tb["msg"]
    if op == nil then
        return
    end

    --加入房间逻辑
    if op == "join" then
        if room_id == nil then
            return
        end

        --只允许同时加入一个房间
        local current_room_id = conn:context_value("current_room_id")
        if current_room_id ~= nil then
            conn:unsubscribe("room:" .. current_room_id)
        end

        conn:subscribe("room:" .. room_id)
        conn:set_context_value("current_room_id", room_id) --保存加入的房间ID
        conn:send('{"status":"success"}')
    end

    --退出房间逻辑
    if op == "quit" then
        local current_room_id = conn:context_value("current_room_id") --取出之前保存的房间ID
        conn:unsubscribe("room:" .. current_room_id)
        conn:send('{"status":"success"}')
    end

    --发送消息逻辑
    if op == "send" then
        if msg == nil then
            return
        end
        local client_id = conn:client_id()
        local current_room_id = conn:context_value("current_room_id") --取出之前保存的房间ID
        conn:send('{"status":"success"}')
        mix.mesh.publish("room:" .. current_room_id, '{"client_id":"' .. client_id .. '","msg":"' .. msg .. '"}')
    end
end
```

- 发送广播：只需要在服务端给任意一个节点的 ApiServer 发送一下HTTP请求，就可以给该room发送广播消息。

```shell
curl --location --request POST 'http://127.0.0.1:6789/v1/mesh/publish' \
--header 'Content-Type: application/json' \
--data-raw '{
    "c": "room:1002",
    "d": "{\"type\":\"broadcast\",\"msg\":\"Hello,World!\"}"
}'
```

## 测试

- `连接` 使用websocket测试工具连接 `ws://127.0.0.1:6790/chat`
- `发送` 发送消息 `{"op":"join","room_id":1002}`
- `接收` 收到回复 `{"status":"success"}` 表示加入房间成功
- `发送` 发送消息 `{"op":"send","msg":"Hello,World!"}`
- `接收` 收到回复 `{"status":"success"}` 表示发送成功
- `接收` 房间内所有人收到消息 `{"client_id":"1627581697287520263","msg":"Hello,World!"}`
- `广播` 执行curl命令给通道 `room:1002` 发送广播
- `接收` 房间内所有人收到消息 `{"type":"broadcast","msg":"Hello,World!"}`
- `退出` 发送消息 `{"op":"quit"}`
- `接收` 收到回复 `{"status":"success"}` 表示退出成功，将不会接收新的消息
