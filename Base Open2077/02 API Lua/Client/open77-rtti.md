# Open77.rtti (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-rtti.md) · [Page web](https://open2077.net/docs/api/client/open77-rtti)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

5 functions.

## call

```lua
Open77.rtti.call(class, fn, instance, ...)
```

`GAME`: Requires a live game instance.

Invokes a reflected game function using supported Lua values.

Client-only; requires `rtti.native`. Call from a running resource callback or thread; candidate loading and stop handlers return resource_not_active. Instance is an owned ref for a method and nil for a static function. Supply every parameter, including optional ones. Supports scalars, enums, 64-bit IDs and strong IScriptable handles; out parameters, arrays, arbitrary structs, script_ref and weak handles are refused. Signatures are limited to 15 parameters. Returned refs remain pinned until release or owner stop/reload. SDK calls participate in installed hooks; other engine entry points and worker-thread calls are outside hook coverage.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `class` | `string|nil` | required | — |
| `fn` | `string` | required | — |
| `instance` | `userdata|nil` | required | — |
| `...` | `any` | required | — |

Returns: `result (nil for void or a null handle), or nil, reason on failure`

```lua
AddEventHandler("example:rtti", function()
    local value, reason = Open77.rtti.call(nil, "AbsF", nil, -3.5)
    if reason then print(reason) else print(value) end
end)
```

[RTTI calls & hooks guide](../../01%20Documentation/rtti.md)

Registered in `ResourceHost.cpp` (line 26202) as `LuaRttiCall`.

## hook

```lua
Open77.rtti.hook(class, fn, handler)
```

`GAME`: Requires a live game instance.

Registers a synchronous hook for supported SDK calls to a game function.

Client-only; requires `rtti.native`. Requires a running resource. handler(ctx, ...args) receives ctx.name, ctx.instance and ctx.callOriginal. The continuation accepts replacement arguments, is valid only during the callback and may be called once. Return a replacement or call through; nil passes through for value-returning functions and skips an uncalled void original. Errors preserve an already-completed original result. Same-function recursion bypasses hooks. Callbacks cannot yield and obey Lua execution quotas. Borrowed instance/argument/original-result refs expire after dispatch. Hooks are removed on owner stop/reload. Coverage is game-thread SDK CStack Execute calls, including rtti.call; it does not cover every internal VM/native invocation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `class` | `string|nil` | required | — |
| `fn` | `string` | required | — |
| `handler` | `function` | required | — |

Returns: `hook id, or nil, reason`

```lua
AddEventHandler("example:hook", function()
    local id = assert(Open77.rtti.hook(nil, "AbsF", function(ctx, value)
        return ctx.callOriginal(value) + 10
    end))
    assert(Open77.rtti.call(nil, "AbsF", nil, -3) == 13)
    Open77.rtti.unhook(id)
end)
```

[RTTI calls & hooks guide](../../01%20Documentation/rtti.md)

Registered in `ResourceHost.cpp` (line 26203) as `LuaRttiHook`.

## release

```lua
Open77.rtti.release(ref)
```

`GAME`: Requires a live game instance.

Releases a pinned RTTI object reference.

Client-only; requires `rtti.native`. Accepts the opaque ref userdata returned by rtti.call or borrowed during a hook. Releasing an expired ref is harmless and returns true. Later calls with it fail with ref_released. Owned call results are also released on resource stop/reload; Lua garbage collection alone does not release them. Hook-borrowed refs expire at dispatch end even when the Lua userdata is retained.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `ref` | `userdata` | required | — |

Returns: `true, or nil, reason`

```lua
AddEventHandler("example:release", function()
    local ref = assert(Open77.rtti.call(nil, "CompareBuilder::Make;", nil))
    Open77.rtti.release(ref)
end)
```

[RTTI calls & hooks guide](../../01%20Documentation/rtti.md)

Registered in `ResourceHost.cpp` (line 26205) as `LuaRttiRelease`.

## resolve

```lua
Open77.rtti.resolve(class, fn)
```

`GAME`: Requires a live game instance.

Reads the RTTI signature of a native or scripted game function.

Client-only; requires `rtti.native`. Use nil or an empty class for a global and its decorated name. Methods accept decorated names or unambiguous short names. Returns a table with name, static, returnType and params ({name, type}). Metadata may describe unsupported types; resolve alone does not prove callability. Errors include function_not_found, ambiguous_function and rtti_unavailable.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `class` | `string|nil` | required | — |
| `fn` | `string` | required | — |

Returns: `signature table, or nil, reason`

```lua
local signature, reason = Open77.rtti.resolve(nil, "AbsF")
if signature then print(signature.name, signature.returnType) else print(reason) end
```

[RTTI calls & hooks guide](../../01%20Documentation/rtti.md)

Registered in `ResourceHost.cpp` (line 26201) as `LuaRttiResolve`.

## unhook

```lua
Open77.rtti.unhook(id)
```

`GAME`: Requires a live game instance.

Removes a hook owned by the calling resource.

Client-only; requires `rtti.native`. Returns true only when this resource removes its hook. Unknown IDs and another resource's IDs return false. Removing a hook during dispatch takes effect for later calls; the current dispatch retains its callback snapshot. Stop/reload automatically removes the owner's hooks.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `boolean; or nil, reason when permission is denied`

```lua
AddEventHandler("example:remove-hook", function()
    local id = assert(Open77.rtti.hook(nil, "AbsF", function(ctx) return ctx.callOriginal() end))
    assert(Open77.rtti.unhook(id))
    assert(not Open77.rtti.unhook(id))
end)
```

[RTTI calls & hooks guide](../../01%20Documentation/rtti.md)

Registered in `ResourceHost.cpp` (line 26204) as `LuaRttiUnhook`.
