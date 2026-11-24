# Open77.state (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-state.md) · [Page web](https://open2077.net/docs/api/server/open77-state)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

9 functions.

## allowRequest

```lua
Open77.state.allowRequest(selector, key, validator)
```

`NETWORK`: Uses the network backend.

Opts one bag key in to client write requests, behind a validator.

Requires `state.write`. The client never writes a bag: it submits through `Open77.state.request` and this validator decides. `validator(source, value)` answers `accepted [, normalisedValue]`, and the value the bag stores is the one the validator returned -- clamp or rewrite here. Anything but true refuses. The selector may be a kind string to cover every bag of that kind, or one bag. The validator is retired when the resource stops.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `selector` | `table|string` | required | — |
| `key` | `string` | required | — |
| `validator` | `function` | required | — |

Returns: `true, otherwise false`, `reason: invalid_state_key, invalid_state_selector, validator_required, permission_denied:state.write`

```lua
-- Let a client tune their own radio, clamped to the valid band
Open77.state.allowRequest("player", "radioFrequency", function(source, value)
    if type(value) ~= "number" then return false end
    return true, math.max(88.0, math.min(108.0, value))
end)
```

[Replicated state bags guide](../../01%20Documentation/state-bags.md)

Registered in `LuaResourceRuntime.State.cs` (line 433) as `LuaResourceRuntime.Bootstrap`.

## clear

```lua
Open77.state.clear()
```

`NETWORK`: Uses the network backend.

Drops the stored state.

No permission. `save(nil)` under a clearer name: the next VM to start sees nothing and initialises from scratch.

Returns: `true, or false`, `reason`

```lua
-- A season reset: the next reload starts from scratch
RegisterCommand("season-reset", function()
    Open77.state.clear()
end, true)
```

[Replicated state bags guide](../../01%20Documentation/state-bags.md)

Registered in `LuaResourceRuntime.cs` (line 8162) as `LuaResourceRuntime.Bootstrap`.

## denyRequest

```lua
Open77.state.denyRequest(selector, key)
```

`NETWORK`: Uses the network backend.

Withdraws a client-writable bag key.

Requires `state.write`. Later client requests for that key are refused with `not_requestable`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `selector` | `table|string` | required | — |
| `key` | `string` | required | — |

Returns: `true, otherwise false`, `reason: invalid_state_key, invalid_state_selector`

```lua
-- The radio feature is switched off: clients may no longer request the key
Open77.state.denyRequest("player", "radioFrequency")
```

[Replicated state bags guide](../../01%20Documentation/state-bags.md)

Registered in `LuaResourceRuntime.State.cs` (line 434) as `LuaResourceRuntime.Bootstrap`.

## entity

```lua
Open77.state.entity(kind, [id])
```

`NETWORK`: Uses the network backend.

The replicated state bag of one vehicle, NPC or prop.

Requires `state.write` to write, nothing to read. Accepts `(kind, id)` or `{ kind = , id = }`, where kind is `vehicle`, `npc` or `prop`. The audience is the owning registry's own viewer set, so the bag follows the entity through stream-in, stream-out and bucket changes, and it dies with the entity.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `kind` | `string|table` | required | — |
| `id` | `integer|string` | optional | — |

Returns: `bag handle`, `reason: invalid_state_selector`

```lua
-- Replicate a car's fuel level to everyone who can see the car
Open77.state.entity("vehicle", vehicleId):set("fuel", 61.5)
```

[Replicated state bags guide](../../01%20Documentation/state-bags.md)

Registered in `LuaResourceRuntime.State.cs` (line 430) as `LuaResourceRuntime.Bootstrap`.

## load

```lua
Open77.state.load()
```

`NETWORK`: Uses the network backend.

Reads back the value the previous VM stored.

No permission. Answers `nil` on a fresh start -- which is the signal to initialise rather than resume -- and `nil, reason` when the stored value cannot be decoded.

Returns: `value, or nil`, `reason`

```lua
-- Resume the round state after a hot reload
local saved = Open77.state.load()
round = saved and saved.round or 1
```

