# Citizen (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/citizen.md) · [Page web](https://open2077.net/docs/api/server/citizen)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

8 functions.

## Await

```lua
Citizen.Await(promise)
```

`SHARED`: Available without a live game instance.

Waits for a promise; exactly promise:await().

Provided because ported code writes it this way. Anything that is not a promise answers nil, invalid_promise instead of raising, and the await itself needs a managed task: outside one it answers nil, await_requires_scheduler_coroutine rather than parking the tick.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `promise` | `Open77.Promise` | required | — |

Returns: `the resolved values, or nil, reason`

```lua
-- Await a server export promise, FiveM spelling
Citizen.CreateThread(function()
    local pending = Open77.exports.call("bank", "balance", playerId)
    local balance = Citizen.Await(pending)
    print("balance", balance)
end)
```

[FiveM compatibility guide](../../01%20Documentation/fivem-compatibility.md)

Registered in `LuaResourceRuntime.Compat.cs` (line 183) as `LuaResourceRuntime.Bootstrap`.

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
-- Same function as ClearTick, FiveM spelling
local tick = Citizen.SetTick(function() checkZones() end)
AddEventHandler("event:ended", function() Citizen.ClearTick(tick) end)
```

[FiveM compatibility guide](../../01%20Documentation/fivem-compatibility.md)

Registered in `LuaResourceRuntime.Compat.cs` (line 181) as `LuaResourceRuntime.Bootstrap`.

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

```lua
-- Cancel the round timer when everyone left
local timer = Citizen.SetTimeout(60000, function() endRound() end)
AddEventHandler("arena:empty", function() Citizen.ClearTimeout(timer) end)
```

[FiveM compatibility guide](../../01%20Documentation/fivem-compatibility.md)

Registered in `LuaResourceRuntime.Compat.cs` (line 179) as `LuaResourceRuntime.Bootstrap`.

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
-- A ported FiveM server loop runs unchanged
Citizen.CreateThread(function()
    while true do
        Citizen.Wait(60000)
        saveEverything()
    end
end)
```

[FiveM compatibility guide](../../01%20Documentation/fivem-compatibility.md)

Registered in `LuaResourceRuntime.Compat.cs` (line 176) as `LuaResourceRuntime.Bootstrap`.

## SetTick

```lua
Citizen.SetTick(body)
```

`SHARED`: Available without a live game instance.

Alias of SetTick.

Citizen holds the very same function values the bare globals do, not copies, so a ported resource may use either spelling and a stack trace names the same function. See SetTick for the task budget and the five-consecutive-failure cancellation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `body` | `function` | required | — |

Returns: `tick id, or nil, reason`

```lua
-- Per-tick work, FiveM spelling
Citizen.SetTick(function()
    for _, playerId in ipairs(Open77.players.all()) do checkZone(playerId) end
end)
```

[FiveM compatibility guide](../../01%20Documentation/fivem-compatibility.md)

Registered in `LuaResourceRuntime.Compat.cs` (line 180) as `LuaResourceRuntime.Bootstrap`.

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
-- Respawn 8 seconds after death, FiveM spelling
Citizen.SetTimeout(8000, function() Open77.players.respawn(playerId, { position = hospital }) end)
```

[FiveM compatibility guide](../../01%20Documentation/fivem-compatibility.md)

Registered in `LuaResourceRuntime.Compat.cs` (line 178) as `LuaResourceRuntime.Bootstrap`.

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
Citizen.Trace("player", source, "bought", itemId)
```

[FiveM compatibility guide](../../01%20Documentation/fivem-compatibility.md)

Registered in `LuaResourceRuntime.Compat.cs` (line 182) as `LuaResourceRuntime.Bootstrap`.

## Wait

```lua
Citizen.Wait(milliseconds)
```

`SHARED`: Available without a live game instance.

Alias of Wait.

Citizen holds the very same function values the bare globals do, not copies, so a ported resource may use either spelling and a stack trace names the same function. Only usable inside a managed task, and never inside a cancellable event handler, which runs inline during the host drain.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `milliseconds` | `integer` | required | — |

```lua
-- A countdown in chat
Citizen.CreateThread(function()
    for n = 3, 1, -1 do
        Open77.chat.broadcast(("%d..."):format(n))
        Citizen.Wait(1000)
    end
end)
```

[FiveM compatibility guide](../../01%20Documentation/fivem-compatibility.md)

Registered in `LuaResourceRuntime.Compat.cs` (line 177) as `LuaResourceRuntime.Bootstrap`.
