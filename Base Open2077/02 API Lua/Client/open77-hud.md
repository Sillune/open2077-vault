# Open77.hud (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-hud.md) · [Page web](https://open2077.net/docs/api/client/open77-hud)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

11 functions.

## clearNotify

```lua
Open77.hud.clearNotify([channel])
```

`GAME`: Requires a live game instance.

Empties one vanilla notification channel.

Client-only; requires `ui.vanilla.hud`. The channel is "ingame" (default) or "menu". There is one vanilla slot per channel, so this clears whatever is in it, whoever put it there.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `channel` | `string` | optional | — |

Returns: `true on success, otherwise false`, `failure reason`

```lua
-- Clear the vanilla toast slot when the prompt no longer applies
AddEventHandler("delivery:done", function()
    Open77.hud.clearNotify("ingame")
end)
```

[HUD visibility guide](../../01%20Documentation/hud-visibility.md)

Registered in `ResourceHost.cpp` (line 24999) as `LuaHudClearNotification`.

## components

```lua
Open77.hud.components()
```

`GAME`: Requires a live game instance.

Lists the supported stock HUD component names.

Requires `ui.vanilla.hud`. Returns the thirteen canonical names suitable for `setVisible`, `isVisible`, or a complete custom-HUD replacement loop. The `all` selector is not listed: it is a selector, not a component.

Returns: `array of canonical component names, or nil`, `reason on failure`

```lua
-- Hide every stock component except the minimap
for _, name in ipairs(Open77.hud.components() or {}) do
    if name ~= "minimap" then Open77.hud.setVisible(name, false) end
end
```

[HUD visibility guide](../../01%20Documentation/hud-visibility.md)

Registered in `ResourceHost.cpp` (line 24980) as `LuaHudComponents`.

## help

```lua
Open77.hud.help(text, [durationMs])
```

`GAME`: Requires a live game instance.

Vanilla on-screen help text. UNSUPPORTED on this build.

Client-only; requires `ui.vanilla.hud`. Validates its arguments (duration 100..60000 ms) and then refuses with `unsupported_channel` without entering engine code. The global behind it, MPDisplayOnscreenMessage, belongs to Cyberpunk's dormant multiplayer HUD and closed the process with an access violation when invoked on 2.31. The name stays bound so a resource written against it keeps working the day the channel is real; today, use Open77.hud.notify.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `text` | `string` | required | — |
| `durationMs` | `number` | optional | — |

Returns: `false`, `unsupported_channel`

```lua
-- Unsupported on 2.31: always false, "unsupported_channel". Use Open77.hud.notify instead.
local ok, reason = Open77.hud.help("HOLD F", 2000)
if not ok then Open77.hud.notify("Hold F to open the trunk") end
```

[HUD visibility guide](../../01%20Documentation/hud-visibility.md)

Registered in `ResourceHost.cpp` (line 24996) as `LuaHudNotification`.

## isVisible

```lua
Open77.hud.isVisible(component)
```

`GAME`: Requires a live game instance.

Reads the effective visibility of one stock HUD component.

Requires `ui.vanilla.hud`. The result already combines hide claims from every resource. Names are case-insensitive and aliases such as `map`, `time`, `hp`, `ammo`, `weaponAmmo`, `speed`, `quest`, `vision`, `toasts`, `reticle` and `hub` are accepted. The selector `all` answers true only when every component is visible.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `component` | `string` | required | — |

Returns: `boolean, or nil`, `reason on failure`

```lua
-- Draw our own health bar only when the stock one is hidden
if Open77.hud.isVisible("health") == false then drawCustomHealthBar() end
```

[HUD visibility guide](../../01%20Documentation/hud-visibility.md)

Registered in `ResourceHost.cpp` (line 24978) as `LuaHudIsVisible`.

## log

```lua
Open77.hud.log(text)
```

`GAME`: Requires a live game instance.

Adds one line to Cyberpunk's activity log.

