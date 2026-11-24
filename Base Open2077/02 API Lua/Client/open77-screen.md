# Open77.screen (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-screen.md) · [Page web](https://open2077.net/docs/api/client/open77-screen)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

14 functions.

## cancel

```lua
Open77.screen.cancel(id)
```

`GAME`: Requires a live game instance.

Cancels an owned native transition and restores the image immediately.

Client-only; requires screen.effects. Ownership includes resource generation and host instance. Never clears a native game fade that has taken over. Emits open77:screen:cancelled with reason=requested on success.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |

Returns: `true, or false, reason`

```lua
-- The player pressed ESC: abort the fade and restore the image now
Open77.screen.cancel(fadeId)
```

[Native fades & transitions guide](../../01%20Documentation/screen-transitions.md)

Registered in `ResourceHost.cpp` (line 25327) as `LuaScreenCancel`.

## capture

```lua
Open77.screen.capture([options])
```

`GAME`: Requires a live game instance.

Takes an in-process screenshot of the game's own rendered frame and resolves a promise with the encoded bytes.

Client-only; requires screen.capture. At the next Present the swap chain's back buffer is copied on the game's own direct queue into a readback buffer, fenced, converted and encoded off the game thread, and the promise resolves with { bytes (a binary string), sizeBytes, width, height, regionWidth, regionHeight, format, screenWidth, screenHeight, sourceFormat, includeOverlay, copyMicros, encodeMicros, totalMicros }. Options: format ('jpeg' default, or 'png'), quality (1..100, default 85, JPEG only), region = { x, y, w, h } in pixels (omitted = the whole frame; a region past an edge is region_invalid, never clamped), includeOverlay (default false = the game alone, before Open77's own overlay pass; true = after it, so nameplates, WebUI pages and the console are in the picture), settleFrames (0..30 Presents to skip first), maxBytes (encoded ceiling, default 8 MiB, at most 16 MiB), scale (0.05..1, default 1: the output shrunk to that fraction of the region through the encoder's own Fant scaler; width/height are then the scaled size and regionWidth/regionHeight what was copied). One capture in flight per resource: a second call before the first settles is capture_in_flight. The picture is the swap chain the game presents -- never the desktop, never another window. Reasons: permission_denied:screen.capture, screenshot_unavailable_on_this_host, device_unavailable (no swap chain hooked -- the overlay is not up), format_unsupported (a request for anything but jpeg/png, or a surface this build cannot convert), region_invalid, invalid_quality, invalid_settle_frames, invalid_max_bytes, invalid_scale, invalid_capture_option:<key>, capture_in_flight, capture_limit; the promise rejects with capture_too_large, encode_failed:<stage>, gpu_timeout, cancelled, resource_stopped. See the Screenshots guide.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table` | optional | — |

Returns: `promise, or nil, reason`

```lua
-- A 64x64 corner of the frame as JPEG, awaited inside a thread
CreateThread(function()
    local shot, reason = Open77.screen.capture({ format = "jpeg", quality = 80, region = { x = 0, y = 0, w = 64, h = 64 } })
    if not shot then return print("capture refused: " .. tostring(reason)) end
    local result, why = shot:await()
    if not result then return print("capture failed: " .. tostring(why)) end
    print(("%dx%d %s, %d bytes (copy %d us, encode %d us)"):format(
        result.width, result.height, result.format, result.sizeBytes, result.copyMicros, result.encodeMicros))
end)
```

[Native fades & transitions guide](../../01%20Documentation/screen-transitions.md)

Registered in `ResourceHost.cpp` (line 25335) as `LuaScreenCapture`.

## catalog

```lua
Open77.screen.catalog()
```

`GAME`: Requires a live game instance.

Lists supported native screen transition presets and backend availability.

Client-only; requires screen.effects. Current preset is fade, with backend=native_quest_fade, available, color, maxDurationMs, maxHoldMs and maxTimeoutMs. Availability verifies this executable's guarded native entry, not whether a world is ready or another fade is busy. Glitch remains unsupported.

Returns: `presets, or nil, reason`

```lua
-- Check the fade backend is available before relying on it
local presets = Open77.screen.catalog()
if presets and presets.fade and presets.fade.available then print("fade ready") end
```

[Native fades & transitions guide](../../01%20Documentation/screen-transitions.md)

Registered in `ResourceHost.cpp` (line 25329) as `LuaScreenCatalog`.

## fadeIn

```lua
Open77.screen.fadeIn(id, [options])
```

`GAME`: Requires a live game instance.

Returns the image for this resource's active native fade.

Client-only; requires screen.effects. options.durationMs is 0..10000 (default 500). Returns true and, as a second value, an Open77.Promise that resolves when the image is back. Can reverse an outgoing fade; does not extend the safety deadline. Rejects another owner, stale generation or an already-running return. open77:screen:finished marks the same boundary.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |
| `options` | `table` | optional | — |

Returns: `true, promise, or false, reason`

```lua
-- Bring the image back after the teleport landed
CreateThread(function()
    local ok, back = Open77.screen.fadeIn(fadeId, { durationMs = 800 })
    if back then back:await() end
    Open77.hud.notify("Welcome to the garage")
end)
```

[Native fades & transitions guide](../../01%20Documentation/screen-transitions.md)

Registered in `ResourceHost.cpp` (line 25325) as `LuaScreenFadeIn`.

## fadeOut

```lua
Open77.screen.fadeOut([options])
```

`GAME`: Requires a live game instance.

Applies a native black or color-tinted quest fade (colored rendering is engine-controlled).

Client-only; requires screen.effects. Returns an owned string ID and, as a SECOND value, an Open77.Promise that resolves with the state table when the screen is actually covered and rejects with the transition's reason if it ends first. Options: durationMs (0..10000, default 500), color={r,g,b,a=255}, timeoutMs (1000..60000, default at least 15000). Resource cleanup and a bounded real-time deadline restore the image, and both fail the promise rather than leave a caller waiting. See the Native screen transitions guide.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table` | optional | — |

