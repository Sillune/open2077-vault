# Open77.combat (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-combat.md) · [Page web](https://open2077.net/docs/api/server/open77-combat)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

11 functions.

## createScope

```lua
Open77.combat.createScope(definition)
```

`NETWORK`: Uses the network backend.

Enable scoped PvP for an explicit ready roster.

Requires combat.scope.control. definition={bucket,players={...}} accepts2..32 unique positive player IDs currently combat-capable in that bucket; a player belongs to at most one scope. Limits64 scopes globally/8 per resource. Scope can permit PvP while global PvP is off but keeps team, life, invulnerability and Lua veto rules. Stop/disconnect/bucket changes remove ownership/participants. Creator must enforce opt-in; no automatic team or bucket change occurs.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `definition` | `table` | required | — |

Returns: `scope ID string`, `nil, reason on rejection`

```lua
local scope, err = Open77.combat.createScope({bucket=arenaBucket,players={firstPlayer,secondPlayer}})
if not scope then print(err); return end
-- Retain scope for teardown.
```

Registered in `LuaResourceRuntime.cs` (line 8951) as `LuaResourceRuntime.Bootstrap`.

## getVehicleHits

```lua
Open77.combat.getVehicleHits(playerId, [sinceMs])
```

`NETWORK`: Uses the network backend.

Returns the car impacts the server recorded on one player, with the culprit it found for each.

Requires `players.stats.read` (or the legacy `players.damage.read` / `players.life.read`), like every other read of the damage ledger. When a car strikes a player only the victim's game sees it, so its client reports a self-targeted environment hit; the server then finds, from the canonical vehicle poses it accepted over the last second, the vehicle whose path came within reach of the impact point while moving toward it, and its seated driver. Nothing the victim claims names a driver. The list is oldest first and holds at most 32 entries (at most one per second), none older than ten minutes; it is released when the player disconnects. `sinceMs` (the `GetGameTimer()` clock) keeps the entries at or after it. Each entry is the table `open77:playerHitByVehicle` hands its handler: `{ victim, driver, vehicle, passengers, physicsOwner, reason, speed, vehicleSpeed, distance, position = {x, y, z}, time, damage, outcome, credited, lethal, downed }`. `driver` and `vehicle` are `0` when none was found, and `reason` says why: `driver`, `npc_driver`, `unowned_vehicle` (parked, rolling or stewarded, nobody at the wheel), `no_vehicle_found` (NPC traffic or a claim with no car near) or `unverified_position`. A passenger is listed in `passengers`, never as the driver. `outcome` is `damaged`, `cancelled` (a damage arbiter) or the protection that refused it (`god_mode`, `friendly_fire_disabled`, `same_team`); `damage` is what the ledger took. The damage itself stays the victim's own environment hit unless the server enables `combat.vehicleHitCreditsDriver` (see Player stats, Car impacts).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `sinceMs` | `number` | optional | — |

Returns: `array of impact tables (empty when none), or nil`, `reason: permission_denied:players.stats.read, invalid_player_id, invalid_argument, combat_unavailable`

```lua
-- A culprit log for car impacts: every one lands in this resource's KVP
-- store, and /carhits <player> prints what the server still holds for them.
AddEventHandler("open77:playerHitByVehicle", function(victim, driver, vehicle, impact)
    local line = ("victim %d hit by driver %d in vehicle %d at %.1f m/s (%s, %s)"):format(
        victim, driver, vehicle, impact.speed, impact.reason, impact.outcome)
    print(line)
    local count = Open77.kvp.increment("carhits:count", 1)
    if count then Open77.kvp.set("carhits:" .. count, line) end
end)

RegisterCommand("carhits", function(source, args)
    local player = tonumber(args[1]) or source
    local hits, reason = Open77.combat.getVehicleHits(player)
    if not hits then print("carhits: " .. reason) return end
    for _, hit in ipairs(hits) do
        print(("%d ms: driver %d, vehicle %d, %.1f m/s, %s, %.0f HP"):format(
            hit.time, hit.driver, hit.vehicle, hit.speed, hit.outcome, hit.damage))
    end
end, true)
```

Registered in `LuaResourceRuntime.VehicleHits.cs` (line 113) as `LuaResourceRuntime.Bootstrap`.

## offDamage

```lua
Open77.combat.offDamage(handler)
```

`NETWORK`: Uses the network backend.

Removes a previously installed damage arbiter.

Requires `combat.config`. Pass the value `onDamage` returned. Removing a handler that is not installed is a no-op.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `handler` | `function` | required | — |

