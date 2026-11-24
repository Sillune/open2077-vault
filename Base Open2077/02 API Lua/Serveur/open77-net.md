# Open77.net (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-net.md) · [Page web](https://open2077.net/docs/api/server/open77-net)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

9 functions.

## callClient

```lua
Open77.net.callClient(playerId, name, [...])
```

`NETWORK`: Uses the network backend.

Asks one client a question and returns a promise for the answer.

Requires network.events. Targets the handler that resource registered on that client; the table form { resource = "other", name = "x", timeout = ms } names another resource or sets a deadline (default 10,000 ms, range 100..120,000). There is no broadcast: callClient(-1, ...) answers callback_broadcast_unsupported, because one question cannot have many answers. Up to 29 arguments and 48 KiB per frame, 256 questions open per resource. Rejects with callback_not_found, callback_timeout, callback_target_stopped and the rest of the callback reason list. Aliased as Open77.callbacks.callClient.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `name` | `string|table` | required | — |
| `...` | `any` | optional | — |

Returns: `Open77.Promise, or nil, reason`

```lua
-- Ask one client to confirm before selling
CreateThread(function()
    local pending, reason = Open77.net.callClient(source, "confirmPurchase", "ammo_box", 50)
    if not pending then return print(reason) end
    local confirmed = pending:await()
    if confirmed then sell(source, "ammo_box") end
end)
```

Registered in `LuaResourceRuntime.Callbacks.cs` (line 497) as `LuaResourceRuntime.Bootstrap`.

## callClientAwait

```lua
Open77.net.callClientAwait(playerId, name, [...])
```

`NETWORK`: Uses the network backend.

Asks one client a question and waits for the answer.

Requires network.events. Open77.net.callClient followed by :await(), so it must run inside a CreateThread, a command handler or another managed task. A client that refuses, times out or disconnects yields nil, reason -- never a partial answer. Aliased as Open77.callbacks.callClientAwait.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `name` | `string|table` | required | — |
| `...` | `any` | optional | — |

Returns: `the client handler's return values, or nil, reason`

```lua
local accepted = Open77.net.callClientAwait(source, "confirmPurchase", item, price)
if accepted ~= true then return end
```

Registered in `LuaResourceRuntime.Callbacks.cs` (line 498) as `LuaResourceRuntime.Bootstrap`.

## cancelLatent

```lua
Open77.net.cancelLatent(id)
```

`NETWORK`: Uses the network backend.

Stops a latent stream that is still sending.

`true` when the stream was still sending; `false, reason` otherwise: `not_found` (never this resource's, or forgotten a minute after it finished), `already_finished`. Nothing is sent to the client: it drops its partial copy on its own after a minute without a frame, so a cancelled stream never delivers a truncated event.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |

Returns: `true, or false`, `reason on failure`

```lua
-- A player who leaves the shop no longer needs the catalogue in flight
AddEventHandler('shop:closed', function(playerId)
    local id = pending[playerId]
    if id then Open77.net.cancelLatent(id); pending[playerId] = nil end
end)
```

Registered in `LuaResourceRuntime.Latent.cs` (line 408) as `LuaResourceRuntime.Bootstrap`.

## emitClient

```lua
Open77.net.emitClient(event, playerId, ...)
```

`NETWORK`: Uses the network backend.

Sends an event to one client or every client; the same function as `TriggerClientEvent`.

Requires `network.events`. A `playerId` of `-1` broadcasts. Same 32-argument, 48 KiB envelope as the inbound direction.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `playerId` | `integer|string, or -1 to broadcast` | required | — |
| `...` | `values` | required | — |

Returns: `boolean`

```lua
-- Namespaced spelling of TriggerClientEvent: tell one player, or everyone with -1
Open77.net.emitClient("bank:deposit", playerId, 250)
Open77.net.emitClient("server:announce", -1, "Restart in 5 minutes")
```

Registered in `LuaResourceRuntime.cs` (line 8165) as `LuaResourceRuntime.Bootstrap`.

## emitLatent

```lua
Open77.net.emitLatent(name, target, bytesPerSecond, [...])
```

`NETWORK`: Uses the network backend.

Sends a client event too large for one net event, paced at a byte rate; the same function as `TriggerLatentClientEvent`.

The same call as `TriggerClientEvent` with FiveM's bytes-per-second argument in third place, for a payload the 48 KiB event envelope cannot carry: up to 4 MiB, cut into 40 KiB frames on a reserved name, paced at the requested rate (clamped to 1 KiB/s .. 2.5 MiB/s, and never more than 64 frames per second for all of a resource's streams together), reassembled by the client host and delivered to `RegisterNetEvent` handlers under the ORIGINAL name -- the receiving resource cannot tell it was latent. `target` is one player id or `-1` for everybody connected. Requires `network.events`. Answers the stream id, or `nil, reason`: `permission_denied:network.events`, `invalid_event_name`, `reserved_*_event`, `invalid_player_id`, `player_not_found`, `invalid_rate`, `event_argument_limit`, `latent_stream_limit` (16 in flight), `latent_payload_not_serializable`, `latent_payload_too_large` (over 4 MiB, nothing sent). Read progress with `Open77.net.latentStatus(id)`; stop it with `cancelLatent(id)`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `target` | `integer player id, or -1 for everybody` | required | — |
| `bytesPerSecond` | `number` | required | — |
| `...` | `event arguments` | optional | — |

