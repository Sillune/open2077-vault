# Open77.weapons (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-weapons.md) · [Page web](https://open2077.net/docs/api/client/open77-weapons)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

22 functions.

## activate

```lua
Open77.weapons.activate(target, [options])
```

`GAME`: Requires a live game instance.

Alias of `Open77.weapons.setActive`.

Requires `player.weapons.edit`; see `setActive` for template placement and asynchronous completion semantics. Secondary slots 4–6 require a compatible enabled dual-wield pair on Stable 152 or later; await completion between policy, assignments and activation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `integer 1..6|string` | required | — |
| `options` | `table` | optional | — |

Returns: `request id, or nil`, `reason`

```lua
-- Draw slot 1 when the round starts
RegisterNetEvent("arena:roundStarted", function()
    Open77.weapons.activate(1)
end)
```

Registered in `ResourceHost.cpp` (line 24945) as `LuaWeaponSetActive`.

## all

```lua
Open77.weapons.all()
```

`GAME`: Requires a live game instance.

Alias of `Open77.weapons.snapshot`.

Requires `player.weapons.read`; results arrive asynchronously as slot-state events followed by completion. Client state rows append reloading after capacity. The booleans are strings; collect rows by request ID and publish only after successful completion. See [Dual wielding](https://open2077.net/docs/api/client/dual-wielding.md#ammunition-and-hud). Stable 156 reads reserve, magazine and capacity for inactive/holstered ranged slots, including never-drawn weapons. Completion follows all native reads; unreadable ranged caches fail with `ammo_cache_unavailable`. Empty/non-ammo slots retain unavailable fields (-1).

Returns: `request id, or nil`, `reason`

```lua
-- Same as snapshot: ask for every slot's verified state
Open77.weapons.all()
AddEventHandler("open77:weapons:state", function(requestId, slot, record)
    print("slot", slot, record ~= "" and record or "empty")
end)
```

Registered in `ResourceHost.cpp` (line 24955) as `LuaWeaponSnapshot`.

## applyBlast

```lua
Open77.weapons.applyBlast(blast)
```

`GAME`: Requires a live game instance.

Applies a server-relayed weapon blast to the vehicles this client simulates.

Client-only; requires `player.weapons.edit`. Only the client that simulates a vehicle can move it: an impulse on another client's observer replica is overwritten by the owner's motion stream. A server resource that accepted a blast therefore finds the vehicles it reaches (`Open77.vehicles.nearby`) and their physics owner (`Open77.vehicles.owner`), and sends each owner the centre, its own copy of the approved profile and that owner's vehicle ids; the owner's client calls this. Accepts {x, y, z, radius (0-60, above zero), push (0-40), lift (0-40), falloff (0-4, default 1), vehicles (0-64 positive server vehicle ids), characters (optional boolean)}; at least one vehicle or characters = true is required, anything else returns nil, invalid_weapon_blast. With characters, every living, unseated network NPC this client presents inside the radius is also knocked down once with the native hit-reaction knockdown (the one remote player proxies play for a car impact), which throws the body a few metres and lets it get up; a copy whose native health has not been projected yet is skipped. The server sends the blast to every nearby player, so the authority copy and each observer copy fall together, and an observer copy that settles more than 1.5 m from its authority walks back to it once the reaction is over. A forced ragdoll with an impulse was measured to do nothing on a living network NPC and is not used. The blast is queued and applied on the game thread to each listed vehicle that this client simulates in its current authority epoch, from this client's own pose, with the same mass-scaled impulse and (1 - distance / radius)^falloff weight as a local blast. A listed vehicle whose authority grant is still in flight is retried for 0.75 s; one outside the radius on this client's pose is skipped. Never moves a vehicle another client simulates, a frozen vehicle or a character. Returns true when queued, or nil/reason (invalid_weapon_blast, weapon_blast_queue_full with 16 blasts pending). tuning() reports relayedBlasts, relayedImpulses, relayedExpired, relayedRefused and relayedCharacters, including when this resource owns no tuning profile.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `blast` | `table` | required | — |

Returns: `true or nil`, `reason on failure`

```lua
RegisterNetEvent("myblast:owned", function(value)
    local ok, reason = Open77.weapons.applyBlast({x = value.x, y = value.y, z = value.z,
        radius = value.radius, push = value.push, lift = value.lift, vehicles = value.vehicles})
    if not ok then print("relayed blast refused: " .. tostring(reason)) end
end)
```

[Blast relay guide](../../01%20Documentation/weapons-api.md#explosions-on-every-screen-the-blast-relay)

Registered in `ResourceHost.cpp` (line 24953) as `LuaWeaponApplyBlast`.

## assign

```lua
Open77.weapons.assign(record, slot, [options])
```

`GAME`: Requires a live game instance.

Assigns an exact TweakDB weapon template to a standard loadout slot.

Requires `player.weapons.edit`. `slot` is 1..6. Options are `active` (default false) and `addToInventory` (default true). The return value only confirms that the validated request was queued; listen for `open77:weapons:completed` for the state verified by REDscript. Secondary slots 4–6 require a compatible enabled dual-wield pair on Stable 152 or later; await completion between policy, assignments and activation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `record` | `string` | required | — |
| `slot` | `integer 1..6` | required | — |
| `options` | `table` | optional | — |

Returns: `request id, or nil`, `reason`

```lua
-- A pistol in slot 1, drawn immediately
local request, reason = Open77.weapons.assign("Items.Preset_Lexington_Default", 1, { active = true })
if not request then print("assign refused: " .. reason) end
```

Registered in `ResourceHost.cpp` (line 24943) as `LuaWeaponAssign`.

## clearTuning

```lua
Open77.weapons.clearTuning()
```

`GAME`: Requires a live game instance.

Disables this resource's weapon tuning and restores its modifiers.

Requires a compatible development client exposing native weapon tuning with inventory-instance restoration. Client-only; requires `player.weapons.edit`. Disables the calling resource's profile and removes only the modifier handles it owns. Returns true when accepted, or nil/reason on failure; it is a successful no-op when this resource owns no profile and never clears another resource's tuning. If a holstered weapon's StatsObject is temporarily absent, exact handles remain queued against its inventory identity, even after the engine entity disappears. Drawing that same inventory item allows cleanup on its current entity; a different copy of the same record is not the same item. Read tuning().status and pendingModifiers to distinguish idle from restoring and wait for zero pending modifiers before reporting completion. Does not remove granted weapons, ammunition or components. Resource stop and failure also release the profile and retain deferred cleanup. The queue is bounded to 32 pending bindings; saturation refuses further changes with restore_queue_full.

Returns: `true or nil`, `reason on failure`

```lua
-- Clear while this weapon is drawn, before switching weapons.
local ok, reason = Open77.weapons.clearTuning()
if not ok then print(reason) end
local state = Open77.weapons.tuning()
if state then print(state.status, state.pendingModifiers) end
```

[Weapon customization & workshop](../../01%20Documentation/weapon-customization.md)

Registered in `ResourceHost.cpp` (line 24951) as `LuaWeaponClearTuning`.

## components

```lua
Open77.weapons.components(slot)
```

`GAME`: Requires a live game instance.

Lists every attachment slot of the weapon in a loadout slot, taken or empty.

Requires `player.weapons.read`. Native-parity E3. Read off the live item -- `GetUsedSlotsOnItem` / `GetEmptySlotsOnItem` -- so the answer is the blueprint of the weapon actually there, not a table. One `open77:weapons:part` row per slot arrives before the completion: `(requestId, slot, attachmentSlot, attachmentSlotId, taken, base, record, tweakDbId)`, where `base == "true"` marks the weapon's own receiver, barrel or magazine and `record` names the installed part when `taken`. `attachmentSlot` is spelled from the 2.31 weapon-part vocabulary when the shipping build's reverse name table cannot; `attachmentSlotId` always travels. Refused by name: `invalid_weapon_slot` now, `weapon_slot_empty` from the bridge. See [Weapon Lua API](https://open2077.net/docs/api/client/weapons-api.md#weapon-components-scopes-muzzles-and-mods).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `slot` | `integer 1..6` | required | — |

Returns: `requestId, or nil`, `reason`

```lua
-- Which mod slots does the rifle in slot 1 have, and what is in them?
local requestId = Open77.weapons.components(1)
AddEventHandler("open77:weapons:part", function(id, slot, attachmentSlot, _, taken, base, record)
  if tostring(id) == tostring(requestId) and base ~= "true" then
    print(attachmentSlot, taken == "true" and record or "empty")
  end
end)
```

Registered in `ResourceHost.cpp` (line 24960) as `LuaWeaponComponents`.

## gadgets

```lua
Open77.weapons.gadgets()
```

`GAME`: Requires a live game instance.

Lists the quick slots: which gadget is in each, how many, and which one is on the throw hotkey.

Requires `player.weapons.read`. Native-parity E4. One `open77:weapons:gadget` row per quick slot arrives before the completion: `(requestId, quickSlot, record, tweakDbId, quantity, active, difference)`; an empty slot has an empty record and quantity `-1`. The same event also carries the consume half without being asked: a row with `requestId == "0"` is the bridge's inventory listener reporting that the engine changed a gadget stack -- `difference` negative for a throw, `quantity` the count left. The bundled `open77_weapons` resource turns those into `open77:weapons:gadgetConsumed(record, tweakDbId, remaining)` locally and into the server's `onGadgetConsumed`. See [Weapon Lua API](https://open2077.net/docs/api/client/weapons-api.md#gadgets-grenades-in-the-quick-slots).

Returns: `requestId, or nil`, `reason`

```lua
-- Print the quick slots, and watch grenades being thrown
Open77.weapons.gadgets()
AddEventHandler("open77:weapons:gadget", function(id, slot, record, _, quantity, active, difference)
  if tostring(id) == "0" then print("stack changed", record, difference, "left", quantity)
  else print("quick slot", slot, record, quantity, active) end
end)
```

Registered in `ResourceHost.cpp` (line 24963) as `LuaWeaponGadgets`.

## giveGadget

```lua
Open77.weapons.giveGadget(record, [count], [options])
```

`GAME`: Requires a live game instance.

Grants a counted stack of grenades (or any quick-slot gadget) and puts it on the throw hotkey.

Requires `player.weapons.edit`. The record must be a `Gadget_Record` -- every 2.31 grenade is one (`Items.GrenadeFragRegular`, `Items.GrenadeEMPRegular`, `Items.Preset_Grenade_Smoke_Default`, ...; the `category = grenade` rows of `docs/generated/weapons-2.31.csv`); a weapon answers `template_is_not_gadget`. `count` is 1..999 units added to the stack. Unless `options.equip` is false, the stack is placed in quick slot 1 (`GameplayEquipRequest`) and assigned to `EHotkey.RB` (`HotkeyAssignmentRequest`) and the completion is accepted only when both took. The completion carries `result.gadget = { slot, record, tweakDbId, quantity, active }` after one `open77:weapons:gadget` row. Refused by name: `invalid_gadget_template`, `template_is_not_gadget`, `invalid_gadget_count` now; `unsupported_gadget_area`, `item_creation_failed`, `equip_rejected`, `hotkey_rejected` from the bridge. See [Weapon Lua API](https://open2077.net/docs/api/client/weapons-api.md#gadgets-grenades-in-the-quick-slots).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `record` | `string a grenade / gadget record` | required | — |
| `count` | `integer 1..999, default 1` | optional | — |
| `options` | `table `{ equip = true }`` | optional | — |

Returns: `requestId, or nil`, `reason`

```lua
-- Three frag grenades, equipped and on the throw key
local requestId, reason = Open77.weapons.giveGadget("Items.GrenadeFragRegular", 3)
print(requestId or reason)
```

Registered in `ResourceHost.cpp` (line 24961) as `LuaWeaponGiveGadget`.

## holster

```lua
Open77.weapons.holster()
```

`GAME`: Requires a live game instance.

Holsters the currently drawn right-hand weapon.

Requires `player.weapons.edit`. Loadout assignments and the selected equipment-area item are unchanged; completion reports whether the right-hand slot became empty.

Returns: `request id, or nil`, `reason`

```lua
-- Entering the shop holsters the weapon
AddEventHandler("shop:entered", function()
    Open77.weapons.holster()
end)
```

Registered in `ResourceHost.cpp` (line 24948) as `LuaWeaponHolster`.

## remove

```lua
Open77.weapons.remove(slot)
```

`GAME`: Requires a live game instance.

Removes the weapon assigned to one standard slot.

Requires `player.weapons.edit`. This unequips the item but does not delete it from inventory. The empty slot is verified asynchronously.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `slot` | `integer 1..6` | required | — |

Returns: `request id, or nil`, `reason`

```lua
-- The round is over: empty slot 1
Open77.weapons.remove(1)
```

Registered in `ResourceHost.cpp` (line 24946) as `LuaWeaponRemove`.

## removeComponent

```lua
Open77.weapons.removeComponent(slot, attachmentSlot)
```

`GAME`: Requires a live game instance.

Empties one attachment slot of the weapon in a loadout slot; the part goes back to the inventory.

Requires `player.weapons.edit`. Native-parity E3. Queues the vanilla `RemoveItemPart` request for the named `AttachmentSlots.*` record and reports once the slot is visibly empty. A slot that is already empty is accepted rather than refused. The weapon's own base parts -- receiver, barrel, magazine, the rows `open77:weapons:part` flags with `base == "true"` -- are refused as `base_part_slot`: removing one breaks the weapon. Immediate refusals: `invalid_weapon_slot`, `invalid_attachment_slot`; from the bridge: `weapon_slot_empty`, `slot_not_on_weapon`, `remove_rejected`. See [Weapon Lua API](https://open2077.net/docs/api/client/weapons-api.md#weapon-components-scopes-muzzles-and-mods).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `slot` | `integer 1..6` | required | — |
| `attachmentSlot` | `string an `AttachmentSlots.*` record` | required | — |

Returns: `requestId, or nil`, `reason`

```lua
-- Take the silencer off the pistol in slot 2
local requestId, reason = Open77.weapons.removeComponent(2, "AttachmentSlots.PowerModule")
print(requestId or reason)
```

Registered in `ResourceHost.cpp` (line 24959) as `LuaWeaponRemoveComponent`.

## setActive

```lua
Open77.weapons.setActive(target, [options])
```

`GAME`: Requires a live game instance.

Selects and draws a standard weapon by slot or TweakDB template.

Requires `player.weapons.edit`. A numeric target must already contain a weapon. A template target reuses an assigned instance or equips it first; options accept fallback `slot` 1..6 and `addToInventory` (default true). Completion is asynchronous through `open77:weapons:completed`. Secondary slots 4–6 require a compatible enabled dual-wield pair on Stable 152 or later; await completion between policy, assignments and activation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `integer 1..6|string` | required | — |
| `options` | `table` | optional | — |

Returns: `request id, or nil`, `reason`

```lua
-- The 2 key draws whatever is in slot 2
RegisterKeyMapping("weapon_slot_2", "Weapon slot 2", "2", function()
    Open77.weapons.setActive(2)
end)
```

Registered in `ResourceHost.cpp` (line 24944) as `LuaWeaponSetActive`.

## setAmmo

```lua
Open77.weapons.setAmmo(slot, amounts)
```

`GAME`: Requires a live game instance.

Sets exact spare and/or loaded ammunition for a weapon slot.

Requires `player.weapons.edit`. `amounts` accepts integer `reserve` and `magazine` counts plus optional `activate`. At least one count is required; omitted counts are preserved. The operation verifies the shared ammo total, spare reserve, magazine, and live capacity asynchronously through `open77:weapons:completed`. When both hands share an ammo record, reserve is shared and the total includes both loaded magazines. Editing one magazine preserves the other; await completion before a dependent change. Native dual reloads with too little shared reserve can still allocate rounds inconsistently. On Stable 156 or later, inactive/holstered slots, including never-drawn assigned weapons, can be read and updated without drawing or changing selection. `activate = true` explicitly selects/draws. For an inactive slot, reserve is inventory total minus that slot's magazine and currently drawn other-hand magazines; other inactive magazines are saved counts, not separate pools. Native jobs are ordered and asynchronous; await completion. Slot changes, invalid capacity, disconnect or body replacement invalidate the job. See [weapon ammunition](../../01%20Documentation/weapons-api.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `slot` | `integer 1..6` | required | — |
| `amounts` | `table { reserve?, magazine?, activate? }` | required | — |

Returns: `request id, or nil`, `reason`

```lua
-- Refill the pistol: full magazine plus 60 spare rounds
Open77.weapons.setAmmo(1, { magazine = 12, reserve = 60 })
```

Registered in `ResourceHost.cpp` (line 24949) as `LuaWeaponSetAmmo`.

## setComponent

```lua
Open77.weapons.setComponent(slot, part, [options])
```

`GAME`: Requires a live game instance.

Installs a part (scope, muzzle, generic mod) into an attachment slot of the weapon in a loadout slot.

Requires `player.weapons.edit`. Native-parity E3. A part is an `Items.*` record tagged `itemPart` (`Items.w_att_scope_short_01`, `Items.w_silencer_01`, `Items.w_muzzle_brake_01`, ...; 928 on 2.31, see `docs/generated/weapon-parts-2.31.csv`) and the slot an `AttachmentSlots.*` record (`Scope`, `PowerModule` for muzzles, `Power_Handgun_WeaponMod1`/`2` for generic mods). The bridge script installs it through the vanilla `ItemModificationSystem` -- the same request the inventory screen queues -- so the part is real: visible on the weapon, kept across holstering, listed by the vanilla screen. Without `options.attachmentSlot` the first of the part's own placement slots this weapon declares is used, empty ones first. A part the player does not own is created for the install; a part already in the slot is swapped out and deleted, as the vanilla screen does. The returned request id proves validation and queuing only; the outcome arrives on `open77:weapons:completed` with `result.part`, preceded by one `open77:weapons:part` row for the slot. Refused by name: `invalid_weapon_slot`, `invalid_part_template`, `template_is_not_part`, `invalid_attachment_slot` now; `weapon_slot_empty`, `slot_not_on_weapon`, `part_does_not_fit_weapon`, `part_does_not_fit_slot`, `base_part_slot`, `install_rejected` from the bridge. Other players do not see the part on the owner's proxy: the proxy weapon is rebuilt from the snapshot's `ItemID`, which does not encode parts. See [Weapon Lua API](https://open2077.net/docs/api/client/weapons-api.md#weapon-components-scopes-muzzles-and-mods).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `slot` | `integer 1..6` | required | — |
| `part` | `string an `Items.*` part record` | required | — |
| `options` | `table `{ attachmentSlot = "AttachmentSlots.Scope" }`` | optional | — |

Returns: `requestId, or nil`, `reason`

```lua
-- A short scope on the rifle in slot 1; the verified outcome arrives later
local requestId, reason = Open77.weapons.setComponent(1, "Items.w_att_scope_short_01",
  { attachmentSlot = "AttachmentSlots.Scope" })
if not requestId then print("refused:", reason) end
AddEventHandler("open77:weapons:completed", function(id, operation, accepted, why)
  if tostring(id) == tostring(requestId) then print(operation, accepted, why) end
end)
```

Registered in `ResourceHost.cpp` (line 24958) as `LuaWeaponSetComponent`.

## setDualWieldAllowed

```lua
Open77.weapons.setDualWieldAllowed(enabled)
```

`GAME`: Requires a live game instance.

Enable or disable native dual wield for a player incarnation.

Queues native dual-wield permission for the current player incarnation. true enables the authenticated-session perk gate; false clears secondary slots 4–6 without deleting inventory items. Reapply after reconnect/body replacement. Wait for open77:weapons:completed before assigning and activating a compatible pair. Requires matching Stable 152 client/server builds and the updated bundled open77_weapons resource. See [Dual wielding](https://open2077.net/docs/api/client/dual-wielding.md). Requires player.weapons.edit. Completion arguments are strings; accepted must equal "true".

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `request id, or nil`, `reason`

```lua
local requestId, reason = Open77.weapons.setDualWieldAllowed(true)
if not requestId then print(reason); return end
-- Await open77:weapons:completed before assigning the two weapons.
```

[Dual wielding guide](../../01%20Documentation/dual-wielding.md)

Registered in `ResourceHost.cpp` (line 25902) as `LuaWeaponSetDualWieldAllowed`.

## setTuning

```lua
Open77.weapons.setTuning(record, options)
```

`GAME`: Requires a live game instance.

Configures a resource-owned tuning profile for the local held weapon.

Requires a compatible development client exposing native weapon tuning with inventory-instance restoration. Client-only; requires `player.weapons.edit`. Accepts an exact `Items.*` record and a complete options table: omitted fields reset to their defaults. Supports reloadSpeed, fireRate, recoil, spread, damage, magazineCapacity, projectilesPerShot, aimSpeed, chargeSpeed, smartProjectileSpeed and the blastRadius/blastPush/blastLift/blastFalloff/blastCooldown weapon-impact controls. Unknown keys, nonnumeric or nonfinite values and out-of-range multipliers are refused. Returns true when accepted, or nil/reason; read tuning() to distinguish active, waiting_for_weapon and waiting_for_restore. Only one resource owns tuning at a time; another receives weapon_tuning_owned_by_another_resource. Does not equip a weapon, rewrite shared TweakDB records, change projectile types or override server damage policy. Supported inputs are bound-weapon entity hits, actual native impact positions from GameEffectExecutor_StimOnHit, and tracked-projectile collision/explosion callbacks. No synthetic crosshair raycast is substituted. Bare-terrain Comrade coverage remains unverified; generic hitscan ground detection and grenades without the bound weapon object are not promised. Vehicle impulses require a streamed, unfrozen vehicle whose current physics owner is this client. The local open77:weaponBlast(owner,record,sequence,x,y,z) event uses string arguments after cooldown acceptance; it grants no network character authority. Resources request character launches separately through server policy. Smart projectile speed applies only to smart weapons. Clearing, switching or stopping the resource removes its own modifiers, with deferred restoration when the engine temporarily retires a holstered weapon's stats.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `record` | `string` | required | — |
| `options` | `table` | required | — |

Returns: `true or nil`, `reason on failure`

```lua
local ok, reason = Open77.weapons.setTuning("Items.Preset_Lexington_Default", { reloadSpeed = 2, recoil = 0.5 })
if not ok then print(reason) end
```

[Weapon customization & workshop](../../01%20Documentation/weapon-customization.md)

Registered in `ResourceHost.cpp` (line 24950) as `LuaWeaponSetTuning`.

## shotCounters

```lua
Open77.weapons.shotCounters()
```

`GAME`: Requires a live game instance.

Reads cumulative local discharges for each hand.

Requires `player.weapons.read`. Returns cumulative native discharges as `{ generation, right, left, total }`, or `nil, reason`. Counts real discharges, including free shots, rather than ammo cost; reloads and administrative ammo edits do not add shots. Holstering and switching preserve counts. A new character incarnation resets them and changes generation; compare generation before subtracting. Local observation only, with no implicit server broadcast or damage authority. See [shot counters](../../01%20Documentation/weapons-api.md#counting-shots-between-polls).

Returns: `table { generation, right, left, total }, or nil`, `reason`

```lua
local shots, reason = Open77.weapons.shotCounters()
if shots then print(shots.generation, shots.right, shots.left) end
```

Registered in `ResourceHost.cpp` (line 15101) as `LuaWeaponShotCounters`.

## slots

```lua
Open77.weapons.slots()
```

`GAME`: Requires a live game instance.

Lists the six supported standard weapon slots.

Requires `player.weapons.read`. Slot numbers are one-based and deliberately exclude QuickSlot grenades, WeaponHeavy items, and ArmsCW cyberware. Since Stable 152 on game 2.5, slots 1–3 are primary and 4–6 are their respective secondary partners. See [Dual wielding](https://open2077.net/docs/api/client/dual-wielding.md).

Returns: `{ 1, 2, 3, 4, 5, 6 }, or nil`, `reason`

```lua
-- Print the standard slots this API manages
for _, slot in ipairs(Open77.weapons.slots() or {}) do print("slot", slot) end
```

Registered in `ResourceHost.cpp` (line 24942) as `LuaWeaponSlots`.

## snapshot

```lua
Open77.weapons.snapshot()
```

`GAME`: Requires a live game instance.

Requests the verified state of every standard weapon slot.

Requires `player.weapons.read`. Emits one `open77:weapons:state` event per available slot and then `open77:weapons:completed`. TweakDB IDs remain opaque hexadecimal strings and are the presence authority; `record` may be empty for items equipped outside this API because shipping builds do not retain every reverse TweakDB name. Client state rows append reloading after capacity. The booleans are strings; collect rows by request ID and publish only after successful completion. See [Dual wielding](https://open2077.net/docs/api/client/dual-wielding.md#ammunition-and-hud). Stable 156 reads reserve, magazine and capacity for inactive/holstered ranged slots, including never-drawn weapons. Completion follows all native reads; unreadable ranged caches fail with `ammo_cache_unavailable`. Empty/non-ammo slots retain unavailable fields (-1).

Returns: `request id, or nil`, `reason`

```lua
-- Ask for the verified state, then read it from the events
Open77.weapons.snapshot()
AddEventHandler("open77:weapons:state", function(requestId, slot, record, tweakDbId, active, drawn)
    print(slot, record, active, drawn)
end)
```

Registered in `ResourceHost.cpp` (line 24954) as `LuaWeaponSnapshot`.

## takeGadget

```lua
Open77.weapons.takeGadget([record], [count])
```

`GAME`: Requires a live game instance.

Takes grenades back: some units of a record, or every unit of the gadget on the throw hotkey.

Requires `player.weapons.edit`. Native-parity E4. With no record the active gadget (`GetActiveGadget`) is taken; with no count every unit is. A quick slot left with nothing in it is unequipped so the next `gadgets()` does not name a stack of zero. The completion carries `result.gadget` with the remaining `quantity`. Refused by name: `invalid_gadget_template`, `template_is_not_gadget`, `invalid_gadget_count` now; `gadget_not_owned`, `no_active_gadget`, `inventory_update_rejected` from the bridge. See [Weapon Lua API](https://open2077.net/docs/api/client/weapons-api.md#gadgets-grenades-in-the-quick-slots).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `record` | `string a grenade / gadget record; nil for the active gadget` | optional | — |
| `count` | `integer 1..999, or -1 / nil for every unit` | optional | — |

Returns: `requestId, or nil`, `reason`

```lua
-- Confiscate whatever is on the throw key
local requestId, reason = Open77.weapons.takeGadget()
print(requestId or reason)
```

Registered in `ResourceHost.cpp` (line 24962) as `LuaWeaponTakeGadget`.

## tuning

```lua
Open77.weapons.tuning()
```

`GAME`: Requires a live game instance.

Reads this resource's native weapon tuning state and diagnostics.

Requires a compatible development client exposing native weapon tuning with inventory-instance restoration. Client-only; requires `player.weapons.read`. Returns a table or nil/reason. status distinguishes idle, waiting_for_weapon, active, restoring, waiting_for_restore and native failure states; active reports whether the matching weapon is bound. An owned profile includes record, opaque string weaponEntity, modifiers, native stats and cumulative blasts, impulsesQueued, failures and foreignSkipped counters. projectileContacts counts supported tracked-projectile contacts only; trackedProjectiles is the number still being tracked. Native entity hits and GameEffectExecutor_StimOnHit do not increment projectileContacts, so an accepted blast can leave it at zero. Neither counter establishes a character launch or observed movement. pendingModifiers counts this resource's deferred cleanup. Idle/restoring results can omit bound-weapon fields. Counters describe queued work rather than measured movement, and a native statistic does not prove that every weapon's animation or projectile logic consumes it. Treat weaponEntity as an opaque string, never as a Lua number.

Returns: `state table or nil`, `reason on failure`

```lua
local state, reason = Open77.weapons.tuning()
if state then
    print(state.status, state.active, state.impulsesQueued or 0)
else
    print(reason)
end
```

[Weapon customization & workshop](../../01%20Documentation/weapon-customization.md)

Registered in `ResourceHost.cpp` (line 24952) as `LuaWeaponTuning`.

## unequip

```lua
Open77.weapons.unequip(slot)
```

`GAME`: Requires a live game instance.

Alias of `Open77.weapons.remove`.

Requires `player.weapons.edit`; the inventory instance is retained.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `slot` | `integer 1..6` | required | — |

Returns: `request id, or nil`, `reason`

```lua
-- Confiscate slot 3 (alias of remove)
Open77.weapons.unequip(3)
```

Registered in `ResourceHost.cpp` (line 24947) as `LuaWeaponRemove`.
