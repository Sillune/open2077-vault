# Open77.EventVerdict (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-eventverdict.md) · [Page web](https://open2077.net/docs/api/server/open77-eventverdict)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

3 functions.

## await

```lua
Open77.EventVerdict.await()
```

`SHARED`: Available without a live game instance.

Waits for the event to have been offered to every resource, then answers the verdict.

Settles one tick after publication. Needs a managed coroutine -- a CreateThread, a command or an event handler -- and answers false, await_requires_scheduler_coroutine outside one rather than blocking the server tick.

Returns: `true when a handler cancelled, false otherwise`, `false, reason outside a managed coroutine`

```lua
-- The chat authority offers the message to every resource, then acts on the
-- verdict. `await` needs a managed task, which a net event handler is.
RegisterNetEvent("chat:said", function(text)
    local name = Open77.players.name(source)
    local verdict = TriggerCancellableEvent("chatMessage", source, name, text)
    if verdict ~= nil and verdict:await() then return end   -- a handler vetoed it
    Open77.chat.send(-1, { author = name, text = text, playerId = source })
end)
```

[Cancellable events](../../01%20Documentation/server-api.md#cancellable-events)

Registered in `LuaResourceRuntime.Events.cs` (line 225) as `LuaResourceRuntime.Bootstrap`.

## canceled

```lua
Open77.EventVerdict.canceled()
```

`SHARED`: Available without a live game instance.

The verdict itself, once it has settled.

Answers nil plus the status while the event is still in flight, so a caller that must not yield can poll instead of awaiting. True means some handler, in this resource or another, called CancelEvent.

Returns: `boolean, or nil, status`

```lua
-- Poll the verdict from a tick instead of awaiting it
local verdict = TriggerCancellableEvent("shop:beforeBuy", source, itemId)
SetTick(function()
    local cancelled = verdict and verdict:canceled()
    if cancelled ~= nil then
        if not cancelled then sell(source, itemId) end
        verdict = nil
    end
end)
```

[Cancellable events](../../01%20Documentation/server-api.md#cancellable-events)

Registered in `LuaResourceRuntime.Events.cs` (line 220) as `LuaResourceRuntime.Bootstrap`.

## status

```lua
Open77.EventVerdict.status()
```

`SHARED`: Available without a live game instance.

Whether the verdict has settled, without waiting.

`pending` while the host is still offering the event to the other resources, `settled` once every running resource has been given it, `unknown` when the handle has been released or has expired. A verdict nobody reads is forgotten 30 seconds after it settles.

Returns: `"pending", "settled" or "unknown"`

```lua
-- Is the verdict in yet?
local verdict = TriggerCancellableEvent("shop:beforeBuy", source, itemId)
print(verdict and verdict:status()) -- "pending" until the next tick
```

[Cancellable events](../../01%20Documentation/server-api.md#cancellable-events)

Registered in `LuaResourceRuntime.Events.cs` (line 216) as `LuaResourceRuntime.Bootstrap`.
