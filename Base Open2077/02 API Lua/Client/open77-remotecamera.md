# Open77.remoteCamera (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-remotecamera.md) · [Page web](https://open2077.net/docs/api/client/open77-remotecamera)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

15 functions.

## bindHud

```lua
Open77.remoteCamera.bindHud(cameraId, rect)
```

`GAME`: Requires a live game instance.

Shows one source in a native HUD rectangle and returns the binding id.

Client-only; requires `camera.capture`. `rect` is `{x, y, width, height}` in viewport pixels -- a screen-space viewport, not a DOM element, so it does not track page layout. The returned binding id is opaque and allocated by the host for this resource generation. Multiple bindings may share one source, and capture is demanded only while at least one binding is visible; hiding the last visible binding stops the rendering without destroying the source. The id ends at `unbind()` or when the source is destroyed.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `cameraId` | `string` | required | — |
| `rect` | `table` | required | — |

Returns: `binding id (opaque string), or nil, reason`

```lua
local hud, reason = Open77.remoteCamera.bindHud(cam, {
    x = 80, y = 100, width = 640, height = 360,
})
if hud then
    -- The source keeps existing; only this presentation goes away
    Open77.remoteCamera.updateBinding(hud, { visible = false })
end
```

[Remote cameras & world screens guide](../../01%20Documentation/remote-camera.md)

Registered in `ResourceHost.cpp` (line 25314) as `LuaRemoteCameraBindHud`.

## bindings

```lua
Open77.remoteCamera.bindings([cameraId])
```

`GAME`: Requires a live game instance.

This resource's bindings, optionally only those of one source.

Client-only; requires `camera.capture`. With no argument every binding this resource owns is listed; passing a camera id filters to that source. Owner-scoped like the rest of the namespace, so it cannot be used to read another resource's presentations. Rows are the same snapshots `getBinding()` returns.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `cameraId` | `string` | optional | — |

Returns: `array of binding snapshots`

```lua
-- One source's presentations
for _, binding in ipairs(Open77.remoteCamera.bindings(cam) or {}) do
    print(binding.id, binding.kind, binding.state)
end
```

[Remote cameras & world screens guide](../../01%20Documentation/remote-camera.md)

Registered in `ResourceHost.cpp` (line 25319) as `LuaRemoteCameraBindings`.

## bindWebUI

```lua
Open77.remoteCamera.bindWebUI(cameraId, page, rect)
```

`GAME`: Requires a live game instance.

Shows one source under a resource-owned transparent page and returns the binding id.

Client-only; requires `camera.capture`, and the WebUI layer keeps its own permission. `page` must belong to the calling resource; another resource's page is refused and the provider never accepts one on a caller's behalf. `rect` is `{x, y, width, height}` in viewport pixels -- the image is drawn by a native underlay behind that transparent region, and the resource still owns the page's layout. The engine's dynamic texture stays behind an Ink image: this is not an HTML video element, an image URL, a JavaScript-readable pixel stream or a transferable GPU texture, and no raw graphics pointer or unsynchronized resource lifetime crosses into Lua or JavaScript.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `cameraId` | `string` | required | — |
| `page` | `userdata` | required | — |
| `rect` | `table` | required | — |

Returns: `binding id (opaque string), or nil, reason`

```lua
local page = WebUI.create({
    entry = "web/monitor.html",
    layer = "hud",
    transparent = true,
    visible = true,
})
if page then
    local underlay = Open77.remoteCamera.bindWebUI(cam, page, {
        x = 80, y = 100, width = 640, height = 360,
    })
    print(underlay)
end
```

[Remote cameras & world screens guide](../../01%20Documentation/remote-camera.md)

Registered in `ResourceHost.cpp` (line 25316) as `LuaRemoteCameraBindWebUI`.

## bindWorld

```lua
Open77.remoteCamera.bindWorld(cameraId, placement)
```

`GAME`: Requires a live game instance.

Shows one source on a depth-tested world panel and returns the binding id.

Client-only; requires `camera.capture`. `placement` is `{position, rotation, width, height, model, parent}` with physical dimensions in metres. `model` defaults to `surface`, a frameless display placed just in front of an existing prop; `panel` uses the stock authored housing. `worldModels()` describes the supported profiles, not installed slot availability. Unsupported models are refused rather than substituted. `parent` is `{type=..., id=...}` and makes position/rotation local to a `prop`, `localProp`, `player`, `vehicle` or `entity`; only canonical prop, player and vehicle parents are accepted over the network. Foreground geometry occludes the display. This is not material substitution, UV reuse or a texture painted onto an arbitrary mesh.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `cameraId` | `string` | required | — |
| `placement` | `table` | required | — |

