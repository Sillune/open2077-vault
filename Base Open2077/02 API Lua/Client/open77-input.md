# Open77.input (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-input.md) · [Page web](https://open2077.net/docs/api/client/open77-input)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

18 functions.

## axis

```lua
Open77.input.axis(axis)
```

`GAME`: Requires a live game instance.

Reads one analogue control: a stick, a trigger, the cursor or a cumulative counter.

Requires `input.actions`. Sticks (`padLeftStickX/Y`, `padRightStickX/Y`, also spelled `leftStickX` and so on) return -1..1 with XInput's own dead zone removed and the remainder rescaled, so a stick at rest reads exactly 0 and a stick at the rim exactly 1; stick Y is positive UP. Triggers (`padLeftTrigger`, `padRightTrigger`) return 0..1. `mouseX` and `mouseY` are the cursor, identical to `Open77.input.cursor()`. `mouseWheel`, `mouseWheelX`, `mouseDeltaX` and `mouseDeltaY` are CUMULATIVE counters since the client started -- difference two samples to get what happened between them. They are deliberately not consumed on read: a consume-on-read wheel would give each of two resources part of the same scroll. A missing pad answers `nil, "gamepad_not_connected"` rather than a silent zero, because a zero cannot be told from a stick held at centre. A button name answers `nil, "control_is_a_button"`. Gamepad reads are zero while the game window is not in the foreground.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `axis` | `string` | required | — |

Returns: `number, or nil, reason`

```lua
-- Sticks and triggers are instantaneous; the wheel and the mouse deltas are
-- CUMULATIVE counters, so difference two samples inside a task.
CreateThread(function()
    local before = Open77.input.axis("mouseWheel") or 0
    while true do
        Wait(100)
        local now = Open77.input.axis("mouseWheel") or 0
        if now ~= before then zoom = zoom - (now - before) end
        before = now
        local turn = Open77.input.axis("padRightStickX") or 0   -- -1..1, dead zone removed
        if math.abs(turn) > 0.2 then panCamera(turn) end
    end
end)
```

[Key bindings guide](../../01%20Documentation/keybindings.md)

Registered in `ResourceHost.cpp` (line 25418) as `LuaInputAxis`.

## blockableActions

```lua
Open77.input.blockableActions()
```

`GAME`: Requires a live game instance.

The actions that can really be blocked on this build, with mechanism and confidence.

Requires `input.actions`. Returns the current action vocabulary, mechanism and confidence. Stable 156 supports NextWeapon, Reload, SwitchItem, Grenade, Consumable, RadialWheels, VehicleInsideWheel and PocketRadioWheel. NextWeapon blocks native selection/holstering while preserving held-weapon fire and Lua setActive. Blocking an open wheel cancels its modal state. VehicleWheel remains inferred and cannot lift the enforced CallVehicle policy. Movement, Map, Hub, FastTravel, WeaponWheel remain available; Attack, Melee, Jump and Sprint have inferred confidence. Unknown, unsupported and enforced names fail explicitly. See [input blocking](../../01%20Documentation/input-blocking.md).

Returns: `array of { action, mechanism, confidence, note }, otherwise nil`, `reason: permission_denied:input.actions, action_blocking_unavailable_on_this_host`

```lua
-- Only block what this build can really block
for _, entry in ipairs(Open77.input.blockableActions() or {}) do
    print(entry.action, entry.mechanism, entry.confidence)
end
```

[Blocking player input guide](../../01%20Documentation/input-blocking.md)

Registered in `ResourceHost.cpp` (line 25403) as `LuaBlockableActions`.

## blockAll

```lua
Open77.input.blockAll(spec)
```

`GAME`: Requires a live game instance.

Takes the player's whole gameplay control stream, sparing named exceptions.

Requires `input.blockAll`, a separate permission from `input.actions` because it is a different blast radius. Consumes the entire gameplay action stream on the player puppet and asserts every blockable lever, minus `except`. Call with `{ except = { "Movement" } }` or nothing to acquire, `false` to release this resource's claim. Chat and voice keep working -- they run on focused browser input, not on the gameplay action stream -- so a blocked player can still say so. Vanilla menus are NOT closed or prevented: menu shortcuts arrive on a different controller alongside the pause menu, which must stay reachable, so block `Map` and `Hub` explicitly. `except` names vocabulary actions, never raw engine actions, and is validated in full before anything is taken.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `spec` | `table { except = { string } } | boolean | nil` | required | — |

