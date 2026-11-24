# Open77.stats (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-stats.md) · [Page web](https://open2077.net/docs/api/server/open77-stats)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

20 functions.

## get

```lua
Open77.stats.get(playerId)
```

`NETWORK`: Uses the network backend.

A player's combined health and stamina snapshot.

Requires `players.stats.read`. `GetPlayerStats` is the same function. The client exposes the same read names and the same nested pool shape, but no setters -- the server owns both pools. See [Player health and stamina](https://open2077.net/docs/api/server/player-stats.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `stats snapshot, or nil`

```lua
-- /stats: both pools in one read
RegisterCommand("stats", function(source)
    local stats = Open77.stats.get(source)
    if stats then
        Open77.chat.send(source, ("HP %d / %d, stamina %d / %d"):format(
            stats.health.value, stats.health.maximum, stats.stamina.value, stats.stamina.maximum))
    end
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 8896) as `LuaResourceRuntime.Bootstrap`.

## getHealth

```lua
Open77.stats.getHealth(playerId)
```

`NETWORK`: Uses the network backend.

A player's health pool.

Requires `players.stats.read`. Returns `{ value, current, maximum, max, fraction, percentage, regenEnabled, regenPerSecond, regenerating }`, or `nil` for an unknown player. The duplicate spellings are deliberate: they let resources written against either convention read the same table.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `health pool, or nil`

```lua
-- A medic sees the patient's health before treating
local hp = Open77.stats.getHealth(patientId)
if hp and hp.fraction < 0.5 then
    Open77.chat.send(medicId, ("Patient at %d %%"):format(hp.percentage))
end
```

Registered in `LuaResourceRuntime.cs` (line 8897) as `LuaResourceRuntime.Bootstrap`.

## getStamina

```lua
Open77.stats.getStamina(playerId)
```

`NETWORK`: Uses the network backend.

A player's stamina pool.

Requires `players.stats.read`. Same shape as the health pool.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `stamina pool, or nil`

```lua
-- Refuse the sprint-heavy job to an exhausted player
local stamina = Open77.stats.getStamina(source)
if stamina and stamina.fraction < 0.3 then
    return Open77.chat.send(source, "Catch your breath first.")
end
```

Registered in `LuaResourceRuntime.cs` (line 8905) as `LuaResourceRuntime.Bootstrap`.

## health

```lua
Open77.stats.health(playerId)
```

`NETWORK`: Uses the network backend.

Alias of `Open77.stats.getHealth`.

Requires `players.stats.read`. The shorter spelling of the same read, matching the client's.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `health pool, or nil`

```lua
-- Short spelling, same read
local hp = Open77.stats.health(playerId)
print(hp and hp.value)
```

Registered in `LuaResourceRuntime.cs` (line 8901) as `LuaResourceRuntime.Bootstrap`.

## restore

```lua
Open77.stats.restore(playerId, pool)
```

`NETWORK`: Uses the network backend.

Fills one pool.

Requires `players.stats.apply`. `pool` is `health` or `stamina`. Filling health does not revive a dead player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `pool` | `string (health|stamina)` | required | — |

Returns: `true, or false`, `reason`

```lua
-- A pool name from the item definition: "health" or "stamina"
RegisterNetEvent("items:use", function(item)
    if consumables[item] then Open77.stats.restore(source, consumables[item].pool) end
end)
```

Registered in `LuaResourceRuntime.cs` (line 8923) as `LuaResourceRuntime.Bootstrap`.

## restoreHealth

```lua
Open77.stats.restoreHealth(playerId)
```

`NETWORK`: Uses the network backend.

Fills the health pool.

Requires `players.stats.apply`. `RestorePlayerHealth` is the same function; `restore(playerId, "health")` is the pool-generic spelling.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Round start: full health for everyone in the arena
for _, playerId in ipairs(Open77.players.inBucket(ARENA_BUCKET)) do
    Open77.stats.restoreHealth(playerId)
end
```

Registered in `LuaResourceRuntime.cs` (line 8941) as `LuaResourceRuntime.Bootstrap`.

## restoreStamina

```lua
Open77.stats.restoreStamina(playerId)
```

`NETWORK`: Uses the network backend.

Fills the stamina pool.

Requires `players.stats.apply`. `RestorePlayerStamina` is the same function; `restore(playerId, "stamina")` is the pool-generic spelling.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `true, or false`, `reason`

```lua
-- An energy drink
RegisterNetEvent("items:energyDrink", function()
    Open77.stats.restoreStamina(source)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8946) as `LuaResourceRuntime.Bootstrap`.

## set

```lua
Open77.stats.set(playerId, pool, value)
```

`NETWORK`: Uses the network backend.

Sets one pool's current value, in points.

Requires `players.stats.apply`. `pool` is `health` or `stamina`; anything else answers `false, invalid_stat_pool`. Setting health to zero routes through life authority, so it is a real death.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `pool` | `string (health|stamina)` | required | — |
| `value` | `number` | required | — |

Returns: `true, or false`, `reason: invalid_stat_pool among others`

```lua
-- Everyone starts the hardcore round at 50 health
for _, playerId in ipairs(Open77.players.inBucket(ARENA_BUCKET)) do
    Open77.stats.set(playerId, "health", 50)
end
```

Registered in `LuaResourceRuntime.cs` (line 8913) as `LuaResourceRuntime.Bootstrap`.

## setHealth

```lua
Open77.stats.setHealth(playerId, value)
```

`NETWORK`: Uses the network backend.

Explicit health setter.

Requires `players.stats.apply`. `SetPlayerHealth` is the same function; `set(playerId, "health", value)` is the pool-generic spelling. Zero routes through life authority.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `value` | `number` | required | — |

Returns: `true, or false`, `reason`

```lua
-- /sethp <id> <value>
RegisterCommand("sethp", function(source, args)
    local target, value = tonumber(args[1]), tonumber(args[2])
    if target and value then Open77.stats.setHealth(target, value) end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8939) as `LuaResourceRuntime.Bootstrap`.

## setHealthMax

```lua
Open77.stats.setHealthMax(playerId, maximum)
```

`NETWORK`: Uses the network backend.

Explicit health maximum setter.

Requires `players.stats.apply`. `SetPlayerMaxHealth` is the same function; `setMax(playerId, "health", maximum)` is the pool-generic spelling.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `maximum` | `number` | required | — |

Returns: `true, or false`, `reason`

```lua
-- A perk: 150 maximum health
Open77.stats.setHealthMax(playerId, 150)
```

Registered in `LuaResourceRuntime.cs` (line 8940) as `LuaResourceRuntime.Bootstrap`.

## setHealthRegenEnabled

```lua
Open77.stats.setHealthRegenEnabled(playerId, enabled)
```

`NETWORK`: Uses the network backend.

Turns health regeneration on or off.

Requires `players.stats.apply`. `SetPlayerRegenEnabled` is the same function.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Bleeding: no regeneration until bandaged
Open77.stats.setHealthRegenEnabled(playerId, false)
RegisterNetEvent("items:bandage", function()
    Open77.stats.setHealthRegenEnabled(source, true)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8942) as `LuaResourceRuntime.Bootstrap`.

## setHealthRegenRate

```lua
Open77.stats.setHealthRegenRate(playerId, pointsPerSecond)
```

`NETWORK`: Uses the network backend.

Sets the health regeneration rate.

Requires `players.stats.apply`. `SetPlayerRegen` is the same function. Zero disables it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `pointsPerSecond` | `number` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Hardcore server: health never comes back on its own
AddEventHandler("onPlayerReady", function(playerId)
    Open77.stats.setHealthRegenRate(playerId, 0)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8943) as `LuaResourceRuntime.Bootstrap`.

## setMax

```lua
Open77.stats.setMax(playerId, pool, maximum)
```

`NETWORK`: Uses the network backend.

Sets one pool's canonical maximum and clamps its current value.

Requires `players.stats.apply`. `pool` is `health` or `stamina`. Lowering the maximum below the current value clamps it rather than leaving an impossible reading.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `pool` | `string (health|stamina)` | required | — |
| `maximum` | `number` | required | — |

Returns: `true, or false`, `reason`

```lua
-- The pool name comes from the perk definition
Open77.stats.setMax(playerId, perk.pool, perk.maximum)
```

Registered in `LuaResourceRuntime.cs` (line 8918) as `LuaResourceRuntime.Bootstrap`.

## setRegenEnabled

```lua
Open77.stats.setRegenEnabled(playerId, pool, enabled)
```

`NETWORK`: Uses the network backend.

Turns one pool's regeneration on or off.

Requires `players.stats.apply`. `enabled` must be a boolean; anything else answers `false, invalid_argument`. Distinct from a rate of zero: the configured rate is remembered.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `pool` | `string (health|stamina)` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Pause stamina regeneration while the player is carrying a body
Open77.stats.setRegenEnabled(playerId, "stamina", false)
```

Registered in `LuaResourceRuntime.cs` (line 8928) as `LuaResourceRuntime.Bootstrap`.

## setRegenRate

```lua
Open77.stats.setRegenRate(playerId, pool, pointsPerSecond)
```

`NETWORK`: Uses the network backend.

Sets one pool's regeneration rate, in points per second.

Requires `players.stats.apply`. Zero disables regeneration for that pool.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `pool` | `string (health|stamina)` | required | — |
| `pointsPerSecond` | `number` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Athletic perk: stamina regenerates twice as fast
Open77.stats.setRegenRate(playerId, "stamina", 20)
```

Registered in `LuaResourceRuntime.cs` (line 8934) as `LuaResourceRuntime.Bootstrap`.

## setStamina

```lua
Open77.stats.setStamina(playerId, value)
```

`NETWORK`: Uses the network backend.

Explicit stamina setter.

Requires `players.stats.apply`. `SetPlayerStamina` is the same function; `set(playerId, "stamina", value)` is the pool-generic spelling.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `value` | `number` | required | — |

Returns: `true, or false`, `reason`

```lua
-- The player just ran a marathon: stamina to 10
Open77.stats.setStamina(playerId, 10)
```

Registered in `LuaResourceRuntime.cs` (line 8944) as `LuaResourceRuntime.Bootstrap`.

## setStaminaMax

```lua
Open77.stats.setStaminaMax(playerId, maximum)
```

`NETWORK`: Uses the network backend.

Explicit stamina maximum setter.

Requires `players.stats.apply`. `SetPlayerMaxStamina` is the same function; `setMax(playerId, "stamina", maximum)` is the pool-generic spelling.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `maximum` | `number` | required | — |

Returns: `true, or false`, `reason`

```lua
-- A "runner" class gets a bigger stamina pool
Open77.stats.setStaminaMax(playerId, 150)
```

Registered in `LuaResourceRuntime.cs` (line 8945) as `LuaResourceRuntime.Bootstrap`.

## setStaminaRegenEnabled

```lua
Open77.stats.setStaminaRegenEnabled(playerId, enabled)
```

`NETWORK`: Uses the network backend.

Turns stamina regeneration on or off.

Requires `players.stats.apply`. `SetPlayerStaminaRegenEnabled` is the same function.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Exhaustion: no stamina regeneration for 20 seconds
Open77.stats.setStaminaRegenEnabled(playerId, false)
SetTimeout(20000, function() Open77.stats.setStaminaRegenEnabled(playerId, true) end)
```

Registered in `LuaResourceRuntime.cs` (line 8947) as `LuaResourceRuntime.Bootstrap`.

## setStaminaRegenRate

```lua
Open77.stats.setStaminaRegenRate(playerId, pointsPerSecond)
```

`NETWORK`: Uses the network backend.

Sets the stamina regeneration rate.

Requires `players.stats.apply`. `SetPlayerStaminaRegen` is the same function. Zero disables it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `pointsPerSecond` | `number` | required | — |

Returns: `true, or false`, `reason`

```lua
-- A heavy backpack halves stamina regeneration
Open77.stats.setStaminaRegenRate(playerId, 5)
```

Registered in `LuaResourceRuntime.cs` (line 8948) as `LuaResourceRuntime.Bootstrap`.

## stamina

```lua
Open77.stats.stamina(playerId)
```

`NETWORK`: Uses the network backend.

Alias of `Open77.stats.getStamina`.

Requires `players.stats.read`. The shorter spelling of the same read, matching the client's.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `stamina pool, or nil`

```lua
-- Short spelling, same read
local stamina = Open77.stats.stamina(playerId)
print(stamina and stamina.percentage)
```

Registered in `LuaResourceRuntime.cs` (line 8909) as `LuaResourceRuntime.Bootstrap`.
