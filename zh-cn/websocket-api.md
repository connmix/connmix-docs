## API 协议

掌握该协议，用户可以自己定制 connmix 客户端，普通用户直接使用官方客户端即可。

> 协议：`websocket` 默认端口：`6789` 路径: `/ws/v1`

### Ping/Pong

Request

```json
{
  "m": "server.ping",
  "p": [],
  "i": 12345
}
```

Response

```json
{
  "E": null,
  "r": {
    "s": "server.pong"
  },
  "i": 12345
}
```

### 消费队列

Request

```json
{
  "m": "queue.pop",
  "p": [
    "foo"
  ],
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

```json
{
  "e": "queue.pop",
  "p": {
    "c": "1463819408261513216",
    "t": "foo",
    "d": {
      "ctx": [],
      "msg": "foo"
    }
  },
  "id": null
}
```

### 取消队列消费

> 暂时只支持取消全部

Request

```json
{
  "m": "queue.unpop",
  "p": [],
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
    "c": 202,
    "m": "*****"
  },
  "i": 12345
}
```

### 远程执行 `conn` 设置上下文

Request

- `set_context_value`

```json
{
  "m": "conn.call",
  "p": [
    {
      "c": "1458271556210786304",
      "f": "set_context_value",
      "a": [
        "user_id",
        10000
      ]
    }
  ],
  "i": 12345
}
```

- `set_context`

```json
{
  "m": "conn.call",
  "p": [
    {
      "c": "1458271556210786304",
      "f": "set_context",
      "a": [
        {
          "user_id": 10000
        }
      ]
    }
  ],
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

### 远程执行 `conn` 订阅频道

Request

- `subscribe`

```json
{
  "m": "conn.call",
  "p": [
    {
      "c": "1463784730422935552",
      "f": "subscribe",
      "a": [
        "channel1",
        "channel2"
      ]
    }
  ],
  "i": 12345
}
```

- `unsubscribe`

```json
{
  "m": "conn.call",
  "p": [
    {
      "c": "1463784730422935552",
      "f": "unsubscribe",
      "a": [
        "channel1",
        "channel2"
      ]
    }
  ],
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

```json
{
  "m": "mesh.send",
  "p": [
    {
      "c": "1463786910324359168",
      "d": "123456789"
    }
  ],
  "id": 12345
}
```

Response

```json
{
  "E": null,
  "r": {
    "s": true,
    "f": 0,
    "t": 1
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

```json
{
  "m": "mesh.publish",
  "p": [
    {
      "c": "channel1",
      "d": "123456789"
    }
  ],
  "i": 12345
}
```

Response

```json
{
  "E": null,
  "r": {
    "s": true,
    "f": 0,
    "t": 9
  },
  "i": 12345
}
```

Error

- `101` 无效参数

```json
{
  "E": {
    "c": 401,
    "m": "*****"
  },
  "i": 12345
}
```