Returns: `true, otherwise false`, `reason: permission_denied:input.blockAll, invalid_action_arguments, unknown_action, action_not_blockable, action_already_enforced, action_blocking_unavailable_on_this_host`

```lua
-- A cutscene: take every gameplay control except looking around
Open77.input.blockAll({ except = { "Movement" } })
AddEventHandler("cutscene:ended", function() Open77.input.blockAll(false) end)
```

[Blocking player input guide](../../01%20Documentation/input-blocking.md)

Registered in `ResourceHost.cpp` (line 25402) as `LuaBlockAllInput`.

## blocks

```lua
Open77.input.blocks()
```

`GAME`: Requires a live game instance.

Who currently holds which input blocks.

Requires `input.actions`. Returns `{ actions = { { action, owners } }, blockAll = { { owner, except } } }`. This is how an admin or debug resource proves a claim really came back: the failure this API most has to avoid is a player left unable to act by a resource that stopped without tidying up.

Returns: `{ actions, blockAll }, otherwise nil`, `reason: permission_denied:input.actions, action_blocking_unavailable_on_this_host`

```lua
-- Debug: who is holding a block right now?
local blocks = Open77.input.blocks()
for _, row in ipairs(blocks and blocks.actions or {}) do
    print(row.action, table.concat(row.owners, ", "))
end
```

[Blocking player input guide](../../01%20Documentation/input-blocking.md)

Registered in `ResourceHost.cpp` (line 25404) as `LuaActionBlocks`.

## cursor

```lua
Open77.input.cursor()
```

`GAME`: Requires a live game instance.

Read the OS cursor relative to the foreground game window.

Requires input.actions. Returns normalized x/y, pixelX/pixelY, client-area width/height, inBounds and captured. captured is true when any WebUI owns keyboard or cursor input. The coordinates may fall outside [0,1]; check inBounds. Returns nil, game_not_focused on focus loss, or a cursor/viewport availability reason. Does not show, capture or unlock the mouse: use an owned WebUI page with setFocus. WebUI uses its own virtual pointer; for exact WebUI clicks pass event.clientX/innerWidth and event.clientY/innerHeight to screenRaycast instead. See screen-picking.md.

Returns: `cursor table, or nil, reason`

```lua
local cursor, reason = Open77.input.cursor()
if cursor and cursor.inBounds then print(cursor.x, cursor.y) end
```

[Key bindings guide](../../01%20Documentation/keybindings.md)

Registered in `ResourceHost.cpp` (line 19514) as `LuaInputCursor`.

## devices

```lua
Open77.input.devices()
```

`GAME`: Requires a live game instance.

Lists the mouse and gamepad controls, and whether this machine can answer for each.

Requires `input.actions`. Each row is `{ name, device, kind, available, reason }`: `device` is `mouse` or `gamepad`, `kind` is `button` or `axis`, and `name` is the canonical spelling to pass to `isDown` or `axis` (aliases resolve but are not listed twice). A control this machine cannot answer for is present and marked `available = false` with a reason -- an unplugged pad is `gamepad_not_connected` -- rather than omitted, so a resource discovers the gap instead of finding out from an axis that silently reads zero. Keyboard keys are not listed: their vocabulary belongs to `isDown` and `registerKeyMapping`. Same honesty contract as `Open77.input.blockableActions()`.

Returns: `array of control tables, or nil, reason`

```lua
for _, c in ipairs(Open77.input.devices() or {}) do
    if not c.available then print(c.name, c.reason) end
end
```

[Key bindings guide](../../01%20Documentation/keybindings.md)

Registered in `ResourceHost.cpp` (line 25423) as `LuaInputDevices`.

## isActionBlocked

```lua
Open77.input.isActionBlocked(action)
```

`GAME`: Requires a live game instance.

Reads whether one curated action is blocked right now, for anyone.

Requires `input.actions`. Effective state, combining every resource's claims and any standing `blockAll` -- not the caller's own claim, so a resource that released its block still reads true while another resource holds one. Actions Open77 enforces unconditionally read as blocked; `Scanner` and `Quickhack` follow the multiplayer policy, `PhotoMode` is always blocked.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `action` | `string (see blockableActions)` | required | — |

Returns: `boolean, otherwise nil`, `reason: permission_denied:input.actions, unknown_action, action_not_blockable, action_blocking_unavailable_on_this_host`

