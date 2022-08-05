# Lua API

## `mix`

| 方法                                       | 描述                                    |
|------------------------------------------|---------------------------------------|
| s, err = mix.json_encode(data)           |                                       |
| tb, err = mix.json_decode(str)           |                                       |
| s = mix.base64_encode(data)              |                                       |
| s, err = mix.base64_decode(str)          |                                       |
| s, err = mix.php_pack(format, args+)     |                                       |
| tb, err = mix.php_unpack(format, data)   |                                       |
| tb = mix.str_split(str, sep)             |                                       |
| tb, err = mix.url_parse(url)             |                                       |
| s = mix.bytes_tostring(bytes_table)      |                                       |
| s = mix.md5(data)                        |                                       |
| s = mix.md5_bin(data)                    |                                       |
| s = mix.sha1(data)                       |                                       |
| s = mix.sha1_bin(data)                   |                                       |
| b = mix.log(mix.INFO, msg)               | mix.DEBUG,mix.INFO,mix.WARN,mix.ERROR |

## `mix.server`

| 方法                                                   | 描述                         |
|-------------------------------------------------------|-----------------------------|
| s = mix.server.option(name)                           |                             |

## `mix.conn`

| 方法                           | 描述                    |
|------------------------------|-----------------------|
| conn, err = mix.websocket()  | websocket 协议获取用户端连接对象 |
| conn, err = mix.socket.tcp() | custom 协议获取用户端连接对象    |

| 方法                                             | 描述        |
|------------------------------------------------|-----------|
| n = conn:client_id()                           |           |
| s = conn:remote_addr()                         |           |
| tb = conn:context()                            |           |
| any = conn:context_value("key")                |           |
| any = conn:wait_context_value("user_id")       | 阻塞等待返回值   |
| conn:set_context({ user_id = 10000 })          | 可远程执行,本节点 |
| conn:set_context_value("user_id", 10000)       | 可远程执行,本节点 |
| err = conn:subscribe("channel1", "channel2")   | 可远程执行,本节点 |
| err = conn:unsubscribe("channel1", "channel2") | 可远程执行,本节点 |
| err = conn:send(data[, type])                  |           |
| err = conn:close()                             |           |
| conn:clear_buffer()                            |           |

## `mix.mesh`

| 方法                                                    | 描述                          |
|-------------------------------------------------------|-----------------------------|
| err = mix.mesh.send(client_id, data[, type])          | 可远程执行,任意节点 (type=text,binary) |
| success, fail = mix.mesh.publish(channel, data[, type]) | 可远程执行,任意节点 (type=text,binary) |

## `mix.queue`

| 方法                                   | 描述                                    |
|--------------------------------------|---------------------------------------|
| n, err = mix.queue.push(topic, data) | yaml中配置max,size                       |

## `mix.redis`

| 方法                                      | 描述                       |
|-----------------------------------------|--------------------------|
| redis, err = mix.redis.connection(name) | 获取对应配置的连接对象, yaml中配置连接信息 |

| 方法                                              | 描述           |
|-------------------------------------------------|--------------|
| any, err = redis:command("set", "key", "value") |              |

## `mix.http`

| 方法                                                    | 描述  |
|-------------------------------------------------------|-----|
| resp, err = mix.http.request(method, url [, options]) |     |

options

| Name    | Type          | Description                                                                                                                          |
|---------|---------------|--------------------------------------------------------------------------------------------------------------------------------------|
| query   | String        | URL encoded query params                                                                                                             |
| cookies | Table         | Additional cookies to send with the request                                                                                          |
| headers | Table         | Additional headers to send with the request                                                                                          |
| timeout | Number/String | Request timeout. Number of seconds or String such as "1h"                                                                            |
| auth    | Table         | Username and password for HTTP basic auth. Table keys are *user* for username, *pass* for passwod. `auth={user="user", pass="pass"}` |

## `mix.buffer`

| 方法                   | 描述  |
|----------------------|-----|
| s = buf:tostring()   |     |
| s, err = buf:read(2) |     |
| n = buf:position()   |     |
| n = buf:length()     |     |
