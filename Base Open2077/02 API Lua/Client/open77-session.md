# Open77.session (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-session.md) · [Page web](https://open2077.net/docs/api/client/open77-session)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

18 functions.

## characterBootstrap

```lua
Open77.session.characterBootstrap()
```

`GAME`: Requires a live game instance.

Reads the current character-bootstrap state.

Returns the current client-local character bootstrap projection as phase, body family, error and generation. The table is a snapshot; polling it does not advance the transaction.

Returns: `value`, `reason (when the first is nil)`

```lua
-- A custom character-creation flow watches the bootstrap phase
local bootstrap = Open77.session.characterBootstrap()
if bootstrap then print("bootstrap phase:", bootstrap.phase, "family:", bootstrap.bodyFamily) end
```

Registered in `ResourceHost.cpp` (line 24737) as `LuaSessionCharacterBootstrap`.

## failCharacterBootstrap

```lua
Open77.session.failCharacterBootstrap([reason])
```

`GAME`: Requires a live game instance.

Marks character bootstrap as failed.

Marks the active client-local character bootstrap as failed with an optional bounded diagnostic reason. It requires player.appearance.edit and does not disconnect the network session.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `reason` | `string` | optional | `"character_bootstrap_failed"` |

```lua
-- The creator was cancelled: mark the bootstrap as failed with a reason
Open77.session.failCharacterBootstrap("player_cancelled")
```

Registered in `ResourceHost.cpp` (line 24741) as `LuaSessionFailCharacterBootstrap`.

## isMenuOpen

```lua
Open77.session.isMenuOpen()
```

`GAME`: Requires a live game instance.

Whether any modal layer -- Open77's or the game's -- owns the screen.

Requires `session.menus`. True when a gameplay loop should stop reading input: an Open77 WebUI surface holding the keyboard, any vanilla menu scenario, photo mode, the appearance mirror, the pre-game character creator, or the Open77 developer console. It is the same disjunction the window procedure already uses to route the escape key, extended by photo mode and the console, so the read cannot disagree with the behaviour the player sees. **What it cannot see**: the vanilla half is fed by the gameplay idle scenario's wrappers, so a screen that does not pass through that scenario -- radial and hub wheels, pre-game menus other than the creator -- never reports itself. A resource that needs certainty about its own modal should still track its own and compare `menuState().overlayResource`. Pairs with the `open77:menuStateChanged` event, which is raised from the same snapshot.

Returns: `boolean, or nil, reason`

```lua
CreateThread(function()
  while true do
    Wait(0)
    if not Open77.session.isMenuOpen() then pollGameplayInput() end
  end
end)
```

Registered in `ResourceHost.cpp` (line 24750) as `LuaSessionIsMenuOpen`.

## isPauseMenuOpen

```lua
Open77.session.isPauseMenuOpen()
```

`GAME`: Requires a live game instance.

Whether the VANILLA pause menu is on screen.

Requires `session.menus`. The FiveM `IsPauseMenuActive` equivalent, and it means the pause menu specifically -- not the inventory, the map, the journal or the death screen, which `isMenuOpen()` covers. Read from the scenario name the script bridge already carries (`MenuScenario_PauseMenu`). On a server whose own pause resource has taken over, the vanilla menu never opens and this stays false while the Open77 overlay is up; `isMenuOpen()` is the question that covers both, and `menuState().overlayResource` names which resource has the screen.

Returns: `boolean, or nil, reason`

```lua
-- Pause the custom HUD timer while the vanilla pause menu is up
SetTick(function()
    if not Open77.session.isPauseMenuOpen() then timer = timer + 1 end
end)
```

Registered in `ResourceHost.cpp` (line 24749) as `LuaSessionIsPauseMenuOpen`.

## launcherContext

```lua
Open77.session.launcherContext()
```

`GAME`: Requires a live game instance.

What the launcher told this game process at boot.

Returns a table { connectEndpoint, master, serverId, modsDigest, launched } read once from the environment the OPEN//77 launcher sets when it starts the game (OP77_CONNECT, OP77_CONNECT_MASTER, OP77_WORLD_SERVER, OP77_WORLD_MODS_DIGEST), every value validated against a strict alphabet. All strings are empty and `launched` is false when the game was started without the launcher (Steam, GOG, a shortcut): that is a state, not an error. `modsDigest` is the SHA-256 of the required-mod set the launcher installed for the world it booted into, the same digest the master relays as `requiredModsDigest` in the catalogue, so the shell can decide whether this boot can join a world without a relaunch. Returns nil plus a reason only when no game backend is attached.

Returns: `table, or nil`, `reason on failure`

```lua
-- Was the game started by the launcher, and for which server?
local ctx = Open77.session.launcherContext()
if ctx and ctx.launched then print("launched for server", ctx.serverId) end
```

Registered in `ResourceHost.cpp` (line 24745) as `LuaSessionLauncherContext`.

## loadPristine

```lua
Open77.session.loadPristine()
```

`GAME`: Requires a live game instance.

Loads the local development world from Open77's bundled pristine save.

No network session is armed by this operation.

Returns: `true on success, otherwise false`, `reason`

