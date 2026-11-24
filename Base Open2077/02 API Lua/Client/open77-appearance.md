# Open77.appearance (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-appearance.md) · [Page web](https://open2077.net/docs/api/client/open77-appearance)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

8 functions.

## apply

```lua
Open77.appearance.apply(snapshot)
```

`GAME`: Requires a live game instance.

Applies a previously captured appearance snapshot to the local player.

Requires `player.appearance.edit`. The runtime validates the schema, build, catalog digest, gender, option identities, choice bounds, and duplicates before touching the game.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `snapshot` | `table` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Restore the look the server saved for this character
RegisterNetEvent("character:appearance", function(snapshot)
    local ok, reason = Open77.appearance.apply(snapshot)
    if not ok then print("appearance not applied: " .. reason) end
end)
```

Registered in `ResourceHost.cpp` (line 24819) as `LuaAppearanceApply`.

## capture

```lua
Open77.appearance.capture()
```

`GAME`: Requires a live game instance.

Captures the local player's complete validated appearance snapshot.

Requires `player.appearance.read`. The snapshot is bound to schema version 1, game build 2.31, a catalog digest, gender hash, and at most 256 head/body/arms options.

Returns: `table { schemaVersion, gameBuild, catalogDigest, gender, options }, or nil`, `reason`

```lua
-- Save the look after the player leaves the ripperdoc
AddEventHandler("appearance:editorClosed", function()
    local snapshot = Open77.appearance.capture()
    if snapshot then TriggerServerEvent("character:saveAppearance", snapshot) end
end)
```

Registered in `ResourceHost.cpp` (line 24818) as `LuaAppearanceCapture`.

## captureBody

```lua
Open77.appearance.captureBody()
```

`GAME`: Requires a live game instance.

Capture the local native body appearance groups.

Requires player.appearance.read. Returns {family,groups}, with native part/name hashes and resource key pairs. This readback is for trusted projection services, not a committed character edit or visual proof. Use the selected durable identity adapter for persistence.

Returns: `body table, or nil, reason`

```lua
local body, err = Open77.appearance.captureBody()
if body then print(body.family, #body.groups) else print(err) end
```

Registered in `ResourceHost.cpp` (line 24823) as `LuaAppearanceCaptureBody`.

## finishCommit

```lua
Open77.appearance.finishCommit()
```

`GAME`: Requires a live game instance.

Finishes the current appearance commit transaction.

Requires `player.appearance.edit`. Call after a successful apply/persistence flow so the native editor can release its commit state.

Returns: `true on success, otherwise false`, `reason`

```lua
-- The server confirmed the save: release the editor's commit state
RegisterNetEvent("character:appearanceSaved", function()
    Open77.appearance.finishCommit()
end)
```

Registered in `ResourceHost.cpp` (line 24820) as `LuaAppearanceFinish`.

## isOpen

```lua
Open77.appearance.isOpen()
```

`GAME`: Requires a live game instance.

Whether the local appearance editor is open.

Requires `player.appearance.read`; returns false when the permission or backend is unavailable.

Returns: `boolean`

```lua
-- Hide the HUD while the editor is open
SetTick(function()
    hud.visible = not Open77.appearance.isOpen()
end)
```

Registered in `ResourceHost.cpp` (line 24817) as `LuaAppearanceIsOpen`.

## open

```lua
Open77.appearance.open([options])
```

`GAME`: Requires a live game instance.

Opens the local Cyberpunk appearance editor.

Requires `player.appearance.edit`. The optional value may be a mode string or `{ mode = string }`; `ripperdoc` is used by default.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `string|table` | optional | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Press E at the ripperdoc to open the editor
RegisterKeyMapping("ripperdoc", "Talk to the ripperdoc", "E", function()
    if nearRipperdoc then Open77.appearance.open("ripperdoc") end
end)
```

Registered in `ResourceHost.cpp` (line 24816) as `LuaAppearanceOpen`.

## switchBodyFamily

```lua
Open77.appearance.switchBodyFamily(gender, [resumeEditor])
```

`GAME`: Requires a live game instance.

Starts a guarded body-family transition.

Starts a guarded transition to the requested body family and optionally resumes the native editor afterward. It requires player.appearance.edit; completion is consumed with takeBodyFamilyTransition.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `gender` | `string` | required | — |
| `resumeEditor` | `boolean` | optional | — |

```lua
-- The player chose the other body family in the creator; resume the editor after
Open77.appearance.switchBodyFamily(true)
```

Registered in `ResourceHost.cpp` (line 24821) as `LuaAppearanceSwitchBodyFamily`.

## takeBodyFamilyTransition

```lua
Open77.appearance.takeBodyFamilyTransition()
```

`GAME`: Requires a live game instance.

Consumes the pending body-family transition result.

Consumes the pending body-family transition result once, returning its serialized payload or nil while no result is ready. It requires player.appearance.edit.

Returns: `value`, `reason (when the first is nil)`

```lua
-- Poll for the transition result once it is ready
CreateThread(function()
    while true do
        local result = Open77.appearance.takeBodyFamilyTransition()
        if result then print("transition finished"); break end
        Wait(250)
    end
end)
```

Registered in `ResourceHost.cpp` (line 24822) as `LuaAppearanceTakeBodyFamilyTransition`.