Returns: `binding id (opaque string), or nil, reason`

```lua
-- Frameless display standing three metres in front of the player
local view = Open77.camera.view()
if view then
    local panel = Open77.remoteCamera.bindWorld(cam, {
        position = { x = view.position.x + 3.0, y = view.position.y, z = view.position.z },
        rotation = { yaw = 90, pitch = 0, roll = 0 },
        width = 2.4,
        height = 1.35,
    })
    print(panel)
end
```

[Remote cameras & world screens guide](../../01%20Documentation/remote-camera.md)

Registered in `ResourceHost.cpp` (line 25315) as `LuaRemoteCameraBindWorld`.

## budget

```lua
Open77.remoteCamera.budget()
```

`GAME`: Requires a live game instance.

The effective admission limits and current usage, read-only.

Client-only; requires `camera.capture`. Reports the global and caller-scoped limits and usage, including allocations that are still retiring, so a caller can decide whether to request another source before it is refused with `camera_slots_exhausted`. There is no public setter: the budget is host/operator policy, and the provider service must not change it either.

Returns: `budget table`

```lua
-- Read-only: the operator owns the ceiling, the caller only reads it
print(Open77.json.encode(Open77.remoteCamera.budget()))
```

[Remote cameras & world screens guide](../../01%20Documentation/remote-camera.md)

Registered in `ResourceHost.cpp` (line 25312) as `LuaRemoteCameraBudget`.

## capabilities

```lua
Open77.remoteCamera.capabilities()
```

`GAME`: Requires a live game instance.

Reports output, presentation support, admission limits and asset requirements.

Client-only; requires `camera.capture`. Reports the fixed output, supported presentation kinds, source and binding limits, expected slot templates, TweakDB availability and world readiness. Storage capacity is not the number of installed slot graphs; asset availability is resolved when a source or presentation is created. Read-only: admission limits are host/operator policy.

Returns: `capabilities table`

```lua
local capabilities = Open77.remoteCamera.capabilities()
print(Open77.json.encode(capabilities))
```

[Remote cameras & world screens guide](../../01%20Documentation/remote-camera.md)

Registered in `ResourceHost.cpp` (line 25311) as `LuaRemoteCameraCapabilities`.

## create

```lua
Open77.remoteCamera.create(view)
```

`GAME`: Requires a live game instance.

Queues one independent capture source and returns its opaque id.

Client-only; requires `camera.capture`. `view` is `{position={x,y,z}, rotation={yaw,pitch,roll}, fov, range}` in world metres and degrees: `position` is required, rotation defaults to zero, FOV to 60 (1..120) and range to 100 (1..500). The id belongs to the calling resource generation; it is neither an engine id nor a server camera id. Admission is asynchronous: `pending` does not mean active capture. Follow `open77:remoteCamera:stateChanged` or read `get()`. Failure is `nil, reason`; `camera_slots_exhausted` means the shared budget is full and `camera_record_unavailable` means the source record could not be prepared. Missing or incompatible packed assets can also fail later component configuration. Nothing renders until a visible binding demands it, and acceptance is not proof of a fresh frame.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `view` | `table` | required | — |

Returns: `camera id (opaque string), or nil, reason`

```lua
-- One source in the loading bay, rendered without taking the player's view
local cam, reason = Open77.remoteCamera.create({
    position = { x = -699.094, y = 2164.29, z = 54.7051 },
    rotation = { yaw = 135, pitch = 0, roll = 0 },
    fov = 60,
    range = 100,
})
if not cam then
    Open77.log.warn("no source: " .. tostring(reason))
else
    Open77.log.info("source " .. cam .. " queued")
end
```

[Remote cameras & world screens guide](../../01%20Documentation/remote-camera.md)

Registered in `ResourceHost.cpp` (line 25306) as `LuaRemoteCameraCreate`.

## destroy

```lua
Open77.remoteCamera.destroy(cameraId)
```

`GAME`: Requires a live game instance.

Retires a source and its bindings.

