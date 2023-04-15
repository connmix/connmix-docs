# Lua API

Through the built-in lua libraries of connmix, we can write complex business logic and customize network communication protocols.

### `mix`

| Method | Description |
|---------------------------------------------|------ ------------------------------------|
| s, err = mix.json_encode(data) | |
| tb, err = mix.json_decode(str) | |
| s = mix.base64_encode(data) | |
| s, err = mix.base64_decode(str) | |
| s, err = mix.php_pack(format, args+) | |
| tb, err = mix.php_unpack(format, data) | |
| tb = mix.str_split(str, sep) | |
| tb, err = mix.url_parse(url) | |
| s = mix.bytes_tostring(bytes_table) | |
| s = mix.md5(data) | |
| s = mix.md5_bin(data) | |
| s = mix.sha1(data) | |
| s = mix.sha1_bin(data) | |
| b = mix.log(mix.INFO, msg) | mix.DEBUG,mix.INFO,mix.WARN,mix.ERROR|

### `mix.server`

| Method                      | Description |
|-----------------------------|-------------|
| s = mix.server.option(name) |             |

### `mix.conn`

| Protocol | Method | Description |
|-----------|---------------------------------|------ -----------------|
| socket | conn, err = mix.socket.tcp() | socket protocol to obtain the client connection object |
| websocket | conn, err = mix.websocket() | websocket protocol to get the client connection object |

| Protocol | Method | Description |
|-----------|------------------------------------- -----------|------------------|
| all | n = conn:client_id() | |
| all | s = conn:remote_addr() | |
| all | tb = conn:context() | |
| all | any = conn:context_value("key") | |
| all | any = conn:wait_context_value("user_id") | block waiting for return value |
| all | conn:set_context({ user_id = 10000 }) | Can be executed remotely, this node |
| all | conn:set_context_value("user_id", 10000) | Can be executed remotely, on this node |
| all | err = conn:subscribe("channel1", "channel2") | can be executed remotely, this node |
| all | err = conn:unsubscribe("channel1", "channel2") | can be executed remotely, this node |
| all | err = conn:close() | |
| socket | conn:clear_buffer() | |
| socket | err = conn:send(data) | |
| websocket | err = conn:send(data[, type]) | type=text,binary |

### `mix.mesh`

| Protocol | Method | Description |
|-----------|------------------------------------- --------------------|----------------------------- --|
| socket | err = mix.mesh.send(client_id, data) | remote execution, any node |
| socket | success, fail = mix.mesh.publish(channel, data) | can be executed remotely, any node |
| websocket | err = mix.mesh.send(client_id, data[, type]) | can be executed remotely, any node (type=text,binary) |
| websocket | success, fail = mix.mesh.publish(channel, data[, type]) | can be executed remotely, any node (type=text,binary) |

### `mix.queue`

| Method | Description |
|----------------------------------------|---------- -----------------------------|
| n, err = mix.queue.push(topic, data) | Configure max, size in yaml |

### `mix.redis`

| Method | Description |
|--------------------------------------------|------- -------------------|
| redis, err = mix.redis.connection(name) | Get the connection object corresponding to the configuration, and configure the connection information in yaml |

| Method                                          | Description |
|-------------------------------------------------|-------------|
| any, err = redis:command("set", "key", "value") |             |

### `mix.http`

| Method | Description |
|------------------------------------------------- ------|-----|
| resp, err = mix.http.request(method, url [, options]) | |

options

| Name | Type | Description |
|---------|---------------|----------------------- -------------------------------------------------- ----------|
| query | String | URL encoded query params |
| cookies | Table | Additional cookies to send with the request |
| body | String | Request body. |
| headers | Table | Additional headers to send with the request |
| timeout | Number/String | Request timeout. Number of seconds or String such as "1h" |
| auth | Table | Username and password for HTTP basic auth. Table keys are *username*, *password*. |

response

| Name | Type | Description |
|--------------|--------|----------------------- --------------------------------------|
| body | String | The HTTP response body |
| body_size | Number | The size of the HTTP response body in bytes |
| headers | Table | The HTTP response headers |
| cookies | Table | The cookies sent by the server in the HTTP response |
| status_code | Number | The HTTP response status code |
| url | String | The final URL the request ended pointing to after redirects |

### `mix.buffer`

| Method               | Description |
|----------------------|-------------|
| s = buf:tostring()   |             |
| s, err = buf:read(2) |             |
| n = buf:position()   |             |
| n = buf:length()     |             |