```lua
-- The "double damage" event is over: remove the arbiter
local arbiter = Open77.combat.onDamage(function(event) return event.amount * 2 end)
AddEventHandler("event:ended", function()
    Open77.combat.offDamage(arbiter)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8966) as `LuaResourceRuntime.Bootstrap`.

## onDamage

```lua
Open77.combat.onDamage(handler)
```

`NETWORK`: Uses the network backend.

Installs a synchronous damage arbiter and returns it.

Requires `combat.config`. The handler receives one event table `{ victim, attacker, amount, attackKind, bodyPart, weaponTdbId, distance }` for every candidate damage application. A car impact on a player also carries `source = "vehicle"`, `vehicle` and `driver` (the vehicle and seated driver the server found, `0` when none), `speed` (server-observed relative m/s) and `vehicleReason` (`driver`, `npc_driver`, `unowned_vehicle`, `no_vehicle_found`, `unverified_position`); `attacker` is the victim itself unless the server option `combat.vehicleHitCreditsDriver` credits the driver. Returning `false` cancels the damage and the first refusal wins; returning a number rewrites the amount for the arbiters that follow; anything else leaves it alone. Handlers run inside a `pcall`, so one that raises is skipped rather than dropping the hit. Keep the body short: this runs on the server tick, in the damage path.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `handler` | `function(event)` | required | — |

Returns: `the handler, so it can be stored for offDamage`

```lua
local arbiter = Open77.combat.onDamage(function(event)
  if event.bodyPart == "head" then return event.amount * 2 end
end)
```

Registered in `LuaResourceRuntime.cs` (line 8961) as `LuaResourceRuntime.Bootstrap`.

## removeScope

```lua
Open77.combat.removeScope(id)
```

`NETWORK`: Uses the network backend.

Remove this resource's scoped PvP permission.

Requires combat.scope.control. Accepts the32-character scope ID returned by createScope. Does not alter global PvP or revive/teleport participants. Missing or foreign scopes fail; ordinary combat policy applies after removal.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |

Returns: `true`, `nil, reason on rejection`

```lua
-- The duel is over: drop the scoped PvP permission
AddEventHandler("duel:ended", function()
    local ok, err = Open77.combat.removeScope(duelScope)
    if not ok then print(err) end
end)
```

Registered in `LuaResourceRuntime.cs` (line 8956) as `LuaResourceRuntime.Bootstrap`.

## setDamageMultiplier

```lua
Open77.combat.setDamageMultiplier(multiplier)
```

`NETWORK`: Uses the network backend.

Scales every damage application on the server.

Requires `combat.config`. Range 0 to 100; `0` makes the server effectively damage-free without touching god mode. Applied before the headshot, weapon and kind multipliers.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `multiplier` | `number (0..100)` | required | — |

Returns: `true, or false`, `reason`

```lua
-- /dmg <factor>: an admin halves or doubles all damage for the round
RegisterCommand("dmg", function(source, args)
    local factor = tonumber(args[1])
    if factor then Open77.combat.setDamageMultiplier(factor) end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8973) as `LuaResourceRuntime.Bootstrap`.

## setFriendlyFire

```lua
Open77.combat.setFriendlyFire(enabled)
```

`NETWORK`: Uses the network backend.

Enables or disables global player-versus-player damage.

Requires `combat.config`. Players carrying the same nonzero team remain protected even when friendly fire is on, so a team mode does not have to arbitrate every hit itself. For a duel between two named players while global PvP stays off, use `createScope` instead.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, or false`, `reason`

```lua
-- PvP is on everywhere on this server
Open77.combat.setFriendlyFire(true)
```

Registered in `LuaResourceRuntime.cs` (line 8971) as `LuaResourceRuntime.Bootstrap`.

## setHeadshotMultiplier

```lua
Open77.combat.setHeadshotMultiplier(multiplier)
```

`NETWORK`: Uses the network backend.

Scales damage from hits on the head.

Requires `combat.config`. Range 0 to 100. Combines with the global multiplier rather than replacing it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `multiplier` | `number (0..100)` | required | — |

Returns: `true, or false`, `reason`

```lua
-- A "one in the head" round: headshots do five times the damage
Open77.combat.setHeadshotMultiplier(5)
```

Registered in `LuaResourceRuntime.cs` (line 8974) as `LuaResourceRuntime.Bootstrap`.

## setKindDamageMultiplier

```lua
Open77.combat.setKindDamageMultiplier(kind, multiplier)
```

`NETWORK`: Uses the network backend.

Overrides the damage of one attack kind.

Requires `combat.config`. Kinds are `ranged`, `melee` and `explosion`, range 0 to 100. Useful for a melee-only round without removing weapons from anyone.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `kind` | `string (ranged|melee|explosion)` | required | — |
| `multiplier` | `number (0..100)` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Fists-only round: guns and explosions do nothing, melee does double
Open77.combat.setKindDamageMultiplier("ranged", 0)
Open77.combat.setKindDamageMultiplier("explosion", 0)
Open77.combat.setKindDamageMultiplier("melee", 2)
```

Registered in `LuaResourceRuntime.cs` (line 8976) as `LuaResourceRuntime.Bootstrap`.

## setTeam

```lua
Open77.combat.setTeam(playerId, teamId)
```

`NETWORK`: Uses the network backend.

Assigns a player to a combat team.

Requires `combat.config`. Team ids are non-negative; `0` means no team. Two players on the same nonzero team never damage each other, whatever the friendly-fire setting.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `teamId` | `integer` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Split the arena players into two teams; team-mates cannot hurt each other
for i, playerId in ipairs(Open77.players.inBucket(ARENA_BUCKET)) do
    Open77.combat.setTeam(playerId, (i % 2) + 1)
end
```

Registered in `LuaResourceRuntime.cs` (line 8972) as `LuaResourceRuntime.Bootstrap`.

## setWeaponDamageMultiplier

```lua
Open77.combat.setWeaponDamageMultiplier(weaponTdbId, multiplier)
```

`NETWORK`: Uses the network backend.

Overrides the damage of one weapon record.

Requires `combat.config`. Range 0 to 100, keyed by the weapon's TweakDB id, so a single gun can be tuned without rebalancing everything.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `weaponTdbId` | `string` | required | — |
| `multiplier` | `number (0..100)` | required | — |

Returns: `true, or false`, `reason`

```lua
-- The Lexington is a training pistol here: a quarter of its damage
Open77.combat.setWeaponDamageMultiplier("Items.Preset_Lexington_Default", 0.25)
```

Registered in `LuaResourceRuntime.cs` (line 8975) as `LuaResourceRuntime.Bootstrap`.