```lua
-- A development helper: load the bundled pristine world without connecting
RegisterCommand("devworld", function()
    Open77.session.loadPristine()
end, false)
```

Registered in `ResourceHost.cpp` (line 24735) as `LuaSessionLoadPristine`.

## loadScreen

```lua
Open77.session.loadScreen()
```

`SHARED`: Available without a live game instance.

Describes the connected server's declared loading screen.

Returns availability and the server name so the trusted shell can choose the verified server page or its built-in fallback. This function does not create the WebUI surface; the shell uses `WebUI.create({ loadScreen = true })` for that.

Returns: `table { available, server? }`

```lua
-- The trusted shell decides between the server's page and the fallback
local screen = Open77.session.loadScreen()
if screen.available then print("loading screen from", screen.server) end
```

Registered in `ResourceHost.cpp` (line 24738) as `LuaSessionLoadScreen`.

## locale

```lua
Open77.session.locale()
```

`GAME`: Requires a live game instance.

The language the game runs in, as the engine's own settings store it.

Requires `session.locale`. Returns `{ code, gameLanguage, voiceLanguage, subtitles, chromium, revision }`. `code` is a BCP-47 tag derived from the engine's `/language OnScreen` value (`en-US`, `fr-FR`, `pt-BR`, `zh-CN`); `gameLanguage`, `voiceLanguage` and `subtitles` are the three raw engine codes exactly as the settings store them (`en-us`, `jp-jp`, `kr-kr`) -- the same values the vanilla settings screen compares against -- so a resource that needs the audio language, or meets a code the mapping has not seen, always has the source. `chromium` is the operating system's UI language, what a WebUI page's `navigator.language` (the JS `Open77.getLocale()`) has always answered: it is not the game's language and is kept for contrast only. `nil, locale_not_reported` until the script bridge has answered once -- it is asked at every world-ready and again, rate-limited, on every call here, so a language changed at the main menu between two sessions re-reports without a restart (the engine only allows changing it pre-game). The same value is pushed to the server once per world-ready for `Open77.players.locale`.

Returns: `table, or nil`, `reason: permission_denied:session.locale, locale_not_reported, game_unavailable_on_this_host`

```lua
-- Pick the greeting file for the player's own language
local locale = Open77.session.locale()
local language = locale and locale.code:sub(1, 2) or "en"
local greetings = { fr = "Bienvenue", de = "Willkommen", en = "Welcome" }
print(greetings[language] or greetings.en)
```

Registered in `ResourceHost.cpp` (line 24753) as `LuaSessionLocale`.

## menuState

```lua
Open77.session.menuState()
```

`GAME`: Requires a live game instance.

Every modal layer at once, sampled in one instant.

Requires `session.menus`. Returns `open`, `pauseMenu`, `vanillaMenu`, `scenario`, `overlay`, `overlayResource`, `photoMode`, `appearanceEditor`, `characterCreator`, `console` and `source`. One call rather than several because the answer a script wants is the disjunction, and a caller assembling it from separate reads would sample them at different instants and could see it half-true. `source` names the highest-priority layer in the same order the escape key is routed: `overlay`, `creator`, `vanilla`, `photoMode`, `appearance`, `console`, `none`. `scenario` is the vanilla scenario name, empty during gameplay. This is also the payload of `open77:menuStateChanged(open, pauseMenu, source, scenario)`, deliberately: the event and the read describe the same snapshot. The event is an EDGE and is not synthesised at startup -- read this once when your resource starts.

Returns: `table, or nil, reason`

```lua
AddEventHandler("open77:menuStateChanged", function(open, pause, source)
    if open == "1" then pauseLoop(source) else resumeLoop() end
end)
local now = Open77.session.menuState()
if now and now.open then pauseLoop(now.source) end
```

Registered in `ResourceHost.cpp` (line 24751) as `LuaSessionMenuState`.

## openLauncher

```lua
Open77.session.openLauncher()
```

`GAME`: Requires a live game instance.

Returns server selection to the desktop launcher.

Client-only; requires network.client. Requests the installed launcher's opening through the native backend. Returns false and a reason when the launcher cannot be opened or this host has no game backend. Use the connection workspace's normal return-to-launcher flow when also closing the game.

Returns: `boolean success`, `reason on failure`

Registered in `ResourceHost.cpp` (line 24746) as `LuaSessionOpenLauncher`.

## openSettings

```lua
Open77.session.openSettings()
```

`GAME`: Requires a live game instance.

Opens the game's own settings screen without exposing the pause menu.

Requests the client-side Open77 settings interface. It only changes local presentation and does not modify server configuration or another player's settings.

Returns: `true on success, otherwise false`, `reason`

```lua
-- A "Settings" entry in a custom pause menu
RegisterNetEvent("pause:settings", function()
    Open77.session.openSettings()
end)
```

Registered in `ResourceHost.cpp` (line 24734) as `LuaSessionOpenSettings`.

## quitGame

```lua
Open77.session.quitGame()
```

`GAME`: Requires a live game instance.

Exits Cyberpunk 2077 to the desktop.

