# Configuration Protocol

Connmix supports two ways to parse network communication protocols:

- Built-in protocol: `websocket`
- Custom protocol: `socket`

The `lua` directory stores all network communication protocol scripts.

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

## `websocket` Protocol

> protocol: websocket

The built-in protocol is parsed directly using the Go language, so it has better performance. Please try to choose the built-in protocol.

- Open the entry file `lua\entry.websocket.lua`
- `prettyprint` is a library in the Lua ecosystem, and many Lua ecosystem libraries can be used directly.
- A websocket protocol Lua entry file must contain these methods: `on_handshake`, `on_close`, and `on_message`.
- We can write business logic in the entry file like writing code. The built-in package name prefixed with `mix.` is provided by Connmix's Lua API, and it can be used to complete many advanced developments.
- In the `on_message` method, we can process the received user messages, such as pushing the message to Redis or subscribing to a channel to achieve active push.

```lua
require("utils.prettyprint")
local mix_log = mix.log
local mix_DEBUG = mix.DEBUG

function on_handshake(client_id, headers)
    --mix_log(mix_DEBUG, "on_handshake client_id: " .. client_id)
end

function on_close(client_id, err)
    --print(err)
end

function on_message(msg)
   --print(msg)
   local conn = mix.websocket()
   conn:send('Hello,World!')
end
```

## `socket` Protocol

> protocol: socket

The custom protocol is parsed using the Lua language, so it has more flexibility. Users can define their own specific binary protocols for their companies. With the Lua API, we can write very complex protocols, such as a WebSocket protocol package provided in the installation package.

- Open the entry file `lua\entry.socket.lua`
- The file uses the official provided `protocols/websocket` Lua protocol
- `prettyprint` is a library in the Lua ecosystem, and many Lua ecosystem libraries can be used directly.
- Custom protocol Lua entry files must contain these methods: `on_connect`, `on_close`, `protocol_input`, `protocol_decode`, `protocol_encode`, and `on_message`:
- We can write business logic in the entry file like writing code. The built-in package name prefixed with `mix.` is provided by Connmix's Lua API, and it can be used to complete many advanced developments.
- In the `on_message` method, we can process the received user messages, such as pushing the message to Redis or subscribing to a channel to achieve active push.

```lua
require("utils.prettyprint")
local mix_log = mix.log
local mix_DEBUG = mix.DEBUG
local websocket = require("protocols.websocket")

function on_connect(client_id)
    mix_log(mix_DEBUG, "on_connect client_id: " .. client_id)
end

function on_handshake(client_id, headers)
    --mix_log(mix_DEBUG, "on_handshake client_id: " .. client_id)
end

function on_close(client_id, err)
    --print(err)
end

--buf is an object, a copy
--The return value must be int, return the length of the package termination 0=continue to wait,-1=disconnect
function protocol_input(buf)
    return websocket.input(buf, on_handshake)
end

--The return value supports any type, when the return data is nil, on_message will not be triggered
function protocol_decode(str)
    return websocket.decode(str)
end

--The return value must be string, when the return data is not string, or empty, an error will be returned when sending the message
function protocol_encode(str)
    return websocket.encode(str)
end

--msg is of any type, the value is equal to the return value of protocol_decode
function on_message(msg)
    --print(msg)
    websocket.send_text('Hello,World!')
end
```
