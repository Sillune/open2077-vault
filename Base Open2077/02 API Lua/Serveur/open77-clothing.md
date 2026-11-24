# Open77.clothing (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-clothing.md) · [Page web](https://open2077.net/docs/api/server/open77-clothing)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

5 functions.

## clear

```lua
Open77.clothing.clear(playerId)
```

`NETWORK`: Uses the network backend.

Asks a player's client to empty every clothing slot.

Asynchronous, same relay and completion contract as the other clothing calls.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `request id, or nil`, `reason`

```lua
-- Prison intake: strip every clothing slot
Open77.clothing.clear(prisonerId)
```

Registered in `LuaResourceRuntime.cs` (line 8291) as `LuaResourceRuntime.Bootstrap`.

## equip

```lua
Open77.clothing.equip(playerId, record, [options])
```

`NETWORK`: Uses the network backend.

Asks a player's client to equip one clothing record.

Asynchronous. The calling resource must declare `network.events`; the request is relayed to the official `open77_equipment` client package. Returns a request id prefixed with the resource name -- only this resource's VM can match the completion. Listen for the resource-local `open77:clothing:completed` event with `playerId, requestId, operation, accepted, reason, result`; an unanswered request completes with `request_timeout` after ten seconds. Completion is acceptance by the client relay, not proof that other observers see the change.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `record` | `string` | required | — |
| `options` | `table` | optional | — |

Returns: `request id, or nil`, `reason`

```lua
-- Give the new officer the uniform jacket
RegisterNetEvent("police:onDuty", function()
    Open77.clothing.equip(source, "Items.Jacket_01_basic_01")
end)
```

Registered in `LuaResourceRuntime.cs` (line 8279) as `LuaResourceRuntime.Bootstrap`.

## requestSnapshot

```lua
Open77.clothing.requestSnapshot(playerId)
```

`NETWORK`: Uses the network backend.

Asks a player's client for its current clothing state.

Asynchronous. The completion event's `result` carries rows with `slot`, `attachmentSlot`, `equipped` and an optional `record`/`tweakDbId`. This is a read of the client's live registry, so it is the honest answer to "what is actually on them" after a write has settled.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `request id, or nil`, `reason`

```lua
-- Ask the client what it really wears, and read the completion
Open77.clothing.requestSnapshot(playerId)
AddEventHandler("open77:clothing:completed", function(playerId, requestId, operation, accepted, reason, result)
    if operation == "snapshot" and accepted then print(json.encode(result)) end
end)
```

Registered in `LuaResourceRuntime.cs` (line 8292) as `LuaResourceRuntime.Bootstrap`.

## set

```lua
Open77.clothing.set(playerId, wardrobe, [options])
```

`NETWORK`: Uses the network backend.

Applies a whole slot-to-record wardrobe to a player in one request.

Asynchronous, same relay and completion contract. `wardrobe` maps slot to record, for example `{ outer_chest = "Items.Jacket_01_basic_01" }`; `false` empties a slot and an omitted slot is left unchanged. An active wardrobe outfit on the client can hide what this equips.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `wardrobe` | `table` | required | — |
| `options` | `table` | optional | — |

Returns: `request id, or nil`, `reason`

```lua
-- A whole uniform in one request; omitted slots stay as they are
Open77.clothing.set(source, {
    outer_chest = "Items.Jacket_01_basic_01",
    head = false, -- take the hat off
})
```

Registered in `LuaResourceRuntime.cs` (line 8287) as `LuaResourceRuntime.Bootstrap`.

## unequip

```lua
Open77.clothing.unequip(playerId, slot)
```

`NETWORK`: Uses the network backend.

Asks a player's client to empty one clothing slot.

Asynchronous, same relay, same completion event and ten-second timeout as `equip`. Slot names follow the relay's equipment registry.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `slot` | `string` | required | — |

Returns: `request id, or nil`, `reason`

```lua
-- Confiscate the helmet
Open77.clothing.unequip(playerId, "head")
```

Registered in `LuaResourceRuntime.cs` (line 8283) as `LuaResourceRuntime.Bootstrap`.
