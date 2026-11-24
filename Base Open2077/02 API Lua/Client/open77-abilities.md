# Open77.abilities (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-abilities.md) · [Page web](https://open2077.net/docs/api/client/open77-abilities)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

8 functions.

## approveSlam

```lua
Open77.abilities.approveSlam(request, sequence, allowed)
```

`GAME`: Requires a live game instance.

Admits or refuses a pending slam activation.

Requires `player.abilities.project`. `allowed` must be a boolean -- there is no third answer, and a non-boolean is refused as `invalid_approval` rather than read as false. The sequence must match a pending activation on this owned request.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `request` | `integer (request id)` | required | — |
| `sequence` | `integer` | required | — |
| `allowed` | `boolean` | required | — |

Returns: `true, or nil`, `reason: invalid_request, invalid_sequence, invalid_approval`

```lua
-- The support resource admits the slam the server just accepted
RegisterNetEvent("slam:accepted", function(sequence)
    Open77.abilities.approveSlam(slamRequest, sequence, true)
end)
```

[Ground Slam guide](../../01%20Documentation/ground-slam.md)

Registered in `ResourceHost.cpp` (line 24861) as `LuaAbilitySlamOperation`.

## cancelSlam

```lua
Open77.abilities.cancelSlam(request, sequence, reason)
```

`GAME`: Requires a live game instance.

Cancels a slam activation, with a reason.

Requires `player.abilities.project`. The reason is a string of 1 to 96 characters and is required: an unexplained cancellation would leave nothing in the log to explain a slam that did not happen.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `request` | `integer (request id)` | required | — |
| `sequence` | `integer` | required | — |
| `reason` | `string (1..96 characters)` | required | — |

Returns: `true, or nil`, `reason: invalid_request, invalid_sequence, invalid_reason`

```lua
-- The server refused the slam: cancel the native activation with a reason
RegisterNetEvent("slam:refused", function(sequence, reason)
    Open77.abilities.cancelSlam(slamRequest, sequence, reason or "server_refused")
end)
```

[Ground Slam guide](../../01%20Documentation/ground-slam.md)

Registered in `ResourceHost.cpp` (line 24861) as `LuaAbilitySlamOperation`.

## configureSlam

```lua
Open77.abilities.configureSlam(options)
```

`GAME`: Requires a live game instance.

Configures the calling resource's native Ground Slam projector.

