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

