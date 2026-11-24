# Open77.hacking (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-hacking.md) · [Page web](https://open2077.net/docs/api/server/open77-hacking)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

12 functions.

## action

```lua
Open77.hacking.action(actionId)
```

`NETWORK`: Uses the network backend.

Reads one hacking action by id: the current upload or its bounded terminal receipt.

Requires `players.hacking.read`. A completed, blocked or cancelled action is kept for a bounded time so a resource that only holds the id can still learn how it ended; after that the answer is `nil`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `actionId` | `string` | required | — |

Returns: `action table, or nil when unknown or expired`, `reason on failure`

```lua
-- Report how the upload ended, from the id kept when it started
local action = Open77.hacking.action(actionId)
if action then print("upload status:", action.status) end
```

Registered in `LuaResourceRuntime.cs` (line 8044) as `LuaResourceRuntime.Bootstrap`.

## cancel

```lua
Open77.hacking.cancel(actionId)
```

`NETWORK`: Uses the network backend.

Cancels this provider's active upload.

Requires `players.hacking.cancel`. Only an upload this resource started can be cancelled; the victim is released and the actor keeps whatever the acceptance already cost.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `actionId` | `string` | required | — |

Returns: `{ ok = true }, or nil`, `reason on failure`

```lua
-- The victim entered a safe zone: cancel our upload on them
RegisterNetEvent("safezone:entered", function()
    if activeUploads[source] then Open77.hacking.cancel(activeUploads[source]) end
end)
```

Registered in `LuaResourceRuntime.cs` (line 8052) as `LuaResourceRuntime.Bootstrap`.

## capabilities

```lua
Open77.hacking.capabilities()
```

`NETWORK`: Uses the network backend.

Reports the hacking schema, profiles, authority model and hard limits.

