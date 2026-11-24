# Open77.camera (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-camera.md) · [Page web](https://open2077.net/docs/api/client/open77-camera)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

29 functions.

## activate

```lua
Open77.camera.activate(camId, [options])
```

`GAME`: Requires a live game instance.

Takes the view with a scripted camera, optionally blending into it.

Client-only; requires `camera.script`. `options.blendMs` (0..60000) interpolates from wherever the view currently is; omit it for a cut. Returns `true` and, as a SECOND value, an `Open77.Promise` that resolves when the blend completes and REJECTS with a reason if the shot is released first -- `player_died`, `resource_stopped`, `camera_superseded` and the rest. A cut settles before the call returns and hands back no promise. Refused with `camera_held_by:<resource>` when another resource has the view (never silently stolen), with `camera_unavailable` when the player is dead or a perspective transition is in flight, and with `invalid_argument` past the 250 m streaming range.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `camId` | `number` | required | — |
| `options` | `table` | optional | — |

Returns: `true and an Open77.Promise, or false, reason`

```lua
-- A one-second blend into a fixed camera looking at the stage
CreateThread(function()
    local cam = Open77.camera.create({ position = vector3(-1442.2, 127.4, 20.0), lookAt = vector3(-1448.2, 96.1, 17.5) })
    local ok, blend = Open77.camera.activate(cam, { blendMs = 1000 })
    if ok and blend then blend:await() end
    Wait(5000)
    Open77.camera.deactivate({ blendMs = 1000 })
end)
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25272) as `LuaCameraActivate`.

## aimRay

```lua
Open77.camera.aimRay([maxDistance], [options])
```

`GAME`: Requires a live game instance.

The ray from the camera along its forward vector: what the reticle points at.

Traces from the camera along its forward vector. Requires `world.query`. Synchronous native raycast on the client game thread. Defaults to Static and Dynamic groups; the nearer hit wins. Stable 156 adds `groups`, 1..16 case-sensitive native names such as Vehicle, Static and Terrain, checked against the engine registry. It replaces static/dynamic flags; combining them fails with `collision_groups_conflict`. Unknown names return `invalid_collision_group`; malformed lists return `invalid_collision_groups`. With `entities = true`, entity identity comes from the hit proxy's actual native owner, never a nearest-object guess. Geometry without an owner has no entity; unavailable owner lookup returns `ray_entity_lookup_unavailable`. The optional entity contains engineEntity, className, kind, position, distance and a canonical playerId/vehicleId/npcId/propId when owned. `ignore` is unsupported and returns `ray_ignore_unavailable`. See [world queries](../../01%20Documentation/world-queries.md#choosing-collision-groups).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `maxDistance` | `number (metres, default 200)` | optional | — |
| `options` | `table { static?, dynamic?, groups?, entities? }` | optional | — |

Returns: `table { hit, position?, normal?, material?, distance?, entity? }, or nil`, `reason`

```lua
local ray = Open77.camera.aimRay(100)
if ray and ray.hit then Open77.blips.create({ position = ray.position, label = "Here" }) end
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25266) as `LuaCameraAimRay`.

## attach

```lua
Open77.camera.attach([camId], [entity], [offset])
```

`GAME`: Requires a live game instance.

Parents a scripted camera to an entity, or restores the player's own camera.

Attaches the Open77 camera to a game entity using the supplied local offset and rotation. Camera ownership is local to the resource generation; attaching replaces the resource's previous detached/attached camera state.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `camId` | `number` | optional | — |
| `entity` | `entity` | optional | — |
| `offset` | `vector3` | optional | — |

Returns: `true, or false, reason`