Client support API requiring player.abilities.project and a running resource. options may contain ground and air booleans (at least one true), staminaManaged=true, maxDurationMs from 500 through 15000 and finite maxFallSpeed greater than zero through 60. These are adapter readiness bounds, not arbitrary motion or damage controls. Returns a configuration request handle or nil, reason; native projection acknowledgement and server admission are still required. Server creators normally use Open77.abilities.define/grant through the optional support resource. No paid implant or vanilla perk progression is changed. See ground-slam.md; a returned handle does not prove playable native movement.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table` | required | — |

Returns: `request handle, or nil, reason`

```lua
-- The support resource prepares the native ground-slam projector once
slamRequest = Open77.abilities.configureSlam({
    ground = true, air = true, staminaManaged = true,
    maxDurationMs = 5000, maxFallSpeed = 30,
})
```

[Ground Slam guide](../../01%20Documentation/ground-slam.md)

Registered in `ResourceHost.cpp` (line 24856) as `LuaAbilityConfigureSlam`.

## presentSlam

```lua
Open77.abilities.presentSlam(playerId, activationId, sequence, phase, airborne, elapsedMs)
```

`GAME`: Requires a live game instance.

Projects a correlated, server-accepted Ground Slam phase for an observer.

Client support API requiring player.abilities.project and a running resource. Accepts a positive safe-integer playerId, a 32-hex-character activation ID, uint32 sequence greater than zero, phase, airborne boolean and elapsedMs from zero through 30000. Phases are windup, descent, contact, impact, recovery, complete, cancelled or rejected. It owns presentation only and never authorizes movement, damage or entitlement. Owner-scoped cleanup, incarnation checks and Dash/Slam pose arbitration remain native responsibilities. true means the adapter accepted the presentation request; actual rendering and sound require live observation. See ground-slam.md for the authoritative public workflow.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `activationId` | `string` | required | — |
| `sequence` | `integer` | required | — |
| `phase` | `string` | required | — |
| `airborne` | `boolean` | required | — |
| `elapsedMs` | `integer` | required | — |

Returns: `true, or nil, reason`

```lua
-- Replay a remote player's slam phase the server relayed to this observer
RegisterNetEvent("slam:phase", function(playerId, activationId, sequence, phase, airborne, elapsedMs)
    Open77.abilities.presentSlam(playerId, activationId, sequence, phase, airborne, elapsedMs)
end)
```

[Ground Slam guide](../../01%20Documentation/ground-slam.md)

Registered in `ResourceHost.cpp` (line 24857) as `LuaAbilityPresentSlam`.

## releaseSlam

```lua
Open77.abilities.releaseSlam()
```

`GAME`: Requires a live game instance.

Releases this resource's ground-slam ownership.

Requires `player.abilities.project`. Takes no arguments: the owner is the calling resource, which cannot be supplied. Resource stop releases it for you, so a reloading gamemode never leaves the slam claimed.

Returns: `true, or nil`, `reason: abilities_backend_unavailable`

```lua
-- The mini-game ends: give the slam projector back
AddEventHandler("minigame:ended", function()
    Open77.abilities.releaseSlam()
end)
```

[Ground Slam guide](../../01%20Documentation/ground-slam.md)

Registered in `ResourceHost.cpp` (line 24861) as `LuaAbilitySlamOperation`.

## requestSlam

```lua
Open77.abilities.requestSlam(request)
```

`GAME`: Requires a live game instance.

Asks the native layer to begin a slam and returns its sequence.

Requires `player.abilities.project`. The returned sequence identifies this activation for `approveSlam` and `cancelSlam`; it is how a late answer is stopped from acting on the wrong slam.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `request` | `integer (request id)` | required | — |

Returns: `sequence (integer), or nil`, `reason: invalid_request, abilities_backend_unavailable`

```lua
-- The player pressed the slam key: begin the native activation and remember its sequence
RegisterKeyMapping("slam", "Ground slam", "L", function()
    local sequence = Open77.abilities.requestSlam(slamRequest)
    if sequence then pendingSlam = sequence end
end)
```

[Ground Slam guide](../../01%20Documentation/ground-slam.md)

Registered in `ResourceHost.cpp` (line 24861) as `LuaAbilitySlamOperation`.

## slamActivity

```lua
Open77.abilities.slamActivity()
```

`GAME`: Requires a live game instance.

The local character's current ground-slam activity, with its recent history.

Requires `player.abilities.read`. Returns `{ request, sequence, phaseSequence, impactSequence, elapsedMs, phase, mode, reason, grounded, position, contact, verticalSpeed, history }`, where `history` holds up to sixteen recent phase receipts. It is what the presenting resource replays through `presentSlam`; it is observation, not authority.

Returns: `table, or nil`, `reason: permission_denied:player.abilities.read, abilities_backend_unavailable`

```lua
-- Show the current slam phase on a debug overlay
local activity = Open77.abilities.slamActivity()
if activity then print(activity.phase, activity.verticalSpeed) end
```

[Ground Slam guide](../../01%20Documentation/ground-slam.md)

Registered in `ResourceHost.cpp` (line 24861) as `LuaAbilitySlamOperation`.

## slamState

```lua
Open77.abilities.slamState(request)
```

`GAME`: Requires a live game instance.

The phase of a ground-slam configuration request.

Requires `player.abilities.read`. Takes the request id `configureSlam` returned and answers the native phase plus a reason. One dispatcher serves the six slam operations, so they share identical permission, lifetime and handle checks.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `request` | `integer (request id)` | required | — |

Returns: `phase string`, `reason`

```lua
-- Is the projector ready before the key is enabled?
if Open77.abilities.slamState(slamRequest) == "ready" then enableSlamKey() end
```

[Ground Slam guide](../../01%20Documentation/ground-slam.md)

Registered in `ResourceHost.cpp` (line 24861) as `LuaAbilitySlamOperation`.