Client-only; requires `ui.vanilla.hud`. Calls `gameActivityLogSystem.AddLog`. Confirmed working on 2.31.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `text` | `string` | required | — |

Returns: `true on success, otherwise false`, `failure reason`

```lua
-- A line in Cyberpunk's activity log when a contract is accepted
RegisterNetEvent("jobs:accepted", function(title)
    Open77.hud.log("Contract accepted: " .. title)
end)
```

[HUD visibility guide](../../01%20Documentation/hud-visibility.md)

Registered in `ResourceHost.cpp` (line 24991) as `LuaHudNotification`.

## menu

```lua
Open77.hud.menu(preset)
```

`GAME`: Requires a live game instance.

Queues a vanilla menu notification preset.

Client-only; requires `ui.vanilla.hud`. A menu preset carries its own localized title and therefore takes no text. Accepts vendor_not_enough_money, not_enough_money, inventory_action_blocked, crafting_not_enough_materials, upgrade_level_too_low, no_perk_points, perks_locked, perks_max_level, no_attribute_points, in_combat, in_combat_explicit, crafting_quickhack, crafting_ammo_cap, player_level_too_low, inventory_full, face_unequip_blocked, tutorial_unequip_blocked and no_junk. Only visible while the menu layer that owns the queue is mounted.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `preset` | `string` | required | — |

Returns: `true on success, otherwise false`, `failure reason`

```lua
-- The player cannot afford the item: the vanilla "not enough money" preset
RegisterNetEvent("shop:tooPoor", function()
    Open77.hud.menu("not_enough_money")
end)
```

[HUD visibility guide](../../01%20Documentation/hud-visibility.md)

Registered in `ResourceHost.cpp` (line 24998) as `LuaHudMenuNotification`.

## notify

```lua
Open77.hud.notify(text, [options])
```

`GAME`: Requires a live game instance.

Queues Cyberpunk's own five-second side notification.

Client-only; requires `ui.vanilla.hud`. Writes to the real vanilla pipeline (`UIInGameNotificationEvent`), not to a WebUI surface, so it looks native and needs no resource dependency. Options: `replace` (replace what is on screen instead of queueing) and `preset` (one of generic, action_blocked, combat_blocked, cant_save_action, cant_save_combat, cant_save_quest, cant_save_death, save_slots_full, save_space_full, photo_mode_disabled, sandevistan_in_call, expansion_installed). Text is required unless a preset is given, since a preset carries its own localized title. The five-second duration is the engine's and cannot be changed. Hiding the `vanillaNotifications` HUD component hides this stack, including your own toasts.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `text` | `string` | required | — |
| `options` | `table` | optional | — |

Returns: `true on success, otherwise false`, `failure reason`

```lua
-- A native five-second toast when the bounty is paid
RegisterNetEvent("bounty:paid", function(amount)
    Open77.hud.notify(("BOUNTY CLAIMED: %d eddies"):format(amount))
end)
```

[HUD visibility guide](../../01%20Documentation/hud-visibility.md)

Registered in `ResourceHost.cpp` (line 24990) as `LuaHudNotification`.

## setCinematic

```lua
Open77.hud.setCinematic(enabled, [height])
```

`GAME`: Requires a live game instance.

Hides gameplay HUD and nonfocused resource interfaces with animated letterbox bars.

Client-only; requires `ui.vanilla.hud`. enabled must be a boolean; optional height is a finite fraction of the viewport for EACH black bar, 0.01..0.4 (default 0.12). true holds a resource-instance cinematic claim; false releases only the caller's claim. Multiple resources cooperate and the greatest requested height wins. Masks native HUD including minimap, nonfocused Open77 WebUI pages including watermark/chat history/Freeroam HUD, and native world labels/debug overlays. Focused chat or menus remain accessible when opened, as does the developer console. Rendering is gated without destroying pages or changing their visibility state. Existing HUD hide claims, including the same resource's ordinary claims, survive release. Stop/reload, coroutine failure and host teardown clean up automatically. Does not change camera, pause gameplay, mute audio or affect other players. Distinct from the open77_uikit letterbox-only helper. Errors: `permission_denied:ui.vanilla.hud`, `invalid_cinematic_arguments`, `invalid_cinematic_height`, `cinematic_unavailable_on_this_host`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |
| `height` | `number` | optional | `0.12` |

