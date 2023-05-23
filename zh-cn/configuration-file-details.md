# 配置文件详解

- 没有特殊标注的配置，都无法热更新，需要重启网关才能生效

```yaml
logging:
  filename: logs/connmix.log
  maxFiles: 7
  level: debug # (debug,info,warn,error) 修改后立即生效

prometheus:
  summary:
    maxAge: 600
    ageBuckets: 5
    bufCap: 500

licenses:
  activationCode:
  server: https://connmix.com

access: # 修改后立即生效
  users:
#    - username: user
#      password: pass
#      roles:
#        - read
#        - write
#      host: * # 172.*.*.*

center:
  registry:
    port: 6786
    bind: 0.0.0.0
  apiServer:
    port: 6787
    bind: 0.0.0.0

engine:
  centerRegistry: localhost:6786
  meshPoint:
    port: 6788
    bind: 0.0.0.0
  apiServer:
    port: 6789
    bind: 0.0.0.0
    readBufferSize: 40096  # 修改后对新连接生效
    writeBufferSize: 40096 # 修改后对新连接生效
    readTimeout: 60        # 修改后对新连接生效
    writeTimeout: 60       # 修改后对新连接生效
  luavm:
    queue:
      topicSize: 2000
      maxTopics: 100
    redis:
      - name: default
        addr: localhost:6379
        password:
        db: 0
        poolSize: 20
        minIdleConns: 10
        maxConnAge: 0
        idleTimeout: 300
  servers:
    - port: 6790
      bind: 0.0.0.0
      protocol: websocket    # (websocket,socket)
      options:
        - name: path
          value: /chat
      entry: lua/entry.websocket.lua
      byteCodeExpires: 10    # 设置0关闭热更新
      readBufferSize: 1024   # 修改后对新连接生效
      writeBufferSize: 1024  # 修改后对新连接生效
      readTimeout: 60        # 修改后对新连接生效
      writeTimeout: 10       # 修改后对新连接生效
```
