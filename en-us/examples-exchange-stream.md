# WebSocket digital exchange market system

Digital currency exchanges have gradually matured in recent years. Unlike the previous barbaric era, the technology is becoming more and more advanced. Although the blockchain is decentralized, the exchange is a centralized system, which is no different from the Internet industry; and Different from the traditional financial industry, WebSocket is widely used in the market system in digital currency exchanges, and is usually used for Web/App to obtain real-time transaction information such as market prices, indexes, and user transactions;

[CONNMIX](https://connmix.com/) is a distributed long-connection gateway based on go and lua, which can build a high-performance exchange market push system in a short time.

## Preconditions

- [Install Gateway](/en-us/install-engine.md)
- [Pubsub](/en-us/pubsub.md)
- [Quick Start](/en-us/start-debug.md)

## demand analysis

Usually the exchange push system includes the following parts:

- `Public interface` No need to log in, you can get market data at will, including orderbook, trades, mark_price, ticker, kline and other information
- `Private interface` needs to send a message to log in to get the push of personal information such as balance, order, position, account, etc.

## Module design

It can be seen from the following module design that CONNMIX makes the requirement design more pure, and programmers do not need to consider user connection processing, but only need to focus on business logic.

| module name | description | tools |
|-----------------------------|------------------- -------------------------------------------------- --------------------------------------|------------- ----|
| User connection push | Responsible for connecting with users, subscribing to a certain channel according to the business, and realizing active push according to the channel | CONNMIX |
| Public push calculation orderbook | Get book snapshots of the matching system at regular intervals, compare them to get the changed positions, and push the calculation results to `<symbol>@depth` | java,php,go ... |
| Public push calculation trades | Obtain real-time trades information from the matching system through UDP/TCP, and push it to `<symbol>@trades` after frequency reduction and merger | java,php,go ... |
| Public push calculation mark_price | Mark price push usually includes index price, estimated settlement price, funding rate and other information, which need to be calculated according to the product formula and pushed to `<symbol>@markprice` | java,php,go ... |
| Public push calculation ticker | Usually records opening, high and low within 24 hours plus the current latest price, quantity, and quota and pushes it to `<symbol>@ticker` at regular intervals | java,php,go ... |
| Public push calculation kline | This push is usually called in the front-end TradingView, and the back-end program records the opening, high, low, and closing within a certain time interval, and regularly pushes it to `<symbol>@kline` | java,php ,go... |
| Private push login design | Just add an interface for ws login in the existing system, and return the corresponding uid | java, php, go ... |
| Private push balance, order, position | Change information of balance, order, position is obtained by the matching system, uid is extracted from the information, and then sent to `<uid>@balance`, `<uid>@order`, ` <uid>@position` | java, php, go ... |
| Private push account | Account changes are usually triggered by rest, extract the uid from the information, and then send it to `<uid>@account` | java,php,go ... |

## Interaction protocol design

message design

| function | json format |
|--------|----------------------------------------- ----------------|
| Login | {"op":"auth","token":"***"} |
| Subscribe channel | {"op":"subscribe","channel":"BTCUSDT@depth"} |
| Unsubscribe Channel | {"op":"unsubscribe","channel":"BTCUSDT@depth"} |
| User channel message | {"event":"BTCUSDT@depth","data":{"bids":[],"asks":[]}} |
| Success | {"result":true} |
| Error | {"code":1,"msg":"Error"} |

channel design

| Function | Channel | CONNMIX Internal Channel Format |
|------------|----------------------|------------- ---------|
| orderbook | `<symbol>@depth` | `<symbol>@depth` |
| trades | `<symbol>@trades` | `<symbol>@trades` |
| mark_price | `<symbol>@markprice` | `<symbol>@markprice` |
| ticker | `<symbol>@ticker` | `<symbol>@ticker` |
| kline | `<symbol>@kline` | `<symbol>@kline` |
| balance | `@balance` | `<uid>@balance` |
| order | `@order` | `<uid>@order` |
| position | `@position` | `<uid>@position` |
| account | `@account` | `<uid>@account` |

## Change setting

In the `options` option of the `connmix.yaml` configuration file, modify the url path of the websocket

```
options:
   - name: path
     value: /exchange-stream
```

## CONNMIX encoding

Modify the `on_message` method of `entry.websocket.lua` as follows:

- When the message is of auth type, call the `auth_url` interface to get the uid through the token and save it in the context
- When the message is subscribe or unsubscribe, the uid is taken from the context and spliced into the internal channel, and the corresponding channel is subscribed/unsubscribed

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
         -- x-www-form-urlencoded
         -- body = 'token=' .. token
         -- application/json
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
     local inner_channel = ""
     if channel_type == "balance" or channel_type == "order" or channel_type == "position" or channel_type == "account" then
         --Requires login to subscribe
         if uid == nil then
             conn:send('{"code":1,"msg":"Not Auth"}')
             return
         end
         inner_channel = string. format("%d@%s", uid, channel_type)
     else
         inner_channel = channel_raw
     end
    
     -- Set prefix for security
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

### Internal message push

Complete the active message push of the corresponding channel in the framework of the existing system

- You can execute the following http request in the resident program written in the spring and laravel framework to complete the push
- If you send requests very frequently, you can use [websocket-api push](en-us/websocket-api?id=grid-publishing-it-can-be-sent-to-all-client-connections-that-have-subscribed-to-these-channels-in-the-entire-grid-1) Improve performance

```
curl --request POST 'http://127.0.0.1:6789/v1/mesh/publish' \
--header 'Content-Type: application/json' \
--data-raw '{
     "c": "ex:1001@balance",
     "d": "{\"event\":\"@balance\",\"data\":{\"uid\":1001,\"balance\":[{\"currency\":\" BTC\",\"available\":100,\"unavailable\":100}]}}"
}'
```

## test

Use the websocket test tool to connect and test

- Connect to `ws://127.0.0.1:6790/exchange-stream`
- Send `{"op":"auth","token":"***"}`
- received `{"result":true}`
- send `{"op":"subscribe","channel":"@balance"}`
- received `{"result":true}`
- Execute curl active push
- Receive `{"event":"@balance","data":{"uid":1001,"balance":[{"currency":"BTC","available":100,"unavailable":100 }]}}`