Returns: `true on success, otherwise false`, `reason on failure`

```lua
-- Manifest: permissions { "ui.vanilla.hud" }
local ok, reason = Open77.hud.setCinematic(true, 0.12)
if not ok then print(reason) end
-- At the end of the sequence, release only this resource's claim.
Open77.hud.setCinematic(false)
```

[HUD visibility guide](../../01%20Documentation/hud-visibility.md)

Registered in `ResourceHost.cpp` (line 24977) as `LuaHudSetCinematic`.

## setVisible

```lua
Open77.hud.setVisible(component, visible)
```

`GAME`: Requires a live game instance.

Hides or restores one stock HUD component for this resource.

Requires `ui.vanilla.hud`. Passing `false` adds this resource's hide claim; passing `true` releases only its own claim. Another resource may still keep the component hidden. Claims are released automatically when the resource stops or reloads. Canonical components are `minimap`, `compass`, `clock`, `health`, `stamina`, `weapon`, `speedometer`, `questTracker`, `phone`, `scanner`, `vanillaNotifications`, `crosshair` and `hubMenu`; the selector `all` claims or releases every one of them at once. Names are case-insensitive. `scanner` also refuses scanner activation, because an invisible scanner would still hold aim snap and the movement restriction.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `component` | `string` | required | — |
| `visible` | `boolean` | required | — |

Returns: `true on success, otherwise false`, `effective visibility, or failure reason`

```lua
-- Hide the minimap during the race; release only our own claim afterwards
AddEventHandler("race:started", function() Open77.hud.setVisible("minimap", false) end)
AddEventHandler("race:finished", function() Open77.hud.setVisible("minimap", true) end)
```

[HUD visibility guide](../../01%20Documentation/hud-visibility.md)

Registered in `ResourceHost.cpp` (line 24976) as `LuaHudSetVisible`.

## state

```lua
Open77.hud.state()
```

`GAME`: Requires a live game instance.

Returns the complete effective stock-HUD visibility snapshot.

Requires `ui.vanilla.hud`. Keys always use canonical component names and values are booleans after all resources' claims have been combined. The `all` selector is not a component and never appears here.

Returns: `table keyed by canonical component name, or nil`, `reason on failure`

```lua
-- Debug: print the effective visibility of every component
for name, visible in pairs(Open77.hud.state() or {}) do print(name, visible) end
```

[HUD visibility guide](../../01%20Documentation/hud-visibility.md)

Registered in `ResourceHost.cpp` (line 24979) as `LuaHudState`.

## subtitle

```lua
Open77.hud.subtitle(text, [durationMs])
```

`GAME`: Requires a live game instance.

Vanilla narration subtitle. UNSUPPORTED on this build.

Client-only; requires `ui.vanilla.hud`. Same story as Open77.hud.help: validated, then refused with `unsupported_channel`. MPDisplayNarrationEventEntry comes from the same dormant CPO group and is deliberately never invoked.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `text` | `string` | required | — |
| `durationMs` | `number` | optional | — |

Returns: `false`, `unsupported_channel`

```lua
-- Unsupported on 2.31: always false, "unsupported_channel". Use Open77.hud.notify instead.
local ok, reason = Open77.hud.subtitle("...", 3000)
if not ok then Open77.hud.notify("Dispatch: all units respond") end
```

[HUD visibility guide](../../01%20Documentation/hud-visibility.md)

Registered in `ResourceHost.cpp` (line 24997) as `LuaHudNotification`.
