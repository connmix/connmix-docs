# HTTP / WebSocket API protocol

In existing systems written in any language, functions such as active push and data reporting to MQ can be completed by calling the API interface.

## HTTP API

> Protocol: `http` Default Port: `6789` Path: `/v1`

### Grid sending: client connections that can be sent to any machine in the entire grid

> POST `/v1/mesh/send`

request

- `c` client client_id
- the data sent by `d`
- `t` message type: binary type needs to use base64 encoding data (t=text,binary)

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

error

- `101` invalid argument
- `401` failed to send

```json
{
   "E": {
     "c": 401,
     "m": "*****"
   }
}
```

### Grid publishing: it can be sent to all client connections that have subscribed to these channels in the entire grid

> POST `/v1/mesh/publish`

request

- `c` channel
- the data sent by `d`
- `t` message type: binary type needs to use base64 encoding data (t=text,binary)

```json
{
   "c": "channel1",
   "d": "data"
}
```

Response

- `s` number of successes
- `f` number of failures

```json
{
   "E": null,
   "r": {
     "s": 9,
     "f": 0
   }
}
```

error

- `101` invalid argument

```json
{
   "E": {
     "c": 101,
     "m": "*****"
   }
}
```

## WebSocket API

> Protocol: `websocket` Default Port: `6789` Path: `/ws/v1`

### Ping/Pong

Sending a ping frame will reply a pong frame, javascript can use the following methods

request

```
ping
```

Response

```
pong
```

### Consumption Queue

request

- topic array consumed by `t`

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

error

- `101` Invalid argument
- `201` consumption failed
- `202` repeated consumption

```json
{
   "E": {
     "c": 202,
     "m": "*****"
   },
   "i": 12345
}
```

event

- `n` node node_id
- `c` client client_id
- `t` which topic the message came from
- the data of the `d` message

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

### Cancel queue consumption

Note: Temporarily only supports cancel all

request

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

error

- `203` Failed to cancel consumption

```json
{
   "E": {
     "c": 203,
     "m": "*****"
   },
   "i": 12345
}
```

### Remote setting connection context

request

> `set_context_value`

- `c` client client_id
- the name of the function to be executed by `f`
- `a` parameter to execute the function

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

- `c` client client_id
- the name of the function to be executed by `f`
- `a` parameter to execute the function

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

error

- `101` Invalid parameter
- `301` execution failed

```json
{
   "E": {
     "c": 301,
     "m": "*****"
   },
   "i": 12345
}
```

### Remote operation connection subscription channel

request

> `subscribe`

- `c` client client_id
- the name of the function to be executed by `f`
- `a` parameter to execute the function

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

- `c` client client_id
- the name of the function to be executed by `f`
- `a` parameter to execute the function

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

error

- `101` Invalid argument
- `301` execution failed

```json
{
   "E": {
     "c": 301,
     "m": "*****"
   },
   "i": 12345
}
```

### Grid sending: client connections that can be sent to any machine in the entire grid

request

- `c` client client_id
- the data sent by `d`
- `t` message type: binary type needs to use base64 encoding data (t=text,binary)

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

error

- `101` invalid argument
- `401` failed to send

```json
{
   "E": {
     "c": 401,
     "m": "*****"
   },
   "i": 12345
}
```

### Grid publishing: it can be sent to all client connections that have subscribed to these channels in the entire grid

request

- `c` channel
- the data sent by `d`
- `t` message type: binary type needs to use base64 encoding data (t=text,binary)

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

- `s` number of successes
- `f` number of failures

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

error

- `101` invalid argument

```json
{
   "E": {
     "c": 101,
     "m": "*****"
   },
   "i": 12345
}
```
