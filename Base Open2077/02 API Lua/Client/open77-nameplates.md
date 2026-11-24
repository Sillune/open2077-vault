# Open77.nameplates (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-nameplates.md) · [Page web](https://open2077.net/docs/api/client/open77-nameplates)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

10 functions.

## clear

```lua
Open77.nameplates.clear()
```

`GAME`: Requires a live game instance.

Removes every nameplate override and talking assertion owned by the calling resource.

Requires `ui.nameplates`; other owners are unaffected. Only the per-player state goes: this resource's `style`, `stream` and `render(true)` stay until it stops. Clients released before the native plate themes also switched off the resource's own `stream` and `render` here, so a resource that must run on them removes its overrides one by one with `remove` instead.

Returns: `true on success`

```lua
-- Leaving the arena: drop every plate override this resource set
AddEventHandler("arena:left", function()
    Open77.nameplates.clear()
end)
```

Registered in `ResourceHost.cpp` (line 28132) as `LuaNameplateClear`.

## remove

```lua
Open77.nameplates.remove(playerId)
```

`GAME`: Requires a live game instance.

Removes the calling resource's nameplate override for one player.

Requires `ui.nameplates`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- The player left the team: back to the default plate
RegisterNetEvent("team:left", function(playerId)
    Open77.nameplates.remove(playerId)
end)
```

Registered in `ResourceHost.cpp` (line 28131) as `LuaNameplateRemove`.

## render

```lua
Open77.nameplates.render(enabled)
```

`GAME`: Requires a live game instance.

Draws remote-player nameplates natively, with no WebUI surface at all.

Requires `ui.nameplates`. `stream` had already taken Lua off the per-frame path, and plates still trailed the world: what was left is the CEF chain itself -- IPC, a page repaint on the page's own timer, an offscreen raster, a shared-texture handoff, then the composite -- and every stage buffers. `render(true)` draws the plate in the frame that presents it, from a projection taken on that same frame, so there is nothing left to buffer and no page to create.

Guard the call rather than invoking it bare: a client older than the native drawer has no `render` at all, and indexing nil takes the whole resource down instead of falling back. `Open77.nameplates.render ~= nil and Open77.nameplates.render(true)` is the shipped form, with `stream(page)` and then a `Wait(33)` poll loop as the two fallbacks.

`render(false)` stops it. The drawer is released on stop, reload and world exit anyway, but asking explicitly is what keeps a reload from leaving one frame painted.

The drawer draws the server's theme (`Open77.nameplates.style`), places each plate between game ticks so it stays on the body during a camera turn, scales it with distance (`nearScale`, `farScale`) and fades it near the edge of its range (`fadeFrom`). It shows only players the camera can see: every plate is line-of-sight tested against walls, floors, closed doors and the ground, with the hysteresis described under `style`. A plate shows talking when a resource asserted `setTalking` or the voice engine plays the player above a small threshold, and the voice bars follow his level. Every `set()` option applies here. `showDistance` and the theme's drawing keys (fonts, chip, pin, voice indicator) only apply on this path; a page fed by `stream` receives `sublabel`, `accent`, `bar` and `level` as data and draws them itself.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true when the plugin is drawing them, otherwise false`, `reason`

```lua
-- Let the plugin draw the plates natively (guarded for older clients)
if Open77.nameplates.render then
    Open77.nameplates.render(true)
end
```

Registered in `ResourceHost.cpp` (line 28136) as `LuaNameplateRender`.

## resetStyle

```lua
Open77.nameplates.resetStyle()
```

`GAME`: Requires a live game instance.

Drops the calling resource's nameplate style.

Requires `ui.nameplates`. The look returns to the style of the most recent other resource that set one, or to the defaults when none did. Resetting a style that was never set succeeds. A stopped or reloaded resource's style is dropped on its own, so this is for a resource that wants the usual look back while it keeps running.

Returns: `true on success, otherwise false`, `reason`

```lua
-- The neon night is over: give the plates back the server's usual look
AddEventHandler("neonNight:ended", function()
    Open77.nameplates.resetStyle()
end)
```

Registered in `ResourceHost.cpp` (line 28139) as `LuaNameplateResetStyle`.

## set

```lua
Open77.nameplates.set(playerId, options)
```

`GAME`: Requires a live game instance.

Creates or updates the calling resource's nameplate override for a remote player.

Requires `ui.nameplates`. Options:

- `label`: replaces the player's name;
- `sublabel`: a second line under it (a job, a squad, a status);
- `color`: the label colour;
- `accent`: this plate's accent -- the side bar, the border, the pin, the voice bars and the bar fill follow it unless the theme pins them;
- `bar`: a health or progress bar, 0..1 (above 1 is clamped); `false` or absent draws none;
- `barColor`: the bar fill;
- `maxDistance`: range in metres, 1-250;
- `visible`: `false` withholds the plate;
- `showDistance`: adds the distance to the second line on the native drawer;
- `throughWalls`: exempts this player from the line-of-sight test, so his plate shows through walls.