[Replicated state bags guide](../../01%20Documentation/state-bags.md)

Registered in `LuaResourceRuntime.cs` (line 8162) as `LuaResourceRuntime.Bootstrap`.

## offChange

```lua
Open77.state.offChange(subscription)
```

`NETWORK`: Uses the network backend.

Stops a state-bag change subscription.

No permission. Takes the id `Open77.state.onChange` returned.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `subscription` | `integer` | required | — |

Returns: `true, otherwise false`, `reason: invalid_subscription`

```lua
-- Stop watching once the match ends
local sub = Open77.state.onChange("global", "round", function(_, _, value) print("round", value) end)
AddEventHandler("arena:matchOver", function() Open77.state.offChange(sub) end)
```

[Replicated state bags guide](../../01%20Documentation/state-bags.md)

Registered in `LuaResourceRuntime.State.cs` (line 432) as `LuaResourceRuntime.Bootstrap`.

## onChange

```lua
Open77.state.onChange(selector, key, handler)
```

`NETWORK`: Uses the network backend.

Watches state-bag changes matching a selector and a key.

No permission. The selector is a bag handle, a `{ kind = , id = }` table, a bare kind string for every bag of that kind, or nil for every bag; the key is a key name or nil for every key. The handler runs at a tick boundary as `fn(selector, key, value, previous)` and may Wait. Only subscribed resources are touched, and subscriptions are retired when the resource stops.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `selector` | `table|string|nil` | required | — |
| `key` | `string|nil` | required | — |
| `handler` | `function` | required | — |

Returns: `subscription id, or nil`, `reason: invalid_state_selector, invalid_state_key`

```lua
-- Log every job change on any player
Open77.state.onChange("player", "job", function(selector, key, value, previous)
    print(("player %s job: %s -> %s"):format(tostring(selector.id), tostring(previous and previous.name), tostring(value and value.name)))
end)
```

[Replicated state bags guide](../../01%20Documentation/state-bags.md)

Registered in `LuaResourceRuntime.State.cs` (line 431) as `LuaResourceRuntime.Bootstrap`.

## player

```lua
Open77.state.player(id)
```

`NETWORK`: Uses the network backend.

The replicated state bag of one player.

Requires `state.write` to write, nothing to read. Replicated to every player in that player's routing bucket, including the subject; a player in another bucket never receives it. The bag dies with the player's session.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer|string` | required | — |

Returns: `bag handle`, `reason: invalid_state_selector`

```lua
-- Replicate the player's job so every client in the bucket can read it
Open77.state.player(source):set("job", { name = "police", grade = 3 })
```

[Replicated state bags guide](../../01%20Documentation/state-bags.md)

Registered in `LuaResourceRuntime.State.cs` (line 429) as `LuaResourceRuntime.Bootstrap`.

## save

```lua
Open77.state.save(value)
```

`NETWORK`: Uses the network backend.

Stores this resource's state so it survives a reload.

No permission. **Not a key-value store**: it holds one value per resource, round-tripped through JSON by the host, so store plain data -- functions, coroutines and metatables do not survive, and cycles are rejected. Passing `nil` clears it. Call it **when the state changes, not on stop**: a reload prepares the successor VM *before* stopping this one, so a write from a stopping resource is refused and returns false. It deliberately does not survive the resource going down: `stop`, `restart` and `refresh` drop it, which keeps an operator's way of saying "come up as you would at boot".

| Parameter | Type | Required | Default |
|---|---|---|---|
| `value` | `any JSON-encodable value, or nil to clear` | required | — |

Returns: `true, or false`, `reason: unserialisable_state among others`

```lua
-- Save the round state every time it changes, so a reload resumes it
local function setRound(n)
    round = n
    Open77.state.save({ round = round })
end
```

[Replicated state bags guide](../../01%20Documentation/state-bags.md)

Registered in `LuaResourceRuntime.cs` (line 8162) as `LuaResourceRuntime.Bootstrap`.
