# Open77.events (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-events.md) · [Page web](https://open2077.net/docs/api/client/open77-events)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

3 functions.

## emit

```lua
Open77.events.emit(event, [...])
```

`SHARED`: Available without a live game instance.

Publishes an event in this resource's VM; the same function as TriggerEvent.

Emits an event on the resource-local client bus and invokes matching local handlers. It never sends a packet; use the network event API for server communication.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `...` | `any` | optional | — |

Returns: `true, or false, reason`

```lua
-- Tell the rest of this resource that the shop closed
Open77.events.emit("shop:closed")
```

Registered in `ResourceHost.cpp` (line 25556) as `LuaTriggerEvent`.

## off

```lua
Open77.events.off(handler)
```

`SHARED`: Available without a live game instance.

Removes a handler registered with `on`.

Unregisters a local event handler id previously returned by `on`/`AddEventHandler`. The id is the only argument -- there is no event name parameter. Handler ids are owned by a resource generation and cannot remove another resource's callback.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `handler` | `integer` | required | — |

Returns: `boolean`

```lua
-- Listen for the first "ready" only
local handler
handler = Open77.events.on("hud:ready", function()
    print("HUD ready")
    Open77.events.off(handler)
end)
```

Registered in `ResourceHost.cpp` (line 24645) as `LuaRemoveEventHandler`.

## on

```lua
Open77.events.on(event, handler)
```

`SHARED`: Available without a live game instance.

Listens for a local event.

Same as the `AddEventHandler` global. Events emitted by the plugin — `open77:pauseKey`, `open77:menuStateChanged`, `open77:inspector:on` — arrive here.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `handler` | `function` | required | — |

Returns: `handler id`

```lua
-- React to the plugin's own menu-state event
Open77.events.on("open77:menuStateChanged", function(state)
    print("menu open:", state.open)
end)
```

Registered in `ResourceHost.cpp` (line 24644) as `LuaAddEventHandler`.