```lua
-- Parent a scripted camera to the local player's body, one metre behind and above
Open77.camera.attach(camId, 0, vector3(0.0, -1.0, 1.0))
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25262) as `LuaCameraAttachEntity`.

## cameras

```lua
Open77.camera.cameras()
```

`GAME`: Requires a live game instance.

This resource's scripted cameras, and whether it holds the view.

Client-only; requires `camera.script`. Owner-scoped for the camera list, like `Open77.screen.state`. `holder` is NOT owner-scoped: it names whoever has the view, including another resource, because that is the one question a `camera_held_by:` refusal leaves a caller asking. `phase` is `idle`, `blending_in`, `holding` or `blending_out`.

Returns: `table { held, holder, phase, activeCamera, blendRemainingMs, cameras }`

```lua
-- Only take the view if nobody else holds it
local cams = Open77.camera.cameras()
if cams.held and cams.holder ~= GetCurrentResourceName() then
    print("view is held by", cams.holder)
end
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25278) as `LuaCameraScriptState`.

## clearOrbit

```lua
Open77.camera.clearOrbit()
```

`GAME`: Requires a live game instance.

Releases this resource's preview orbit and framing.

Requires camera.preview. Clears the calling resource's preview orbit and releases preview ownership. It is a successful no-op if no preview is owned; it refuses to clear another resource's preview with camera_preview_owned.

Returns: `true, or false, reason`

```lua
-- The wardrobe preview closes: release the orbit
AddEventHandler("wardrobe:closed", function()
    Open77.camera.clearOrbit()
end)
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25251) as `LuaCameraClearOrbit`.

## configureThirdPerson

```lua
Open77.camera.configureThirdPerson(options)
```

`GAME`: Requires a live game instance.

Configures the playable third-person camera's framing, anchor, FOV and shoulder.

Client-only; requires `camera.style`. Acquires an exclusive resource-instance style or patches the caller's existing configuration. `style` selects `classic`, `shoulder` or `centered` and resets that preset before applying other fields; without it, fields patch the current configuration (classic on first use). `anchor` is `eyes` or `body` (feet), stable gameplay pivots rather than animated bones. `hip`, `aim` and `explore` accept distance (0.8..8 metres), offset (vector3 or complete x/y/z table: view-right -2..2, view-forward -1..1, world-up -1.5..3 metres), and fov (0 to restore the captured gameplay lens, otherwise 40..110 native component degrees). Other fields: shoulder (`left`/`right`), transition (0..3 seconds, default 0.3), runSpeed (0.5..15 m/s, default 2.5), runFov and sprintFov (0 disabled or 40..110). Aiming overrides sprint, which overrides running. Collision and native-camera safety still win. Unknown fields and invalid/non-finite values are rejected atomically. Does not enable TPP: use `Open77.perspective.setThirdPerson` separately. Stop/reload, coroutine failure and host teardown release the claim. Errors include `permission_denied:camera.style`, `camera_style_unavailable`, `invalid_camera_style_fields`, `invalid_camera_style` and `camera_style_owned`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table` | required | — |

Returns: `true on success, otherwise false`, `reason on failure`

```lua
-- Manifest: permissions { "camera.style" }
local ok, reason = Open77.camera.configureThirdPerson({
    style = "shoulder", transition = 0.35, runFov = 72, sprintFov = 78
})
if ok then
    local enabled, why = Open77.perspective.setThirdPerson(true)
    if not enabled then Open77.camera.resetThirdPerson(); print(why) end
else
    print(reason)
end
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25253) as `LuaCameraConfigureThirdPerson`.

## create

```lua
Open77.camera.create(options)
```

`GAME`: Requires a live game instance.

Creates a scripted camera and returns its id.

Client-only; requires `camera.script`. Nothing is rendered until `activate`. Options: `position` (world vector3, required unless `attachTo`), `attachTo` (entity; `0` is the local player's body), `offset` (vector3 in the parent's frame, X right / Y forward / Z up; also accepted as `position`), `lookAt` (a world vector3 OR an entity, re-solved every frame), `rotation` (quaternion `{x,y,z,w}` or Euler degrees `{pitch,yaw,roll}`, used when there is no `lookAt`), `fov` (5..170 degrees; omit or 0 to leave the engine's field of view alone). 16 cameras per resource, 64 per client. See the Scripted cameras guide.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table` | required | — |

Returns: `camera id, or nil, reason`

