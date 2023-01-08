# HTTP / WebSocket API 协议

可以在任意语言编写的现有系统中，通过调用API接口即可完成主动推送、上报数据到MQ等功能。

## HTTP API

> 协议：`http` 默认端口：`6789` 路径: `/v1`

### 网格发送：可以发送给整个网格的任意机器的客户端连接

> POST `/v1/mesh/send`

Request

- `c` 客户端 client_id
- `d` 发送的数据
- `t` 消息类型: binary类型需要使用base64编码data (t=text,binary)

```json
{
  "c": "1463786910324359168",
  "d": "data"
}
```

Response

```json
{
  "E": null,
  "r": {
    "s": true
  }
}
```

Error

- `101` 无效参数
- `401` 发送失败

```json
{
  "E": {
    "c": 401,
    "m": "*****"
  }
}
```

### 网格发布：可以发送给整个网格内所有订阅了这些频道的客户端连接

> POST `/v1/mesh/publish`

Request

- `c` 通道
- `d` 发送的数据
- `t` 消息类型: binary类型需要使用base64编码data (t=text,binary)

```json
{
  "c": "channel1",
  "d": "data"
}
```

Response

- `s` 成功数量
- `f` 失败数量

```json
{
  "E": null,
  "r": {
    "s": 9,
    "f": 0
  }
}
```

Error

- `101` 无效参数

```json
{
  "E": {
    "c": 101,
    "m": "*****"
  }
}
```

## WebSocket API

> 协议：`websocket` 默认端口：`6789` 路径: `/ws/v1`

### Ping/Pong

发送ping帧会回复pong帧，javascript可以使用以下方式

Request

```
ping
```

Response

```
pong
```

### 消费队列

Request

- `t` 消费的topic数组

```json
{
  "m": "queue.pop",
  "p": {
    "t": [
      "topic1"
    ]
  },
  "i": 12345
}
```

Response

```json
{
  "E": null,
  "r": {
    "s": true
  },
  "i": 12345
}
```

Error

- `101` 参数无效
- `201` 消费失败
- `202` 重复消费

```json
{
  "E": {
    "c": 202,
    "m": "*****"
  },
  "i": 12345
}
```

Event

- `n` 节点 node_id
- `c` 客户端 client_id
- `t` 消息来自哪个 topic
- `d` 消息的数据

```json
{
  "e": "queue.pop",
  "p": {
    "n": 1,
    "c": "1463819408261513216",
    "t": "topic1",
    "d": {
      "frame": {
        "type": "text",
        "data": "json str",
        "finish": true
      }
    }
  },
  "id": null
}
```

### 取消队列消费

注意：暂时只支持取消全部

Request

```json
{
  "m": "queue.unpop",
  "p": {},
  "i": 12345
}
```

Response

```json
{
  "E": null,
  "r": {
    "s": true
  },
  "i": 12345
}
```

Error

- `203` 取消消费失败

```json
{
  "E": {
    "c": 203,
    "m": "*****"
  },
  "i": 12345
}
```

### 远程设置连接上下文

Request

> `set_context_value`

- `c` 客户端 client_id
- `f` 执行的函数名称
- `a` 执行函数的参数

```json
{
  "m": "conn.call",
  "p": {
    "c": "1458271556210786304",
    "f": "set_context_value",
    "a": [
      "user_id",
      10000
    ]
  },
  "i": 12345
}
```

> `set_context`

- `c` 客户端 client_id
- `f` 执行的函数名称
- `a` 执行函数的参数

```json
{
  "m": "conn.call",
  "p": {
    "c": "1458271556210786304",
    "f": "set_context",
    "a": [
      {
        "user_id": 10000
      }
    ]
  },
  "i": 12345
}
```

Response

```json
{
  "E": null,
  "r": {
    "s": true
  },
  "i": 12345
}
```

Error

- `101` 参数无效
- `301` 执行失败

```json
{
  "E": {
    "c": 301,
    "m": "*****"
  },
  "i": 12345
}
```

### 远程操作连接订阅频道

Request

> `subscribe`

- `c` 客户端 client_id
- `f` 执行的函数名称
- `a` 执行函数的参数

```json
{
  "m": "conn.call",
  "p": {
    "c": "1463784730422935552",
    "f": "subscribe",
    "a": [
      "channel1",
      "channel2"
    ]
  },
  "i": 12345
}
```

> `unsubscribe`

- `c` 客户端 client_id
- `f` 执行的函数名称
- `a` 执行函数的参数

```json
{
  "m": "conn.call",
  "p": {
    "c": "1463784730422935552",
    "f": "unsubscribe",
    "a": [
      "channel1",
      "channel2"
    ]
  },
  "i": 12345
}
```

Response

```json
{
  "E": null,
  "r": {
    "s": true
  },
  "i": 12345
}
```

Error

- `101` 参数无效
- `301` 执行失败

```json
{
  "E": {
    "c": 301,
    "m": "*****"
  },
  "i": 12345
}
```

### 网格发送：可以发送给整个网格的任意机器的客户端连接

Request

- `c` 客户端 client_id
- `d` 发送的数据
- `t` 消息类型: binary类型需要使用base64编码data (t=text,binary)

```json
{
  "m": "mesh.send",
  "p": {
    "c": "1463786910324359168",
    "d": "data"
  },
  "id": 12345
}
```

Response

```json
{
  "E": null,
  "r": {
    "s": true
  },
  "i": 12345
}
```

Error

- `101` 无效参数
- `401` 发送失败

```json
{
  "E": {
    "c": 401,
    "m": "*****"
  },
  "i": 12345
}
```

### 网格发布：可以发送给整个网格内所有订阅了这些频道的客户端连接

Request

- `c` 通道
- `d` 发送的数据
- `t` 消息类型: binary类型需要使用base64编码data (t=text,binary)

```json
{
  "m": "mesh.publish",
  "p": {
    "c": "channel1",
    "d": "data"
  },
  "i": 12345
}
```

Response

- `s` 成功数量
- `f` 失败数量

```json
{
  "E": null,
  "r": {
    "s": 9,
    "f": 0
  },
  "i": 12345
}
```

Error

- `101` 无效参数

```json
{
  "E": {
    "c": 101,
    "m": "*****"
  },
  "i": 12345
}
```