An option you omit follows the server's theme (`Open77.nameplates.style`), `color` and `maxDistance` included: an override that only renames a player keeps the server's palette and range instead of freezing one into his plate. `label` and `sublabel` are at most 96 bytes with no control characters, and colours are `#RRGGBB` or `#RRGGBBAA`; anything else answers `invalid_argument`. A string option may be `nil` or `false` for unset; any other type, a `maxDistance` or `bar` that is not a number, or `options` that is not a table answers `false` and `invalid_argument:<key>` (`invalid_argument` for the table) instead of raising. `inf` and `nan` are refused.

`throughWalls` is for squadmates and objectives, never a default: on any other player it is a wallhack.

Each call replaces this resource's whole override for that player, so restate every option you want to keep. A player another resource already overrides answers `owned_by_another_resource`. Overrides are owner-scoped and cleaned up with the resource.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `options` | `table` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Squadmates: green accent, their role on a second line, found through walls
RegisterNetEvent("squad:member", function(playerId, name, role)
    local ok, reason = Open77.nameplates.set(playerId, {
        label = name,
        sublabel = role,
        accent = "#3DDC84",
        maxDistance = 150,
        throughWalls = true,
    })
    if not ok then
        print("squad plate refused: " .. tostring(reason))
    end
end)
```

Registered in `ResourceHost.cpp` (line 28130) as `LuaNameplateSet`.

## setTalking

```lua
Open77.nameplates.setTalking(playerId, talking)
```

`GAME`: Requires a live game instance.

Shows or clears the native speaker icon on one remote player's nameplate.

Requires `ui.nameplates`. Talking assertions are owner-scoped and automatically released on stop; several voice/HUD resources may assert the same player without racing. The integrated voice package drives this from authenticated native talker state.

The plate also reads the voice engine itself while a voice session is up: it shows talking when any resource asserted it **or** the player is audible above a small threshold, and the voice bars follow his smoothed level either way. An assertion therefore turns the indicator on; clearing it does not silence a player the voice engine still hears.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `talking` | `boolean` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Show the speaker icon while the voice package reports the player talking
for _, talker in ipairs(Open77.voice.talkers() or {}) do
    Open77.nameplates.setTalking(talker.playerId, talker.speaking == true)
end
```

Registered in `ResourceHost.cpp` (line 28137) as `LuaNameplateTalking`.

## snapshot

```lua
Open77.nameplates.snapshot()
```

`GAME`: Requires a live game instance.

Returns the currently rendered remote-player nameplates.

Requires `ui.nameplates`. Every plate the camera can see this frame, from the same occlusion-tested pass the native drawer and `stream(page)` use: a player behind a wall, a floor or the ground is not returned once his plate has faded out. Screen coordinates and distance describe the most recent presentation snapshot.

Each entry carries `id` (a string), `label`, `sublabel`, `color` (the resolved label colour as `#RRGGBBAA`: the override's when one is set, else the theme's), `accent` (resolved, `#RRGGBBAA`), `x` and `y` (0..1 of the viewport, from the top left), `distance` in metres, `talking`, `level` (smoothed voice level, 0..1), `alpha` (the appear and occlusion fade, 0..1), `bar` (only when the override set one) and `occluded` (the latest line-of-sight verdict). A plate can be `occluded` and still have `alpha > 0` for the few frames it takes to fade out.

Returns: `array of { id, label, sublabel, color, accent, x, y, distance, talking, level, alpha, bar?, occluded }, or nil`, `reason`

```lua
-- Draw a health bar under each rendered plate, fading with it
for _, plate in ipairs(Open77.nameplates.snapshot() or {}) do
    healthBars.place(plate.id, plate.x, plate.y + 0.02, plate.alpha)
end
```

Registered in `ResourceHost.cpp` (line 28133) as `LuaNameplateSnapshot`.

## stopStream

```lua
Open77.nameplates.stopStream()
```

`GAME`: Requires a live game instance.

Stops the calling resource's native nameplate stream.

Requires `ui.nameplates`. Stopping a stream that does not exist succeeds, so teardown is idempotent. The stream is also dropped automatically on stop, reload, world exit, and when its page is destroyed.

Returns: `true on success, otherwise false`, `reason`

```lua
-- Stop streaming plates when the HUD page is closed
AddEventHandler("hud:closed", function()
    Open77.nameplates.stopStream()
end)
```

Registered in `ResourceHost.cpp` (line 28135) as `LuaNameplateStopStream`.

## stream

```lua
Open77.nameplates.stream(page)
```

`GAME`: Requires a live game instance.

Streams the native nameplate projection to a page, once per game frame.

Requires `ui.nameplates` and a WebUI page this resource owns. The projection was always native and always correct; relaying it from a Lua timer is what made plates lag and freeze. This publishes the same `nameplates:update` event, and the keys an existing page reads are unchanged -- `{ players = { { id, label, color, x, y, distance, talking } } }` -- so the page needs no edit and the resource can delete its `Wait(33)` loop. An identical frame is not re-sent.

