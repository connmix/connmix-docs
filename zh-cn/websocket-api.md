# API Server

协议：`websocket` 默认端口：`6789` 路径: `/ws/v1`

### Ping

Request

```json
{"method":"server.ping","params":[],"id":12345}
```

Response

```json
{"error":null,"result":["pong"],"id":12345}
```

### 消费队列

Request

```json
{"method":"queue.consume","params":["foo"],"id":12345}
```

Response

```json
{"error":null,"result":[{"success":true}],"id":12345}
```

Push

```json
{"method":"queue.pop","params":[{"client_id":"1463819408261513216","queue":"foo","data":{"ctx":[],"msg":"foo"}}],"id":null}
```

### 取消队列消费

暂时只支持取消全部

Request

```json
{"method":"queue.unconsume","params":[],"id":12345}
```

Response

```json
{"error":null,"result":[{"success":true}],"id":12345}
```

### 远程执行 `conn` 设置上下文

Request

```json
{"method":"conn.call","params":[{"client_id":"1458271556210786304","method":"set_context_value","params":["user_id",10000]}],"id":12345}
```

Response

```json
{"error":null,"result":[{"success":true}],"id":12345}
```

### 远程执行 `conn` 订阅频道

Request

```json
{"method":"conn.call","params":[{"client_id":"1463784730422935552","method":"subscribe","params":["channel1","channel2"]}],"id":12345}
```

Response

```json
{"error":null,"result":[{"success":true}],"id":12345}
```

### 网格发送：可以发送给整个网格的任意机器的客户端连接

Request

```json
{"method":"mesh.send","params":[{"client_id":"1463786910324359168","data":"123456789"}],"id":12345}
```

Response

```json
{"error":null,"result":[{"fail":0,"success":true,"total":1}],"id":12345}
```

### 网格发布：可以发送给整个网格内所有订阅了这些频道的客户端连接

Request

```json
{"method":"mesh.publish","params":[{"channel":"channel1","data":"123456789"}],"id":12345}
```

Response

```json
{"error":null,"result":[{"fail":0,"success":true,"total":9}],"id":12345}
```
