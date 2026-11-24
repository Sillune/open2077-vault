# Open77.time (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-time.md) · [Page web](https://open2077.net/docs/api/client/open77-time)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

1 function.

## monotonic

```lua
Open77.time.monotonic()
```

`SHARED`: Available without a live game instance.

Monotonic clock, in seconds.

Never goes backwards, unlike wall-clock time: this is the one to measure a duration with. Multiply by 1000 only when an API explicitly expects milliseconds.

Returns: `number`

```lua
-- Measure how long the menu stayed open
local openedAt = Open77.time.monotonic()
AddEventHandler("menu:closed", function()
    print(("menu open for %.1f s"):format(Open77.time.monotonic() - openedAt))
end)
```

Registered in `ResourceHost.cpp` (line 24577) as `LuaMonotonic`.
