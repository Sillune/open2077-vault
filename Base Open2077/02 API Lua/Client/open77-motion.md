# Open77.motion (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-motion.md) · [Page web](https://open2077.net/docs/api/client/open77-motion)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

7 functions.

## knockdown

```lua
Open77.motion.knockdown(entity, x, y, distance)
```

`GAME`: Requires a live game instance.

Queue an owned native reaction on an exact local body.

Requires player.motion.project. entity is a local registry handle (1 for the owner), not a network player ID. Direction is normalized; distance is0..6 m and the local owner must be alive, ready, on foot and grounded. Returns an owned request ID. Owner phase starts pending, then active after two valid consecutive native-down observations within1.5s. Remote projection starts its native action immediately. No damage, teleport or server authority is granted.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `integer` | required | — |
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `distance` | `number` | required | — |

Returns: `request ID`, `nil, reason on rejection`

```lua
local request, err = Open77.motion.knockdown(1, 0, 1, 0)
if not request then print(err); return end
print(Open77.motion.state(request))
```

[Cyberware authority & motion guide](../../01%20Documentation/cyberware.md)

Registered in `ResourceHost.cpp` (line 24890) as `LuaMotionKnockdown`.

## launch

```lua
Open77.motion.launch(entity, x, y, push, lift)
```

`GAME`: Requires a live game instance.

Queue an owned native character launch on an exact local body.

Requires player.motion.project and a matching development client. entity is a local registry handle (1 for the local player), not a network player ID. The finite nonzero XY direction is normalized; push is 0..20 m/s and lift is 0..14 m/s, with at least one positive. Returns a resource-owned request ID or nil/reason. The local owner must be ready, alive, unmounted and grounded; its request starts pending and activation requires observed movement after one queued native 3D impulse. On a remote body, only the reaction pose is presented; displacement follows owner snapshots. Use state(request) and stop(request) for this resource's request. This projection primitive does not grant server authority, damage or a teleport. Use the matching open77_cyberware projector for network lease and incarnation checks.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `integer` | required | — |
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `push` | `number` | required | — |
| `lift` | `number` | required | — |

Returns: `request ID`, `nil, reason on rejection`

```lua
-- Client projection only: invoked after the server has granted motion.
local request, reason = Open77.motion.launch(1, 1, 0, 12, 10)
if request then print(Open77.motion.state(request)) else print(reason) end
```

