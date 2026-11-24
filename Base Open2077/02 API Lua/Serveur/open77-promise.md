# Open77.Promise (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-promise.md) · [Page web](https://open2077.net/docs/api/server/open77-promise)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

5 functions.

## await

```lua
Open77.Promise.await()
```

`SHARED`: Available without a live game instance.

Waits for the promise to settle and returns its values.

Needs a managed coroutine; outside one it answers nil, await_requires_scheduler_coroutine instead of blocking the server tick. Returns the callee's values on success and nil, reason on rejection, so a caller never has to pcall it.

Returns: `the resolved values, or nil, reason`

```lua
-- Await a bank export from another resource
CreateThread(function()
    local pending = Open77.exports.call("bank", "balance", playerId)
    local balance, reason = pending and pending:await()
    print(balance or reason)
end)
```

Registered in `LuaResourceRuntime.Compat.cs` (line 113) as `LuaResourceRuntime.Bootstrap`.

## next

```lua
Open77.Promise.next([onResolved], [onRejected])
```

`SHARED`: Available without a live game instance.

Schedules a continuation for when the promise settles.

Not chainable: it returns the id of the task it schedules, not a new promise, so p:next(a):next(b) does not work and sequences belong in a CreateThread with :await(). A rejection is recognised by shape -- the nil, <string reason> pair :await() returns for one -- which is this codebase's failure convention everywhere, so a promise deliberately resolved with a leading nil followed by a string reads as rejected. Answers nil, invalid_promise_handler for a non-function and nil, task_limit_or_stopping when no task slot is available.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `onResolved` | `function` | optional | — |
| `onRejected` | `function` | optional | — |

Returns: `task id, or nil, reason`

```lua
-- Continue when the client answers, without a thread
local pending = Open77.net.callClient(playerId, "confirmPurchase", "ammo_box", 50)
if pending then
    pending:next(function(confirmed) if confirmed then sell(playerId, "ammo_box") end end)
end
```

Registered in `LuaResourceRuntime.Compat.cs` (line 148) as `LuaResourceRuntime.Bootstrap`.

## reject

```lua
Open77.Promise.reject([reason])
```

`SHARED`: Available without a live game instance.

Settles a promise you created with promise.new as failed.

Same shape and same refusals as resolve. A non-string reason becomes promise_rejected, because the reason has to survive as text for the awaiter's nil, reason pair to mean anything.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `reason` | `string` | optional | — |

Returns: `true, or false, reason`

```lua
-- A promise this resource created is rejected when the timer runs out
local answer = promise.new()
SetTimeout(10000, function() answer:reject("timeout") end)
```

Registered in `LuaResourceRuntime.Compat.cs` (line 135) as `LuaResourceRuntime.Bootstrap`.

## resolve

```lua
Open77.Promise.resolve([...])
```

`SHARED`: Available without a live game instance.

Settles a promise you created with promise.new, with the values the awaiter receives.

Only a promise this resource created can be settled by hand: one returned by Open77.exports.call or Open77.net.callClient answers false, promise_not_local, and settling twice answers false, promise_already_settled. Values are passed through untouched on the server, where the client copies them across a serialization boundary.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `...` | `any` | optional | — |

Returns: `true, or false, reason`

```lua
-- Resolve a promise you created when the database answers
local ready = promise.new()
Open77.database.ready(function() ready:resolve(true) end)
CreateThread(function() print("db ready:", ready:await()) end)
```

Registered in `LuaResourceRuntime.Compat.cs` (line 126) as `LuaResourceRuntime.Bootstrap`.

## status

```lua
Open77.Promise.status()
```

`SHARED`: Available without a live game instance.

State of the promise, without waiting.

The one handle type covers both an export call and a question asked of a client, so the same four states describe either.

Returns: `"pending", "resolved", "rejected" or "cancelled"`

```lua
-- Poll a client question from a tick
SetTick(function()
    if pending and pending:status() ~= "pending" then
        print("answered:", pending:status())
        pending = nil
    end
end)
```

Registered in `LuaResourceRuntime.Compat.cs` (line 107) as `LuaResourceRuntime.Bootstrap`.
