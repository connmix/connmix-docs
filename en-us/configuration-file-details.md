# Configuration File Explanation

- Configurations without special annotations cannot be hot-updated and require a gateway restart to take effect.

```yaml
logging:
  filename: logs/connmix.log
  maxFiles: 7
  level: debug # (debug,info,warn,error) Takes effect after modification

prometheus:
  summary:
    maxAge: 600
    ageBuckets: 5
    bufCap: 500

licenses:
  activationCode:
  server: https://connmix.com

access: # Takes effect after modification
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
    readBufferSize: 40096  # Takes effect for new connections after modification
    writeBufferSize: 40096 # Takes effect for new connections after modification
    readTimeout: 60        # Takes effect for new connections after modification
    writeTimeout: 60       # Takes effect for new connections after modification
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
      byteCodeExpires: 10    # Set to 0 to disable hot updates
      readBufferSize: 1024   # Takes effect for new connections after modification
      writeBufferSize: 1024  # Takes effect for new connections after modification
      readTimeout: 60        # Takes effect for new connections after modification
      writeTimeout: 10       # Takes effect for new connections after modification
```
