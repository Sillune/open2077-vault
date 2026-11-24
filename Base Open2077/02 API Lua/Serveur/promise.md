# promise (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/promise.md) · [Page web](https://open2077.net/docs/api/server/promise)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

1 function.

## new

```lua
promise.new()
```

`SHARED`: Available without a live game instance.

Creates a promise this resource settles itself.

The same Open77.Promise object Open77.exports.call and Open77.net.callClient return, so the runtime has one promise type rather than two. A promise created here keeps its value and can be awaited repeatedly, where the client's single-consumer export promise cannot; code that awaits once is portable across both. See the FiveM compatibility guide.

Returns: `Open77.Promise`

```lua
local p = promise.new()
CreateThread(function() Wait(250) p:resolve("done") end)
print(p:await())
```

[FiveM compatibility guide](../../01%20Documentation/fivem-compatibility.md#promises)

Registered in `LuaResourceRuntime.Compat.cs` (line 165) as `LuaResourceRuntime.Bootstrap`.
