# Lua API

通过 connmix 内置的这些 lua 库，我们可以编写复杂的业务逻辑，同时也可以自定义网络通讯协议。

### `mix`

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

### `mix.server`

| 方法                          | 描述  |
|-----------------------------|-----|
| s = mix.server.option(name) |     |

### `mix.conn`

| 协议        | 方法                           | 描述                    |
|-----------|------------------------------|-----------------------|
| socket    | conn, err = mix.socket.tcp() | socket协议获取用户端连接对象     |
| websocket | conn, err = mix.websocket()  | websocket 协议获取用户端连接对象 |

| 协议        | 方法                                             | 描述               |
|-----------|------------------------------------------------|------------------|
| all       | n = conn:client_id()                           |                  |
| all       | s = conn:remote_addr()                         |                  |
| all       | tb = conn:context()                            |                  |
| all       | any = conn:context_value("key")                |                  |
| all       | any = conn:wait_context_value("user_id")       | 阻塞等待返回值          |
| all       | conn:set_context({ user_id = 10000 })          | 可远程执行,本节点        |
| all       | conn:set_context_value("user_id", 10000)       | 可远程执行,本节点        |
| all       | err = conn:subscribe("channel1", "channel2")   | 可远程执行,本节点        |
| all       | err = conn:unsubscribe("channel1", "channel2") | 可远程执行,本节点        |
| all       | err = conn:close()                             |                  |
| socket    | conn:clear_buffer()                            |                  |
| socket    | err = conn:send(data)                          |                  |
| websocket | err = conn:send(data[, type])                  | type=text,binary |

### `mix.mesh`

| 协议        | 方法                                                      | 描述                            |
|-----------|---------------------------------------------------------|-------------------------------|
| socket    | err = mix.mesh.send(client_id, data)                    | 可远程执行，任意节点                    |
| socket    | success, fail = mix.mesh.publish(channel, data)         | 可远程执行，任意节点                    |
| websocket | err = mix.mesh.send(client_id, data[, type])            | 可远程执行，任意节点 (type=text,binary) |
| websocket | success, fail = mix.mesh.publish(channel, data[, type]) | 可远程执行，任意节点 (type=text,binary) |

### `mix.queue`

| 方法                                   | 描述                                    |
|--------------------------------------|---------------------------------------|
| n, err = mix.queue.push(topic, data) | yaml中配置max,size                       |

### `mix.redis`

| 方法                                      | 描述                       |
|-----------------------------------------|--------------------------|
| redis, err = mix.redis.connection(name) | 获取对应配置的连接对象, yaml中配置连接信息 |

| 方法                                              | 描述           |
|-------------------------------------------------|--------------|
| any, err = redis:command("set", "key", "value") |              |

### `mix.http`

| 方法                                                    | 描述  |
|-------------------------------------------------------|-----|
| resp, err = mix.http.request(method, url [, options]) |     |

options

| Name    | Type          | Description                                                                       |
|---------|---------------|-----------------------------------------------------------------------------------|
| query   | String        | URL encoded query params                                                          |
| cookies | Table         | Additional cookies to send with the request                                       |
| body    | String        | Request body.                                                                     |
| headers | Table         | Additional headers to send with the request                                       |
| timeout | Number/String | Request timeout. Number of seconds or String such as "1h"                         |
| auth    | Table         | Username and password for HTTP basic auth. Table keys are *username*, *password*. |

response

| Name        | Type   | Description                                                 |
|-------------|--------|-------------------------------------------------------------|
| body        | String | The HTTP response body                                      |
| body_size   | Number | The size of the HTTP reponse body in bytes                  |
| headers     | Table  | The HTTP response headers                                   |
| cookies     | Table  | The cookies sent by the server in the HTTP response         |
| status_code | Number | The HTTP response status code                               |
| url         | String | The final URL the request ended pointing to after redirects |

### `mix.buffer`

| 方法                   | 描述  |
|----------------------|-----|
| s = buf:tostring()   |     |
| s, err = buf:read(2) |     |
| n = buf:position()   |     |
| n = buf:length()     |     |
