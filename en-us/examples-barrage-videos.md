# WebSocket video barrage

It is very simple to use WebSocket to make a stand-alone barrage system, but when the stand-alone performance reaches the bottleneck and needs to be expanded to cluster deployment, it will face many distributed problems. If you use [CONNMIX](https://connmix.com/), you don’t need to worry For these problems, a high-performance distributed WebSocket cluster can be completed with very little code.

## Preconditions

- [Install Gateway](/en-us/install-engine.md)
- [Pubsub](/en-us/pubsub.md)
- [Quick Start](/en-us/start-debug.md)

## Design ideas

- After the client successfully connects to ws, it subscribes to a video ID channel `video:<videoid>`
- In the interface for sending barrage, call the `/v1/mesh/publish` interface of any connmix node to send real-time barrage to the corresponding `video_id`, and all ws clients subscribed to this channel will receive the message.
- The above are all incremental push designs, and the full amount is usually obtained through a full amount api interface when the page is loaded.

## Interaction protocol design

- When user sends 100001@video we execute subscribe video:100001 channel in lua code.
- When the front end switches videos, it can first send a cancel barrage message, and then send a new subscription barrage message.

| function | json format |
|------|------------------------------------------ --------------------------------------|
| Subscribe Barrage | {"op":"subscribe","channel":"100001@video"} |
| Unsubscribe Barrage | {"op":"unsubscribe","channel":"100001@video"} |
| Barrage event | {"event":"@video","data":{"uid":1001,"video_id":100001,"msg":"Hello,World!"}} |
| Success | {"result":true} |
| Error | {"code":1,"msg":"Error"} |

## Change setting

In the `options` option of the `connmix.yaml` configuration file, modify the url path of the websocket

```
options:
   - name: path
     value: /barrage-videos
```

## CONNMIX encoding

Modify the `on_message` method of `entry.websocket.lua` as follows:

- When subscribe and unsubscribe messages are received, subscribe/unsubscribe the corresponding channel

```lua
function on_message(msg)
     --print(msg)
     if msg["type"] ~= "text" then
         conn:close()
         return
     end

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
     local video_id = channel_table[1] --Lua's table index starts from 1 by default
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

## API encoding

Next, implement active barrage push in the framework of the existing system

- You can write a barrage sending interface in the framework of spring and laravel
- After verifying the user identity in this interface, execute the following http request to complete the push
- If you send requests very frequently, you can use [websocket-api push](en-us/websocket-api?id=%e7%bd%91%e6%a0%bc%e5%8f%91%e5%b8% 83%ef%bc%9a%e5%8f%af%e4%bb%a5%e5%8f%91%e9%80%81%e7%bb%99%e6%95%b4%e4%b8%aa% e7%bd%91%e6%a0%bc%e5%86%85%e6%89%80%e6%9c%89%e8%ae%a2%e9%98%85%e4%ba%86%e8% bf%99%e4%ba%9b%e9%a2%91%e9%81%93%e7%9a%84%e5%ae%a2%e6%88%b7%e7%ab%af%e8%bf% 9e%e6%8e%a5-1) Improve performance

```
curl --request POST 'http://127.0.0.1:6789/v1/mesh/publish' \
--header 'Content-Type: application/json' \
--data-raw '{
     "c": "video:100001",
     "d": "{\"event\":\"@video\",\"data\":{\"uid\":1001,\"video_id\":100001,\"msg\":\" Hello, World!\"}}"
}'
```

## test

Use the websocket test tool to connect and test

- Connect to `ws://127.0.0.1:6790/barrage-videos`
- send `{"op":"subscribe","channel":"100001@video"}`
- received `{"result":true}`
- Execute curl active push
- Received `{"event":"@video","data":{"uid":1001,"video_id":100001,"msg":"Hello,World!"}}`
