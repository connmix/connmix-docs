# WebSocket User Message Center

Using WebSocket as a message center is usually implemented by using middleware such as kafka and redis. Using [CONNMIX](https://connmix.com/) does not need to use middleware, and at the same time, distributed cluster capabilities do not need to worry about a large number of users Performance problems caused by the increase.

## Preconditions

- [Install Gateway](/en-us/install-engine.md)
- [Pubsub](/en-us/pubsub.md)
- [Quick Start](/en-us/start-debug.md)

## Design ideas

- The client sends a message to perform login after the ws connection is successful, uses lua to call the business api interface to parse the uid in the login token data, and then saves the uid to the context of the connection.
- After successful login, send a message to subscribe to a user ID channel `user:<uid>`, and the uid is taken from the context.
- In the interface for sending user messages, call the `/v1/mesh/publish` interface of any connmix node to send a real-time message to the corresponding `uid`, and all ws clients that subscribe to this channel will receive the message.
- The above are all incremental push designs, and the full amount is usually obtained through a full amount api interface when the page is loaded.

## Interaction protocol design

- You must be logged in to subscribe and unsubscribe
- When user sends @user we execute subscribe user:\<uid\> channel in lua code.

| function | json format |
|--------|----------------------------------------- --------------------|
| Login | {"op":"auth","token":"***"} |
| Subscribe User Messages | {"op":"subscribe","channel":"@user"} |
| Unsubscribe User Messages | {"op":"unsubscribe","channel":"@user"} |
| User message event | {"event":"@user","data":{"uid":1001,"msg":"Hello,World!"}} |
| Success | {"result":true} |
| Error | {"code":1,"msg":"Error"} |

## Change setting

In the `options` option of the `connmix.yaml` configuration file, modify the url path of the websocket

```
options:
   - name: path
     value: /message-center
```

## CONNMIX encoding

Modify the `on_message` method of `entry.websocket.lua` as follows:

- When the message is of auth type, call the `auth_url` interface to get the uid through the token and save it in the context
- When the message is subscribe or unsubscribe, take out the uid from the context, and perform subscription/unsubscription corresponding to the channel

```lua
function on_message(msg)
     --print(msg)
     if msg["type"] ~= "text" then
         conn:close()
         return
     end

     local auth_url = "http://127.0.0.1:8000/websocket_auth" --fill in the api interface address for parsing token
     local conn = mix. websocket()

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
         --x-www-form-urlencoded
         -- body = 'token=' .. token
         --application/json
         -- body = '{"token:"' .. token .. '"}'
         local resp, err = mix.http.request("POST", auth_url, {
             body = '{"token:"' .. token .. '"}'
         })
         if err then
             mix_log(mix_DEBUG, "http. request error: " .. err)
             conn:close()
             return
         end
         if resp.status_code ~= 200 then
             mix_log(mix_DEBUG, "http. request status_code: " .. resp["status_code"])
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

## API encoding

### WebSocket login interface

Write a login information verification interface `/websocket_auth` in the framework of the existing system, which is used for ws login to obtain user uid

- Interface input parameter: token

```json
{"token":"***"}
```

- Interface output parameter: uid

```json
{"uid":1001}
```

### Send message interface

Implement proactive message push within the framework of existing systems

- An interface for sending user messages can be written in the framework of spring and laravel
- After verifying the user identity in this interface, execute the following http request to complete the push
- If you send requests very frequently, you can use [websocket-api push](en-us/websocket-api?id=grid-publishing-it-can-be-sent-to-all-client-connections-that-have-subscribed-to-these-channels-in-the-entire-grid-1) Improve performance

```
curl --request POST 'http://127.0.0.1:6789/v1/mesh/publish' \
--header 'Content-Type: application/json' \
--data-raw '{
     "c": "user:1001",
     "d": "{\"event\":\"@user\",\"data\":{\"uid\":1001,\"msg\":\"Hello,World!\"}}"
}'
```

## test

Use the websocket test tool to connect and test

- Connect to `ws://127.0.0.1:6790/message-center`
- Send `{"op":"auth","token":"***"}`
- received `{"result":true}`
- send `{"op":"subscribe","channel":"@user"}`
- received `{"result":true}`
- Execute curl active push
- Received `{"event":"@user","data":{"uid":1001,"msg":"Hello,World!"}}`
