# 安装协议

- `lua` 目录存放了全部的网络通讯协议脚本
- `lua\protocols` 里存放的是 connmix 官方开源的通用网络协议
- 下载更多开源协议：https://github.com/connmix/protocols

```
└── lua
    ├── entry.lua
    ├── prettyprint.lua
    └── protocols
        ├── jsonrpc.lua
        ├── text.lua
        └── websocket.lua
```

- 打开入口文件 `lua\entry.lua`
- 该入口使用了 `protocols/websocket` 协议
- `require("prettyprint")` 是 lua 生态的一个库，很多lua生态的库可以直接使用
- 每个 lua 入口文件都必须包含这些方法：`init`、`on_connect`、`protocol_input`、`on_close`、`protocol_input`、`protocol_decode`、`protocol_encode`、`on_message`
- 我们可以像写代码一样在入口文件中编写一些协议处理的逻辑，其中 `mix.` 前缀的内置包名是 connmix 提供的 Lua API，可以使用它完成很多高级开发。
- 比如：下面代码中我们在 `init` 方法中创建了一个 100 尺寸的 `foo` 内存队列。
- 在 `on_message` 方法中我们将收到的消息 json 序列化后 push 到了内存队列 `foo` 中。
- 接下来我们就可以通过 ApiServer/Client SDK 来消费这个数据，并针对 client_id 来响应数据。

```lua
require("prettyprint")
local mix_log = mix.log
local mix_DEBUG = mix.DEBUG
local websocket = require("protocols/websocket")
local queue_name = "foo"

function init()
    mix.queue.new(queue_name, 100)
end

function on_connect(conn)
    --print(conn:client_id())
end

function on_close(err, conn)
    --print(err)
end

--bufcopy为一个对象
--返回值必须为int, 返回包截止的长度 0=继续等待,-1=断开连接
function protocol_input(bufcopy, conn)
    return websocket.input(bufcopy, conn, "/")
end

--返回值支持任意类型, 当返回数据为nil时，on_message将不会被触发
function protocol_decode(str, conn)
    return websocket.decode(conn)
end

--返回值必须为string, 当返回数据不是string, 或者为空, 发送消息时将返回失败错误
function protocol_encode(str, conn)
    return websocket.encode(str)
end

--data为任意类型, 值等于protocol_decode返回值
function on_message(data, conn)
    --print(data)
    if data["type"] ~= "text" then
        return
    end

    s, err = mix.json_encode({ msg = data, ctx = conn:context() })
    if err then
       mix_log(mix_DEBUG, "json_encode error: " .. err)
       return
    end

    n, err = mix.queue.push(queue_name, s)
    if err then
       mix_log(mix_DEBUG, "queue push error: " .. err)
       return
    end
end
```