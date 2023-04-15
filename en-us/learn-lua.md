# Learn Lua in One Day

Lua is a lightweight scripting language written in standard C and open in source code form. Its design purpose is to be embedded in applications, providing flexible extensions and customization features for applications.

> CONNMIX is driven by a golang-powered `lua vm`. The Lua API is executed at the golang code level.

## Language Features

Lua has some similarities with other languages and some unique features.

### Multiple return values and `nil` like in `Go`

Usually, the first return value is the result, and the second return value is the error information. When there is no error information, it returns nil.

```lua
function foo()
    return 123, nil
end

function bar()
    return nil, "error msg"
end
```

### Inequality is `~=`

Lua's inequality is different from other languages.

```lua
local err = baz()
if err ~= nil then
    -- error handling
end
```

### Array `table` indexes start from `1`

```lua
local channel_table = mix.str_split("100001@video", "@")
local video_id = channel_table[1]
```

### String concatenation is `..`

```lua
mix_log(mix_DEBUG, "invalid channel: " .. channel_raw)
```

### Modules and package methods are called with `.`

```lua
local resp, err = mix.http.request("POST", auth_url, {})
```

### Object methods are called with `:`

```lua
local err = conn:subscribe("user:" .. uid)
```

## More Tutorials

- `Beginner Tutorial` https://www.runoob.com/lua/lua-tutorial.html
- `w3cschool` https://www.w3cschool.cn/lua/