```lua
-- A security-camera shot in the corner of the room, pointed at the door
local cam = Open77.camera.create({
    position = vector3(-1442.2, 127.4, 21.0),
    lookAt = vector3(-1448.2, 96.1, 17.5),
    fov = 70,
})
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25268) as `LuaCameraCreate`.

## deactivate

```lua
Open77.camera.deactivate([options])
```

`GAME`: Requires a live game instance.

Gives the view back to the player, optionally blending out.

Client-only; requires `camera.script`. `options.blendMs` blends back to the player's own eyes and finishes exactly on them, so the restore writes the value already on screen. Only the holder may. Returns `true` and an `Open77.Promise` for a blended hand-back; an instant one is already finished and hands back none.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table` | optional | — |

Returns: `true and an Open77.Promise, or false, reason`

```lua
-- Give the view back after the cinematic, blending over half a second
Open77.camera.deactivate({ blendMs = 500 })
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25273) as `LuaCameraDeactivate`.

## destroy

```lua
Open77.camera.destroy(camId)
```

`GAME`: Requires a live game instance.

Destroys a scripted camera.

Client-only; requires `camera.script`. Destroying the camera you are looking through is legal and IS a release: the view comes back. Refusing would let a resource paint itself into a corner, and silently keeping the view would be worse.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `camId` | `number` | required | — |

Returns: `true, or false, reason`

```lua
-- Clean up the cinematic camera when it is no longer needed
Open77.camera.destroy(cam)
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25274) as `LuaCameraDestroy`.

## detach

```lua
Open77.camera.detach(x, y, z)
```

`GAME`: Requires a live game instance.

Detaches the camera from the body, with an offset in the body's own space.

Detaches the Open77 camera from its target while retaining a controllable camera view. It does not synchronize a camera to other players and is automatically restored during resource cleanup.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `z` | `number` | required | — |

Returns: `true on success, otherwise false`, `reason for the refusal`

```lua
-- A selfie view: the player's own camera moved two metres in front of them
Open77.camera.detach(0.0, 2.0, 1.6)
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25258) as `LuaCameraDetach`.

## detachFrom

```lua
Open77.camera.detachFrom(camId)
```

`GAME`: Requires a live game instance.

Un-parents a scripted camera from its entity.

Client-only; requires `camera.script`. NOT spelled `detach`: `Open77.camera.detach()` already means 'move the player's own camera off their eyes' and all of its arguments are optional, so there is no arity left to overload. The camera keeps the pose it had; the next `setTransform` places it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `camId` | `number` | required | — |

Returns: `true, or false, reason`

```lua
-- Stop following the car; the camera stays where it was
Open77.camera.detachFrom(cam)
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25271) as `LuaCameraDetachFrom`.

## follow

```lua
Open77.camera.follow(entity, options)
```

`GAME`: Requires a live game instance.

Rides an entity: a camera behind it, aimed at it, both re-solved every frame.

Client-only; requires `camera.script`. Entity `0` is the local player's body. `options`: `distance` (metres behind, 0..100, default 4.0), `height` (metres above, -50..50, default 1.5), `side` (metres right, -50..50, default 0), `fov` (5..170 degrees; omit or 0 to leave it alone), `blendMs` (0..60000). The offsets are in the TARGET's own frame, so `distance` means behind him rather than north of him. ONE camera per resource: calling `follow` again retargets it and returns the same id, superseding the previous call's promise with `camera_superseded`. `Open77.camera.unfollow` gives the view back and destroys it. Not a new way to hold the view -- it is `create` + `lookAt` + `activate`, so every release rule on the Scripted cameras page applies unchanged. The world streams around the player's BODY, not around the lens, so a distant target renders an empty frame: move the body there first. See the Scripted cameras guide.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `number` | required | — |
| `options` | `table` | required | — |

Returns: `camera id and a blend promise, or nil, reason`