Client-only; requires `camera.capture`. Retirement is asynchronous: the id goes through `destroying` and ends with a terminal `removed` event, and the slot is not reusable until the native resources and bindings finish cleanup. After the terminal event the id is stale -- `get()` answers `nil, "stale_id"` -- and a caller never held the engine id, so it cannot reach the native object by mistake. Bindings are removed with the source; `unbind()` is how a caller retires one presentation and keeps the source.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `cameraId` | `string` | required | — |

Returns: `true, or false, reason`

```lua
-- Bindings go with the source
local ok, reason = Open77.remoteCamera.destroy(cam)
if not ok then
    Open77.log.warn("destroy refused: " .. tostring(reason))
end
```

[Remote cameras & world screens guide](../../01%20Documentation/remote-camera.md)

Registered in `ResourceHost.cpp` (line 25310) as `LuaRemoteCameraDestroy`.

## get

```lua
Open77.remoteCamera.get(cameraId)
```

`GAME`: Requires a live game instance.

Snapshot of one owned source: requested pose, asynchronous state and demand.

Client-only; requires `camera.capture`. The snapshot carries the requested pose as direct fields (`position` keyed `x`/`y`/`z`, `rotation` keyed `yaw`/`pitch`/`roll`, `fov`, `range`) plus `id`, `state`, `reason`, `width`, `height` and `captureEnabled`. States are `pending`, `active`, `failed` and `destroying`; `active` means the native object is established, not that a GPU frame is fresh and not that the remote world around the camera is complete. A terminal `removed` event invalidates the id, after which this returns `nil, "stale_id"`. Ids are opaque: no raw engine handle is exposed.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `cameraId` | `string` | required | — |

Returns: `source snapshot, or nil, reason`

```lua
-- `pending` is queued, `active` is established -- neither is a fresh frame
local source = Open77.remoteCamera.get(cam)
if source then
    print(source.state, source.width, source.height, source.captureEnabled)
end
```

[Remote cameras & world screens guide](../../01%20Documentation/remote-camera.md)

Registered in `ResourceHost.cpp` (line 25308) as `LuaRemoteCameraGet`.

## getBinding

```lua
Open77.remoteCamera.getBinding(bindingId)
```

`GAME`: Requires a live game instance.

Snapshot of one binding: source, kind, requested options, state and reason.

Client-only; requires `camera.capture`. The snapshot carries `id`, `cameraId`, `kind`, `state`, `reason` and `visible`, with the kind's requested options flattened onto the same row. Binding states are the source states plus `hidden` and `unavailable`; `unavailable` is not proof that the source is gone, only that this presentation cannot be materialised right now. `removed` is terminal and invalidates the id: a later call answers `nil, "stale_binding_id"`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bindingId` | `string` | required | — |

Returns: `binding snapshot, or nil, reason`

```lua
local binding = Open77.remoteCamera.getBinding(panel)
if binding then
    print(binding.kind, binding.state, binding.visible)
end
```

[Remote cameras & world screens guide](../../01%20Documentation/remote-camera.md)

Registered in `ResourceHost.cpp` (line 25318) as `LuaRemoteCameraGetBinding`.

## list

```lua
Open77.remoteCamera.list()
```

`GAME`: Requires a live game instance.

Every capture source this resource owns.

Client-only; requires `camera.capture`. Owner-scoped: another resource's sources are never listed, and neither are native objects the host holds for internal reasons. Rows are the same snapshots `get()` returns, including sources that are still retiring -- a retirement that has not completed keeps counting against the budget until it does.

Returns: `array of source snapshots`

```lua
-- Including sources still retiring
for _, source in ipairs(Open77.remoteCamera.list() or {}) do
    print(source.id, source.state)
end
```

[Remote cameras & world screens guide](../../01%20Documentation/remote-camera.md)

Registered in `ResourceHost.cpp` (line 25309) as `LuaRemoteCameraList`.

## unbind

```lua
Open77.remoteCamera.unbind(bindingId)
```

`GAME`: Requires a live game instance.

Retires one presentation and keeps its source alive.

Client-only; requires `camera.capture`. Retirement is asynchronous: the binding enters `destroying`, then a terminal `removed` event invalidates its id. Its source and other bindings remain; rendering continues only while another binding demands it. Use `destroy()` to retire the source and all its presentations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bindingId` | `string` | required | — |

Returns: `true, or false, reason`

```lua
-- Remove just this monitor; the camera keeps running
local ok, reason = Open77.remoteCamera.unbind(panel)
if not ok then
    Open77.log.warn("unbind refused: " .. tostring(reason))
