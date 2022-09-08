# 一天学会 `Lua`

Lua 是一种轻量小巧的脚本语言,用标准C语言编写并以源代码形式开放, 其设计目的是为了嵌入应用程序中,从而为应用程序提供灵活的扩展和定制功能。

## 语言特征

Lua 具有一些和其他语言类似，又有和其他语言都不一样的地方

### 和 `go` 一样具有多返回值、`nil` 

通常第1个返回值是结果，第二个返回值是error信息，当没有错误信息时返回nil

```lua
function foo()
    return 123, nil
end

function bar()
    return nil, "error msg"
end
```

### 不等于是 `~=`

lua的不等于和其他语言都不同

```lua
local err = baz()
if err ~= nil then
    --错误处理
end
```

### 数组 `table` 的索引是从 `1` 开始

```lua
local channel_table = mix.str_split("100001@video", "@")
local video_id = channel_table[1]
```

### 字符串拼接是 `..`

```lua
mix_log(mix_DEBUG, "invalid channel: " .. channel_raw)
```

### 模块与包方法使用 `.` 调用

```lua
local resp, err = mix.http.request("POST", auth_url, {})
```

### 对象方法使用 `:` 调用

```lua
local err = conn:subscribe("user:" .. uid)
```

## 更多教程

- `菜鸟教程` https://www.runoob.com/lua/lua-tutorial.html
- `w3cschool` https://www.w3cschool.cn/lua/