```lua
-- A spectator camera four metres behind another player
local cam, blend = Open77.camera.follow(Open77.players.entity(targetId), { distance = 4.0, height = 1.5, blendMs = 400 })
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25281) as `LuaCameraFollow`.

## lookAt

```lua
Open77.camera.lookAt(camId, target)
```

`GAME`: Requires a live game instance.

Points a scripted camera at a world point or an entity.

Client-only; requires `camera.script`. A vector3 aims at a fixed world point; anything else is read as an entity id, and `0` is the local player's own body. The aim is re-solved EVERY frame, so a tracked entity that moves stays centred -- this is FiveM's `PointCamAtEntity`, not a snapshot. An entity that streams out holds the last aim instead of snapping the camera to the world origin.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `camId` | `number` | required | — |
| `target` | `vector3|entity` | required | — |

Returns: `true, or false, reason`

```lua
-- Keep the camera centred on the car as it drives past
Open77.camera.lookAt(cam, Open77.vehicles.get(vehicleId).entity)
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25270) as `LuaCameraLookAt`.

## orbit

```lua
Open77.camera.orbit(yawDegrees)
```

`GAME`: Requires a live game instance.

Sets this resource's preview orbit yaw on an active third-person rig.

Requires camera.preview. Sets a resource-owned preview orbit yaw in degrees, normalized modulo 360. The yaw must be finite. Returns false/reason if another resource owns the preview, the preview backend is unavailable, or the native operation fails. Use clearOrbit to release the owned preview.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `yawDegrees` | `number` | required | — |

Returns: `true, or false, reason`

```lua
-- Rotate the wardrobe preview with the mouse wheel
RegisterKeyMapping("preview_left", "Rotate preview", "Q", function()
    previewYaw = previewYaw - 15
    Open77.camera.orbit(previewYaw)
end)
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25250) as `LuaCameraOrbit`.

## project

```lua
Open77.camera.project(position)
```

`GAME`: Requires a live game instance.

Projects a world point into normalized WebUI viewport coordinates.

Uses REDengine's active camera projection. `(0,0)` is the top-left and `(1,1)` the bottom-right. `onScreen` is false for points behind the view or beyond the viewport.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `position` | `table { x, y, z }` | required | — |

Returns: `table { x, y, depth, distance, onScreen }, or nil`, `reason`

```lua
-- Place a 2D name tag over a world position
local screen = Open77.camera.project(target)
if screen and screen.onScreen then
    nameTag.setPosition(screen.x, screen.y)
end
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25265) as `LuaCameraProject`.

## resetThirdPerson

```lua
Open77.camera.resetThirdPerson()
```

`GAME`: Requires a live game instance.

Releases this resource's camera style and shake, restoring underlying framing and FOV.

Client-only; requires `camera.style`. Releases the caller's style and shake and blends back to the underlying framing and captured gameplay lens. Does not turn third person off, clear a perspective lock or alter server policy. Safe when no style is configured; another resource's style returns `camera_style_owned` instead of being reset. Other errors: `permission_denied:camera.style`, `camera_style_unavailable`. Stop/reload, coroutine failure and host teardown also release the style automatically.

Returns: `true on success, otherwise false`, `reason on failure`

```lua
-- Give the camera style back without changing the player's FPP/TPP choice.
local ok, reason = Open77.camera.resetThirdPerson()
if not ok then print(reason) end
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25255) as `LuaCameraResetThirdPerson`.

## screenRay

```lua
Open77.camera.screenRay(x, y, [maxDistance])
```

`GAME`: Requires a live game instance.

Unproject a normalized screen point into a world-space ray.

Requires world.query. x/y use normalized top-left viewport coordinates in [0,1]. maxDistance defaults to 100 metres and must be finite in [0.1,1000]. Returns origin, unit direction, endpoint position and maxDistance; this is not a collision result. Uses the active camera and checks projection consistency. Returns nil/reason if the camera or projection is unavailable. Identical global alias: Screen2DPointTo3DSpace. See screen-picking.md and context-menu.md. Requires the new screen-picking client, not the earlier .62 release.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `maxDistance` | `number` | optional | — |

Returns: `ray table, or nil, reason`

```lua
-- The ray under the mouse cursor, 50 m long
local cursor = Open77.input.cursor()
if cursor and cursor.inBounds then
    local ray = Open77.camera.screenRay(cursor.x, cursor.y, 50)
    if ray then print("ray ends at", ray.position.x, ray.position.y, ray.position.z) end
