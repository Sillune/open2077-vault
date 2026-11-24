# Open77.events (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-events.md) · [Page web](https://open2077.net/docs/api/server/open77-events)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

7 functions.

## cancel

```lua
Open77.events.cancel()
```

`NETWORK`: Uses the network backend.

Server only. Vetoes the cancellable event this handler is running under.

Alias of CancelEvent.

Returns: `true, or false, reason`

```lua
-- Veto a chat message from a muted player (alias of CancelEvent)
Open77.events.on("chatMessage", function(playerId, name, text)
    if muted[playerId] then Open77.events.cancel() end
end)
```

Registered in `LuaResourceRuntime.Events.cs` (line 248) as `LuaResourceRuntime.Bootstrap`.

## emit

```lua
Open77.events.emit(event, ...)
```

`NETWORK`: Uses the network backend.

Publishes an event host-wide; the same function as TriggerEvent.

No permission. Since the host-wide bus exists, this reaches every running resource that handles the name, not only this VM: the fan-out is queued and drained at the next tick boundary, in resource-name order. Open77.events.emitLocal is the one that stays inside this VM. Host events (onPlayerReady, onPlayerDisconnected, the resource lifecycle events) arrive through the same handler list a resource uses for its own, so it listens for both the same way.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `...` | `values` | required | — |

Returns: `true, or false, reason`

```lua
-- Announce a robbery to every resource on the server
Open77.events.emit("bank:robbed", source, amount)
```

Registered in `LuaResourceRuntime.Events.cs` (line 245) as `LuaResourceRuntime.Bootstrap`.

## emitCancellable

```lua
Open77.events.emitCancellable(event, [...])
```

`NETWORK`: Uses the network backend.

Server only. Publishes a host-wide cancellable event and returns its verdict handle.

Alias of TriggerCancellableEvent.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `...` | `any` | optional | — |

Returns: `verdict handle, or nil, reason`

```lua
-- Let other resources veto a purchase (alias of TriggerCancellableEvent)
RegisterNetEvent("shop:buy", function(itemId)
    local verdict = Open77.events.emitCancellable("shop:beforeBuy", source, itemId)
    if verdict and verdict:await() then return end -- somebody said no
    sell(source, itemId)
end)
```

Registered in `LuaResourceRuntime.Events.cs` (line 247) as `LuaResourceRuntime.Bootstrap`.

## emitLocal

```lua
Open77.events.emitLocal(event, [...])
```

`NETWORK`: Uses the network backend.

Server only. Fires an event in this resource's VM only.

Alias of TriggerLocalEvent.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `...` | `any` | optional | — |

Returns: `true`

```lua
-- Internal signalling that stays inside this resource
Open77.events.emitLocal("shop:restocked", "ammo", 200)
```

Registered in `LuaResourceRuntime.Events.cs` (line 246) as `LuaResourceRuntime.Bootstrap`.

## off

```lua
Open77.events.off(handlerId)
```

`NETWORK`: Uses the network backend.

Removes a local or network handler; the same function as `RemoveEventHandler`.

No permission. Takes the id `Open77.events.on` or `Open77.net.on` returned.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `handlerId` | `integer` | required | — |

Returns: `boolean`

```lua
-- Stop listening after the first player is ready
local handler
handler = Open77.events.on("onPlayerReady", function(playerId)
    print("first player:", playerId)
    Open77.events.off(handler)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8164) as `LuaResourceRuntime.Bootstrap`.

## on

```lua
Open77.events.on(event, handler)
```

`NETWORK`: Uses the network backend.

Registers an event handler; the same function as AddEventHandler.

No permission. One handler list receives three sources: this VM's Open77.events.emitLocal, the host-wide bus every resource publishes on, and the platform's own lifecycle events. Cross-resource *calls* with a return value are still Open77.exports; the bus carries announcements, not answers. Returns a handler id for Open77.events.off. The limit is 2,048 handlers per resource.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `handler` | `function` | required | — |

Returns: `handler id`

```lua
-- Listen for another resource's announcement
Open77.events.on("bank:robbed", function(robberId, amount)
    Open77.players.setWanted(robberId, 3)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8164) as `LuaResourceRuntime.Bootstrap`.

## wasCanceled

```lua
Open77.events.wasCanceled()
```

`NETWORK`: Uses the network backend.

Server only. Whether the cancellable event being dispatched has already been vetoed.

Alias of WasEventCanceled.

Returns: `boolean`

```lua
-- Log messages an earlier filter already vetoed
Open77.events.on("chatMessage", function(playerId, name, text)
    if Open77.events.wasCanceled() then print("filtered:", text) end
end)
```

Registered in `LuaResourceRuntime.Events.cs` (line 249) as `LuaResourceRuntime.Bootstrap`.