Requires `players.hacking.read`. The grade schema each `define*` accepts, the six hack kinds, the evidence model (clients submit intent and visibility only; the server decides) and the bounds -- read it instead of copying the tables out of [Hacking and counterplay](https://open2077.net/docs/api/server/hacking.md).

Returns: `capabilities table`, `nil, reason on failure`

```lua
-- Print the hack kinds this server supports at startup
local caps = Open77.hacking.capabilities()
if caps then for _, kind in ipairs(caps.kinds or {}) do print(kind) end end
```

Registered in `LuaResourceRuntime.cs` (line 8042) as `LuaResourceRuntime.Bootstrap`.

## clearProtection

```lua
Open77.hacking.clearProtection(playerId)
```

`NETWORK`: Uses the network backend.

Clears only this provider's protection scope on a player.

Requires `players.hacking.policy`. Another provider's safe-area or resistance scope on the same player is untouched.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `{ ok = true }, or nil`, `reason on failure`

```lua
-- Leaving the safe zone lifts our protection scope
RegisterNetEvent("safezone:left", function()
    Open77.hacking.clearProtection(source)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8056) as `LuaResourceRuntime.Bootstrap`.

## configureProtection

```lua
Open77.hacking.configureProtection(playerId, options)
```

`NETWORK`: Uses the network backend.

Owns a safe-area or electrical-resistance scope on a player's current body.

Requires `players.hacking.policy`. `options.safeArea = true` refuses every upload against the player; `options.resistance` (0..1) scales electrical damage. The scope belongs to this provider and to the current body: a new body needs a new call.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `options` | `table { safeArea?, resistance? }` | required | — |

Returns: `{ ok = true }, or nil`, `reason on failure`

```lua
-- Nobody can be hacked inside the safe zone
RegisterNetEvent("safezone:entered", function()
    Open77.hacking.configureProtection(source, { safeArea = true })
end)
```

Registered in `LuaResourceRuntime.cs` (line 8053) as `LuaResourceRuntime.Bootstrap`.

## define

```lua
Open77.hacking.define(definition)
```

`NETWORK`: Uses the network backend.

Registers this resource's hack definition and its grades.

Requires `players.hacking.define`. `definition` is `{ id, version, grades = { { id, range, uploadMs, staminaCost, cooldownMs, damage, statusMs, recoveryMs, lockHacking, nonlethal, kind? } } }`; `kind` selects Short Circuit, Overheat, Cyberware Malfunction, Cripple Movement, Reboot Optics or Weapon Glitch, six kinds of one warned, interruptible upload. Register the matching implant through `Open77.cyberware.define` with the same provider, id, version and grade ids, in the `operating_system` slot. See [Hacking and counterplay](https://open2077.net/docs/api/server/hacking.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `definition` | `table` | required | — |

Returns: `{ ok = true }, or nil`, `reason on failure`

```lua
-- A training-grade Short Circuit quickhack
assert(Open77.hacking.define({
    id = "myserver.short_circuit", version = 1,
    grades = {
        { id = "training", range = 20, uploadMs = 2000, staminaCost = 20, cooldownMs = 5000,
          damage = 25, statusMs = 750, recoveryMs = 4000, lockHacking = false, nonlethal = true },
    },
}))
```

Registered in `LuaResourceRuntime.cs` (line 8039) as `LuaResourceRuntime.Bootstrap`.

## defineIce

```lua
Open77.hacking.defineIce(definition)
```

`NETWORK`: Uses the network backend.

Registers this resource's Self-ICE definition and its grades.

Requires `players.hacking.define`. Grades carry `charges` and `rechargeMs`; a victim whose Self-ICE has a charge blocks an incoming upload and spends it. The matching implant lives in the `self_ice` slot, profile `self_ice`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `definition` | `table` | required | — |

Returns: `{ ok = true }, or nil`, `reason on failure`

```lua
-- A Self-ICE that blocks one upload every 15 seconds
assert(Open77.hacking.defineIce({
    id = "myserver.self_ice", version = 1,
    grades = { { id = "training", charges = 1, rechargeMs = 15000 } },
}))
```

Registered in `LuaResourceRuntime.cs` (line 8040) as `LuaResourceRuntime.Bootstrap`.

## definePurge

```lua
Open77.hacking.definePurge(definition)
```

`NETWORK`: Uses the network backend.

Registers this resource's active-purge definition and its grades.

Requires `players.hacking.define`. Grades carry `staminaCost`, `cooldownMs`, `allowSelf`, `allowAlly`, `range`, `cancelUploads` and `removeStatuses`. The matching implant lives in the `purge` slot, profile `active_purge`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `definition` | `table` | required | — |

Returns: `{ ok = true }, or nil`, `reason on failure`

```lua
-- A purge that also clears an ally within 10 m
assert(Open77.hacking.definePurge({
    id = "myserver.purge", version = 1,
    grades = { { id = "training", staminaCost = 15, cooldownMs = 8000, allowSelf = true, allowAlly = true,
                 range = 10, cancelUploads = true, removeStatuses = true } },
}))
```

Registered in `LuaResourceRuntime.cs` (line 8041) as `LuaResourceRuntime.Bootstrap`.

## purge

```lua
Open77.hacking.purge(playerId, targetId, definitionId, gradeId, options)
```

`NETWORK`: Uses the network backend.

Purges uploads and statuses on a target through the actor's active-purge implant.

Requires `players.hacking.activate`. Goes through the same defence authority as a player's own purge input: the actor must own the grade, the target must be self or an ally the grade allows, and stamina and cooldown are spent. `options.operationId` is required, as for `start`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `targetId` | `integer` | required | — |
| `definitionId` | `string` | required | — |
| `gradeId` | `string` | required | — |
| `options` | `table { operationId }` | required | — |

Returns: `{ ok = true, actionId }, or nil`, `reason on failure`

```lua
-- A medic purges a team-mate's statuses from a command
RegisterCommand("purge", function(source, args)
    local target = tonumber(args[1]) or source
    local result, reason = Open77.hacking.purge(source, target, "myserver.purge", "training",
        { operationId = ("purge:%d:%d"):format(source, GetGameTimer()) })
    if not result then Open77.chat.send(source, "Purge refused: " .. reason) end
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 8049) as `LuaResourceRuntime.Bootstrap`.

## start

```lua
Open77.hacking.start(playerId, targetId, definitionId, gradeId, options)
```

`NETWORK`: Uses the network backend.

Starts an entitled hack upload from one player against a target.

Requires `players.hacking.activate`. The actor must own the matching installed implant grade; the server checks range, visibility evidence, stamina, cooldown, Self-ICE and protection scopes, then runs the warned upload. `options.operationId` is REQUIRED and must be stable across retries of the same operation -- the same id with different arguments is rejected. Progress arrives on `onHackingTransition`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `targetId` | `integer` | required | — |
| `definitionId` | `string` | required | — |
| `gradeId` | `string` | required | — |
| `options` | `table { operationId }` | required | — |

Returns: `{ ok = true, actionId }, or nil`, `reason on failure`

```lua
-- A scripted hack from the player against the target they aimed at
RegisterNetEvent("hack:request", function(targetId)
    local result, reason = Open77.hacking.start(source, targetId, "myserver.short_circuit", "training",
        { operationId = ("hack:%d:%d"):format(source, GetGameTimer()) })
    if not result then return Open77.chat.send(source, "Hack refused: " .. reason) end
    activeUploads[source] = result.actionId
end)
```

Registered in `LuaResourceRuntime.cs` (line 8046) as `LuaResourceRuntime.Bootstrap`.

## state

```lua
Open77.hacking.state(playerId)
```

`NETWORK`: Uses the network backend.

Reads a player's authoritative hacking state.

Requires `players.hacking.read`. Includes the active upload if any and the remaining status windows: `cyberwareSuspendedMs`, `frozenMs`, `malfunctionMs`, `malfunctionBlocks`, `crippledMs`, `crippleHeavy`, `blindedMs` and `weaponGlitchedMs`. This is the server's ledger, not the victim's rendering.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `state table`, `nil, reason on failure`

```lua
-- Is the player still blinded?
local state = Open77.hacking.state(playerId)
if state and state.blindedMs > 0 then print("blind for another", state.blindedMs, "ms") end
```

Registered in `LuaResourceRuntime.cs` (line 8043) as `LuaResourceRuntime.Bootstrap`.

## statuses

```lua
Open77.hacking.statuses(playerId)
```

`NETWORK`: Uses the network backend.

Lists the owned statuses currently affecting a player.

Requires `players.hacking.read`. Same answer as `Open77.statuses.list`: every status any provider applied to this player, with its kind, provider, action id and deadline.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `array of status tables`, `nil, reason on failure`

```lua
-- List every status currently on a player
for _, status in ipairs(Open77.hacking.statuses(playerId) or {}) do
    print(status.kind, status.provider, status.deadline)
end
```

Registered in `LuaResourceRuntime.cs` (line 8045) as `LuaResourceRuntime.Bootstrap`.