end
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 24882) as `LuaCameraScreenRay`.

## screenRaycast

```lua
Open77.camera.screenRaycast(x, y, [maxDistance], [options])
```

`GAME`: Requires a live game instance.

Pick a world surface/entity, with optional visible F7 self targeting.

Requires world.query. Same normalized coordinates and distance limits as screenRay. Optional {static=true,dynamic=true,self=false} strict boolean options; at least one of static/dynamic must be enabled. Physical world, vehicle and NPC-hitbox categories are included, not query-only player sensors. self=true with dynamic=true also picks the active visible F7 body using animated slot capsules, without enabling collision. A closer physical hit wins. This approximation returns hitSource=self_presentation, local playerId and isLocalPlayer=true, but no physical normal/material; hidden/suspended bodies are excluded. Ordinary physical hits add normal, camera distance, material, entityLookupAvailable and optional target. A miss has hit=false and position at the ray endpoint, not a surface. target distinguishes decimal-string engineEntity/local entity handles from canonical playerId, vehicleId, npcId and propId; propId is a decimal string. Never interchange these IDs. NPC records use Character.* when available. No nearest-object fallback or server authority. See screen-picking.md and context-menu.md for contracts and examples.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `maxDistance` | `number` | optional | — |
| `options` | `table` | optional | — |

Returns: `raycast table, or nil, reason`

```lua
local hit, reason = Open77.camera.screenRaycast(0.5, 0.5, 12, {self=true})
if hit and hit.hit and hit.target then
    print(hit.target.kind, hit.target.isLocalPlayer, hit.hitSource)
end
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 24883) as `LuaCameraScreenRaycast`.

## setFov

```lua
Open77.camera.setFov(degrees)
```

`GAME`: Requires a live game instance.

Sets the field of view, in degrees.

Changes the field of view of the camera currently controlled by Open77. The value is validated by the native backend and applies only to this client's presentation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `degrees` | `number` | required | — |

Returns: `true on success, otherwise false`, `reason for the refusal`

```lua
-- A wider view while driving, back to normal on foot
SetTick(function()
    Open77.camera.setFov(Open77.character.isInVehicle() and 95 or 80)
end)
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25263) as `LuaCameraFieldOfView`.

## setTransform

```lua
Open77.camera.setTransform(camId, [position], [rotation])
```

`GAME`: Requires a live game instance.

Moves and/or turns a scripted camera.

Client-only; requires `camera.script`. Two forms: `setTransform(camId, position?, rotation?)` for a per-frame dolly, and `setTransform(camId, { position =, rotation =, fov = })` when the field of view changes too. Setting a rotation CLEARS a previous `lookAt` -- an explicit rotation is an explicit aim, and a stale look-at silently overriding it would be a trap. Cheap enough to call every frame.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `camId` | `number` | required | — |
| `position` | `vector3` | optional | — |
| `rotation` | `table` | optional | — |

Returns: `true, or false, reason`

```lua
-- Dolly the camera forward a little every frame
SetTick(function()
    camZ = camZ + 0.01
    Open77.camera.setTransform(cam, vector3(-1442.2, 127.4, camZ))
end)
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25269) as `LuaCameraSetTransform`.

## shake

```lua
Open77.camera.shake([camId], preset, [amplitude], [milliseconds])
```

`GAME`: Requires a live game instance.

Shakes a scripted camera with a named preset.

Client-only; requires `camera.script`. Presets: `hand`, `drunk`, `explosion`, `earthquake`; an unknown name is refused rather than defaulted. `amplitude` defaults to 1.0 and is capped at 4.0, `ms` defaults to 500. Additive on top of the resolved pose, so it composes with a blend and with a look-at instead of fighting either, and it decays exactly to rest. Omit `camId` to shake the camera this resource currently holds.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `camId` | `number` | optional | — |
| `preset` | `string` | required | — |
| `amplitude` | `number` | optional | — |
| `milliseconds` | `number` | optional | — |

Returns: `true, or false, reason`

```lua
-- An explosion nearby shakes the current camera for a second
RegisterNetEvent("fx:explosion", function()
    Open77.camera.shake(nil, "explosion", 2.0, 1000)
end)
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25275) as `LuaCameraShake`.