Returns: `id, promise, or nil, reason`

```lua
-- Fade to black, teleport, fade back
CreateThread(function()
    local id, covered = Open77.screen.fadeOut({ durationMs = 400 })
    if not id then return end
    covered:await()
    Open77.travel.teleportAndSettle({ x = -1442.2, y = 127.4, z = 18.0 }):await()
    Open77.screen.fadeIn(id, { durationMs = 400 })
end)
```

[Native fades & transitions guide](../../01%20Documentation/screen-transitions.md)

Registered in `ResourceHost.cpp` (line 25324) as `LuaScreenFadeOut`.

## isFaded

```lua
Open77.screen.isFaded()
```

`GAME`: Requires a live game instance.

Is the screen covered right now, whoever covered it.

Client-only; requires screen.effects. UNOWNED: true when the engine's own fade manager reports a completed outgoing fade, whether this resource, another resource, the server relay or a vanilla quest/loading fade caused it. Read it before fading out so a spawn or teleport does not stack a second fade on a screen that is already black. Answers nil, reason rather than a confident false when the guarded backend is unavailable or the call is off the game thread.

Returns: `faded, or nil, reason`

```lua
-- Do not stack a second fade on a screen that is already black
if not Open77.screen.isFaded() then
    Open77.screen.fadeOut({ durationMs = 300 })
end
```

[Native fades & transitions guide](../../01%20Documentation/screen-transitions.md)

Registered in `ResourceHost.cpp` (line 25330) as `LuaScreenIsFaded`.

## isLoading

```lua
Open77.screen.isLoading()
```

`GAME`: Requires a live game instance.

Whether the native loading-screen lifecycle is active.

Client-only; requires screen.read. Observes native opening through delayed hide, including long-distance teleport loads. Returns nil, reason until the guarded observer is available. This is separate from quest fades, background streaming and the Open77 connection cover. See loading-screens.md for events and a complete WebUI example.

Returns: `active, or nil, reason`

```lua
local active, err = Open77.screen.isLoading()
if active ~= nil then print('Native loading:', active) else print(err) end
```

[Custom loading screens guide](../../01%20Documentation/loading-screens.md)

Registered in `ResourceHost.cpp` (line 25418) as `LuaScreenIsLoading`.

## loadingState

```lua
Open77.screen.loadingState()
```

`GAME`: Requires a live game instance.

Read native loading state, cycle ID, kind and real progress.

Client-only; requires screen.read. Returns active, id (opaque string), revision, kind, progressKnown, progress (nil until reported) and elapsedMs. Subscribe to open77:loadingScreen:started/changed/finished; each event receives one snapshot. Register handlers then query initial state to handle a resource starting mid-load. Progress 100% never implies native hide. The state persists across resource reloads, and network messages cannot forge these events. See loading-screens.md.

Returns: `state, or nil, reason`

```lua
local function apply(s)
    if s then print('Loading:', s.active, s.kind, s.progress) end
end
AddEventHandler('open77:loadingScreen:changed', apply)
apply(Open77.screen.loadingState())
```

[Custom loading screens guide](../../01%20Documentation/loading-screens.md)

Registered in `ResourceHost.cpp` (line 25419) as `LuaScreenLoadingState`.

## mugshot

```lua
Open77.screen.mugshot([target], [options])
```