Each player also carries `sublabel`, `accent` (resolved, `#RRGGBBAA`), `level` (smoothed voice level, 0..1), `alpha` (the appear and occlusion fade, 0..1: multiply your own opacity by it) and `bar` (0..1, or `-1` when no `set()` gave one). `color` is always filled, as `#RRGGBBAA`: the override's when one is set, else the theme's label colour. `id` is a string.

The roster is the occlusion-tested one the native drawer uses: a player behind a wall, a floor or the ground leaves the payload once his plate has faded out, unless `set(id, { throughWalls = true })` exempts him or the server's style sets `occlusion = false`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `page` | `WebUI.Page` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Feed the native plate projection to this resource's HUD page every frame
Open77.nameplates.stream(hudPage)
```

Registered in `ResourceHost.cpp` (line 28134) as `LuaNameplateStream`.

## style

```lua
Open77.nameplates.style(theme)
```

`GAME`: Requires a live game instance.

Sets the look of every native nameplate: font, chip, colours, accent, voice indicator, pin, distance scaling and occlusion.

Requires `ui.nameplates`. A web page could always draw any plate a designer wanted, and paid for it twice: the plate trailed the body through the CEF chain, and a page anchor was never occlusion-tested. `style` keeps the drawer native and makes the look data, so a server reproduces its own design without a page. It restyles the plates `render(true)` draws -- the shipped `open77_nameplates` resource turns that on -- for every player at once; per-player details stay in `set()`.

The keys are applied over this resource's previous style, or over the defaults the first time, **all or nothing**: on the first key it cannot apply nothing changes, and the call answers `false` and a reason naming the key -- `unknown_style_key:<key>` for a typo, `invalid_style_value:<key>` for a wrong type, an unparseable colour or an unknown name. `invalid_style` means the argument is not a table or has more than 64 keys, `invalid_style_key` a key that is not a string. Numbers out of range are clamped, not refused; `inf` and `nan` are refused as `invalid_style_value:<key>`.

The most recent resource to call it decides the look of every plate. When that resource stops, the style of the one before it returns; `resetStyle()` hands the look back without stopping.

Colours are `#RRGGBB`, `#RRGGBBAA`, or `false` for transparent. `border`, `borderTalking`, `voiceActive`, `pinColor` and `barColor` follow the plate's accent while unset or set to `"accent"`, which is what lets a per-player `accent` from `set()` recolour one plate without restating the palette. Defaults are the Open77 design system: Chakra Petch in upper case, cyan accent `#22D8E2`, a chamfered chip. Fonts: `chakra`/`ui` and `rajdhani`/`body` are embedded in the plugin; `segoe`/`system` and `bahnschrift`/`condensed` are read from Windows and fall back to the UI face.

Keys -- label: `font`, `size`, `letterSpacing`, `uppercase`, `textColor`, `textShadow`, `sublabelColor`, `sublabelScale`; chip: `shape` (`chamfer`, `rounded`, `pill`, `square`, `none`), `corner`, `paddingX`, `paddingY`, `background`, `backgroundTalking`, `border`, `borderTalking`, `borderWidth`; accent: `accent`, `accentSide` (`left`, `top`, `bottom`, `none`), `accentWidth`; voice: `voice` (`bars`, `speaker`, `none`), `voiceIdle`, `voiceActive`, `voiceIdleVisible`; pin: `pin`, `pinWidth`, `pinColor`, `tip` (`diamond`, `dot`, `none`), `tipSize`, `tipGlow`, `tipPulse`; bar: `barHeight`, `barBackground`, `barColor`; distance: `nearScale`, `farScale`, `fadeFrom`, `fade` (milliseconds), `maxDistance`; behaviour: `occlusion`, `offset`, `showDistance`, `scale`. Types, defaults and ranges are in the *Official resource exports* guide, under `open77_nameplates`.

`maxDistance`, `offset`, `occlusion` and `fade` also shape what `stream(page)` and `snapshot()` return, because they decide where a plate sits and which players are listed.

**Occlusion.** Every plate is line-of-sight tested natively, from the camera to the player's head and then to his chest, against static geometry, doors and terrain -- walls, floors, closed doors, the ground; never vehicles or people -- crossing up to two panes of glass. A plate shows on the first clear test, hides after at least two blocked tests spanning 120 ms or more, never flashes on a player first seen behind a wall, and fades between the two over `fade`. `occlusion = false` turns the test off for the whole server; `set(id, { throughWalls = true })` exempts one player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `theme` | `table` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- A yellow-accent RP look for every plate on this server
local ok, reason = Open77.nameplates.style({
    font = "rajdhani",
    size = 16,
    uppercase = false,
    accent = "#FCEE0A",
    textColor = "#FFF8D6",
    background = "#14120ACC",
    backgroundTalking = "#2A2608E0",
    shape = "rounded",
    corner = 4,
    accentSide = "bottom",
    voice = "speaker",
    tip = "dot",
    maxDistance = 25,
})
if not ok then
    print("nameplate style refused: " .. tostring(reason))
end
```

Registered in `ResourceHost.cpp` (line 28138) as `LuaNameplateStyle`.