## shakeThirdPerson

```lua
Open77.camera.shakeThirdPerson(options)
```

`GAME`: Requires a live game instance.

Starts or replaces a finite positional shake on the caller's playable TPP style.

Client-only; requires `camera.style` and a style already owned via configureThirdPerson. options: amplitude (vector3 or complete x/y/z table, each 0..0.2 metres, default 0.02/0.01/0.015), frequency (0.1..20 Hz, default 8), duration (0.05..10 seconds, default 0.5). Axes are view-right, view-forward, world-up. Shakes move the pivot before collision resolution; they do not rotate aim, inject roll or change player position. Natural attack/release envelope; a new shake replaces the old one rather than stacking. Wall-clock expiry continues while another camera owns the view. Unknown fields and invalid/non-finite values are refused. Errors: `permission_denied:camera.style`, `camera_style_unavailable`, `invalid_camera_shake`, `camera_style_not_owned`. Cleanup follows the owning style's lifecycle; do not restart it every frame.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table` | required | — |

Returns: `true on success, otherwise false`, `reason on failure`

```lua
-- First acquire a style; an empty options table uses shake defaults.
local configured, reason = Open77.camera.configureThirdPerson({ style = "shoulder" })
if configured then
    Open77.camera.shakeThirdPerson({ amplitude = vector3(0.03, 0.015, 0.02), frequency = 8, duration = 0.65 })
else
    print(reason)
end
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25256) as `LuaCameraShakeThirdPerson`.

## stopShake

```lua
Open77.camera.stopShake(camId)
```

`GAME`: Requires a live game instance.

Stops a scripted camera's shake immediately.

Requires `camera.script`. Cancels the shake started by `Open77.camera.shake` on that camera and leaves the camera itself where it is -- it is not a deactivate, and a camera with no shake running accepts it without complaint, so a cleanup path may call it unconditionally. Refuses with `permission_denied:camera.script`, `camera_unavailable_on_this_host` when the host exposes no scripted-camera backend, `invalid_camera_id`, or `camera_not_found` for a camera this resource does not own. Shakes are released with the camera, so a resource that stops does not leave the view trembling.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `camId` | `number` | required | — |

Returns: `true, or false, reason`

```lua
-- Cut the earthquake short when the player enters the bunker
Open77.camera.stopShake(cam)
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25276) as `LuaCameraStopShake`.

## stopThirdPersonShake

```lua
Open77.camera.stopThirdPersonShake()
```

`GAME`: Requires a live game instance.

Immediately stops only the calling resource's third-person camera shake.

Client-only; requires `camera.style`. Safe when the caller owns no shake and never stops another resource's effect. Keeps the style and selected perspective unchanged. Explicit stop is immediate, unlike the natural expiry envelope. Errors: `permission_denied:camera.style`, `camera_style_unavailable`.

Returns: `true on success, otherwise false`, `reason on failure`

```lua
local ok, reason = Open77.camera.stopThirdPersonShake()
if not ok then print(reason) end
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25257) as `LuaCameraStopThirdPersonShake`.

## thirdPerson

```lua
Open77.camera.thirdPerson(enabled, [distance], [height])
```

`GAME`: Requires a live game instance.

Controls the legacy camera-only third-person path.

Legacy camera-only control, not the playable perspective arbiter. Prefer `Open77.perspective.setThirdPerson(enabled, force)` to switch or lock the playable FPP/TPP view, and `Open77.camera.configureThirdPerson(options)` for shoulder framing, anchors, movement FOV and shake. The legacy call changes the camera without coordinating all body, weapon and input owners. Turning it off restores the view the game had before the call, rather than some default.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |
| `distance` | `number` | optional | — |
| `height` | `number` | optional | — |

Returns: `true on success, otherwise false`, `reason for the refusal`

```lua
-- A shoulder view during the fashion show, restored afterwards
AddEventHandler("show:started", function() Open77.camera.thirdPerson(true, 2.5, 1.7) end)
AddEventHandler("show:ended", function() Open77.camera.thirdPerson(false) end)
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25252) as `LuaCameraThirdPerson`.

