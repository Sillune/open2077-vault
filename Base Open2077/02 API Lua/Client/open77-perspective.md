# Open77.perspective (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-perspective.md) · [Page web](https://open2077.net/docs/api/client/open77-perspective)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

7 functions.

## applyPolicy

```lua
Open77.perspective.applyPolicy(policy, [perspective])
```

`GAME`: Requires a live game instance.

Applies a third-person policy received from the server.

`disabled`, `allowed`, `default` or `forced`, with the pinned perspective for `forced` (a pin with nothing pinned is refused rather than quietly becoming first person). Requires the `perspective.policy` permission -- not because it exposes anything, since all of this is client-local presentation, but because it is the one call that can take the choice away from the player. It deliberately cannot express the bridge's fifth value, `lab`, which disengages the arbiter entirely: a server able to reach that could leave a client with nothing governing the camera. The bundled `open77_perspective` resource is the intended caller, on receipt of `open77:perspective:policy`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `policy` | `string` | required | — |
| `perspective` | `string` | optional | — |

Returns: `true on success, otherwise false`, `reason for the refusal`

```lua
-- The bundled perspective resource applies what the server decided
RegisterNetEvent("perspective:policy", function(policy, perspective)
    Open77.perspective.applyPolicy(policy, perspective)
end)
```

Registered in `ResourceHost.cpp` (line 25387) as `LuaPerspectiveApplyPolicy`.

## clearThirdPersonOverride

```lua
Open77.perspective.clearThirdPersonOverride()
```

`GAME`: Requires a live game instance.

Releases this resource's temporary FPP/TPP lock without changing player preference.

Client-only; no permission required and no arguments accepted. Idempotent: releases only the calling resource instance's override, never another resource's. Preserves the player's pre-lock preference, camera style and server perspective policy. Native-camera safety and server policy still determine the effective view after release. Stop/reload, coroutine failure and host teardown also release the owner's override automatically. To release a lock AND request a different view, use setThirdPerson(enabled, false) instead. Errors: `invalid_perspective_arguments`, `perspective_unavailable_on_this_host`.

Returns: `true on success, otherwise false`, `reason on failure`

```lua
local ok, reason = Open77.perspective.clearThirdPersonOverride()
if not ok then print(reason) end
```

Registered in `ResourceHost.cpp` (line 25389) as `LuaPerspectiveClearThirdPersonOverride`.

## get

```lua
Open77.perspective.get()
```

`GAME`: Requires a live game instance.

Returns the perspective actually in force.

`fpp` or `tps`, read from the committed camera owner rather than from the request -- intention is not evidence, which is the same rule `camera.state` had to learn. An engine-owned view (a vehicle camera, a cutscene) reads `fpp`, because that is what the player sees; `Open77.perspective.state().camera` keeps the distinction.

Returns: `"fpp" or "tps", or nil`, `reason`

```lua
-- Adjust the crosshair for the view actually in force
SetTick(function()
    crosshair.big = Open77.perspective.get() == "tps"
end)
```

Registered in `ResourceHost.cpp` (line 25384) as `LuaPerspectiveGet`.

## set

```lua
Open77.perspective.set(mode)
```

`GAME`: Requires a live game instance.

Asks the perspective arbiter for first or third person.

Records the local player's standing request. It is a REQUEST, not a switch: the perspective arbiter coordinates camera, body, input, weapon, effects and audio. Native vehicle/workspot/death/photo-mode safety and server policy take priority; a server-policy refusal is reported as `refused_by_policy` while remembering the request. A temporary resource lock created by setThirdPerson(enabled, true) is different: it refuses an opposing request without changing the pre-lock preference, and does not record a same-view request either. Accepts `fpp`/`first`/`1` and `tps`/`tpp`/`third`/`3`. Use setThirdPerson for a boolean switch or resource-owned lock and configureThirdPerson for camera style. Do not bypass the arbiter with legacy camera.thirdPerson, which changes only the camera owner.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `mode` | `string` | required | — |

Returns: `true on success, otherwise false`, `reason for the refusal`

```lua
-- Third person on foot when the player joins; the arbiter still decides
AddEventHandler("onClientResourceStart", function(name)
    if name == GetCurrentResourceName() then Open77.perspective.set("tps") end
end)
```

