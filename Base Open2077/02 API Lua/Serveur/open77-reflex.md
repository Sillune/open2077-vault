# Open77.reflex (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-reflex.md) · [Page web](https://open2077.net/docs/api/server/open77-reflex)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

6 functions.

## cancel

```lua
Open77.reflex.cancel(playerId)
```

`NETWORK`: Uses the network backend.

Ends a player's running overdrive now; the grant and its cooldown are unchanged.

Requires `players.reflex.manage`. The client stops the boost on the next frame and every attached layer and the nameplate marker clear. No activation running answers `no_activation`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `{ ok = true }, or nil`, `reason`

```lua
-- A referee ends a boost mid-round
RegisterCommand("stopboost", function(source, args)
    local target = tonumber(args[1])
    local result, reason = Open77.reflex.cancel(target)
    print(result and "stopped" or reason)
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8027) as `LuaResourceRuntime.Bootstrap`.

## capabilities

```lua
Open77.reflex.capabilities()
```

`NETWORK`: Uses the network backend.

The supported profile, the two tiers and the advertised boundary.

Requires `players.reflex.read`. Reports `profile = "reflex_overdrive"`, `sharedRealTimeWorld = true`, `slowsBullets = false`, `slowsOtherPlayers = false`, `touchesTimeDilation = false` and `maximumDurationMs = 15000` -- the boundary a server maker can quote to players.

Returns: `capabilities table`

```lua
local caps = Open77.reflex.capabilities()
assert(caps.touchesTimeDilation == false)
print(caps.profile, caps.maximumDurationMs)
```

Registered in `LuaResourceRuntime.cs` (line 8029) as `LuaResourceRuntime.Bootstrap`.

## current

```lua
Open77.reflex.current(playerId)
```

`NETWORK`: Uses the network backend.

The player's overdrive activity and projection, or nil without a grant.

Requires `players.reflex.read`. Returns `projection` (`status` is `pending`, `ready` or `removed`, plus the `definition`), `phase`, `charges`, `remainingMs`, `cooldownUntil`, `activation` and `ownedByCaller`, a server-derived boolean for the calling resource that is accurate across projection restarts.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `activity table, or nil`

```lua
-- Only show the HUD hint once the native side is ready
local activity = Open77.reflex.current(player)
if activity and activity.projection.status == "ready" then
    Open77.chat.send(player, "Overdrive ready: tap X")
end
```

Registered in `LuaResourceRuntime.cs` (line 8028) as `LuaResourceRuntime.Bootstrap`.

## define

```lua
Open77.reflex.define(definition)
```

`NETWORK`: Uses the network backend.

Registers a reflex-overdrive definition: a tier plus its economy.

Requires `players.reflex.define`. `profile` is exactly `reflex_overdrive`; `config` picks `tier` (`reflex` or `reflex_heavy`), `inputKey` (the default the client registers as the rebindable `reflex_overdrive` action), `presentation` (`native`, `silent`, `none`), `durationMs` 500-15000 (never longer than `cooldownMs`), `cooldownMs` 1000-600000, `maxCharges` 1-3, `chargeRegenMs`, `staminaCost` 0-300 and `heatCost` 0-100. A definition cannot name a stat: the client owns the two tier plans and their ceilings. Out-of-range values are refused, not clamped. Versions are immutable. No clock is touched anywhere in this feature.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `definition` | `table {id, version, profile, config}` | required | — |

Returns: `{ ok = true }, or nil`, `reason`

```lua
-- Define once at start; grant per player later
CreateThread(function()
    local result, reason = Open77.reflex.define({
        id = "myserver.overdrive", version = 1, profile = "reflex_overdrive",
        config = {
            tier = "reflex", inputKey = "x", presentation = "native",
            durationMs = 8000, cooldownMs = 20000, maxCharges = 1,
            chargeRegenMs = 30000, staminaCost = 25,
        },
    })
    assert(result and result.ok, reason)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8024) as `LuaResourceRuntime.Bootstrap`.

## grant

```lua
Open77.reflex.grant(playerId, definition)
```

`NETWORK`: Uses the network backend.

Grants a defined overdrive to one player for the session.

Requires `players.reflex.manage`. Acceptance is entitlement, not readiness: check `current(player).projection.status == "ready"` before presenting it as usable. Needs the bound cyberware character and an alive, ready, unmounted body -- a grant sent before the player is incarnated is refused `body_unavailable`. Only the granting resource may revoke or cancel it. A session capability, not purchased equipment.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `definition` | `string (definition id)` | required | — |

Returns: `{ ok = true }, or nil`, `reason`

```lua
-- Grant when the player buys the implant
RegisterNetEvent("shop:buyOverdrive", function()
    local result, reason = Open77.reflex.grant(source, "myserver.overdrive")
    if not result then Open77.chat.send(source, "Not granted: " .. tostring(reason)) end
end)
```

Registered in `LuaResourceRuntime.cs` (line 8025) as `LuaResourceRuntime.Bootstrap`.

## revoke

```lua
Open77.reflex.revoke(playerId)
```

`NETWORK`: Uses the network backend.

Removes this resource's overdrive grant from a player.

Requires `players.reflex.manage`. Paid implants are untouched; a running boost ends on the release path. Another resource's grant answers `grant_owned_by_another_resource`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `{ ok = true }, or nil`, `reason`

```lua
-- Lose the implant on death in a hardcore mode
AddEventHandler("playerDied", function(player)
    Open77.reflex.revoke(player)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8026) as `LuaResourceRuntime.Bootstrap`.
