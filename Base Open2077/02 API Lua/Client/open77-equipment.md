# Open77.equipment (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-equipment.md) · [Page web](https://open2077.net/docs/api/client/open77-equipment)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

11 functions.

## apply

```lua
Open77.equipment.apply(slots, [options])
```

`GAME`: Requires a live game instance.

Applies a table of local equipment slot values.

Client-only; requires player.equipment.edit. Returns true on accepted native mutation, or nil, reason; acceptance is not rendered completion. Values are record strings or false to clear. Options: allowRestricted=false and replace=false. With replace=false omitted slots are preserved; replace=true also clears omitted slots. Worn-set mutations run sequentially and may partially apply before an error; this is not an atomic transaction.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `slots` | `table` | required | — |
| `options` | `table` | optional | — |

Returns: `true, or nil, reason`

```lua
-- Put on a whole outfit at once, clearing anything the outfit does not name
Open77.equipment.apply({
    Head = "Items.Cap_01_basic_01",
    OuterChest = false,        -- take the jacket off
}, { replace = false })
```

Registered in `ResourceHost.cpp` (line 24886) as `LuaEquipmentApply`.

## attached

```lua
Open77.equipment.attached(slot)
```

`GAME`: Requires a live game instance.

Reads the item actually attached at a local slot.

Client-only; requires player.equipment.read. Failures return nil, reason. Returns a record string or false when no item is attached. Attachment alone does not prove the expected garment appearance has settled.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `slot` | `string` | required | — |

Returns: `record or false, or nil, reason`

```lua
-- Is the helmet really on the body yet?
local worn = Open77.equipment.attached("Head")
print(worn or "bare head")
```

Registered in `ResourceHost.cpp` (line 24881) as `LuaEquipmentAttached`.

## equip

```lua
Open77.equipment.equip(record, [slot], [options])
```

`GAME`: Requires a live game instance.

Equips a record in the local worn set.

Client-only; requires player.equipment.edit. Returns true on accepted native mutation, or nil, reason; acceptance is not rendered completion. Call equip(record[, slot][, options]); omitted slot is inferred by the native record adapter. options.allowRestricted defaults false. Read registry/attached/visualSettled separately when completion matters.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `record` | `string` | required | — |
| `slot` | `string` | optional | — |
| `options` | `table` | optional | — |

Returns: `true, or nil, reason`

```lua
-- The wardrobe menu: put on the selected cap
RegisterNetEvent("wardrobe:pick", function(record)
    local ok, reason = Open77.equipment.equip(record)
    if not ok then Open77.hud.notify("Cannot wear that: " .. reason) end
end)
```

Registered in `ResourceHost.cpp` (line 24884) as `LuaEquipmentEquip`.

## family

```lua
Open77.equipment.family()
```

`GAME`: Requires a live game instance.

Reads the local equipment body family.

Client-only; requires player.equipment.read. Failures return nil, reason. Returns the native adapter body-family name, used to filter compatible records.

Returns: `family, or nil, reason`

```lua
-- Only list records this body can wear
local family = Open77.equipment.family()
local records = Open77.equipment.records({ slot = "Head", family = family })
```

Registered in `ResourceHost.cpp` (line 24879) as `LuaEquipmentFamily`.

## info

```lua
Open77.equipment.info(record)
```

`GAME`: Requires a live game instance.

Looks up metadata for one equipment record.

Client-only; requires player.equipment.read. Failures return nil, reason. The record name is 1..160 bytes. Result fields: record, tweakDbId, slot, attachmentSlot, catalogScope, supportsMale, supportsFemale, defaultSelectable and nonvisual. This lookup does not equip the item.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `record` | `string` | required | — |

Returns: `info, or nil, reason`

```lua
-- Show which slot an item goes into before offering it
local info = Open77.equipment.info("Items.Cap_01_basic_01")
if info then print(info.slot, info.supportsMale, info.supportsFemale) end
```

Registered in `ResourceHost.cpp` (line 24878) as `LuaEquipmentInfo`.

## records

```lua
Open77.equipment.records([options])
```

`GAME`: Requires a live game instance.

Lists equipment records with optional slot and body-family filters.