`GAME`: Requires a live game instance.

A head-and-shoulders picture of a body: a scripted camera cut in front of the face, one frame captured through it, the view handed straight back.

Client-only; requires screen.capture, camera.script and world.transform. target is an Open77 entity id (0 or nil = the local player) or a { engineEntity = ... } table -- exactly what Open77.character.frame accepts. Composed in Lua from the public calls: the head is the body's own head slot (Open77.character.bonePosition) or headHeight up its up axis; a scripted camera is created distance metres ahead of the face looking at it with fov, activated with blendMs = 0, a frame is captured through it after settleFrames Presents, then the camera is deactivated and destroyed whatever happened. Options: format, quality, region, includeOverlay, settleFrames (default 3), maxBytes, scale (as capture -- scale = 0.25 makes a ~4 KiB thumbnail the server's own route can take), crop (0..1, default 0.5: the side of the centred square crop as a fraction of the shorter screen edge, used when no region is given), fov (default 35), distance (default 0.7 m), headHeight (default 1.62 m). Resolves with the capture table plus camera = { position, lookAt, fov }. Refuses with camera_busy while anybody holds the view or a blend is running (it never steals), target_not_streamed for a body this client cannot see, not_incarnated before the player has a body, and everything capture and Open77.camera.create refuse. What the picture shows depends on the body the engine renders through that camera: another player's puppet and an NPC are full bodies; the local player's own head is whatever body the perspective arbiter leaves in place while a scripted camera holds the view. See the Screenshots guide.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `integer|table` | optional | — |
| `options` | `table` | optional | — |

Returns: `promise, or nil, reason`

```lua
-- An MDT mugshot of the player you are looking at. Another player's puppet and
-- an NPC are full bodies; the LOCAL player in first person has no body to
-- photograph, so target somebody else or go third person before shooting one.
CreateThread(function()
    local aimed = Open77.character.aimedEntity()
    if not aimed or aimed.entity == 0 then return print("aim at a player first") end
    local shot, reason = Open77.screen.mugshot(aimed.entity, { fov = 32, crop = 0.4, quality = 90 })
    if not shot then return print("mugshot refused: " .. tostring(reason)) end
    local picture, why = shot:await()
    if picture then print(("mugshot %dx%d, %d bytes"):format(picture.width, picture.height, picture.sizeBytes)) end
end)
```

[Native fades & transitions guide](../../01%20Documentation/screen-transitions.md)

Registered in `ResourceHost.cpp` (line 23378) as `LuaPrelude`.

## nativeState

```lua
Open77.screen.nativeState()
```

`GAME`: Requires a live game instance.

The engine's own fade state, unowned.

Client-only; requires screen.effects. Returns faded, fading, out, busy, ready and remainingMs read straight from the native fade manager with no owner filter -- the detailed form of isFaded(). fading separates 'on its way to black' from 'already black'. Engine truth, not an estimate, and not a GPU presentation fence.

Returns: `state, or nil, reason`

```lua
-- Wait until the engine reports the screen fully black
CreateThread(function()
    repeat Wait(50) until (Open77.screen.nativeState() or {}).faded
    print("black now")
end)
```

[Native fades & transitions guide](../../01%20Documentation/screen-transitions.md)

Registered in `ResourceHost.cpp` (line 25331) as `LuaScreenNativeState`.

## state

```lua
Open77.screen.state(id)
```

`GAME`: Requires a live game instance.

Reads this resource generation's active or recent native transition.

Client-only; requires screen.effects. OWNER-SCOPED: it answers for this resource generation's own transition and nothing else -- isFaded() and nativeState() read a fade someone else owns. Returns id, generation, phase, reason, backend, timing, automatic, elapsedMs, remainingMs, timeoutMs, black and color. remainingMs is the native fade timer, not hold time; native-state completion is not a GPU fence or streaming-complete signal. History is bounded to 64 global entries and removed on resource release.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |

Returns: `state, or nil, reason`

```lua
-- Is our own fade still running?
local state = Open77.screen.state(fadeId)
if state then print(state.phase, state.remainingMs) end
```

[Native fades & transitions guide](../../01%20Documentation/screen-transitions.md)

Registered in `ResourceHost.cpp` (line 25328) as `LuaScreenState`.

## surface

```lua
Open77.screen.surface()
```

`GAME`: Requires a live game instance.

What the last Present said about the back buffer: size, DXGI format and whether captures are possible.