The vanilla exit path, reached through the script bridge rather than by terminating the process: the request is ENQUEUED and runs inside the script frame the bridge loop already owns, because a native call into script without a frame kills the process outright. Fails with `script_bridge_not_attached` before the bridge is up, `queue_full` if the bridge queue is saturated, and `game_unavailable_on_this_host` on a host with no game backend. Intended for the trusted pause surface -- `open77_pause` is the only caller -- and not for gameplay code: quitting a session is `Open77.network.disconnect`, which walks the engine back to the main menu instead of closing the game.

Returns: `true on success, otherwise false`, `reason`

```lua
-- The "Quit to desktop" entry of a custom pause surface
local ok, reason = Open77.session.quitGame()
if not ok then print("quit refused: " .. reason) end
```

Registered in `ResourceHost.cpp` (line 24744) as `LuaSessionQuitGame`.

## relaunchViaLauncher

```lua
Open77.session.relaunchViaLauncher(serverId, endpoint)
```

`GAME`: Requires a live game instance.

Hands the player to the launcher for a world this boot cannot serve.

Opens `open77://connect?server=<serverId>&endpoint=<host:port>&relaunch=<pid>` through the OS protocol handler the OPEN//77 launcher registers. The launcher waits for this process to exit, installs the world's required mods into its profile and starts the game again with an auto-connect to the endpoint. Nothing here terminates the game: the caller quits (Open77.session.quitGame) once this returned true, so a missing launcher is reported instead of leaving the player at the desktop. Both arguments are validated (a GUID-shaped server id, a host:port endpoint) and only that one scheme and query shape can ever be launched, so a resource cannot turn this into a generic open-URL or run-program primitive. Returns true, or false plus a reason: invalid_server_id, invalid_endpoint, launcher_not_registered (no handler for the open77 scheme on this machine), launcher_launch_failed:<code>.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `serverId` | `string` | required | — |
| `endpoint` | `string` | required | — |

Returns: `true, otherwise false`, `reason on failure`

```lua
-- This boot lacks the world's mods: hand over to the launcher, then quit
if Open77.session.relaunchViaLauncher(serverId, "play.example.net:27015") then
    Open77.session.quitGame()
end
```

Registered in `ResourceHost.cpp` (line 24747) as `LuaSessionRelaunchViaLauncher`.

## requestCharacterCreator

```lua
Open77.session.requestCharacterCreator()
```

`GAME`: Requires a live game instance.

Requests the native character creator.

Requests the native character creator for the active bootstrap transaction. It requires player.appearance.edit and returns a stable failure when the game backend cannot accept the request.

```lua
-- Open the native character creator for a brand-new character
Open77.session.requestCharacterCreator()
```

Registered in `ResourceHost.cpp` (line 24739) as `LuaSessionRequestCharacterCreator`.

## resetCharacterBootstrap

```lua
Open77.session.resetCharacterBootstrap()
```

`GAME`: Requires a live game instance.

Resets the local character-bootstrap transaction.

Clears the client-local character bootstrap transaction so a fresh bootstrap can begin. It does not delete a server character or change another player's state.

```lua
-- Start the character bootstrap again after a failure
Open77.session.resetCharacterBootstrap()
```

Registered in `ResourceHost.cpp` (line 24736) as `LuaSessionResetCharacterBootstrap`.

## resolveCharacterBootstrap

```lua
Open77.session.resolveCharacterBootstrap(family)
```

`GAME`: Requires a live game instance.

Completes bootstrap with a selected body family.

Completes the active bootstrap with the validated female or male body family. It requires player.appearance.edit and only updates the local bootstrap backend.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `family` | `string` | required | — |

```lua
-- The player picked a body family in a custom menu
Open77.session.resolveCharacterBootstrap("female")
```

Registered in `ResourceHost.cpp` (line 24740) as `LuaSessionResolveCharacterBootstrap`.

## setMenuReady

```lua
Open77.session.setMenuReady(ready)
```

`GAME`: Requires a live game instance.

Declares that this resource is able to serve the pause menu.

The escape key is only intercepted while this is true. A resource that fails to start therefore leaves the game's own menu reachable, rather than stranding the player with none.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `ready` | `boolean` | required | — |

Returns: `boolean`

```lua
-- Only intercept ESC once the pause page is really up
AddEventHandler("pause:pageReady", function()
    Open77.session.setMenuReady(true)
end)
```

Registered in `ResourceHost.cpp` (line 24743) as `LuaSessionMenuReady`.

## takeCharacterCreatorResult

```lua
Open77.session.takeCharacterCreatorResult()
```

`GAME`: Requires a live game instance.

Consumes the pending character-creator result.

Consumes the pending native character-creator result once and returns its serialized payload, or nil when no result is ready. It requires player.appearance.edit.

Returns: `value`, `reason (when the first is nil)`

```lua
-- Poll for the creator's result once it closes
CreateThread(function()
    while true do
        local result = Open77.session.takeCharacterCreatorResult()
        if result then TriggerServerEvent("character:created", result); break end
        Wait(500)
    end
end)
```

Registered in `ResourceHost.cpp` (line 24742) as `LuaSessionTakeCharacterCreatorResult`.
