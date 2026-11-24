# Open77.state (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-state.md) · [Page web](https://open2077.net/docs/api/client/open77-state)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

7 functions.

## entity

```lua
Open77.state.entity(kind, [id])
```

`CLIENT`: Client-only surface.

The replicated state bag of one vehicle, NPC or prop, read-only.

No permission. Accepts `(kind, id)` or `{ kind = , id = }`, where kind is `vehicle`, `npc` or `prop`. The mirror only holds entities this client is streaming.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `kind` | `string|table` | required | — |
| `id` | `integer|string` | optional | — |

Returns: `bag handle, or nil`, `reason: invalid_state_selector`

```lua
-- Read a car's replicated fuel level set by the server
local fuel = Open77.state.entity("vehicle", vehicleId).fuel
if fuel then hud.fuel = fuel end
```

[Replicated state bags guide](../../01%20Documentation/state-bags.md)

Registered in `ResourceHost.cpp` (line 24609) as `LuaStateEntityBag`.

## localPlayer

```lua
Open77.state.localPlayer()
```

`CLIENT`: Client-only surface.

The local player's own replicated state bag.

No permission. Resolved from the live session on every call, so it answers `no_session` before a session exists and `network_unavailable` on a host that does not replicate. `LocalPlayer.state` is the FiveM spelling of the same thing.

Returns: `bag handle, or nil`, `reason: no_session, network_unavailable`

```lua
-- Show the local player's job on the HUD
local me = Open77.state.localPlayer()
if me and me.job then hud.job = me.job.name end
```

[Replicated state bags guide](../../01%20Documentation/state-bags.md)

Registered in `ResourceHost.cpp` (line 24610) as `LuaStateLocalPlayerBag`.

## offChange

```lua
Open77.state.offChange(subscription)
```

`CLIENT`: Client-only surface.

Stops a state-bag change subscription.

No permission. Takes the id returned by `Open77.state.onChange` or by `AddStateBagChangeHandler`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `subscription` | `integer` | required | — |

Returns: `true, otherwise false`

```lua
-- Stop watching the round key when the match is over
local sub = Open77.state.onChange("global", "round", function(_, _, value) hud.round = value end)
AddEventHandler("arena:matchOver", function() Open77.state.offChange(sub) end)
```

[Replicated state bags guide](../../01%20Documentation/state-bags.md)

Registered in `ResourceHost.cpp` (line 24613) as `LuaStateOffChange`.

## onChange

```lua
Open77.state.onChange(selector, key, handler)
```

`CLIENT`: Client-only surface.

Watches state-bag changes matching a selector and a key.

No permission. The selector is a bag handle, a `{ kind = , id = }` table, a bare kind string, or nil for every bag; the key is a key name or nil for every key. The handler runs at a tick boundary as `fn(selector, key, value, previous)`. Subscriptions die with the resource instance.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `selector` | `table|string|nil` | required | — |
| `key` | `string|nil` | required | — |
| `handler` | `function` | required | — |

Returns: `subscription id, or nil`, `reason: invalid_state_selector, invalid_state_key, invalid_handler_or_watcher_limit`

```lua
-- React when the server changes this player's job
Open77.state.onChange({ kind = "player", id = Open77.players.localId() }, "job", function(selector, key, value, previous)
    Open77.hud.notify("New job: " .. tostring(value and value.name))
end)
```

[Replicated state bags guide](../../01%20Documentation/state-bags.md)

Registered in `ResourceHost.cpp` (line 24612) as `LuaStateOnChange`.

## player

```lua
Open77.state.player(id)
```

`CLIENT`: Client-only surface.

The replicated state bag of one player, read-only.

No permission. Accepts a player id as a number or its decimal string. A bag this client is not entitled to reads as empty rather than erroring, so a read is always safe.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer|string` | required | — |

Returns: `bag handle, or nil`, `reason: invalid_state_selector`

```lua
-- Is the player in front of us a cop?
local job = Open77.state.player(otherId).job
if job and job.name == "police" then print("that is a cop") end
```

[Replicated state bags guide](../../01%20Documentation/state-bags.md)

Registered in `ResourceHost.cpp` (line 24608) as `LuaStatePlayerBag`.

## ready

```lua
Open77.state.ready()
```

`CLIENT`: Client-only surface.

Asks the server to resend every state bag this client may see.

Requires `network.events`. Rarely needed: the host asks for a resync itself whenever the server session changes, and the mirror survives a resource restart because it lives in the host rather than in a VM. Use it after a deliberate local reset.

Returns: `true when the request was sent, otherwise false`, `reason: permission_denied:network.events`

```lua
-- After a deliberate local reset, ask for every bag again
Open77.state.ready()
```

[Replicated state bags guide](../../01%20Documentation/state-bags.md)

Registered in `ResourceHost.cpp` (line 24614) as `LuaStateReady`.

## request

```lua
Open77.state.request(selector, key, value)
```

`CLIENT`: Client-only surface.

Asks the server to change one state-bag key.

Requires `network.events`. An acknowledgement of the SEND, never of the outcome: the server runs the owning resource's validator and republishes, so the mirror only moves when the delta comes back. Refused with `not_requestable` unless a server resource opted the key in. Requests are coalesced per key and leave as one net event at most four times a second -- that is what keeps a naive per-keystroke caller from tripping the transport's 32-inbound-events-per-second disconnect.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `selector` | `table|string` | required | — |
| `key` | `string` | required | — |
| `value` | `any` | required | — |

Returns: `true when queued for sending, otherwise false`, `reason: permission_denied:network.events, invalid_state_selector, invalid_state_key, state_value_not_serializable, state_value_too_large, state_request_rate_limit`

```lua
-- Ask the server to change the player's radio frequency (the server validates it)
RegisterNetEvent("radio:tune", function(frequency)
    Open77.state.request({ kind = "player", id = Open77.players.localId() }, "radioFrequency", frequency)
end)
```

[Replicated state bags guide](../../01%20Documentation/state-bags.md)

Registered in `ResourceHost.cpp` (line 24611) as `LuaStateRequest`.