Client-only; requires screen.capture. Returns { ready, width, height, format, pixel, supported }. ready is false before any swap chain has been hooked (the overlay is not up, or it was just torn down for a resize) -- an honest 'not yet', not a failure. format is the DXGI name the swap chain actually presents (R8G8B8A8_UNORM, B8G8R8A8_UNORM, R10G10B10A2_UNORM, R16G16B16A16_FLOAT ...), read from the buffer and never assumed; pixel is the conversion family (rgba8, bgra8, rgb10a2, rgba16f, unsupported); supported false means every capture will refuse with format_unsupported. Use it to size a region before capturing. Reasons: permission_denied:screen.capture, screenshot_unavailable_on_this_host.

Returns: `surface, or nil, reason`

```lua
-- Capture the centre quarter of the frame, whatever the resolution
local surface = Open77.screen.surface()
if surface and surface.ready then
    local w, h = surface.width // 2, surface.height // 2
    Open77.screen.capture({ region = { x = w // 2, y = h // 2, w = w, h = h } })
end
```

[Native fades & transitions guide](../../01%20Documentation/screen-transitions.md)

Registered in `ResourceHost.cpp` (line 25337) as `LuaScreenSurface`.

## transition

```lua
Open77.screen.transition(preset, [options])
```

`GAME`: Requires a live game instance.

Runs a native fade, bounded hold and return sequence.

Client-only; requires screen.effects. Supported preset: fade. Returns an owned string ID and, as a second value, an Open77.Promise that resolves when the whole sequence has finished. Options: durationMs=500, holdMs=250, fadeInMs=500, timeoutMs and opaque RGB color. Durations are integer milliseconds; hold is 0..30000, fades 0..10000. Only one active owner; vanilla requests retain priority. No teleport or network mutation. Glitch is not yet a supported preset.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `preset` | `string` | required | — |
| `options` | `table` | optional | — |

Returns: `id, promise, or nil, reason`

```lua
-- A complete fade-hold-return in one call, for a scene change
CreateThread(function()
    local id, done = Open77.screen.transition("fade", { durationMs = 500, holdMs = 1000, fadeInMs = 500 })
    if done then done:await() end
end)
```

[Native fades & transitions guide](../../01%20Documentation/screen-transitions.md)

Registered in `ResourceHost.cpp` (line 25326) as `LuaScreenTransition`.

## upload

```lua
Open77.screen.upload(capture, url, [options])
```

`NETWORK`: Uses the network backend.

POSTs a capture (or raw image bytes) to a URL and resolves a promise with the HTTP receipt.

Client-only; requires screen.upload. The first argument is the table a capture resolved with (its bytes, format, width and height travel along) or a bare string of image bytes. url must be an absolute http:// or https:// URL with no credentials in it; redirects are never followed, so the host named is the only host reached. Options: encoding ('multipart' default -- one file part plus any fields, the shape screenshot-basic targets expect; or 'json' -- an application/json envelope { format, width, height, sizeBytes, ...fields, data = base64 }, for a target that cannot take a binary body such as the server's own Open77.http.listen route), field (multipart file field, default 'files[]'), filename, headers (up to 16, no CR/LF; Content-Type, Content-Length and Host are the client's), fields (up to 16 extra text fields), timeoutMs (1000..60000, default 15000). The promise resolves with { status, body (the first 64 KiB of the answer), bodyTruncated, sentBytes, elapsedMs } and rejects with request_failed:<stage>:<winhttp code>, timeout or cancelled. Two uploads in flight per resource (upload_in_flight), eight per client (upload_limit); at most 16 MiB per upload (bytes_too_large). Reasons: permission_denied:screen.upload, screenshot_unavailable_on_this_host, bytes_empty, bytes_too_large, invalid_url, invalid_upload_option:<key>, invalid_header, invalid_field, too_many_headers, too_many_fields, upload_in_flight, upload_limit. See the Screenshots guide.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `capture` | `table|string` | required | — |
| `url` | `string` | required | — |
| `options` | `table` | optional | — |

Returns: `promise, or nil, reason`

```lua
-- Report evidence: capture the frame and post it to the server's own route
CreateThread(function()
    local shot = Open77.screen.capture({ region = { x = 0, y = 0, w = 320, h = 180 }, quality = 70 })
    local picture = shot and shot:await()
    if not picture then return end
    local upload = Open77.screen.upload(picture, "https://rp.example.net/mdt/evidence", {
        fields = { reportId = "4711" }, headers = { Authorization = "Bearer token" },
    })
    local receipt, why = upload and upload:await()
    print(receipt and ("uploaded: HTTP %d"):format(receipt.status) or ("upload failed: " .. tostring(why)))
end)
```

[Native fades & transitions guide](../../01%20Documentation/screen-transitions.md)

Registered in `ResourceHost.cpp` (line 25336) as `LuaScreenUpload`.
