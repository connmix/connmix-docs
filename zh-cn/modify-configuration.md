# 修改配置

- 打开配置文件 `conf/connmix.yaml`
- 我们只需要配置 `servers` 配置即可
- 默认已经配置了一个 `6790` 端口的服务
- 该服务采用的协议入口文件为 `lua/entry.lua`
- 字节码缓存时间为 `10` 秒，也就是修改 lua 代码 10s 后自动热更新，配置为 `0` 关闭热更新。

```
  servers:
    - port: 6790
      bind: 0.0.0.0
      entry: lua/entry.lua
      bytecode_expires: 10
      read_buffer_size: 1024  # 修改后对新连接生效
      read_timeout: 60        # 修改后对新连接生效
      write_timeout: 10       # 修改后对新连接生效
```
