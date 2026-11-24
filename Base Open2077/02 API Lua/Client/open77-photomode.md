# Open77.photoMode (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-photomode.md) · [Page web](https://open2077.net/docs/api/client/open77-photomode)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

4 functions.

## close

```lua
Open77.photoMode.close()
```

`GAME`: Requires a live game instance.

Closes Cyberpunk's native photo mode.

Requests the native menu's confirmed exit and UI cleanup. Idempotent when inactive or already closing; a request during the intro is queued until the menu is ready. Success means accepted, not that the transition has finished: isActive() still reports native state and may briefly remain true. Reopening during an accepted close returns photo_mode_transition_pending. A missing active menu returns photo_mode_menu_unavailable; deploy the matching client REDscripts with the DLL.

Returns: `true on success, otherwise false`, `reason for the refusal`

```lua
-- Leave photo mode when the round starts
RegisterNetEvent("arena:roundStarted", function()
    Open77.photoMode.close()
end)
```

Registered in `ResourceHost.cpp` (line 25373) as `LuaPhotoModeClose`.

## isActive

```lua
Open77.photoMode.isActive()
```

`GAME`: Requires a live game instance.

Reports whether REDengine photo mode is active.

Reads `gamePhotoModeSystem::IsPhotoModeActive`; it reports engine state rather than remembering the last Lua request.

Returns: `boolean, or nil`, `reason when unavailable`

```lua
-- Hide the custom HUD while photo mode is up
SetTick(function()
    hud.visible = not Open77.photoMode.isActive()
end)
```

Registered in `ResourceHost.cpp` (line 25375) as `LuaPhotoModeIsActive`.

## open

```lua
Open77.photoMode.open()
```

`GAME`: Requires a live game instance.

Opens Cyberpunk's native photo mode.

This is the sole supported activation path while Open77 is loaded. The native plugin refuses the stock keyboard/controller activation entry, then this call invokes the retained original REDengine implementation directly. It is idempotent when already active; REDengine can still refuse contexts in which photo mode is unavailable.

Returns: `true on success, otherwise false`, `reason for the refusal`

```lua
-- A key owned by this resource opens photo mode (the stock shortcut is disabled)
RegisterKeyMapping("photo", "Photo mode", "F8", function()
    local ok, reason = Open77.photoMode.open()
    if not ok then Open77.hud.notify("Photo mode unavailable: " .. reason) end
end)
```

Registered in `ResourceHost.cpp` (line 25372) as `LuaPhotoModeOpen`.

## setEnabled

```lua
Open77.photoMode.setEnabled(enabled)
```

`GAME`: Requires a live game instance.

Opens or closes the native photo mode.

Equivalent to `open()` for true and `close()` for false. This controls one client-local engine state and is not a resource-owned lease.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true on success, otherwise false`, `reason for the refusal`

```lua
-- A photo-mode toggle key owned by this resource
RegisterKeyMapping("photo", "Photo mode", "F8", function()
    Open77.photoMode.setEnabled(not Open77.photoMode.isActive())
end)
```

Registered in `ResourceHost.cpp` (line 25374) as `LuaPhotoModeSetEnabled`.