[Character launch & weapon blasts](../../01%20Documentation/weapon-customization.md#launch-nearby-characters)

Registered in `ResourceHost.cpp` (line 24894) as `LuaMotionLaunch`.

## predictAction

```lua
Open77.motion.predictAction(source, entity, x, y, expectedDamage)
```

`GAME`: Requires a live game instance.

Predict a temporary reaction on a remote player proxy.

Requires `player.motion.project`. **Experimental.** Starts a presentation-only knockdown on a streamed remote player's proxy for a melee, Ground Slam or quickhack contact, before the server's verdict. `source` is `melee`, `slam` or `hack`; `entity` is the remote player's local registry ID, never the local player; `x`, `y` give the horizontal direction of the hit. The resource must first apply its gamemode's team, safe-zone and crowd-control policy. Native guards require an active session, a living attacker and victim, an available unseated body, no competing pose, known health above `expectedDamage` and no god mode. `expectedDamage` is a conservative lethal bound after gameplay multipliers; use zero only for an authoritative non-lethal or cosmetic configuration. At most 32 predictions are pending, each for 1500 ms. The authorized knockdown adopts the same native reaction; an unadopted prediction stops through the native get-up path on expiry or lifecycle change, or through `refutePrediction`. The call sends no network message and grants no damage, displacement or authority. The bundled `open77_cyberware` resource is the intended caller and applies the server's prediction policy first; see [Prediction](../../01%20Documentation/prediction.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `source` | `string` | required | — |
| `entity` | `integer` | required | — |
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `expectedDamage` | `number` | required | — |

Returns: `request ID or nil`, `reason on refusal`

```lua
-- After local gameplay policy admitted this actual contact:
local request, reason = Open77.motion.predictAction('melee', remoteEntity, 1, 0, 15)
if not request then print(reason) end
```

[Prediction guide](../../01%20Documentation/prediction.md)

Registered in `ResourceHost.cpp` (line 24891) as `LuaMotionPredictAction`.

## predictionStats

```lua
Open77.motion.predictionStats()
```

`GAME`: Requires a live game instance.

Read bounded action-reaction prediction counters.

**Experimental.** Requires player.motion.project. Returns JSON with pending/capacity and melee, slam, hack counters (predicted, adopted, refuted, refused), last adoption/refutation times in milliseconds and lastPredictionQpc on the motion trace clock. Refused counts native admission attempts; the support resource separately counts earlier policy refusals. Counters are process-local and reading them sends no network traffic. See [Prediction](../../01%20Documentation/prediction.md) for the per-family counters of `Open77.prediction.stats`.

Returns: `JSON string or nil`, `reason on refusal`

```lua
local counters, reason = Open77.motion.predictionStats()
if counters then print(counters) else print(reason) end
```

[Prediction guide](../../01%20Documentation/prediction.md)

Registered in `ResourceHost.cpp` (line 24893) as `LuaMotionPredictionStats`.

## refutePrediction

```lua
Open77.motion.refutePrediction(request)
```

`GAME`: Requires a live game instance.

Blend a refused presentation-only prediction back to the live pose.

**Experimental.** Requires player.motion.project. Call it when the server refuses the action a prediction was made for (a refused permit, a refused target, a knockdown the arbiter rejected). request is the ID predictAction returned. An unadopted prediction stops through a 150 ms graph exit instead of playing its whole fall; a prediction the authoritative lease already adopted is left alone and the call answers prediction_not_owned. A prediction that already timed out locally is still blended back while it is kept for a late verdict. This API sends no network message and changes no health, position or authority.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `request` | `integer` | required | — |

Returns: `true on success`, `nil and a reason on refusal (invalid_request, prediction_not_owned, native_cleanup_pending, permission_denied:player.motion.project, motion_backend_unavailable)`

```lua
-- The server refused this target of our slam: blend its predicted fall back.
local ok, reason = Open77.motion.refutePrediction(request)
if not ok and reason ~= 'prediction_not_owned' then print(reason) end
```

[Prediction guide](../../01%20Documentation/prediction.md)

Registered in `ResourceHost.cpp` (line 24892) as `LuaMotionRefutePrediction`.

## state

```lua
Open77.motion.state(request)
```

`GAME`: Requires a live game instance.

Read the phase of this resource's native reaction.

Requires player.motion.project. Phases include pending, active and recovering; an expired/unknown/foreign ID returns nil, reason. active is a native PSM observation for the owner, not proof of physical displacement or the observer rendering. The support resource waits for active before acknowledging server motion.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `request` | `integer` | required | — |

Returns: `phase string`, `nil, reason on rejection`

```lua
local phase, err = Open77.motion.state(request)
if not phase then print(err) else print(phase) end
```

[Cyberware authority & motion guide](../../01%20Documentation/cyberware.md)

Registered in `ResourceHost.cpp` (line 24895) as `LuaMotionState`.

## stop

```lua
Open77.motion.stop(request)
```

`GAME`: Requires a live game instance.

Request cleanup of this resource's native reaction.

Requires player.motion.project. Pending cancellation is latched so cleanup retries cannot later activate it. Cancellation removes the owned status/action but may leave native momentum and get-up in progress. It does not instantly freeze velocity, revive a player, or cancel another resource's request.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `request` | `integer` | required | — |

Returns: `true`, `nil, reason on rejection`

```lua
-- Cancel this resource's knockdown reaction early
local ok, err = Open77.motion.stop(request)
if not ok then print(err) end
```

[Cyberware authority & motion guide](../../01%20Documentation/cyberware.md)

Registered in `ResourceHost.cpp` (line 24896) as `LuaMotionStop`.
