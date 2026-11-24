# Open77.Promise (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-promise.md) · [Page web](https://open2077.net/docs/api/client/open77-promise)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

5 functions.

## await

```lua
Open77.Promise.await()
```

`SHARED`: Available without a live game instance.

Waits for a promise to settle.

Only usable inside a `CreateThread`.

Returns: `resolved value, or nil`, `rejection reason`

```lua
-- Wait for a notification export to answer
CreateThread(function()
    local pending = Open77.exports.call("open77_notifications", "show", { message = "Saved" })
    local result, reason = pending:await()
    if not result then print("show failed: " .. tostring(reason)) end
end)
```

Registered in `ResourceHost.cpp` (line 24427) as `LuaPromiseAwait`.

## next

```lua
Open77.Promise.next([onResolved], [onRejected])
```

`SHARED`: Available without a live game instance.

Schedules a continuation for when the promise settles.

Not chainable: it returns the id of the task it schedules, not a new promise, so p:next(a):next(b) does not work and sequences belong in a CreateThread with :await(). The id cancels a continuation that is still waiting. A rejection is recognised by shape -- the nil, <string reason> pair :await() returns -- which is this codebase's failure convention everywhere, so a promise deliberately resolved with a leading nil followed by a string reads as rejected.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `onResolved` | `function` | optional | — |
| `onRejected` | `function` | optional | — |

Returns: `task id, or nil, reason`

```lua
-- Fire-and-forget: run a continuation when the export answers, without a thread
local pending = Open77.exports.call("open77_notifications", "show", { message = "Saved" })
if pending then
    pending:next(function(result) print("shown", result.ok) end, function(reason) print("failed", reason) end)
end
```

Registered in `ResourceHost.cpp` (line 24544) as `LuaPromiseNext`.

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
-- Reject a promise you created when the player cancels the dialog
local answer = promise.new()
AddEventHandler("dialog:cancelled", function() answer:reject("cancelled") end)
```

Registered in `ResourceHost.cpp` (line 24543) as `LuaPromiseReject`.

## resolve

```lua
Open77.Promise.resolve([...])
```

`SHARED`: Available without a live game instance.

Settles a promise you created with promise.new, with the values the awaiter receives.

Only a promise this resource created can be resolved; one returned by Open77.exports.call or Open77.net.call is settled by the runtime. Settling twice answers false, promise_already_settled, and a promise whose single consumer has already taken its value answers false, promise_cancelled. On the client the values cross the same serialization boundary exports use, so one that cannot be carried is refused with false, promise_value_not_serializable rather than silently dropped.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `...` | `any` | optional | — |

Returns: `true, or false, reason`

```lua
-- Resolve a promise you created when the dialog is answered
local answered = promise.new()
AddEventHandler("dialog:answered", function(yes) answered:resolve(yes) end)
```

Registered in `ResourceHost.cpp` (line 24542) as `LuaPromiseResolve`.

## status

```lua
Open77.Promise.status()
```

`SHARED`: Available without a live game instance.

State of the promise, without waiting.

Returns the promise state without yielding the current coroutine. Use it for polling or diagnostics; `await()` remains the normal way to receive a completed export result.

Returns: `"pending", "resolved" or "rejected"`

```lua
-- Poll a promise from a tick instead of awaiting it
SetTick(function()
    if pending and pending:status() ~= "pending" then
        print("settled:", pending:status())
        pending = nil
    end
end)
```

Registered in `ResourceHost.cpp` (line 24428) as `LuaPromiseStatus`.
