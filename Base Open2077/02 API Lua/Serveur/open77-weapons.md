# Open77.weapons (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-weapons.md) · [Page web](https://open2077.net/docs/api/server/open77-weapons)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

19 functions.

## activate

```lua
Open77.weapons.activate(playerId, target, [options])
```

`NETWORK`: Uses the network backend.

Alias of `Open77.weapons.setActive`.

Asynchronous. The same function under a second name; it accepts both the slot and the record form. Secondary slots 4–6 require a compatible enabled dual-wield pair on Stable 152 or later; await completion between policy, assignments and activation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `target` | `integer (slot) | string (record)` | required | — |
| `options` | `table` | optional | — |

Returns: `request id, or nil`, `reason`

```lua
-- Draw the player's slot 1 when the round starts
Open77.weapons.activate(playerId, 1)
```

Registered in `LuaResourceRuntime.cs` (line 8324) as `LuaResourceRuntime.Bootstrap`.

## assign

```lua
Open77.weapons.assign(playerId, record, slot, [options])
```

`NETWORK`: Uses the network backend.

Assigns a weapon record to one of a player's six slots.

Asynchronous. The calling resource declares `network.events` and depends on `open77_weapons >=0.1.0`; the request is relayed to the official `open77_weapons` client package. `slot` is an integer 1 to 6 and `record` is at most 160 characters. Listen for `open77:weapons:completed` with `playerId, requestId, operation, accepted, reason, result`; replies are matched to the authenticated target and the calling resource, and an unanswered request completes with `request_timeout` after ten seconds. See [Weapon Lua API](https://open2077.net/docs/api/server/weapons-api.md). Secondary slots 4–6 require a compatible enabled dual-wield pair on Stable 152 or later; await completion between policy, assignments and activation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `record` | `string` | required | — |
| `slot` | `integer (1..6)` | required | — |
| `options` | `table` | optional | — |

Returns: `request id, or nil`, `reason: invalid_weapon_template, invalid_weapon_slot, invalid_weapon_options`

```lua
-- Give the new officer a pistol in slot 1 and draw it
RegisterNetEvent("police:onDuty", function()
    Open77.weapons.assign(source, "Items.Preset_Lexington_Default", 1, { active = true })
end)
```

Registered in `LuaResourceRuntime.cs` (line 8295) as `LuaResourceRuntime.Bootstrap`.

## clear

```lua
Open77.weapons.clear(playerId)
```

`NETWORK`: Uses the network backend.

Empties all three of a player's weapon slots in one request.

Asynchronous. Removes all six weapon slots through one owner relay request, secondary slots before primary slots to avoid native promotion leaving a weapon equipped. Already-empty slots are successful. The completion result carries cleared and a slots array of {slot, accepted, reason}; inspect each result for a partial failure. Requires network.events and the open77_weapons resource bundled with Stable 152 or later. See [Dual wielding](https://open2077.net/docs/api/server/dual-wielding.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `requestId, or nil`, `reason`

```lua
-- Arrested: every weapon slot emptied in one request
RegisterCommand("disarm", function(source, args)
    local target = tonumber(args[1])
    if target then Open77.weapons.clear(target) end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8361) as `LuaResourceRuntime.Bootstrap`.

## components

```lua
Open77.weapons.components(playerId, slot)
```

`NETWORK`: Uses the network backend.

The parts installed on the weapon in a player's loadout slot, from the cache, synchronously.

Requires `player.weapons.read`. Native-parity E3. The installed, non-base parts of that slot as the owner last reported them -- the same rows `Open77.weapons.get(playerId).slots[slot].parts` carries -- with `loadoutAgeMs` beside them, because they refresh with the loadout report and not with the 20 Hz snapshot. **A cache, not authority**: as true as the owner's last report. `nil, "weapons_unreported"` until the owner has described its slots at all. See [Weapon Lua API](https://open2077.net/docs/api/server/weapons-api.md#weapon-components-scopes-muzzles-and-mods).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `slot` | `integer 1..6` | required | — |

Returns: `{ slot, record, tweakDbId, equipped, parts, loadoutAgeMs, source }, or nil`, `reason`

```lua
-- A tournament rule: no scopes. Read the cache before letting the player in.
local fitted, reason = Open77.weapons.components(playerId, 1)
if fitted then
  for _, part in ipairs(fitted.parts) do
    if part.attachmentSlot == "AttachmentSlots.Scope" then print(playerId, "has a scope:", part.record) end
  end
else
  print("unknown:", reason)
end
```

Registered in `LuaResourceRuntime.Weapons6.cs` (line 121) as `LuaResourceRuntime.Bootstrap`.

## gadgets

```lua
Open77.weapons.gadgets(playerId)
```

`NETWORK`: Uses the network backend.

A player's quick slots from the cache, synchronously: the count after the last throw, with its own age.

Requires `player.weapons.read`. Native-parity E4. The quick slots as the owner last reported them -- at start, after every give or take, and the moment a throw is observed -- so `slots[n].quantity` is the count after the grenade left the hand and `active` names the record on the throw hotkey. `reportedAgeMs` is this report's own age: the gadget report runs on its own schedule and does not borrow the loadout's clock. **A cache, not authority.** `nil, "weapons_unreported"` when nothing at all has been heard from that player and `nil, "gadgets_unreported"` when the loadout has been but the quick slots have not. Pair it with `onGadgetConsumed(playerId, record, remaining[, tweakDbId])`, the host-wide, forgery-proof event the same report raises. See [Weapon Lua API](https://open2077.net/docs/api/server/weapons-api.md#gadgets-grenades-in-the-quick-slots).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `{ slots, active, reportedAgeMs }, or nil`, `reason`

```lua
-- Dispatch: log every throw with the count left, and refuse a resupply above three
AddEventHandler("onGadgetConsumed", function(playerId, record, remaining)
  print(playerId .. " threw " .. record .. ", " .. remaining .. " left")
end)
local kit = Open77.weapons.gadgets(playerId)
for _, row in ipairs(kit and kit.slots or {}) do
  if row.record == "Items.GrenadeFragRegular" and (row.quantity or 0) >= 3 then print("already stocked") end
end
```

Registered in `LuaResourceRuntime.Weapons6.cs` (line 189) as `LuaResourceRuntime.Bootstrap`.

## get

```lua
Open77.weapons.get(playerId)
```

`NETWORK`: Uses the network backend.

The server's cached view of a player's weapons, answered synchronously.

Requires `player.weapons.read`. **This is a cache, not authority.** Weapons live in the client's REDengine equipment system; the server has no copy of that state and no way to verify one, so every field is as true as the owner's last report -- read it to decide (anticheat, inventory sync, a shop refusing a sale), never to assert. The answer carries **two** ages because its halves refresh at different rates. `reportedAgeMs` dates `active`, `drawn` and `magazine`, which ride the ordinary 20 Hz player snapshot, and `fresh` applies the same two-second rule as `Open77.players.get`. `loadoutAgeMs` dates `slots`, which is pushed only when the loadout changes, so minutes there is normal and means nothing is wrong. `source` is `report` once the owner has described its slots and `snapshot` when only the packet's weapon block has arrived; the call fails with `weapons_unreported` when nothing at all has been heard, which is different from a player carrying nothing. The `onPlayerWeaponChanged` event fires once per change. See [Weapon Lua API](https://open2077.net/docs/api/server/weapons-api.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `{ playerId, slots, active, activeRecord, activeTweakDbId, drawn, magazine, reportedAgeMs, loadoutAgeMs, fresh, source }, or nil`, `reason`

```lua
-- The shop refuses a sale to a player who already carries a Lexington (cached view)
local weapons = Open77.weapons.get(source)
if weapons and weapons.activeRecord == "Items.Preset_Lexington_Default" then
    return Open77.chat.send(source, "You already have one.")
end
```

Registered in `LuaResourceRuntime.cs` (line 8368) as `LuaResourceRuntime.Bootstrap`.

## giveGadget

```lua
Open77.weapons.giveGadget(playerId, record, [count], [options])
```

`NETWORK`: Uses the network backend.

Grants a player a counted stack of grenades (or any quick-slot gadget), equipped and on the throw hotkey.

Uses the weapon relay. `record` must be a `Gadget_Record` -- every 2.31 grenade (`Items.GrenadeFragRegular`, `Items.GrenadeEMPRegular`, `Items.Preset_Grenade_Smoke_Default`, ...; the `category = grenade` rows of `docs/generated/weapons-2.31.csv`). `count` is 1..999 units. Unless `options.equip` is false the owner places the stack in quick slot 1 and on the RB hotkey, and accepts only when both took. The completion carries `result.gadget = { slot, record, tweakDbId, quantity, active }`; the owner's gadget report follows, so `Open77.weapons.gadgets(playerId)` then answers from the cache. Refusals by name: `invalid_player`, `invalid_gadget_template`, `invalid_gadget_count`, `invalid_weapon_options` now; `template_is_not_gadget`, `unsupported_gadget_area`, `item_creation_failed`, `equip_rejected`, `hotkey_rejected`, `request_timeout` on the completion. Like `assign`, this is a presentation grant: commit ownership in your own ledger first. See [Weapon Lua API](https://open2077.net/docs/api/server/weapons-api.md#gadgets-grenades-in-the-quick-slots).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `record` | `string a grenade / gadget record` | required | — |
| `count` | `integer 1..999, default 1` | optional | — |
| `options` | `table `{ equip = true }`` | optional | — |

Returns: `requestId, or nil`, `reason`

```lua
-- /frag: two frag grenades for the caller, equipped
RegisterCommand("frag", function(source)
  local requestId, reason = Open77.weapons.giveGadget(source, "Items.GrenadeFragRegular", 2)
  if not requestId then print("refused:", reason) end
end, false)
```

Registered in `LuaResourceRuntime.Weapons6.cs` (line 146) as `LuaResourceRuntime.Bootstrap`.

## holster

```lua
Open77.weapons.holster(playerId)
```

`NETWORK`: Uses the network backend.

Puts away whatever a player is holding.

Asynchronous, same relay and completion contract as `assign`. It holsters rather than removing: the slots keep their weapons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `request id, or nil`, `reason`

```lua
-- Entering the hospital holsters the weapon
AddEventHandler("hospital:entered", function(playerId)
    Open77.weapons.holster(playerId)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8334) as `LuaResourceRuntime.Bootstrap`.

## remove

```lua
Open77.weapons.remove(playerId, slot)
```

`NETWORK`: Uses the network backend.

Empties one of a player's three weapon slots.

Asynchronous, same relay and completion contract as `assign`. `slot` is an integer 1 to 6.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `slot` | `integer (1..6)` | required | — |

Returns: `request id, or nil`, `reason: invalid_weapon_slot`

```lua
-- Confiscate slot 2
Open77.weapons.remove(playerId, 2)
```

Registered in `LuaResourceRuntime.cs` (line 8327) as `LuaResourceRuntime.Bootstrap`.

## removeComponent

```lua
Open77.weapons.removeComponent(playerId, slot, attachmentSlot)
```

`NETWORK`: Uses the network backend.

Empties one attachment slot of the weapon in a player's loadout slot.

Native-parity E3. The relay asks the owner to queue the vanilla `RemoveItemPart` for the named `AttachmentSlots.*` record; the part returns to that player's inventory and the completion reports once the slot is visibly empty. An already-empty slot is accepted. The weapon's base parts are refused as `base_part_slot`. Refusals by name: `invalid_player`, `invalid_weapon_slot`, `invalid_attachment_slot` now; `weapon_slot_empty`, `slot_not_on_weapon`, `remove_rejected`, `request_timeout` on the completion. See [Weapon Lua API](https://open2077.net/docs/api/server/weapons-api.md#weapon-components-scopes-muzzles-and-mods).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `slot` | `integer 1..6` | required | — |
| `attachmentSlot` | `string an `AttachmentSlots.*` record` | required | — |

Returns: `requestId, or nil`, `reason`

```lua
-- Strip the scope off a player's slot-1 weapon before a no-scope round
local requestId, reason = Open77.weapons.removeComponent(playerId, 1, "AttachmentSlots.Scope")
print(requestId or reason)
```

Registered in `LuaResourceRuntime.Weapons6.cs` (line 95) as `LuaResourceRuntime.Bootstrap`.

## requestComponents

```lua
Open77.weapons.requestComponents(playerId, slot)
```

`NETWORK`: Uses the network backend.

Asks the owner for every attachment slot of the weapon in a loadout slot, taken or empty.

Native-parity E3, the round trip. The owner reads the live item and answers on `open77:weapons:completed` with `result.parts`, one row per attachment slot: `{ slot, attachmentSlot, attachmentSlotId, taken, base, record, tweakDbId }`, `base` marking the weapon's own receiver, barrel or magazine. Use `Open77.weapons.components` for the cached, synchronous answer. Refusals: `invalid_player`, `invalid_weapon_slot` now; `weapon_slot_empty`, `request_timeout` on the completion. See [Weapon Lua API](https://open2077.net/docs/api/server/weapons-api.md#weapon-components-scopes-muzzles-and-mods).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `slot` | `integer 1..6` | required | — |

Returns: `requestId, or nil`, `reason`

```lua
-- Audit a player's slot-1 mods from the live item rather than the cache
Open77.weapons.requestComponents(playerId, 1)
AddEventHandler("open77:weapons:completed", function(who, requestId, operation, accepted, reason, result)
  if operation ~= "components" or not accepted then return end
  for _, part in ipairs(result.parts) do
    if not part.base then print(who, part.attachmentSlot, part.taken and part.record or "empty") end
  end
end)
```

Registered in `LuaResourceRuntime.Weapons6.cs` (line 108) as `LuaResourceRuntime.Bootstrap`.

## requestGadgets

```lua
Open77.weapons.requestGadgets(playerId)
```

`NETWORK`: Uses the network backend.

Asks the owner for its quick slots: one row per slot with the gadget, its count and the hotkey flag.

Native-parity E4, the round trip. The owner answers on `open77:weapons:completed` with `result` as an array of `{ slot, record, tweakDbId, quantity, active, difference }` rows and refreshes the server cache in the same breath. Use `Open77.weapons.gadgets` for the cached, synchronous answer. Refusals: `invalid_player` now, `request_timeout` on the completion. See [Weapon Lua API](https://open2077.net/docs/api/server/weapons-api.md#gadgets-grenades-in-the-quick-slots).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `requestId, or nil`, `reason`

```lua
-- Count a player's grenades from the live inventory
Open77.weapons.requestGadgets(playerId)
AddEventHandler("open77:weapons:completed", function(who, requestId, operation, accepted, reason, result)
  if operation == "gadgets" and accepted then
    for _, row in ipairs(result) do print(who, row.slot, row.record, row.quantity, row.active) end
  end
end)
```

Registered in `LuaResourceRuntime.Weapons6.cs` (line 178) as `LuaResourceRuntime.Bootstrap`.

## requestSnapshot

```lua
Open77.weapons.requestSnapshot(playerId)
```

`NETWORK`: Uses the network backend.

Asks a player's client for its current weapon state.

Asynchronous. The completion event's `result` carries the slot contents and ammunition the client actually holds, which is the honest answer after a write has settled.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `request id, or nil`, `reason`

```lua
-- Ask the client what it really holds, and read the answer from the completion event
Open77.weapons.requestSnapshot(playerId)
AddEventHandler("open77:weapons:completed", function(playerId, requestId, operation, accepted, reason, result)
    if operation == "snapshot" and accepted then print(json.encode(result)) end
end)
```

Registered in `LuaResourceRuntime.cs` (line 8351) as `LuaResourceRuntime.Bootstrap`.

## setActive

```lua
Open77.weapons.setActive(playerId, target, [options])
```

`NETWORK`: Uses the network backend.

Draws a weapon, by slot or by record.

Asynchronous, same relay and completion contract as `assign`. Two forms: a slot number 1 to 6, which takes no options, or a record string with optional options. Passing options with a slot is refused, because the two forms mean different things. Secondary slots 4–6 require a compatible enabled dual-wield pair on Stable 152 or later; await completion between policy, assignments and activation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `target` | `integer (slot) | string (record)` | required | — |
| `options` | `table (record form only)` | optional | — |

Returns: `request id, or nil`, `reason: invalid_weapon_target, invalid_weapon_slot, invalid_weapon_template, invalid_weapon_options`

```lua
-- Force the katana out for the melee round
Open77.weapons.setActive(playerId, "Items.Preset_Katana_Default", { slot = 3 })
```

Registered in `LuaResourceRuntime.cs` (line 8307) as `LuaResourceRuntime.Bootstrap`.

## setAmmo

```lua
Open77.weapons.setAmmo(playerId, slot, amounts)
```

`NETWORK`: Uses the network backend.

Sets a slot's magazine and reserve ammunition.

Asynchronous, same relay and completion contract as `assign`. `amounts` takes `reserve` and `magazine` -- at least one is required, each a non-negative integer up to 1,000,000 -- plus an optional boolean `activate`. When both hands share an ammo record, reserve is shared and the total includes both loaded magazines. Editing one magazine preserves the other; await completion before a dependent change. Native dual reloads with too little shared reserve can still allocate rounds inconsistently. On Stable 156 or later, inactive/holstered slots, including never-drawn assigned weapons, can be read and updated without drawing or changing selection. `activate = true` explicitly selects/draws. For an inactive slot, reserve is inventory total minus that slot's magazine and currently drawn other-hand magazines; other inactive magazines are saved counts, not separate pools. Native jobs are ordered and asynchronous; await completion. Slot changes, invalid capacity, disconnect or body replacement invalidate the job. See [weapon ammunition](../../01%20Documentation/weapons-api.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `slot` | `integer (1..6)` | required | — |
| `amounts` | `table { reserve?, magazine?, activate? }` | required | — |

Returns: `request id, or nil`, `reason: invalid_weapon_slot, invalid_weapon_ammo`

```lua
-- An ammo box refills slot 1
RegisterNetEvent("items:ammoBox", function()
    Open77.weapons.setAmmo(source, 1, { magazine = 12, reserve = 120 })
end)
```

Registered in `LuaResourceRuntime.cs` (line 8335) as `LuaResourceRuntime.Bootstrap`.

## setComponent

```lua
Open77.weapons.setComponent(playerId, slot, record, [options])
```

`NETWORK`: Uses the network backend.

Installs a part (scope, muzzle, generic mod) on the weapon in a player's loadout slot.

Native-parity E3, over the same relay as every weapon mutation: a request id now, the verified outcome later on `open77:weapons:completed` with `result.part = { attachmentSlot, attachmentSlotId, record, tweakDbId }`. The part is an `Items.*` record tagged `itemPart` (`Items.w_att_scope_short_01`, `Items.w_silencer_01`, `Items.w_muzzle_brake_01`; 928 on 2.31, `docs/generated/weapon-parts-2.31.csv`) and the optional `options.attachmentSlot` an `AttachmentSlots.*` record (`Scope`, `PowerModule` for muzzles, `Power_Handgun_WeaponMod1`/`2` for generic mods). The owning client installs it through the vanilla `ItemModificationSystem`, so it is a real part on a real weapon; a missing part is created, a part already in the slot is swapped out and deleted as the vanilla screen does. Without a slot the client picks the first of the part's placement slots the weapon declares, empty ones first. The owner's next loadout report carries the part, so `Open77.weapons.components(playerId, slot)` then answers without a round trip. Refusals by name: `invalid_player`, `invalid_weapon_slot`, `invalid_part_template`, `invalid_attachment_slot` now; `weapon_slot_empty`, `slot_not_on_weapon`, `part_does_not_fit_weapon`, `part_does_not_fit_slot`, `base_part_slot`, `install_rejected`, `request_timeout` on the completion. **Other players do not see the part**: the proxy weapon is rebuilt from the snapshot's `ItemID`, which does not encode parts. See [Weapon Lua API](https://open2077.net/docs/api/server/weapons-api.md#weapon-components-scopes-muzzles-and-mods).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `slot` | `integer 1..6` | required | — |
| `record` | `string an `Items.*` part record` | required | — |
| `options` | `table `{ attachmentSlot = "AttachmentSlots.PowerModule" }`` | optional | — |

Returns: `requestId, or nil`, `reason`

```lua
-- /silence: the armoury fits a silencer to whatever the caller holds in slot 1
RegisterCommand("silence", function(source)
  local requestId, reason = Open77.weapons.setComponent(source, 1, "Items.w_silencer_01",
    { attachmentSlot = "AttachmentSlots.PowerModule" })
  if not requestId then print("refused:", reason) end
end, false)
AddEventHandler("open77:weapons:completed", function(playerId, requestId, operation, accepted, reason, result)
  if operation == "setComponent" then
    print(playerId, accepted and ("fitted " .. tostring(result.part and result.part.record)) or reason)
  end
end)
```

Registered in `LuaResourceRuntime.Weapons6.cs` (line 75) as `LuaResourceRuntime.Bootstrap`.

## setDualWieldAllowed

```lua
Open77.weapons.setDualWieldAllowed(playerId, enabled)
```

`NETWORK`: Uses the network backend.

Enable or disable native dual wield for a player incarnation.

Queues native dual-wield permission for the current player incarnation. true enables the authenticated-session perk gate; false clears secondary slots 4–6 without deleting inventory items. Reapply after reconnect/body replacement. Wait for open77:weapons:completed before assigning and activating a compatible pair. Requires matching Stable 152 client/server builds and the updated bundled open77_weapons resource. See [Dual wielding](https://open2077.net/docs/api/server/dual-wielding.md). Server resources need network.events and dependency open77_weapons >=0.1.0. Completion is (playerId, requestId, operation, accepted, reason, result), with boolean accepted.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `request id, or nil`, `reason`

```lua
local requestId, reason = Open77.weapons.setDualWieldAllowed(playerId, true)
if not requestId then print(reason); return end
-- Await open77:weapons:completed before assigning the two weapons.
```

[Dual wielding guide](../../01%20Documentation/dual-wielding.md)

Registered in `LuaResourceRuntime.cs` (line 8443) as `LuaResourceRuntime.Bootstrap`.

## takeGadget

```lua
Open77.weapons.takeGadget(playerId, [record], [count])
```

`NETWORK`: Uses the network backend.

Takes grenades back from a player: some units of a record, or every unit of the active gadget.

Native-parity E4, over the weapon relay. Without a record the owner takes the gadget on its throw hotkey; without a count, every unit. A quick slot left empty is unequipped. The completion carries `result.gadget` with the remaining `quantity`. Refusals by name: `invalid_player`, `invalid_gadget_template`, `invalid_gadget_count` now; `gadget_not_owned`, `no_active_gadget`, `inventory_update_rejected`, `request_timeout` on the completion. See [Weapon Lua API](https://open2077.net/docs/api/server/weapons-api.md#gadgets-grenades-in-the-quick-slots).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `record` | `string a grenade / gadget record; nil for the active gadget` | optional | — |
| `count` | `integer 1..999, or -1 / nil for every unit` | optional | — |

Returns: `requestId, or nil`, `reason`

```lua
-- Arrested: whatever is on the throw key is confiscated
local requestId, reason = Open77.weapons.takeGadget(playerId)
print(requestId or reason)
```

Registered in `LuaResourceRuntime.Weapons6.cs` (line 166) as `LuaResourceRuntime.Bootstrap`.

## unequip

```lua
Open77.weapons.unequip(playerId, slot)
```

`NETWORK`: Uses the network backend.

Alias of `Open77.weapons.remove`.

Asynchronous. The same function under the name the clothing API uses for the same idea.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `slot` | `integer (1..6)` | required | — |

Returns: `request id, or nil`, `reason`

```lua
-- Same as remove: take the weapon out of slot 1
Open77.weapons.unequip(playerId, 1)
```

Registered in `LuaResourceRuntime.cs` (line 8333) as `LuaResourceRuntime.Bootstrap`.
