# 常见问题

### 如何获取 websocket 连接的 GET 参数

使用 `mix.parse_url` 方法解析，[更多Lua API](zh-cn/lua-api)

```lua
function on_handshake(client_id, headers, server)
    print(server)
    local tb, err = mix.parse_url(server["RequestUri"])
    print(tb, err)
    local foo = tb["query"]["foo"] --bar
end
```

打印结果

```
{
        "Method" = "GET"
        "RequestUri" = "/chat?foo=bar"
        "ServerProtocol" = "HTTP/1.1"
        "Host" = "127.0.0.1:6790"
        "RemoteAddr" = "127.0.0.1:59506"
}

{
        "scheme" = ""
        "opaque" = ""
        "user" = ""
        "host" = ""
        "path" = "/chat"
        "raw_path" = ""
        "force_query" = false
        "raw_query" = "foo=bar"
        "fragment" = ""
        "raw_fragment" = ""
        "query" = [table: 0x1400044e7e0]
        {
                "foo" = "bar"
        }
}
```

### 如何安全的在公网使用控制台

在 `connmix.yaml` 配置文件中，我们可以单独配置一个只读账户给控制台使用，而程序内部使用读写账户并限制内网ip段，然后就可以安全的将 `center` 和 `engine` 的 `apiServer` 端口暴露在公网中，这样控制台就可以正常连接使用了。 

> 如果有成熟的运维团队，推荐在内网使用控制台(生产内网中win/ubuntu机器的浏览器访问，通过跳板机)，或者使用prometheus

```yaml
access: # 修改后立即生效
  users:
    - username: user1
      password: pass1
      roles:
        - read
      host: '*'
    - username: user2
      password: pass2
      roles:
        - read
        - write
      host: 172.*.*.*
```
