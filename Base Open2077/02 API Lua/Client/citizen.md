# Citizen (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/citizen.md) · [Page web](https://open2077.net/docs/api/client/citizen)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

8 functions.

## Await

```lua
Citizen.Await(promise)
```

`SHARED`: Available without a live game instance.

Waits for a promise; exactly promise:await().

Provided because ported code writes it this way. It needs a managed task like any await: outside one it answers nil, await_requires_scheduler_coroutine rather than parking the VM.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `promise` | `Open77.Promise` | required | — |

Returns: `the resolved values, or nil, reason`

```lua
-- A FiveM-style await of an export promise
Citizen.CreateThread(function()
    local pending = Open77.exports.call("open77_notifications", "show", { message = "Hello" })
    local result = Citizen.Await(pending)
    print(result and result.ok)
end)
```

[FiveM compatibility guide](../../01%20Documentation/fivem-compatibility.md)

Registered in `ResourceHost.cpp` (line 24563) as `LuaPromiseAwait`.

## ClearTick

```lua
Citizen.ClearTick(id)
```

`SHARED`: Available without a live game instance.

Alias of ClearTick.

Citizen holds the very same function values the bare globals do, not copies, so a ported resource may use either spelling and a stack trace names the same function.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `boolean`

```lua
-- Same function as ClearTick, under the FiveM name
local tick = Citizen.SetTick(function() drawMarker() end)
AddEventHandler("delivery:done", function() Citizen.ClearTick(tick) end)
```

[FiveM compatibility guide](../../01%20Documentation/fivem-compatibility.md)

Registered in `ResourceHost.cpp` (line 24562) as `LuaClearTick`.

## ClearTimeout

```lua
Citizen.ClearTimeout(id)
```

`SHARED`: Available without a live game instance.

Alias of ClearTimeout.

Citizen holds the very same function values the bare globals do, not copies, so a ported resource may use either spelling and a stack trace names the same function.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `boolean`

```lua
-- Cancel a pending timer, FiveM spelling
local timer = Citizen.SetTimeout(5000, function() closeMenu() end)
AddEventHandler("menu:closed", function() Citizen.ClearTimeout(timer) end)
```

[FiveM compatibility guide](../../01%20Documentation/fivem-compatibility.md)

Registered in `ResourceHost.cpp` (line 24560) as `LuaClearTimeout`.

## CreateThread

```lua
Citizen.CreateThread(body)
```

`SHARED`: Available without a live game instance.

Alias of CreateThread.

Citizen holds the very same function values the bare globals do, not copies, so a ported resource may use either spelling and a stack trace names the same function. Citizen.CreateThreadNow is deliberately absent: this scheduler starts a task on the next pass, and resuming one inline from inside a running task would reset that task's instruction counter and deadline mid-flight.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `body` | `function` | required | — |

Returns: `task id`

```lua
-- A ported FiveM loop runs unchanged
Citizen.CreateThread(function()
    while true do
        Citizen.Wait(1000)
        print("still here")
    end
end)
```

[FiveM compatibility guide](../../01%20Documentation/fivem-compatibility.md)

Registered in `ResourceHost.cpp` (line 24557) as `LuaCreateThread`.

## SetTick

```lua
Citizen.SetTick(body)
```

`SHARED`: Available without a live game instance.

Alias of SetTick.

Citizen holds the very same function values the bare globals do, not copies, so a ported resource may use either spelling and a stack trace names the same function. See SetTick for the per-frame budget and the five-consecutive-failure cancellation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `body` | `function` | required | — |

Returns: `tick id, or nil, reason`

```lua
-- Per-frame work, FiveM spelling
Citizen.SetTick(function()
    if Open77.input.isDown("mouse2") then showAimOverlay() end
end)
```

[FiveM compatibility guide](../../01%20Documentation/fivem-compatibility.md)

Registered in `ResourceHost.cpp` (line 24561) as `LuaSetTick`.

## SetTimeout

```lua
Citizen.SetTimeout(milliseconds, body)
```

`SHARED`: Available without a live game instance.

Alias of SetTimeout.

Citizen holds the very same function values the bare globals do, not copies, so a ported resource may use either spelling and a stack trace names the same function.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `milliseconds` | `integer` | required | — |
| `body` | `function` | required | — |

Returns: `timer id`

```lua
-- Hide the hint after 4 seconds
Citizen.SetTimeout(4000, function() Open77.hud.clearNotify() end)
```

[FiveM compatibility guide](../../01%20Documentation/fivem-compatibility.md)

Registered in `ResourceHost.cpp` (line 24559) as `LuaSetTimeout`.

## Trace

```lua
Citizen.Trace(...)
```

`SHARED`: Available without a live game instance.

Writes to the resource log at debug level; the same function as Open77.log.debug.

FiveM's trace channel, mapped onto the log this resource already writes to. It is not a separate sink and has no separate filter.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `...` | `any` | required | — |

```lua
-- Debug-level log line, FiveM spelling
Citizen.Trace("menu opened by key")
```

[FiveM compatibility guide](../../01%20Documentation/fivem-compatibility.md)

Registered in `ResourceHost.cpp` (line 24564) as `LuaLog`.

## Wait

```lua
Citizen.Wait(milliseconds)
```

`SHARED`: Available without a live game instance.

Alias of Wait.

Citizen holds the very same function values the bare globals do, not copies, so a ported resource may use either spelling and a stack trace names the same function. Only usable inside a managed task.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `milliseconds` | `integer` | required | — |

```lua
-- Inside any managed task
Citizen.CreateThread(function()
    Citizen.Wait(500)
    Open77.hud.notify("Half a second later")
end)
```

[FiveM compatibility guide](../../01%20Documentation/fivem-compatibility.md)

Registered in `ResourceHost.cpp` (line 24558) as `LuaWait`.