Client-only; requires player.equipment.read. Failures return nil, reason. Options: slot, family="male" or "female", restricted=false and limit=200 (positive integer, capped at 2000). restricted=true includes records not default-selectable; it does not grant mutation permission. Results use the same metadata shape as info().

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table` | optional | — |

Returns: `records, or nil, reason`

```lua
-- Fill the shop's hat list
for _, item in ipairs(Open77.equipment.records({ slot = "Head", limit = 50 }) or {}) do
    print(item.record)
end
```

Registered in `ResourceHost.cpp` (line 24877) as `LuaEquipmentRecords`.

## registry

```lua
Open77.equipment.registry()
```

`GAME`: Requires a live game instance.

Reads the local worn equipment slot registry.

Client-only; requires player.equipment.read. Failures return nil, reason. Returns a table keyed by slot: record strings or false for empty slots. Registry state is separate from the attached item and its settled visual appearance.

Returns: `slots, or nil, reason`

```lua
-- Save the current outfit so it can be restored after a respawn
local outfit = Open77.equipment.registry()
if outfit then SetResourceKvp("outfit", json.encode(outfit)) end
```

Registered in `ResourceHost.cpp` (line 24880) as `LuaEquipmentRegistry`.

## slots

```lua
Open77.equipment.slots()
```

`GAME`: Requires a live game instance.

Lists the supported clothing slot names and attachment slots.

Client-only; requires player.equipment.read. Failures return nil, reason. Returns nine {name, attachmentSlot} entries: Head, Face, InnerChest, OuterChest, Legs, Feet, Outfit, UnderwearTop and UnderwearBottom.

Returns: `slots, or nil, reason`

```lua
-- Build the wardrobe tabs from the slot list
for _, slot in ipairs(Open77.equipment.slots() or {}) do
    print(slot.name, slot.attachmentSlot)
end
```

Registered in `ResourceHost.cpp` (line 24876) as `LuaEquipmentSlots`.

## tags

```lua
Open77.equipment.tags(slot)
```

`GAME`: Requires a live game instance.

Reads native tags associated with a local equipment slot.

Client-only; requires player.equipment.read. Failures return nil, reason. Returns an array of tag strings for the current wearer and slot.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `slot` | `string` | required | — |

Returns: `tags, or nil, reason`

```lua
-- Does the worn jacket carry a faction tag?
for _, tag in ipairs(Open77.equipment.tags("OuterChest") or {}) do
    if tag == "Maelstrom" then print("wearing gang colours") end
end
```

Registered in `ResourceHost.cpp` (line 24883) as `LuaEquipmentTags`.

## unequip

```lua
Open77.equipment.unequip(slot)
```

`GAME`: Requires a live game instance.

Removes equipment from a local worn slot.

Client-only; requires player.equipment.edit. Returns true on accepted native mutation, or nil, reason; acceptance is not rendered completion. Does not delete the inventory item. Read native visual readiness separately.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `slot` | `string` | required | — |

Returns: `true, or nil, reason`

```lua
-- Take the hat off when entering the church
AddEventHandler("church:entered", function()
    Open77.equipment.unequip("Head")
end)
```

Registered in `ResourceHost.cpp` (line 24885) as `LuaEquipmentUnequip`.

## visualSettled

```lua
Open77.equipment.visualSettled(slot, expected)
```

`GAME`: Requires a live game instance.

Checks native visual readiness for an expected clothing slot value.

Client-only; requires player.equipment.read. Failures return nil, reason. expected must be a record string or false. Reads registry/outfit visibility and actual attachment/appearance readiness; false means not yet settled, whereas nil, reason indicates an error. This native readback is not screenshot proof.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `slot` | `string` | required | — |
| `expected` | `string|boolean` | required | — |

Returns: `settled, or nil, reason`

```lua
-- Wait until the new jacket is visibly on before taking the screenshot
CreateThread(function()
    Open77.equipment.equip("Items.Jacket_01_basic_01", "OuterChest")
    while not Open77.equipment.visualSettled("OuterChest", "Items.Jacket_01_basic_01") do Wait(100) end
    takePhoto()
end)
```

Registered in `ResourceHost.cpp` (line 24882) as `LuaEquipmentVisualSettled`.
