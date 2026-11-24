# promise (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/promise.md) · [Page web](https://open2077.net/docs/api/client/promise)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

1 function.

## new

```lua
promise.new()
```

`SHARED`: Available without a live game instance.

Creates a promise this resource settles itself.

The same Open77.Promise object Open77.exports.call and Open77.net.call return, so the runtime has one promise type rather than two: resolve, reject, next, await and status all work on it. On the client a pending promise has a single consumer -- the first await or next woken by a settle takes the value and the promise then reports cancelled -- while on the server it keeps its value and can be awaited repeatedly. Code that awaits once is portable across both. See the FiveM compatibility guide.

Returns: `Open77.Promise`

```lua
-- A promise resolved by an event, awaited by a thread
local answered = promise.new()
AddEventHandler("dialog:answered", function(yes) answered:resolve(yes) end)
CreateThread(function()
    local yes = answered:await()
    print("player said", yes)
end)
```

[FiveM compatibility guide](../../01%20Documentation/fivem-compatibility.md#promises)

Registered in `ResourceHost.cpp` (line 24548) as `LuaPromiseNew`.
