# Open77.consumables (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-consumables.md) · [Page web](https://open2077.net/docs/api/client/open77-consumables)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

9 functions.

## configure

```lua
Open77.consumables.configure(kind, options)
```

`GAME`: Requires a live game instance.

Configures a grenade or healing kit.

Client-only; requires `player.weapons.edit`. Returns a request ID when queued, not when applied. Wait for open77:weapons:completed with operation consumable before another write to the same kit. Client completion accepted is a string; open77:consumables:state precedes completion. Native refusal includes consumable_busy, invalid_consumable_type and readback mismatches. Failed writes report observed partial state without atomic rollback. Set record, count (0..999), capacity (1..999), equipped and recharge. Unknown options or count above an explicit capacity are rejected. Omitted fields preserve state; capacity grows to fit count if omitted. The first managed write defaults to recharge=false. See [Grenades and healing items](../../01%20Documentation/consumables-api.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `kind` | `"grenade" or "healing"` | required | — |
| `options` | `table` | required | — |

Returns: `requestId or nil`, `reason on immediate refusal`

```lua
CreateThread(function()
    local result, reason = Open77.consumables.configure("grenade", { record = "Items.GrenadeFragRegular", count = 5, capacity = 5, equipped = true, recharge = false })
    if not result then print(reason) end
end)
```

[Grenades & healing items guide](../../01%20Documentation/consumables-api.md)

Registered in `ResourceHost.cpp` (line 24967) as `LuaConsumable`.

## equip

```lua
Open77.consumables.equip(kind, [record])
```

`GAME`: Requires a live game instance.

Equips the selected consumable on its hotkey.

Client-only; requires `player.weapons.edit`. Returns a request ID when queued, not when applied. Wait for open77:weapons:completed with operation consumable before another write to the same kit. Client completion accepted is a string; open77:consumables:state precedes completion. Native refusal includes consumable_busy, invalid_consumable_type and readback mismatches. Failed writes report observed partial state without atomic rollback. Optionally selects a new record. Omitting record uses the retained type and preserves its charges. Fails with no_consumable_type if none is available. See [Grenades and healing items](../../01%20Documentation/consumables-api.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `kind` | `"grenade" or "healing"` | required | — |
| `record` | `string` | optional | — |

Returns: `requestId or nil`, `reason on immediate refusal`

```lua
CreateThread(function()
    local result, reason = Open77.consumables.equip("healing", "Items.FirstAidWhiffV0")
    if not result then print(reason) end
end)
```

[Grenades & healing items guide](../../01%20Documentation/consumables-api.md)

Registered in `ResourceHost.cpp` (line 24967) as `LuaConsumable`.

## get

```lua
Open77.consumables.get(kind)
```

`GAME`: Requires a live game instance.

Reads cached consumable state.

Client-only; requires `player.weapons.read`. Source is native_readback. Returns kind, record, tweakDbId, count, capacity, equipped, recharge, managed, sequence, reportedAgeMs and source. An unresolved native record name can be empty; use tweakDbId to identify it. Age measures time since observation, not a heartbeat. Use snapshot when fresh state is required. See [Grenades and healing items](../../01%20Documentation/consumables-api.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `kind` | `"grenade" or "healing"` | required | — |

Returns: `state table or nil`, `reason on failure`

```lua
CreateThread(function()
    local result, reason = Open77.consumables.get("healing")
    if not result then print(reason) end
end)
```

[Grenades & healing items guide](../../01%20Documentation/consumables-api.md)

Registered in `ResourceHost.cpp` (line 24967) as `LuaConsumable`.

## reset

```lua
Open77.consumables.reset(kind)
```

`GAME`: Requires a live game instance.

Releases managed consumable capacity and regeneration.

Client-only; requires `player.weapons.edit`. Returns a request ID when queued, not when applied. Wait for open77:weapons:completed with operation consumable before another write to the same kit. Client completion accepted is a string; open77:consumables:state precedes completion. Native refusal includes consumable_busy, invalid_consumable_type and readback mismatches. Failed writes report observed partial state without atomic rollback. Restores the captured native regeneration policy and removes this API's capacity modifiers. Preserves the selected item, clamps excess charges to restored capacity, and leaves unrelated inventory and modifiers intact. See [Grenades and healing items](../../01%20Documentation/consumables-api.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `kind` | `"grenade" or "healing"` | required | — |

Returns: `requestId or nil`, `reason on immediate refusal`

```lua
CreateThread(function()
    local result, reason = Open77.consumables.reset("grenade")
    if not result then print(reason) end
end)
```

[Grenades & healing items guide](../../01%20Documentation/consumables-api.md)

Registered in `ResourceHost.cpp` (line 24967) as `LuaConsumable`.

## setCount

```lua
Open77.consumables.setCount(kind, count)
```

`GAME`: Requires a live game instance.

Sets usable consumable charges.

Client-only; requires `player.weapons.edit`. Returns a request ID when queued, not when applied. Wait for open77:weapons:completed with operation consumable before another write to the same kit. Client completion accepted is a string; open77:consumables:state precedes completion. Native refusal includes consumable_busy, invalid_consumable_type and readback mismatches. Failed writes report observed partial state without atomic rollback. Count is 0..999, not an inventory stack size. Zero retains the selected type without allowing a use. Capacity grows when needed. See [Grenades and healing items](../../01%20Documentation/consumables-api.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `kind` | `"grenade" or "healing"` | required | — |
| `count` | `integer` | required | — |

Returns: `requestId or nil`, `reason on immediate refusal`

```lua
CreateThread(function()
    local result, reason = Open77.consumables.setCount("grenade", 5)
    if not result then print(reason) end
end)
```

[Grenades & healing items guide](../../01%20Documentation/consumables-api.md)

Registered in `ResourceHost.cpp` (line 24967) as `LuaConsumable`.

## setRecharge

```lua
Open77.consumables.setRecharge(kind, enabled)
```

`GAME`: Requires a live game instance.

Controls native consumable regeneration.

Client-only; requires `player.weapons.edit`. Returns a request ID when queued, not when applied. Wait for open77:weapons:completed with operation consumable before another write to the same kit. Client completion accepted is a string; open77:consumables:state precedes completion. Native refusal includes consumable_busy, invalid_consumable_type and readback mismatches. Failed writes report observed partial state without atomic rollback. Accepts a strict boolean. False gives a finite charge supply; true enables native regeneration. Native capacity and regeneration modifiers are released on reset or player/session replacement. See [Grenades and healing items](../../01%20Documentation/consumables-api.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `kind` | `"grenade" or "healing"` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `requestId or nil`, `reason on immediate refusal`

```lua
CreateThread(function()
    local result, reason = Open77.consumables.setRecharge("healing", false)
    if not result then print(reason) end
end)
```

[Grenades & healing items guide](../../01%20Documentation/consumables-api.md)

Registered in `ResourceHost.cpp` (line 24967) as `LuaConsumable`.

## setType

```lua
Open77.consumables.setType(kind, record)
```

`GAME`: Requires a live game instance.

Changes the selected consumable type.

Client-only; requires `player.weapons.edit`. Returns a request ID when queued, not when applied. Wait for open77:weapons:completed with operation consumable before another write to the same kit. Client completion accepted is a string; open77:consumables:state precedes completion. Native refusal includes consumable_busy, invalid_consumable_type and readback mismatches. Failed writes report observed partial state without atomic rollback. Preserves charges and equipped state. The record must resolve to the matching grenade, inhaler or injector type; an Items.* name alone is insufficient. See [Grenades and healing items](../../01%20Documentation/consumables-api.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `kind` | `"grenade" or "healing"` | required | — |
| `record` | `string` | required | — |

Returns: `requestId or nil`, `reason on immediate refusal`

```lua
CreateThread(function()
    local result, reason = Open77.consumables.setType("healing", "Items.FirstAidWhiffV0")
    if not result then print(reason) end
end)
```

[Grenades & healing items guide](../../01%20Documentation/consumables-api.md)

Registered in `ResourceHost.cpp` (line 24967) as `LuaConsumable`.

## snapshot

```lua
Open77.consumables.snapshot(kind)
```

`GAME`: Requires a live game instance.

Requests fresh native consumable state.

Client-only; requires `player.weapons.read`. Returns a request ID when queued, not when applied. Wait for open77:weapons:completed with operation consumable before another write to the same kit. Client completion accepted is a string; open77:consumables:state precedes completion. Native refusal includes consumable_busy, invalid_consumable_type and readback mismatches. Failed writes report observed partial state without atomic rollback. Asynchronous readback of the actual selected type, charges, capacity, equipment and regeneration. Use get for a synchronous cached observation. See [Grenades and healing items](../../01%20Documentation/consumables-api.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `kind` | `"grenade" or "healing"` | required | — |

Returns: `requestId or nil`, `reason on immediate refusal`

```lua
CreateThread(function()
    local result, reason = Open77.consumables.snapshot("healing")
    if not result then print(reason) end
end)
```

[Grenades & healing items guide](../../01%20Documentation/consumables-api.md)

Registered in `ResourceHost.cpp` (line 24967) as `LuaConsumable`.

## unequip

```lua
Open77.consumables.unequip(kind)
```

`GAME`: Requires a live game instance.

Clears the consumable hotkey and equipment slots.

Client-only; requires `player.weapons.edit`. Returns a request ID when queued, not when applied. Wait for open77:weapons:completed with operation consumable before another write to the same kit. Client completion accepted is a string; open77:consumables:state precedes completion. Native refusal includes consumable_busy, invalid_consumable_type and readback mismatches. Failed writes report observed partial state without atomic rollback. Preserves selected type and charges so equip can restore the kit. See [Grenades and healing items](../../01%20Documentation/consumables-api.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `kind` | `"grenade" or "healing"` | required | — |

Returns: `requestId or nil`, `reason on immediate refusal`

```lua
CreateThread(function()
    local result, reason = Open77.consumables.unequip("grenade")
    if not result then print(reason) end
end)
```

[Grenades & healing items guide](../../01%20Documentation/consumables-api.md)

Registered in `ResourceHost.cpp` (line 24967) as `LuaConsumable`.