Registered in `ResourceHost.cpp` (line 25383) as `LuaPerspectiveSet`.

## setThirdPerson

```lua
Open77.perspective.setThirdPerson(enabled, [force])
```

`GAME`: Requires a live game instance.

Requests first/third person, optionally locking that view for this resource.

Client-only. enabled and optional force must be strict booleans; force defaults to false. Ordinary true/false requests TPP/FPP without a permission and leaves F7 available, subject to existing locks and server policy. force=true requires `perspective.policy`: (true,true) locks TPP; (false,true) locks FPP. This exclusive resource-instance lock preserves the player's remembered pre-lock preference, including when F7 is pressed. A rival forced request returns `perspective_owned`. force=false first releases the caller's own previous lock, then requests the selected view; it cannot clear another resource's lock. Native vehicle/cutscene/safety states and server disabled/forced policy win. Conflicting server policy rejects a new lock with `refused_by_policy`; existing locks resume after policy permits them again. Stop/reload, coroutine failure and host teardown release the lock automatically. No per-frame loop is needed. Does not configure camera style or broadcast a network view: use server setPolicy for global policy. Other errors: `invalid_perspective_arguments`, `permission_denied:perspective.policy` (forced calls only), `perspective_unavailable_on_this_host`. Inspect state for the effective view rather than assuming acceptance means the TPP camera currently owns the screen.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |
| `force` | `boolean` | optional | `false` |

Returns: `true when accepted, otherwise false`, `reason on failure`

```lua
-- Manifest: permissions { "perspective.policy" }
local ok, reason = Open77.perspective.setThirdPerson(true, true)
if not ok then print(reason) end
-- Later restore the choice the player had before our lock.
Open77.perspective.clearThirdPersonOverride()
```

Registered in `ResourceHost.cpp` (line 25388) as `LuaPerspectiveSetThirdPerson`.

## state

```lua
Open77.perspective.state()
```

`GAME`: Requires a live game instance.

Reports the whole perspective decision.

Client-only read; no permission required. Returns mode, requested, requestExpressed, policy, forced, resourceForced (boolean), resourcePerspective (tps/fpp/none), allowed, engaged, aiming, available, settled, transition, reason, rule, tier, the six owners (camera, body, input, weapon, effects, audio), violations, unsupportedChecks, slideFrames, slideLifted, unsupportedFires and unsupportedSource. resourceForced/resourcePerspective describe a requested local resource lock, not guaranteed ownership of a native vehicle or cutscene camera: inspect mode, camera, reason and settled for the effective view. allowed is false under a disabled/forced server policy or while a resource lock exists; an active resource lock reports reason/rule `resource_forced`. violations counts ticks where the arbiter detected conflicting body/weapon ownership; unsupported fields are diagnostic gate counters. Reads work during a live session and may return nil with `perspective_unavailable_on_this_host` if unsupported.

Returns: `table, or nil`, `reason`

```lua
-- Grey out the "toggle view" button when the server forbids it
local state = Open77.perspective.state()
toggleButton.enabled = state and state.allowed
```

Registered in `ResourceHost.cpp` (line 25386) as `LuaPerspectiveState`.

## toggle

```lua
Open77.perspective.toggle()
```

`GAME`: Requires a live game instance.

Flips the standing request and returns the new mode.

Flips the REQUEST, not the effective view. Native vehicle cameras keep their own rules regardless of the standing on-foot preference. For a player who has never expressed a preference, the effective view is inverted so the first press does not appear to do nothing. A temporary resource lock from setThirdPerson(enabled, true) blocks an opposing toggle and preserves the pre-lock preference; server policy and native-camera safety still take priority. Returns the new fpp/tps request on acceptance, or nil and a refusal reason.

Returns: `the new "fpp" or "tps", or nil`, `reason`

```lua
-- V flips between first and third person
RegisterKeyMapping("toggle_view", "Toggle view", "V", function()
    local mode = Open77.perspective.toggle()
    if mode then Open77.hud.notify(mode == "tps" and "Third person" or "First person") end
end)
```

Registered in `ResourceHost.cpp` (line 25385) as `LuaPerspectiveToggle`.
