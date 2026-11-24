# Open77.dash (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-dash.md) · [Page web](https://open2077.net/docs/api/server/open77-dash)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

6 functions.

## cancel

```lua
Open77.dash.cancel(playerId, activationId)
```

`NETWORK`: Uses the network backend.

Cancels a player's active Dash by its activation id.

Requires `players.dash.manage`. Cancellation keeps bounded server movement ownership during native recovery, until the correlated native-exit acknowledgement or a 1500 ms deadline; `current().phase` reports `recovering` in that window. The charge and stamina spent on acceptance are not refunded.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `activationId` | `string` | required | — |

Returns: `{ ok = true }, or nil`, `reason on failure`

```lua
-- The player entered a safe zone mid-dash
RegisterNetEvent("safezone:entered", function()
    local activity = Open77.dash.current(source)
    if activity and activity.activation then Open77.dash.cancel(source, activity.activation) end
end)
```

Registered in `LuaResourceRuntime.cs` (line 8019) as `LuaResourceRuntime.Bootstrap`.

## capabilities

```lua
Open77.dash.capabilities()
```

`NETWORK`: Uses the network backend.

Reports the supported Dash profile and implementation limits.

Requires `players.dash.read`. The one `profile` (`dash`), the configuration bounds `define` enforces, the presentation presets and the movement model. Read it to build a definition editor rather than hard-coding the table in [Dash / Air Dash](https://open2077.net/docs/api/server/dash.md).

Returns: `capabilities table`, `nil, reason on failure`

```lua
-- Read the bounds before building a definition editor
local caps = Open77.dash.capabilities()
if caps then print(caps.profile) end
```

Registered in `LuaResourceRuntime.cs` (line 8021) as `LuaResourceRuntime.Bootstrap`.

## current

```lua
Open77.dash.current(playerId)
```

`NETWORK`: Uses the network backend.

Reads a player's Dash activity and projection state.

Requires `players.dash.read`. Answers `{ projection, charges, cooldownUntil, nextChargeAt, airUsed, activation, phase, ownedByCaller }` -- times in server monotonic milliseconds -- or `nil` when the player holds no Dash. `ownedByCaller` is server-derived and stays accurate across projection restarts; a definition id or revision alone is not proof of ownership.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `activity table, or nil when absent`, `reason on failure`

```lua
-- Only present the dash key once the projection is ready
local activity = Open77.dash.current(playerId)
if activity and activity.projection.status == "ready" then
    TriggerClientEvent("dash:ready", playerId)
end
```

Registered in `LuaResourceRuntime.cs` (line 8020) as `LuaResourceRuntime.Bootstrap`.

## define

```lua
Open77.dash.define(definition)
```

`NETWORK`: Uses the network backend.

Registers an immutable, versioned Dash definition owned by this resource.

Requires `players.dash.define`. `definition` is `{ id, version, profile = "dash", config }`; `config` takes `inputKey`, `allowGround`, `allowAir`, `requireDoubleJump`, `movementProfile = "native"`, `presentation` (`native` | `silent` | `none`), `staminaCost` (0..300), `cooldownMs` (300..600000), `maxCharges` (1..3), `chargeRegenMs`, `landingRearmMs` (150..1000), `maxAirborneMs` (100..10000) and `maxFallSpeed` (0.1..30). Speed, distance and immunity are not configurable. Ownership comes from the resource, never from a supplied owner name. See [Dash / Air Dash](https://open2077.net/docs/api/server/dash.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `definition` | `table` | required | — |

Returns: `{ ok = true }, or nil`, `reason on failure`

```lua
assert(Open77.dash.define({ id = "myserver.air_dash", version = 1, profile = "dash",
  config = { inputKey = "ctrl", allowGround = false, allowAir = true, staminaCost = 20,
    cooldownMs = 700, maxCharges = 1, chargeRegenMs = 2500, maxFallSpeed = 30 } }))
```

Registered in `LuaResourceRuntime.cs` (line 8016) as `LuaResourceRuntime.Bootstrap`.

## grant

```lua
Open77.dash.grant(playerId, definitionId)
```

`NETWORK`: Uses the network backend.

Grants a player this resource's Dash capability and starts its native projection.

Requires `players.dash.manage`. A success means pending native work, not a usable Dash: read `current(player).projection.status == "ready"` before presenting it. Grants require the authenticated cyberware character binding and an alive, ready, unmounted body. They are session capabilities -- no implant is purchased or replaced -- and cooldown or charge debt survives revoke and regrant within the session.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `definitionId` | `string` | required | — |

Returns: `{ ok = true }, or nil`, `reason on failure`

```lua
-- The player bought the dash implant
RegisterNetEvent("shop:buyDash", function()
    local result, reason = Open77.dash.grant(source, "myserver.dash")
    if not result then Open77.chat.send(source, "Not granted: " .. reason) end
end)
```

Registered in `LuaResourceRuntime.cs` (line 8017) as `LuaResourceRuntime.Bootstrap`.

## revoke

```lua
Open77.dash.revoke(playerId)
```

`NETWORK`: Uses the network backend.

Removes this resource's Dash grant from a player.

Requires `players.dash.manage`. Only the calling resource's capability is removed; another resource's grant and every installed implant stay. Charge and cooldown debt are retained for the session.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `{ ok = true }, or nil`, `reason on failure`

```lua
-- Arrested: no more dashing
Open77.dash.revoke(playerId)
```

Registered in `LuaResourceRuntime.cs` (line 8018) as `LuaResourceRuntime.Bootstrap`.
