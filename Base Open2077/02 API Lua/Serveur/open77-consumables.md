# Open77.consumables (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-consumables.md) · [Page web](https://open2077.net/docs/api/server/open77-consumables)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

9 functions.

## configure

```lua
Open77.consumables.configure(playerId, kind, options)
```

`NETWORK`: Uses the network backend.

Configures a grenade or healing kit.

Server; requires `network.events`. Returns a request ID when queued, not when applied. Wait for open77:weapons:completed with operation consumable before another write to the same kit. Requires the open77_weapons owner relay 0.2.0 and a compatible client. Server completion includes playerId, requestId, operation, boolean accepted, reason and a typed state table. Native refusal includes consumable_busy, invalid_consumable_type and readback mismatches. Failed writes report observed partial state without atomic rollback. Set record, count (0..999), capacity (1..999), equipped and recharge. Unknown options or count above an explicit capacity are rejected. Omitted fields preserve state; capacity grows to fit count if omitted. The first managed write defaults to recharge=false. See [Grenades and healing items](../../01%20Documentation/consumables-api.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `kind` | `"grenade" or "healing"` | required | — |
| `options` | `table` | required | — |

Returns: `requestId or nil`, `reason on immediate refusal`

```lua
-- Call from your server's validated spawn/loadout handler.
local result, reason = Open77.consumables.configure(playerId, "grenade", { record = "Items.GrenadeFragRegular", count = 5, capacity = 5, equipped = true, recharge = false })
if not result then print(reason) end
```

[Grenades & healing items guide](../../01%20Documentation/consumables-api.md)

Registered in `server/src/Open77.Server.Scripting/Runtime/LuaResourceRuntime.Consumables.cs` (line 18) as `LuaResourceRuntime.ConsumablesBootstrap`.

## equip

```lua
Open77.consumables.equip(playerId, kind, [record])
```

`NETWORK`: Uses the network backend.

Equips the selected consumable on its hotkey.

Server; requires `network.events`. Returns a request ID when queued, not when applied. Wait for open77:weapons:completed with operation consumable before another write to the same kit. Requires the open77_weapons owner relay 0.2.0 and a compatible client. Server completion includes playerId, requestId, operation, boolean accepted, reason and a typed state table. Native refusal includes consumable_busy, invalid_consumable_type and readback mismatches. Failed writes report observed partial state without atomic rollback. Optionally selects a new record. Omitting record uses the retained type and preserves its charges. Fails with no_consumable_type if none is available. See [Grenades and healing items](../../01%20Documentation/consumables-api.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `kind` | `"grenade" or "healing"` | required | — |
| `record` | `string` | optional | — |

Returns: `requestId or nil`, `reason on immediate refusal`

```lua
-- Call from your server's validated spawn/loadout handler.
local result, reason = Open77.consumables.equip(playerId, "healing", "Items.FirstAidWhiffV0")
if not result then print(reason) end
```

[Grenades & healing items guide](../../01%20Documentation/consumables-api.md)

Registered in `server/src/Open77.Server.Scripting/Runtime/LuaResourceRuntime.Consumables.cs` (line 43) as `LuaResourceRuntime.ConsumablesBootstrap`.

## get

```lua
Open77.consumables.get(playerId, kind)
```

`NETWORK`: Uses the network backend.

Reads cached consumable state.

Server; requires `player.weapons.read`. Source is owner_report from the authenticated player's client; missing reports return consumable_unreported. Returns kind, record, tweakDbId, count, capacity, equipped, recharge, managed, sequence, reportedAgeMs and source. An unresolved native record name can be empty; use tweakDbId to identify it. Age measures time since observation, not a heartbeat. Use snapshot when fresh state is required. See [Grenades and healing items](../../01%20Documentation/consumables-api.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `kind` | `"grenade" or "healing"` | required | — |

Returns: `state table or nil`, `reason on failure`

```lua
-- Call from your server's validated spawn/loadout handler.
local result, reason = Open77.consumables.get(playerId, "healing")
if not result then print(reason) end
```

[Grenades & healing items guide](../../01%20Documentation/consumables-api.md)

Registered in `server/src/Open77.Server.Scripting/Runtime/LuaResourceRuntime.Consumables.cs` (line 60) as `LuaResourceRuntime.ConsumablesBootstrap`.

## reset

```lua
Open77.consumables.reset(playerId, kind)
```

`NETWORK`: Uses the network backend.

Releases managed consumable capacity and regeneration.

Server; requires `network.events`. Returns a request ID when queued, not when applied. Wait for open77:weapons:completed with operation consumable before another write to the same kit. Requires the open77_weapons owner relay 0.2.0 and a compatible client. Server completion includes playerId, requestId, operation, boolean accepted, reason and a typed state table. Native refusal includes consumable_busy, invalid_consumable_type and readback mismatches. Failed writes report observed partial state without atomic rollback. Restores the captured native regeneration policy and removes this API's capacity modifiers. Preserves the selected item, clamps excess charges to restored capacity, and leaves unrelated inventory and modifiers intact. See [Grenades and healing items](../../01%20Documentation/consumables-api.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `kind` | `"grenade" or "healing"` | required | — |

Returns: `requestId or nil`, `reason on immediate refusal`

```lua
-- Call from your server's validated spawn/loadout handler.
local result, reason = Open77.consumables.reset(playerId, "grenade")
if not result then print(reason) end
```

[Grenades & healing items guide](../../01%20Documentation/consumables-api.md)

Registered in `server/src/Open77.Server.Scripting/Runtime/LuaResourceRuntime.Consumables.cs` (line 54) as `LuaResourceRuntime.ConsumablesBootstrap`.

## setCount

```lua
Open77.consumables.setCount(playerId, kind, count)
```

`NETWORK`: Uses the network backend.

Sets usable consumable charges.

Server; requires `network.events`. Returns a request ID when queued, not when applied. Wait for open77:weapons:completed with operation consumable before another write to the same kit. Requires the open77_weapons owner relay 0.2.0 and a compatible client. Server completion includes playerId, requestId, operation, boolean accepted, reason and a typed state table. Native refusal includes consumable_busy, invalid_consumable_type and readback mismatches. Failed writes report observed partial state without atomic rollback. Count is 0..999, not an inventory stack size. Zero retains the selected type without allowing a use. Capacity grows when needed. See [Grenades and healing items](../../01%20Documentation/consumables-api.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `kind` | `"grenade" or "healing"` | required | — |
| `count` | `integer` | required | — |

Returns: `requestId or nil`, `reason on immediate refusal`

```lua
-- Call from your server's validated spawn/loadout handler.
local result, reason = Open77.consumables.setCount(playerId, "grenade", 5)
if not result then print(reason) end
```

[Grenades & healing items guide](../../01%20Documentation/consumables-api.md)

Registered in `server/src/Open77.Server.Scripting/Runtime/LuaResourceRuntime.Consumables.cs` (line 35) as `LuaResourceRuntime.ConsumablesBootstrap`.

## setRecharge

```lua
Open77.consumables.setRecharge(playerId, kind, enabled)
```

`NETWORK`: Uses the network backend.

Controls native consumable regeneration.

Server; requires `network.events`. Returns a request ID when queued, not when applied. Wait for open77:weapons:completed with operation consumable before another write to the same kit. Requires the open77_weapons owner relay 0.2.0 and a compatible client. Server completion includes playerId, requestId, operation, boolean accepted, reason and a typed state table. Native refusal includes consumable_busy, invalid_consumable_type and readback mismatches. Failed writes report observed partial state without atomic rollback. Accepts a strict boolean. False gives a finite charge supply; true enables native regeneration. Native capacity and regeneration modifiers are released on reset or player/session replacement. See [Grenades and healing items](../../01%20Documentation/consumables-api.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `kind` | `"grenade" or "healing"` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `requestId or nil`, `reason on immediate refusal`

```lua
-- Call from your server's validated spawn/loadout handler.
local result, reason = Open77.consumables.setRecharge(playerId, "healing", false)
if not result then print(reason) end
```

[Grenades & healing items guide](../../01%20Documentation/consumables-api.md)

Registered in `server/src/Open77.Server.Scripting/Runtime/LuaResourceRuntime.Consumables.cs` (line 50) as `LuaResourceRuntime.ConsumablesBootstrap`.

## setType

```lua
Open77.consumables.setType(playerId, kind, record)
```

`NETWORK`: Uses the network backend.

Changes the selected consumable type.

Server; requires `network.events`. Returns a request ID when queued, not when applied. Wait for open77:weapons:completed with operation consumable before another write to the same kit. Requires the open77_weapons owner relay 0.2.0 and a compatible client. Server completion includes playerId, requestId, operation, boolean accepted, reason and a typed state table. Native refusal includes consumable_busy, invalid_consumable_type and readback mismatches. Failed writes report observed partial state without atomic rollback. Preserves charges and equipped state. The record must resolve to the matching grenade, inhaler or injector type; an Items.* name alone is insufficient. See [Grenades and healing items](../../01%20Documentation/consumables-api.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `kind` | `"grenade" or "healing"` | required | — |
| `record` | `string` | required | — |

Returns: `requestId or nil`, `reason on immediate refusal`

```lua
-- Call from your server's validated spawn/loadout handler.
local result, reason = Open77.consumables.setType(playerId, "healing", "Items.FirstAidWhiffV0")
if not result then print(reason) end
```

[Grenades & healing items guide](../../01%20Documentation/consumables-api.md)

Registered in `server/src/Open77.Server.Scripting/Runtime/LuaResourceRuntime.Consumables.cs` (line 39) as `LuaResourceRuntime.ConsumablesBootstrap`.

## snapshot

```lua
Open77.consumables.snapshot(playerId, kind)
```

`NETWORK`: Uses the network backend.

Requests fresh native consumable state.

Server; requires `network.events`. Returns a request ID when queued, not when applied. Wait for open77:weapons:completed with operation consumable before another write to the same kit. Requires the open77_weapons owner relay 0.2.0 and a compatible client. Server completion includes playerId, requestId, operation, boolean accepted, reason and a typed state table. Native refusal includes consumable_busy, invalid_consumable_type and readback mismatches. Failed writes report observed partial state without atomic rollback. Asynchronous readback of the actual selected type, charges, capacity, equipment and regeneration. Use get for a synchronous cached observation. See [Grenades and healing items](../../01%20Documentation/consumables-api.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `kind` | `"grenade" or "healing"` | required | — |

Returns: `requestId or nil`, `reason on immediate refusal`

```lua
-- Call from your server's validated spawn/loadout handler.
local result, reason = Open77.consumables.snapshot(playerId, "healing")
if not result then print(reason) end
```

[Grenades & healing items guide](../../01%20Documentation/consumables-api.md)

Registered in `server/src/Open77.Server.Scripting/Runtime/LuaResourceRuntime.Consumables.cs` (line 54) as `LuaResourceRuntime.ConsumablesBootstrap`.

## unequip

```lua
Open77.consumables.unequip(playerId, kind)
```

`NETWORK`: Uses the network backend.

Clears the consumable hotkey and equipment slots.

Server; requires `network.events`. Returns a request ID when queued, not when applied. Wait for open77:weapons:completed with operation consumable before another write to the same kit. Requires the open77_weapons owner relay 0.2.0 and a compatible client. Server completion includes playerId, requestId, operation, boolean accepted, reason and a typed state table. Native refusal includes consumable_busy, invalid_consumable_type and readback mismatches. Failed writes report observed partial state without atomic rollback. Preserves selected type and charges so equip can restore the kit. See [Grenades and healing items](../../01%20Documentation/consumables-api.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `kind` | `"grenade" or "healing"` | required | — |

Returns: `requestId or nil`, `reason on immediate refusal`

```lua
-- Call from your server's validated spawn/loadout handler.
local result, reason = Open77.consumables.unequip(playerId, "grenade")
if not result then print(reason) end
```

[Grenades & healing items guide](../../01%20Documentation/consumables-api.md)

Registered in `server/src/Open77.Server.Scripting/Runtime/LuaResourceRuntime.Consumables.cs` (line 47) as `LuaResourceRuntime.ConsumablesBootstrap`.