Returns: `stream id string, or nil`, `reason on failure`

```lua
-- A 1 MiB map snapshot to everybody, paced at a megabyte per second so the
-- stream never competes with ordinary gameplay traffic.
local id, reason = Open77.net.emitLatent("map:snapshot", -1, 1024 * 1024, snapshot)
if not id then return print("snapshot not sent: " .. tostring(reason)) end
streams[#streams + 1] = id   -- latentStatus(id) follows it; cancelLatent(id) stops it
```

Registered in `LuaResourceRuntime.Latent.cs` (line 397) as `LuaResourceRuntime.Bootstrap`.

## latentStatus

```lua
Open77.net.latentStatus(id)
```

`NETWORK`: Uses the network backend.

Progress of one latent stream this resource started.

`{ id, name, target, state, sentBytes, totalBytes, sentFrames, totalFrames, progress (0..1), bytesPerSecond, elapsedMs, reason? }` with `state` one of `sending`, `done`, `failed` (reason `player_left`) or `cancelled`. A finished stream keeps answering for a minute, then `nil, not_found`; another resource's stream is never visible. Ungated read.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string (the id emitLatent returned)` | required | — |

Returns: `table, or nil`, `reason on failure`

```lua
-- Tell the player when their catalogue has fully arrived
local id = TriggerLatentClientEvent('shop:catalogue', playerId, 256 * 1024, catalogue)
CreateThread(function()
    while true do
        Wait(500)
        local s = Open77.net.latentStatus(id)
        if not s then return end
        if s.state == 'done' then Open77.chat.send(playerId, 'Catalogue loaded'); return end
        if s.state ~= 'sending' then return end
    end
end)
```

Registered in `LuaResourceRuntime.Latent.cs` (line 403) as `LuaResourceRuntime.Bootstrap`.

## on

```lua
Open77.net.on(event, [handler])
```

`NETWORK`: Uses the network backend.

Registers an authenticated client event; the same function as `RegisterNetEvent`.

Requires `network.events`. During a network handler the global `source` is set from the authenticated connection, never from the client's payload -- that is the whole point of the distinction from `Open77.events.on`. Events accept at most 32 arguments in a 48 KiB JSON envelope.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `handler` | `function` | optional | — |

Returns: `handler id`

```lua
-- Namespaced spelling of RegisterNetEvent; source is the authenticated player
Open77.net.on("shop:buy", function(itemId)
    print("player", source, "wants", itemId)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8165) as `LuaResourceRuntime.Bootstrap`.

## register

```lua
Open77.net.register(name, handler)
```

`NETWORK`: Uses the network backend.

Answers a callback a client sends with Open77.net.call.

Requires network.events. The handler is called as handler(source, ...): `source` is the authenticated player id, taken from the transport and never from the payload, exactly as a RegisterNetEvent handler sets it -- and that is the only thing the caller cannot choose, so re-validate every argument. Use the source argument rather than the global in a handler that yields, because another task can overwrite the global across a Wait. Return values become the answer; raising rejects the caller's promise with the error text. Names are per resource: a client reaches this handler by naming this resource. Up to 256 names and 256 questions being served at once per resource; stopping the resource answers what it was still serving with callback_target_stopped. Aliased as Open77.callbacks.register. See the Network callbacks guide.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `handler` | `function` | required | — |

Returns: `true, or nil, reason`

```lua
Open77.net.register("getStock", function(source, item)
    if type(item) ~= "string" then return nil, "invalid_item" end
    return stock[item] or 0
end)
```

Registered in `LuaResourceRuntime.Callbacks.cs` (line 495) as `LuaResourceRuntime.Bootstrap`.

## unregister

```lua
Open77.net.unregister(name)
```

`NETWORK`: Uses the network backend.

Stops answering a callback name registered with Open77.net.register.

Requires network.events. A later request for that name is refused with callback_not_found on the next frame rather than waiting out the asker's deadline. Aliased as Open77.callbacks.unregister.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |

Returns: `true, or false, reason`

```lua
-- Stop answering a callback when the shop closes
Open77.net.unregister("getStock")
```

Registered in `LuaResourceRuntime.Callbacks.cs` (line 496) as `LuaResourceRuntime.Bootstrap`.
