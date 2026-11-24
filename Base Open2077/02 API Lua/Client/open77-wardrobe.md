# Open77.wardrobe (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-wardrobe.md) · [Page web](https://open2077.net/docs/api/client/open77-wardrobe)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

3 functions.

## activate

```lua
Open77.wardrobe.activate([index])
```

`GAME`: Requires a live game instance.

Activates a native outfit or returns to the worn set.

Client-only; requires player.equipment.edit. Returns true on accepted native mutation, or nil, reason; acceptance is not rendered completion. Pass an outfit index; false, nil or omitted disables the active outfit. Activation does not itself prove all garment visuals have settled.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `index` | `integer|boolean` | optional | — |

Returns: `true, or nil, reason`

```lua
-- Switch to outfit slot 2 from the wardrobe menu
Open77.wardrobe.activate(2)
```

Registered in `ResourceHost.cpp` (line 24920) as `LuaWardrobeActivate`.

## active

```lua
Open77.wardrobe.active()
```

`GAME`: Requires a live game instance.

Reads the active native outfit index.

Client-only; requires player.equipment.read. Failures return nil, reason. Returns the outfit index or false when no outfit is active; false is not an error.

Returns: `index or false, or nil, reason`

```lua
-- Highlight the active outfit in the wardrobe menu
local index = Open77.wardrobe.active()
if index then print("wearing outfit", index) else print("no outfit active") end
```

Registered in `ResourceHost.cpp` (line 24919) as `LuaWardrobeActive`.

## outfit

```lua
Open77.wardrobe.outfit(index)
```

`GAME`: Requires a live game instance.

Returns slot operations bound to a native outfit index.

Client-only; requires player.equipment.read. Failures return nil, reason. Index must be an integer from 0 through 6. The result contains index and bound registry(), equip(record[, slot][, options]), unequip(slot), hide(slot) and apply(slots[, options]) functions; call them with dot syntax. registry requires read permission; mutations separately require player.equipment.edit. hide marks an outfit slot hidden, while unequip removes its override. apply false marks a hidden slot, and replace controls preservation of omitted overrides. Editing does not automatically activate the outfit.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `index` | `integer` | required | — |

Returns: `outfit, or nil, reason`

```lua
-- Edit outfit 1 without activating it
local outfit = Open77.wardrobe.outfit(1)
if outfit then
    outfit.equip("Items.Cap_01_basic_01", "Head")
    outfit.hide("Face")
end
```

Registered in `ResourceHost.cpp` (line 24921) as `LuaWardrobeOutfit`.