```lua
-- Do not draw the map hint if the map is blocked
if not Open77.input.isActionBlocked("Map") then
    Open77.hud.notify("Press M to open the map")
end
```

[Blocking player input guide](../../01%20Documentation/input-blocking.md)

Registered in `ResourceHost.cpp` (line 25401) as `LuaIsActionBlocked`.

## isCaptured

```lua
Open77.input.isCaptured()
```

`GAME`: Requires a live game instance.

Whether another WebUI currently owns keyboard input.

Contextual actions should become inert while this is true so chat and menu typing cannot activate gameplay.

Returns: `boolean`, `reason when permission is refused`

```lua
-- A held key does nothing while the player is typing in chat
SetTick(function()
    if Open77.input.isCaptured() then return end
    if Open77.input.isDown("mouse2") then showAimOverlay() end
end)
```

[Key bindings guide](../../01%20Documentation/keybindings.md)

Registered in `ResourceHost.cpp` (line 25394) as `LuaInputCaptured`.

## isDown

```lua
Open77.input.isDown(control)
```

`GAME`: Requires a live game instance.

Whether a key, mouse button or gamepad button is currently held.

Requires `input.actions`. Keys: A-Z, 0-9, F1-F12, SPACE, ENTER/RETURN, TAB, SHIFT, CTRL/CONTROL, ALT, CAPSLOCK, BACKSPACE, INSERT, DELETE, HOME, END, PAGEUP/PAGEDOWN and arrows. Mouse: mouse1/mouseLeft, mouse2/mouseRight, mouse3/mouseMiddle, mouse4, mouse5. Gamepad: padA/B/X/Y, padUp/Down/Left/Right, padStart, padBack, padLeftShoulder (padLB), padRightShoulder (padRB), padLeftThumb, padRightThumb. Names are case-insensitive; `Open77.input.devices()` lists what this machine can actually answer for. These are PHYSICAL device reads -- they do not know the player's bindings and do not know whether the game is currently consuming the control, which makes them the `GetDisabledControlNormal` flavour rather than the enabled one. An unplugged pad answers `false, "gamepad_not_connected"`; an axis name answers `false, "control_is_an_axis"`. Prefer a press edge or a hold timer over firing every frame.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `control` | `string` | required | — |

Returns: `boolean`, `reason when the control, the device or the permission is refused`

```lua
if Open77.input.isDown("mouse2") and not Open77.input.isCaptured() then
    showAimOverlay()
end
```

[Key bindings guide](../../01%20Documentation/keybindings.md)

Registered in `ResourceHost.cpp` (line 25393) as `LuaInputKeyDown`.

## keyFor

```lua
Open77.input.keyFor(id)
```

`GAME`: Requires a live game instance.

The effective key of one of this resource's own mappings.

