# Quick start - a few lines of code to get websocket real-time communication

## Preconditions

- [Install Gateway](/en-us/install-engine.md)
- [Pubsub](/en-us/pubsub.md)

## Start the gateway

In the development stage, we use the `dev` mode, which will start both Center and Engine nodes together, and **no authorization** also has the execution capability of 2CPU.

```
% bin/connmix dev -f conf/connmix.yaml

  __________________________________________________________________________________________________
   ______________________________________________________________________________/\\\_______________
    _____/\\\\\\\\_____/\\\\_____/\\/\\\\\\____/\\/\\\\\\______/\\\\\__/\\ \\\___\///___/\\\____/\\\_
     ___/\\\//////____/\\\///\\\__\/\\\////\\\__\/\\\////\\\___/ \\\///\\\\\///\\\__/\\\_\///\\\/\\\/__
      __/\\\__________/\\\__\//\\\_\/\\\__\//\\\_\/\\\__\//\\\_\/\\\_ \//\\\__\/\\\_\/\\\___\///\\\/____
       _\//\\\________\//\\\__/\\\__\/\\\___\/\\\_\/\\\___\/\\\_\/\\\__ \/\\\__\/\\\_\/\\\____/\\\/\\\___
        __\///\\\\\\\\__\///\\\\\/___\/\\\___\/\\\_\/\\\___\/\\\_\ /\\\__\/\\\__\/\\\_\/\\\__/\\\/\///\\\_
         ____\////////_____\/////_____\///____\///__\///____\///__\///___\///___\/ //__\///__\///____\///__
        
         connmix1.0.3, go1.17.5, lua5.1+bitop, darwin, arm64

2022-08-05 16:19:31.780199 WARN commands/welcome.go:30 cpu underutilized, max_procs: 2, device_cpus: 8
2022-08-05 16:19:31.780432 INFO center/server.go:46 start the center server 0.0.0.0:6787
2022-08-05 16:19:31.780510 INFO registry/server.go:42 start the registry server (0.0.0.0:6786)
2022-08-05 16:19:31.780571 INFO apiserver/server.go:58 start the api server (0.0.0.0:6789)
2022-08-05 16:19:31.780643 INFO mesh/server.go:42 start the mesh point (0.0.0.0:6788)
2022-08-05 16:19:31.781903 INFO lua/registrycli.go:36 connect to center registry localhost:6786 success
2022-08-05 16:19:31.781925 INFO lua/registrycli.go:80 register engine node_id vsemvqedsc
2022-08-05 16:19:31.782274 INFO registry/registry.go:118 register node 192.168.1.16 node_id vsemvqedsc
2022-08-05 16:19:31.782376 INFO lua/wsserver.go:60 start the websocket server /Users/liujian/Documents/mycode/connmix/lua/entry.websocket.lua (0.0.0.0:6790)
```

Our entry file `lua/entry.websocket.lua` executes a service on port `6790`, using the `websocket` protocol.

## Write server-side logic

To demonstrate a chat room Demo, modify the content of the `on_message` method in the `entry.websocket.lua` file as follows:

- When the user sends `{"op":"join","room_id":1002}`, subscribe the connection to the `room:1002` channel.
- Send a message to the current channel when the user sends `{"op":"send","msg":"Hello,World!"}`.
- Unsubscribe the current channel for the connection when the user sends `{"op":"quit"}`.

> Through [Lua API](/en-us/lua-api) we can write various complex business logic

```lua
function on_message(msg)
     --parameter parsing
     local tb, err = mix.json_decode(msg["data"])
     if err then
         mix_log(mix_DEBUG, "json_decode error: " .. err)
         return
     end
     local conn = mix. websocket()
     local op = tb["op"]
     local room_id = tb["room_id"]
     local msg = tb["msg"]
     if op == nil then
         return
     end

     --add room logic
     if op == "join" then
         if room_id == nil then
             return
         end

         --Only allow to join one room at the same time
         local current_room_id = conn:context_value("current_room_id")
         if current_room_id ~= nil then
             conn:unsubscribe("room:" .. current_room_id)
         end

         conn:subscribe("room:" .. room_id)
         conn:set_context_value("current_room_id", room_id) -- Save the room ID you joined
         conn:send('{"status":"success"}')
     end

     --exit room logic
     if op == "quit" then
         local current_room_id = conn:context_value("current_room_id") --take out the previously saved room ID
         conn:unsubscribe("room:" .. current_room_id)
         conn:send('{"status":"success"}')
     end

     --send message logic
     if op == "send" then
         if msg == nil then
             return
         end
         local client_id = conn:client_id()
         local current_room_id = conn:context_value("current_room_id") --take out the previously saved room ID
         conn:send('{"status":"success"}')
         mix.mesh.publish("room:" .. current_room_id, '{"client_id":"' .. client_id .. '","msg":"' .. msg .. '"}')
     end
end
```

- Send broadcast: You only need to send an HTTP request to the ApiServer of any node on the server side, and then you can send a broadcast message to the room.

```shell
curl --location --request POST 'http://127.0.0.1:6789/v1/mesh/publish' \
--header 'Content-Type: application/json' \
--data-raw '{
     "c": "room:1002",
     "d": "{\"type\":\"broadcast\",\"msg\":\"Hello, World!\"}"
}'
```

## test

- `connect` use websocket test tool to connect `ws://127.0.0.1:6790/chat`
- `send` send message `{"op":"join","room_id":1002}`
- `Receive` receives a reply `{"status":"success"}`, which means joining the room successfully
- `send` Send a message `{"op":"send","msg":"Hello,World!"}`
- `Receive` received a reply `{"status":"success"}`, indicating that the sending was successful
- `Receive` everyone in the room receives the message `{"client_id":"1627581697287520263","msg":"Hello,World!"}`
- `Broadcast` Execute the curl command to send a broadcast to the channel `room:1002`
- `Receive` Everyone in the room receives the message `{"type":"broadcast","msg":"Hello,World!"}`
- `quit` sends the message `{"op":"quit"}`
- `Receive` receives a reply `{"status":"success"}`, indicating that the exit is successful, and no new messages will be received