end
```

[Remote cameras & world screens guide](../../01%20Documentation/remote-camera.md)

Registered in `ResourceHost.cpp` (line 25320) as `LuaRemoteCameraUnbind`.

## update

```lua
Open77.remoteCamera.update(cameraId, patch)
```

`GAME`: Requires a live game instance.

Applies a sparse pose, FOV or range patch to one owned source.

Client-only; requires `camera.capture`. Only the fields the caller names change, and `position`, `rotation`, `fov` and `range` are the same values and limits `create` accepts. The patch is sparse per field but not per axis: a named `position` or `rotation` replaces that value as a whole, so `{ rotation = { yaw = 90 } }` lands with pitch and roll at zero rather than keeping the previous pitch. Pass the full table whenever only one axis should differ. The logical id survives the patch, and so do its bindings. A native source may be rebuilt to apply a patch, and that happens asynchronously, so acceptance is not a smooth per-frame update and not a guarantee of a fresh GPU frame. Mutations return `true` or `false, reason`; a destroyed source answers with a stale-id refusal.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `cameraId` | `string` | required | — |
| `patch` | `table` | required | — |

Returns: `true, or false, reason`

```lua
-- Sparse: the id and every binding survive the patch
local ok, reason = Open77.remoteCamera.update(cam, { fov = 40, range = 60 })
if not ok then
    Open77.log.warn("update refused: " .. tostring(reason))
end
```

[Remote cameras & world screens guide](../../01%20Documentation/remote-camera.md)

Registered in `ResourceHost.cpp` (line 25307) as `LuaRemoteCameraUpdate`.

## updateBinding

```lua
Open77.remoteCamera.updateBinding(bindingId, patch)
```

`GAME`: Requires a live game instance.

Changes what a binding shows or where, without changing its identity.

Client-only; requires `camera.capture`. The patch is sparse and kind-appropriate: `cameraId` selects a source, `visible` sets requested visibility, rectangle fields move/size HUD and WebUI presentations, and `position`/`rotation`/`width`/`height`/`model`/`parent` update world panels. `parent = false` detaches. Position and rotation replace whole vectors: position requires all axes; omitted rotation axes become zero. Kind and WebUI page cannot change. Native changes may be asynchronous. Current limitations: switching an established HUD/WebUI source can leave the previous image mounted; world visibility gates capture demand but does not suppress the mesh's last or shared-source image.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bindingId` | `string` | required | — |
| `patch` | `table` | required | — |

Returns: `true, or false, reason`

```lua
-- Same panel, same id, new place
local ok, reason = Open77.remoteCamera.updateBinding(panel, {
    position = { x = 12.5, y = -4.0, z = 1.2 },
    visible = true,
})
if not ok then
    Open77.log.warn("binding refused: " .. tostring(reason))
end
```

[Remote cameras & world screens guide](../../01%20Documentation/remote-camera.md)

Registered in `ResourceHost.cpp` (line 25317) as `LuaRemoteCameraUpdateBinding`.

## worldModels

```lua
Open77.remoteCamera.worldModels()
```

`GAME`: Requires a live game instance.

Lists supported world-display profiles, dimensions and placement bases.

Client-only; requires `camera.capture`. Rows carry `id`, `defaultWidth`, `defaultHeight`, `frameless`, `sizing` and `basis`. Both built-in profiles default to 1.6 by 0.9 metres. `surface` is a frameless display with `sizing = "display_plane"` and entity-local basis right `+X`, up `+Z`, normal `-Y`, `yawOffset = 0`. `panel` uses the stock housing, `sizing = "mesh_bounds"` and authored mesh-face basis with `yawOffset = 90`; its in-plane axes are not advertised as known. Apply the selected profile's correction once. This catalogue does not scan packed assets: a supported profile may still fail creation if its slot graph is missing or incompatible.

Returns: `array of {id, defaultWidth, defaultHeight, frameless, sizing, basis={space, right, up, normal, yawOffset}}`

```lua
-- `surface` is the frameless default; `panel` keeps the authored housing
for _, model in ipairs(Open77.remoteCamera.worldModels() or {}) do
    print(model.id, model.defaultWidth, model.defaultHeight, model.frameless)
end
```

[Remote cameras & world screens guide](../../01%20Documentation/remote-camera.md)

Registered in `ResourceHost.cpp` (line 25313) as `LuaRemoteCameraWorldModels`.