Requires `input.actions`. Returns the effective key (the player's rebind if any, otherwise the registered default) for a mapping id owned by the calling resource, or nil when no such mapping exists.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |

Returns: `key string, or nil when the id is not registered by this resource`

```lua
-- Show the real key in the prompt, even after the player rebound it
Open77.hud.notify(("Press %s to open the inventory"):format(Open77.input.keyFor("inventory_open") or "I"))
```

[Key bindings guide](../../01%20Documentation/keybindings.md)

Registered in `ResourceHost.cpp` (line 25410) as `LuaInputKeyFor`.

## mappings

```lua
Open77.input.mappings()
```

`GAME`: Requires a live game instance.

Every registered key mapping across all running resources.

Requires `input.actions`. Returns an array of `{ resource, id, name, key, defaultKey, hold, rebound }` records for the rebinding UI. Only serialisable metadata is exposed; the callbacks stay in their owning resource. Stable 156 accepts wheel and gamepad button/trigger mappings; `consume` is an optional boolean on registered mapping specs. See [supported controls](../../01%20Documentation/keybindings.md#key-names).

Returns: `array of mapping records`

```lua
-- A settings page listing every rebindable action
for _, mapping in ipairs(Open77.input.mappings()) do
    print(mapping.resource, mapping.name, mapping.key, mapping.rebound and "(rebound)" or "")
end
```

[Key bindings guide](../../01%20Documentation/keybindings.md)

Registered in `ResourceHost.cpp` (line 25411) as `LuaInputMappings`.

## pointer

```lua
Open77.input.pointer()
```

`GAME`: Requires a live game instance.

Where the pointer is, as 0..1 across the rendered image -- the overlay's own pointer when a WebUI holds the cursor.

Requires `input.actions`. Returns `x, y, source`. Unlike `Open77.input.cursor`, which samples the OS cursor against the game window and answers with a table, this reads the WebUI's VIRTUAL pointer whenever a surface owns the cursor (`source = "overlay"`) -- Cyberpunk keeps recentring the Win32 cursor while it owns the camera, so the OS position says nothing about where the drawn pointer is -- and falls back to the OS cursor otherwise (`source = "system"`). The origin is the TOP LEFT and the units are fractions of the image -- the same frame `Open77.camera.project` answers in and `Open77.camera.unproject` consumes, so hit-testing a projected world point is a subtraction rather than a conversion. It is a ratio and not pixels on purpose: a client-rect read and the engine's back-buffer size disagree on a display with DPI scaling, so every pixel answer would need a qualifier the caller cannot see. `source` is `overlay` when a WebUI surface owns the pointer -- Open77's own virtual cursor, the one CEF and the drawn pointer share -- or `system` when the OS cursor is the only answer. During ordinary gameplay Cyberpunk recentres the OS cursor every frame, so `system` reads near 0.5, 0.5; that is where the cursor is, not a failed read.

Returns: `x, y, source, or nil, reason`

```lua
-- Where is the pointer over the image right now?
local x, y, source = Open77.input.pointer()
if x then print(("%.3f %.3f (%s)"):format(x, y, source)) end
```

[Key bindings guide](../../01%20Documentation/keybindings.md)

Registered in `ResourceHost.cpp` (line 25422) as `LuaInputPointer`.

## rebind

```lua
Open77.input.rebind(resource, id, key)
```

`GAME`: Requires a live game instance.

Sets and persists a new key for any mapping.

Requires `input.actions`. Rebinds the mapping identified by `resource` and `id` to `key` (same key vocabulary as `Open77.input.isDown`), persists it machine-globally, and raises `open77:keybinds:changed`. Used by the pause KEY BINDINGS tab. Stable 156 accepts wheel and gamepad button/trigger mappings; `consume` is an optional boolean on registered mapping specs. See [supported controls](../../01%20Documentation/keybindings.md#key-names).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `resource` | `string` | required | — |
| `id` | `string` | required | — |
| `key` | `string` | required | — |

Returns: `the applied key`, `reason for the refusal`

```lua
-- A settings page rebinds another resource's action
Open77.input.rebind("myinventory", "inventory_open", "TAB")
```

[Key bindings guide](../../01%20Documentation/keybindings.md)

Registered in `ResourceHost.cpp` (line 25412) as `LuaInputRebind`.

## registerKeyMapping

```lua
Open77.input.registerKeyMapping(spec)
```

`GAME`: Requires a live game instance.

Declares a named, rebindable action bound to a key (twin of the RegisterKeyMapping global).

Requires `input.actions`. Accepts `{ id, name, key (or defaultKey), hold?, consume?, onPressed, onReleased? }` or `(id, name, key, onPressed [, onReleased])`. Returns `true, effectiveKey` or `false, reason`. The callback receives the mapping ID; hold mode adds releases. Defaults to observing input; Stable 156 adds `consume = true` to suppress the bound native input while invoking Lua. Keyboard, MOUSE1..5, MOUSEWHEELUP/DOWN and gamepad buttons/triggers are supported; wheel holds emit press/release pulses and triggers use a 0.5 threshold. Gamepad behavior remains experimental. Pause and voice capture support keyboard and mouse buttons; use Lua/rebind for wheel/gamepad controls. Mappings pause while a WebUI owns input. Re-registering preserves saved rebinds; unregister/stop/rebind/focus changes release consumption. See [key mappings](../../01%20Documentation/keybindings.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `spec` | `table` | required | — |

Returns: `boolean: true on installation, false on refusal`, `string: effective key on success, reason on refusal`

```lua
-- A hold-to-talk push key with press and release callbacks
Open77.input.registerKeyMapping({
    id = "radio_ptt",
    name = "Radio push-to-talk",
    key = "CAPSLOCK",
    hold = true,
    onPressed = function() TriggerServerEvent("radio:talk", true) end,
    onReleased = function() TriggerServerEvent("radio:talk", false) end,
})
```

[Key bindings guide](../../01%20Documentation/keybindings.md)

Registered in `ResourceHost.cpp` (line 25408) as `LuaRegisterKeyMapping`.

## reset

```lua
Open77.input.reset(resource, id)
```

`GAME`: Requires a live game instance.

Restores a mapping to its registered default key.

Requires `input.actions`. Drops any saved override for the mapping identified by `resource` and `id`, restores the default key, persists the change, and raises `open77:keybinds:changed`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `resource` | `string` | required | — |
| `id` | `string` | required | — |

Returns: `the default key`, `reason for the refusal`

```lua
-- "Reset to default" button on the settings page
Open77.input.reset("myinventory", "inventory_open")
```

[Key bindings guide](../../01%20Documentation/keybindings.md)

Registered in `ResourceHost.cpp` (line 25413) as `LuaInputReset`.

## setActionBlocked

```lua
Open77.input.setActionBlocked(action, blocked)
```

`GAME`: Requires a live game instance.

Blocks or releases one curated input action for the calling resource.

Requires `input.actions`. Sets or releases this resource's action block; another owner's block remains effective. Claims are released on resource stop. Stable 156 supports NextWeapon, Reload, SwitchItem, Grenade, Consumable, RadialWheels, VehicleInsideWheel and PocketRadioWheel. NextWeapon blocks native selection/holstering while preserving held-weapon fire and Lua setActive. Blocking an open wheel cancels its modal state. VehicleWheel remains inferred and cannot lift the enforced CallVehicle policy. Movement, Map, Hub, FastTravel, WeaponWheel remain available; Attack, Melee, Jump and Sprint have inferred confidence. Unknown, unsupported and enforced names fail explicitly. See [input blocking](../../01%20Documentation/input-blocking.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `action` | `string (see blockableActions)` | required | — |
| `blocked` | `boolean` | required | — |

Returns: `true, otherwise false`, `reason: permission_denied:input.actions, invalid_action_arguments, unknown_action, action_not_blockable, action_already_enforced, action_blocking_unavailable_on_this_host`

```lua
-- No fast travel during the race
AddEventHandler("race:started", function() Open77.input.setActionBlocked("FastTravel", true) end)
AddEventHandler("race:finished", function() Open77.input.setActionBlocked("FastTravel", false) end)
```

[Blocking player input guide](../../01%20Documentation/input-blocking.md)

Registered in `ResourceHost.cpp` (line 25400) as `LuaSetActionBlocked`.

## setNativeActionBlocked

```lua
Open77.input.setNativeActionBlocked(action, blocked)
```

`GAME`: Requires a live game instance.

Blocks or restores one vanilla input action by its engine name.

Requires input.actions. Accepts exactly (actionName, blockedBoolean); supports OpenMapMenu and WeaponWheel. Applies or removes the calling resource's claim, returning boolean/reason. A block remains while any resource claims it; stop/reload/disconnect automatically releases that owner's claims. WeaponWheel prevents native radial entry and cancels an already-open weapon wheel without selecting an item. It does not remap ALT, block other radial menus or disable SwitchItem. Acquire before the activation key is pressed. Other action names return unsupported_native_action. This is not a general-purpose arbitrary native input blocker.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `action` | `string (OpenMapMenu | WeaponWheel)` | required | — |
| `blocked` | `boolean` | required | — |

Returns: `true, otherwise false`, `reason: permission_denied:input.actions, invalid_native_action_arguments, unsupported_native_action, native_action_unavailable`

```lua
-- A custom weapon wheel replaces the vanilla one
Open77.input.setNativeActionBlocked("WeaponWheel", true)
```

[Key bindings guide](../../01%20Documentation/keybindings.md)

Registered in `ResourceHost.cpp` (line 25396) as `LuaSetNativeActionBlocked`.

## unregisterKeyMapping

```lua
Open77.input.unregisterKeyMapping(id)
```

`GAME`: Requires a live game instance.

Removes a key mapping this resource declared.

Requires `input.actions`. Frees the mapping's callbacks and raises `open77:keybinds:changed`. Only affects mappings owned by the calling resource.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |

Returns: `boolean: true when a mapping was removed`

```lua
-- The job ends: its key goes away
Open77.input.unregisterKeyMapping("job_cancel")
```

[Key bindings guide](../../01%20Documentation/keybindings.md)

Registered in `ResourceHost.cpp` (line 25409) as `LuaUnregisterKeyMapping`.
