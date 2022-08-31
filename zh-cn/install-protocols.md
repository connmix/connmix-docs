# 安装协议

connmix 支持两种方式来解析网络通讯协议

- 内置协议：`websocket` 后续还会支持 `mqtt`
- 自定义协议：`socket`

`lua` 目录存放了全部的网络通讯协议脚本

```
├── lua
│   ├── entry.socket.lua
│   ├── entry.websocket.lua
│   ├── protocols
│   │   ├── jsonrpc.lua
│   │   ├── text.lua
│   │   └── websocket.lua
│   └── utils
│       └── prettyprint.lua
```

## `websocket` 协议

> protocol: websocket

内置协议是使用go语言直接解析协议，因此性能更好，大家尽量选择内置协议

- 打开入口文件 `lua\entry.websocket.lua`
- `prettyprint` 是 lua 生态的一个库，很多lua生态的库可以直接使用
- websocket 协议 lua 入口文件都必须包含这些方法：`on_connect`、`on_close`、`on_message`
- 我们可以像写代码一样在入口文件中编写业务处理的逻辑，其中 `mix.` 前缀的内置包名是 connmix 提供的 Lua API，可以使用它完成很多高级开发。
- 在 `on_message` 方法中我们可以处理收到的用户消息，例如：将消息push到redis，或者订阅某个channel来实现主动推送。

```lua
require("utils.prettyprint")
local mix_log = mix.log
local mix_DEBUG = mix.DEBUG
local mix_conn = mix.websocket

function on_connect(client_id, headers)
    mix_log(mix_DEBUG, "on_connect client_id: " .. client_id)
end

function on_close(client_id, err)
    --print(err)
end

function on_message(msg)
   --print(msg)
   local conn = mix_conn()
   conn:send('Hello,World!')
end
```

## `socket` 协议

> protocol: socket

自定义协议是采用lua语言来解析协议，因此具有更多的灵活性，用户可以自己定义属于自己公司的特定二进制协议，通过 lua api 我们可以自己编写非常复杂的协议，例如：安装包内我们就提供了一个lua编写的websocket协议包。

- 打开入口文件 `lua\entry.socket.lua`
- 该文件使用了官方提供的 `protocols/websocket` lua协议
- `prettyprint` 是 lua 生态的一个库，很多lua生态的库可以直接使用
- 自定义协议 lua 入口文件都必须包含这些方法：`on_connect`、`on_close`、`protocol_input`、`protocol_decode`、`protocol_encode`、`on_message`
- 我们可以像写代码一样在入口文件中编写业务处理的逻辑，其中 `mix.` 前缀的内置包名是 connmix 提供的 Lua API，可以使用它完成很多高级开发。
- 在 `on_message` 方法中我们可以处理收到的用户消息，例如：将消息push到redis，或者订阅某个channel来实现主动推送。

```lua
require("utils.prettyprint")
local mix_log = mix.log
local mix_DEBUG = mix.DEBUG
local websocket = require("protocols.websocket")
local mix_conn = mix.socket.tcp

function on_connect(client_id)
    mix_log(mix_DEBUG, "on_connect client_id: " .. client_id)
end

function on_handshake(headers)
    --print(headers)
end

function on_close(client_id, err)
    --print(err)
end

--buf为一个对象，是一个副本
--返回值必须为int, 返回包截止的长度 0=继续等待,-1=断开连接
function protocol_input(buf)
    return websocket.input(buf, on_handshake)
end

--返回值支持任意类型, 当返回数据为nil时，on_message将不会被触发
function protocol_decode(str)
    return websocket.decode(str)
end

--返回值必须为string, 当返回数据不是string, 或者为空, 发送消息时将返回失败错误
function protocol_encode(str)
    return websocket.encode(str)
end

--msg为任意类型, 值等于protocol_decode返回值
function on_message(msg)
    --print(msg)
    websocket.send_text('Hello,World!')
end
```