## thirdPersonState

```lua
Open77.camera.thirdPersonState()
```

`GAME`: Requires a live game instance.

Reads the requested third-person camera style and whether it is active or owned.

Client-only read; no permission required. Returns configured, owned (by the calling resource), active, shaking, appliedFov, and the full configuration: style, anchor, shoulder, hip/aim/explore (distance, offset, fov), transition, runSpeed, runFov and sprintFov. style names the base preset, even after custom patches. active is false while FPP, a native vehicle/scripted camera or wardrobe owns the view; a configured style resumes when the playable TPP rig returns. appliedFov is the last rig-written FOV, not necessarily the current camera's lens; use `Open77.camera.view()` for the rendered view. Returns `nil, camera_style_unavailable` when the backend is absent.

Returns: `snapshot table, or nil on failure`, `reason on failure`

```lua
local state, reason = Open77.camera.thirdPersonState()
if state then
    print(state.style, state.owned, state.active, state.appliedFov)
else
    print(reason)
end
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25254) as `LuaCameraThirdPersonState`.

## unfollow

```lua
Open77.camera.unfollow(options)
```

`GAME`: Requires a live game instance.

Ends a follow camera: blends the view back, then destroys it.

Client-only; requires `camera.script`. `options.blendMs` is 0..60000. Answers `false, "not_following"` when this resource has no follow camera. The camera is destroyed even when the view had already been released by something else, so a follow can never be left behind. See the Scripted cameras guide.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table` | required | — |

Returns: `true and a blend promise, or false, reason`

```lua
-- The spectated player left: blend back to our own eyes
Open77.camera.unfollow({ blendMs = 400 })
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25282) as `LuaCameraUnfollow`.

## unproject

```lua
Open77.camera.unproject(x, y)
```

`GAME`: Requires a live game instance.

The world ray behind a screen point.

Client-only and UNGATED, like `project` and `view`: it reads the camera, it does not move it. `x` and `y` are 0..1 with the origin at the TOP LEFT -- exactly the frame `Open77.camera.project` returns, which is what makes `unproject(project(p))` a round trip. Returns `{ origin, direction }` in world space, `direction` normalised.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `x` | `number` | required | — |
| `y` | `number` | required | — |

Returns: `table { origin, direction }, or nil, reason`

```lua
-- Find what is under the centre of the screen
local ray = Open77.camera.unproject(0.5, 0.5)
if ray then
    local to = vector3(ray.origin) + vector3(ray.direction) * 30
    local hit = Open77.world.raycast(ray.origin, to)
    if hit and hit.hit then print("surface at", hit.distance, "m") end
end
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25277) as `LuaCameraUnproject`.

## view

```lua
Open77.camera.view()
```

`GAME`: Requires a live game instance.

Where the view is: position, forward vector, field of view.

Returns the latest game-thread camera snapshot used by projection and presentation APIs. It is a same-client read and may return `nil, reason` while the game camera is unavailable.

Returns: `position (x, y, z)`, `forward (x, y, z)`, `field of view`

```lua
-- Where is the camera and which way does it face?
local view = Open77.camera.view()
if view then
    print(("camera at %.1f, %.1f, %.1f, fov %d"):format(view.position.x, view.position.y, view.position.z, view.fov))
end
```

[Scripted cameras guide](../../01%20Documentation/cameras.md)

Registered in `ResourceHost.cpp` (line 25264) as `LuaCameraView`.
