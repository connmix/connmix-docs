# 配置文件详解

- 没有特殊标注的配置，都无法热更新，需要重启引擎才能生效

```yaml
log:
  filename: logs/connmix.log
  max_files: 7
  level: info # (debug,info,warn,error) 修改后生效

licenses:
  activation_code:
  server: https://connmix.com

center:
  registry:
    port: 6786
    bind: 0.0.0.0
  server:
    port: 6787
    bind: 0.0.0.0

engine:
  center_registry: 127.0.0.1:6786
  mesh_point:
    port: 6788
    bind: 0.0.0.0
  control:
    protocol: websocket
    port: 6789
    bind: 0.0.0.0
    read_buffer_size: 1024  # 修改后对新连接生效
    write_buffer_size: 1024 # 修改后对新连接生效
    read_timeout: 600       # 修改后对新连接生效
    write_timeout: 10       # 修改后对新连接生效
  servers:
    - port: 6790
      bind: 0.0.0.0
      entry: lua/entry.lua
      bytecode_expires: 10    # 设置0关闭热更新
      read_buffer_size: 1024  # 修改后对新连接生效
      read_timeout: 60        # 修改后对新连接生效
      write_timeout: 10       # 修改后对新连接生效
```
