# Modify Configuration

- Open configuration file `conf/connmix.yaml`
- We only need to configure the `servers` setting
- By default, a service with port `6790` is already configured
- This service uses the built-in `websocket` protocol, and the entry file is `lua/entry.websocket.lua`
- The bytecode cache time is `10` seconds, which means that the modified Lua code will be automatically hot updated after 10 seconds. Set it to `0` to disable hot updates.

```
  servers:
    - port: 6790
      bind: 0.0.0.0
      protocol: websocket    # (websocket,socket)
      options:
        - name: path
          value: /chat
      entry: lua/entry.websocket.lua
      byteCodeExpires: 10    # Set to 0 to disable hot updates
      readBufferSize: 1024   # Takes effect on new connections after modification
      writeBufferSize: 1024  # Takes effect on new connections after modification
      readTimeout: 60        # Takes effect on new connections after modification
      writeTimeout: 10       # Takes effect on new connections after modification
```
