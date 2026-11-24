# Open77.players (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-players.md) · [Page web](https://open2077.net/docs/api/server/open77-players)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

88 functions.

## all

```lua
Open77.players.all()
```

`NETWORK`: Uses the network backend.

Every authenticated player id, ascending.

No permission. `GetPlayers` is the same function. This is the host's roster, so a gamemode no longer needs its own copy assembled from join and leave events.

Returns: `array of player ids`

```lua
-- Announce a restart to everyone currently connected
RegisterCommand("announce", function(source, args)
    local text = table.concat(args, " ")
    for _, playerId in ipairs(Open77.players.all()) do
        Open77.notifications.send(playerId, { type = "warning", message = text })
    end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8733) as `LuaResourceRuntime.Bootstrap`.

## attach

```lua
Open77.players.attach(playerId, options, [expectedRevision])
```

`NETWORK`: Uses the network backend.

Creates or updates an experimental persistent player binding.

Requires `players.attachments.control`. Options: `{ parentType = "player"|"vehicle", parentId, bone?, offset?, rotation?, pose? }`. Bone defaults to the parent root. Offset axes are metres within ±1000; rotation axes are degrees within ±360. Parent/child must be ready and in the same bucket; child cannot be in a vehicle. Rejects cycles, busy participants and foreign ownership. Enforce consent and distance in your own server resource. Omit pose: its client projection is unsupported. Acceptance returns pending state, not proof of rendering; preparation times out after 15 seconds. Active bindings have no duration cap. Optional expectedRevision compares current revision, or 0 for no binding. Resource stop, disconnect, bucket/parent/incarnation changes and vehicle entry release the binding. See [persistent player attachments](../../01%20Documentation/player-interactions.md#persistent-player-attachments-experimental). Experimental: alive-player root/exported-slot following only; arbitrary skeleton bones, corpse transport and custom-bank poses are not reliable supported workflows. Requires matching protocol 1.50 and `open77_player_interactions`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `options` | `table` | required | — |
| `expectedRevision` | `integer` | optional | — |

Returns: `state table, or nil`, `reason`

```lua
local state, reason = Open77.players.attach(playerId, {
    parentType = "vehicle", parentId = vehicleId,
    offset = { x = 0, y = -2, z = 0.5 },
}, 0)
```

Registered in `LuaResourceRuntime.cs` (line 8893) as `PlayerAttachmentOp`.

## ban

```lua
Open77.players.ban(playerId, [reason], [durationSeconds])
```

`NETWORK`: Uses the network backend.

Bans a player's device identity through the master and disconnects them now.

Requires `players.ban`. `BanPlayer` is the same function. Resolves the authenticated device identity and submits a **server-scoped** ban to the master -- different from `Open77.access.ban`, which writes the server's own local list. `durationSeconds` must be strictly positive; omitting it bans permanently. Reasons accept up to 512 UTF-8 bytes. Central publication is best-effort and asynchronous while the local kick is queued immediately.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `reason` | `string` | optional | — |
| `durationSeconds` | `number` | optional | — |

Returns: `true, or false`, `reason: permission_denied:players.ban, invalid_player_id, invalid_reason, invalid_duration, server_unavailable, ban_failed`

```lua
-- /ban <id> <days> <reason>: a temporary ban published through the master
RegisterCommand("ban", function(source, args)
    local target, days = tonumber(args[1]), tonumber(args[2])
    if not target or not days then return Open77.chat.send(source, "Usage: /ban <id> <days> <reason>") end
    local reason = table.concat(args, " ", 3)
    local ok, err = Open77.players.ban(target, reason, days * 86400)
    if not ok then Open77.chat.send(source, "Ban failed: " .. err) end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8759) as `LuaResourceRuntime.Bootstrap`.

## clearTasks

```lua
Open77.players.clearTasks(playerId)
```

`NETWORK`: Uses the network backend.

Ends this resource's motion lease, animation playback and freeze claim on one player.

Requires any of `players.motion.control`, `players.animations.control`, `players.life.freeze`; each half runs only under its own string. FiveM's `ClearPedTasksImmediately`, composed on the three services that already keep per-owner claims and already refuse to release another owner's. Answers `{ motion, animation, freeze, clear }`: `cancelled` / `stopped` / `released` when something of yours ended, `none` when you held nothing, `not_owned` when another resource holds it (left alone), or the owning service's reason (`transition_in_progress` on a freeze during a revive); `clear` is `true` when nothing of yours is left on the body. The animation half covers every playback shape for that player -- emote, placed posture, raw clip, sequence. `nil, reason` only for a bad id or a resource holding none of the three capabilities.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `{ motion, animation, freeze, clear }, or nil`, `reason`

```lua
-- /free <id>: whatever this resource was doing to them, stop it.
RegisterCommand("free", function(source, args)
    local target = tonumber(args[1])
    if not target then return end
    local result = Open77.players.clearTasks(target)
    if result and not result.clear then
        print(("still held: freeze=%s"):format(result.freeze))
    end
end, true)
```

Registered in `LuaResourceRuntime.Players6.cs` (line 310) as `LuaResourceRuntime.Bootstrap`.

## closest

```lua
Open77.players.closest(anchor, [options])
```

`NETWORK`: Uses the network backend.

The nearest player to a point or to another player.

No permission. The first entry `Open77.players.nearby` would return, with the same anchor rules and the same options plus `radius`, which it takes in the options table because it has no positional radius. It answers a **bare nil** -- no second value -- when the search worked and nobody matched, and `nil, reason` when it failed: that is what separates an empty street from a bad argument.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `anchor` | `integer (player id) | table { x, y, z }` | required | — |
| `options` | `table { radius?, bucket?, includeSelf?, limit?, maxAgeMs? }` | optional | — |

Returns: `nearest entry, or nil when nobody matched`, `reason, only on failure`

```lua
-- /rob: rob the closest player within 3 metres
RegisterCommand("rob", function(source)
    local victim = Open77.players.closest(source, { radius = 3 })
    if not victim then return Open77.chat.send(source, "Nobody close enough.") end
    Open77.chat.send(victim.playerId, Open77.players.name(source) .. " is robbing you!")
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 8741) as `LuaResourceRuntime.Bootstrap`.

## damage

```lua
Open77.players.damage(playerId, amount, [options])
```

`NETWORK`: Uses the network backend.

Applies authoritative damage to a player.

Requires `players.stats.apply` (the older spelling `players.damage.apply` is still accepted by the runtime but is not the catalogued name). Options are `attacker`, `cause` (or its alias `kind`) and `weapon`. The damage still passes the combat multipliers, team and scope rules, and every installed `Open77.combat.onDamage` arbiter.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `amount` | `number` | required | — |
| `options` | `table { attacker?, cause?, weapon? }` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- Fall damage reported by a zone trigger: 20 points, no attacker
RegisterNetEvent("parkour:badLanding", function()
    Open77.players.damage(source, 20, { cause = "fall" })
end)
```

Registered in `LuaResourceRuntime.cs` (line 8874) as `LuaResourceRuntime.Bootstrap`.

## detach

```lua
Open77.players.detach(playerId, [expectedRevision])
```

`NETWORK`: Uses the network backend.

Releases this resource's persistent player binding.

Requires `players.attachments.control`. Optional expectedRevision rejects a stale write. Releases only the caller's binding; foreign ownership is refused. Listen for `onPlayerAttachmentChanged(state)` on the server for phase/reason changes. See [persistent player attachments](../../01%20Documentation/player-interactions.md#persistent-player-attachments-experimental). Experimental: alive-player root/exported-slot following only; arbitrary skeleton bones, corpse transport and custom-bank poses are not reliable supported workflows. Requires matching protocol 1.50 and `open77_player_interactions`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `expectedRevision` | `integer` | optional | — |

Returns: `true, or nil`, `reason`

```lua
local current = Open77.players.getAttachment(playerId)
if current then Open77.players.detach(playerId, current.revision) end
```

Registered in `LuaResourceRuntime.cs` (line 8896) as `PlayerAttachmentOp`.

## disconnect

```lua
Open77.players.disconnect(playerId, [reason])
```

`NETWORK`: Uses the network backend.

Disconnects a player with a reason they see.

Requires `players.disconnect`. `DropPlayer` is the same function. The reason is trimmed, must contain no control characters and is limited to 127 UTF-8 bytes so the transport can deliver it without truncation; omitting it uses `Disconnected by server.`. The closure is deferred until the current Lua callback returns, and repeated calls for the same player in one tick are idempotent -- the first reason wins.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `reason` | `string` | optional | — |

Returns: `true, or false`, `reason: permission_denied:players.disconnect, invalid_player_id, invalid_reason, player_not_found, server_unavailable`

```lua
-- /kick <id> <reason>
RegisterCommand("kick", function(source, args)
    local target = tonumber(args[1])
    if not target then return end
    local reason = table.concat(args, " ", 2)
    Open77.players.disconnect(target, reason ~= "" and reason or "Kicked by an admin.")
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8757) as `LuaResourceRuntime.Bootstrap`.

## distance

```lua
Open77.players.distance(a, b)
```

`NETWORK`: Uses the network backend.

The distance in metres between two players, two points, or one of each.

No permission. Each argument is a player id (an integer) or a point (a table with numeric `x`, `y`, `z`); an Open77 `vector3` is a point, because the shared vector type is a plain table with real `x`/`y`/`z` fields. Three-dimensional. A player's position comes from their last accepted snapshot whatever its age -- call `Open77.players.get` first if the age matters to the decision you are about to make. `position_unknown` means the player exists but has never reported a transform.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `a` | `integer (player id) | table { x, y, z }` | required | — |
| `b` | `integer (player id) | table { x, y, z }` | required | — |

Returns: `number (metres), or nil`, `reason: invalid_argument, invalid_position, player_not_found, position_unknown, sessions_unavailable`

```lua
-- Handcuffs only work at arm's length
RegisterCommand("cuff", function(source, args)
    local target = tonumber(args[1])
    if not target then return end
    local metres = Open77.players.distance(source, target)
    if not metres or metres > 2.0 then
        return Open77.chat.send(source, "Get closer.")
    end
    Open77.players.setFrozen(target, true)
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 8756) as `LuaResourceRuntime.Bootstrap`.

## endpoint

```lua
Open77.players.endpoint(playerId)
```

`NETWORK`: Uses the network backend.

The address a player is connecting from, as `ip:port`.

Requires `players.identity.sensitive` -- the only function in this API behind that capability, and the reason it exists. An endpoint is an IP address: it identifies a person outside the game, it is the one field here that can follow someone home, and in most of the world it is regulated personal data. A resource that wants it says so in its manifest, where the operator installing that resource sees it before running it. Folding it into the ungated `Open77.players.identity` would have handed every resource on the server an address list it never asked for. IPv4 renders `203.0.113.9:31337`, IPv6 `[2001:db8::1]:31337`. `endpoint_unavailable` when the transport cannot name an address it is sure of -- it never guesses one.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `"ip:port", or nil`, `reason`

```lua
-- Anti-alt check: log the address a suspicious account joined from
-- (the manifest must declare players.identity.sensitive)
AddEventHandler("onPlayerConnected", function(playerId)
    local endpoint, reason = Open77.players.endpoint(playerId)
    print(("player %d connected from %s"):format(playerId, endpoint or reason))
end)
```

Registered in `LuaResourceRuntime.PlayerConnection.cs` (line 154) as `LuaResourceRuntime.Bootstrap`.

## fade

```lua
Open77.players.fade(playerId, out, [durationMs], [options])
```

`NETWORK`: Uses the network backend.

Cover or uncover one player's screen from the server.

Requires `players.screen`. `out` true fades that player to black and HOLDS; `out` false gives the image back. The client consumes the relay frame itself and drives the same guarded native quest fade `Open77.screen` uses, so the target needs no client resource of its own, and a fade a client resource already owns is never stolen -- the request is refused instead. `durationMs` is 0..10000 (default 500); `options.timeoutMs` is 1000..60000 (default 15000) and is a ceiling the CLIENT enforces on real time, so forgetting the second call costs a pause, never a player stuck looking at black. Bracket a teleport with it: fade, move, fade back.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `out` | `boolean` | required | — |
| `durationMs` | `integer` | optional | — |
| `options` | `table { timeoutMs? }` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- Bracket a teleport with a fade so the player never sees the streaming
CreateThread(function()
    Open77.players.fade(playerId, true, 400)
    Wait(400)
    Open77.players.teleport(playerId, { x = -1442.2, y = 127.4, z = 18.0 }, { fade = false })
    Open77.players.fade(playerId, false, 400)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8859) as `LuaResourceRuntime.Bootstrap`.

## fadeIn

```lua
Open77.players.fadeIn(playerId, [durationMs])
```

`NETWORK`: Uses the network backend.

Give one player's image back.

Requires `players.screen`. `Open77.players.fade(playerId, false, durationMs)`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `durationMs` | `integer` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- The cutscene is over: give the image back
Open77.players.fadeIn(playerId, 800)
```

Registered in `LuaResourceRuntime.cs` (line 8869) as `LuaResourceRuntime.Bootstrap`.

## fadeOut

```lua
Open77.players.fadeOut(playerId, [durationMs], [options])
```

`NETWORK`: Uses the network backend.

Fade one player's screen to black and hold it.

Requires `players.screen`. `Open77.players.fade(playerId, true, durationMs, options)` under the name a spawn or teleport reads better with.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `durationMs` | `integer` | optional | — |
| `options` | `table { timeoutMs? }` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- Knocked out: black screen for five seconds, then back
CreateThread(function()
    Open77.players.fadeOut(playerId, 300)
    Wait(5000)
    Open77.players.fadeIn(playerId, 1000)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8866) as `LuaResourceRuntime.Bootstrap`.

## forceOutOfVehicle

```lua
Open77.players.forceOutOfVehicle(playerId, [vehicleId])
```

`NETWORK`: Uses the network backend.

Forces a player out of a vehicle, overriding the exit lock.

Requires `world.vehicles`. Alias of `Open77.vehicles.forcePlayerOutOfVehicle`. Omitting the vehicle ejects them from whatever they are in. The forced exit overrides `setVehicleExitLocked` and keeps the seat reserved until native confirmation or the bounded transition timeout.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `vehicleId` | `vehicle id` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- The car was sold while someone was sitting in it
Open77.players.forceOutOfVehicle(playerId)
```

Registered in `LuaResourceRuntime.cs` (line 8765) as `LuaResourceRuntime.Bootstrap`.

## get

```lua
Open77.players.get(playerId)
```

`NETWORK`: Uses the network backend.

Everything the host can answer about one player synchronously, with a dated transform.

No permission. The one read a job script makes before it acts. Unlike `Open77.players.position` it never goes quiet on a stale snapshot: it reports `ageMs` and `fresh` and lets the caller pick its own threshold -- `fresh` is true precisely when `position` would have answered, so the old semantics are still reachable from one reading. Always present: `playerId`, `bucket`, `fresh`, `ready`. Present once the player has reported: `ageMs`, `position` `{ x, y, z }`, `heading` (alias `yaw`), `velocity` `{ x, y, z }`, `speed`, `stateFlags`, `flags`, and `supportKind`/`supportId` on a moving support. Present when the host can resolve them: `name`, `identifier`, `userId`. A player who has never sent an accepted snapshot has no `position` and no `ageMs` at all, which is distinguishable from a reading that is merely old. Health (`players.stats.read`), life (`players.life.read`) and vehicle seat (`world.vehicles`) fields are **absent rather than false** without the permission that already guards `getHealth`, `getLifeState` and `getVehicleSeat`. See [the rich read](https://open2077.net/docs/api/server/server-api.md#the-rich-read-and-the-freshness-decision).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `table, or nil`, `reason: invalid_player_id, player_not_found, sessions_unavailable`

```lua
local read = Open77.players.get(playerId)
if read and read.position and (read.ageMs or 0) < 5000 then
  -- your freshness threshold, not ours
end
```

Registered in `LuaResourceRuntime.cs` (line 8739) as `LuaResourceRuntime.Bootstrap`.

## getAttachment

```lua
Open77.players.getAttachment(playerId)
```

`NETWORK`: Uses the network backend.

Reads a player's current persistent binding.

Requires `players.attachments.read`. May read another resource's binding. State includes epoch, revision, id, playerId, incarnation, bucket, parentType, parentId, parentIncarnation, bone, offset, rotation, pose, phase and reason. Preserve returned IDs as strings. Returns nil without a live binding, or nil and a reason on failure. See [persistent player attachments](../../01%20Documentation/player-interactions.md#persistent-player-attachments-experimental). Experimental: alive-player root/exported-slot following only; arbitrary skeleton bones, corpse transport and custom-bank poses are not reliable supported workflows. Requires matching protocol 1.50 and `open77_player_interactions`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `state table, or nil`, `reason on failure`

```lua
local state, reason = Open77.players.getAttachment(playerId)
```

Registered in `LuaResourceRuntime.cs` (line 8899) as `PlayerAttachmentOp`.

## getDriveByState

```lua
Open77.players.getDriveByState(playerId)
```

`NETWORK`: Uses the network backend.

Reads server-validated vehicle drive-by state.

Server; requires players.life.read and a positive connected player ID. Returns enabled, available, active, phase, sequence, duration and elapsed; non-none phases include canonical vehicle and seat. Phases are none, entering, active, exiting; active includes transitions. duration/elapsed are seconds. Actions must belong to the current canonical seat lifetime, bucket and alive player, and expire when stale. enabled is combined server policy; client-only restrictions are not broadcast. A missing action returns none/unavailable, while an unknown player returns nil, player_unavailable. Use matching Unstable 2.31.21-unstable+op77.118 client and server (protocol 1.43), including the animation assets. Passenger support was introduced in .117; car and motorcycle drivers require .118. See [Vehicle drive-by](../../01%20Documentation/drive-by.md). Adds mode (passenger, car_driver, bike_driver) and driverYaw/driverPitch/driverRoll in degrees relative to the seated pose. Drivers use front_left and phase active with zero duration/elapsed. Only interpret mode while active; inactive data defaults to passenger with zero angles.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `state table or nil`, `reason on failure`

```lua
RegisterCommand("drivebystate", function(playerId)
    local state, reason = Open77.players.getDriveByState(playerId)
    if state then print(state.phase, state.vehicle, state.seat) else print(reason) end
end, true)
```

[Passenger drive-by guide](../../01%20Documentation/drive-by.md)

Registered in `LuaResourceRuntime.DriveBy.cs` (line 69) as `LuaResourceRuntime.Bootstrap`.

## getHealth

```lua
Open77.players.getHealth(playerId)
```

`NETWORK`: Uses the network backend.

A player's health, armor and god-mode snapshot.

Requires `players.stats.read` (the older spelling `players.damage.read` is still accepted by the runtime but is not the catalogued name). `GetPlayerHealth` is the same function. New resources should prefer `Open77.stats.getHealth`, which returns the richer nested pool the client also exposes.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `health snapshot, or nil`

```lua
-- /hp <id>: an admin reads a player's health
RegisterCommand("hp", function(source, args)
    local hp = Open77.players.getHealth(tonumber(args[1]) or source)
    if hp then Open77.chat.send(source, ("Health %d / %d"):format(hp.health, hp.maxHealth)) end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8873) as `LuaResourceRuntime.Bootstrap`.

## getHoloCallEyes

```lua
Open77.players.getHoloCallEyes(playerId)
```

`NETWORK`: Uses the network backend.

Read whether any server resource holds a player's holocall eye-glow lease.

Requires players.holocall.read and an explicit connected network player ID. This reads authoritative cosmetic intent, not a native render acknowledgement. False means no lease; nil, reason means the query failed. See holocall-eyes.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `boolean enabled, or nil, reason`

```lua
-- Is the player already on a call before starting a new one?
local enabled = Open77.players.getHoloCallEyes(playerId)
if not enabled then Open77.players.setHoloCallEyes(playerId, true, { durationMs = 30000 }) end
```

[Blue holocall eyes guide](../../01%20Documentation/holocall-eyes.md)

Registered in `LuaResourceRuntime.cs` (line 8729) as `LuaResourceRuntime.Bootstrap`.

## getLifeState

```lua
Open77.players.getLifeState(playerId)
```

`NETWORK`: Uses the network backend.

A player's canonical life snapshot.

Requires `players.life.read`. `GetPlayerLifeState` is the same function. Answers a bare `nil` (no reason) without the permission, for a player with no life record yet (the continue screen) or while the life service is down. `phase` is `alive`, `dead`, `revivepending`, `respawnpending` or `recovering` -- **no underscore**, unlike the client spelling. Prefer `isDead` over comparing the string. The table carries, in the order the runtime sets them: `playerId`, `revision`, `phase`, `bucket`, `changedAtServerTick`, `killerPlayerId` and its alias `killer` (an integer, **`0` when nobody killed the player -- never `nil`**), `cause` (`unknown`, `firearm`, `melee`, `explosion`, `fall`, `vehicle`, `environment` or `script`), `weapon` (the death's weapon record string: what `kill` was given, default `resource:<name>`; `tweak:<hex>` or `""` for the damage arbiter's own verdict), `health`, `yaw`, `position { x, y, z }`, `impulse { x, y, z }`, `graceMilliseconds` and its alias `graceMs`, `flags`, and the booleans `ghosted`, `frozen`, `visible`. No `fallDamage` here -- that bit is on `Open77.players.get`. **Not every `dead` is a player going down.** The gamemode's join spawn (`weapon = "freeroam:join_spawn"`), `open77_playerstate` (`playerstate:restore`) and `open77_admin`'s `tp`/`goto`/`bring` (`open77_admin:<verb>`) all move a player through a scripted kill (`cause = "script"`) followed by a respawn; a fresh connection therefore reports `dead` once, milliseconds after the readiness gate opens. A death-driven resource reads `cause` and `weapon` and skips `script` deaths it did not cause itself. `Open77.players.teleport` moves a living player with no transition at all.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `life snapshot table { playerId, revision, phase, bucket, changedAtServerTick, killerPlayerId, killer, cause, weapon, health, yaw, position, impulse, graceMilliseconds, graceMs, flags, ghosted, frozen, visible }, or nil`

```lua
-- Count only real deaths: the join spawn and an admin move are scripted kills too
AddEventHandler("onPlayerLifeStateChanged", function(playerId, revision, phase, reason)
    if phase ~= "dead" then return end
    local life = Open77.players.getLifeState(playerId)
    if not life or life.cause == "script" then return end   -- freeroam:join_spawn, open77_admin:tp, ...
    local killer = life.killer ~= 0 and life.killer or nil   -- 0, never nil, when there is no killer
    recordDeath(playerId, killer, life.cause, life.weapon)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8771) as `LuaResourceRuntime.Bootstrap`.

## getModel

```lua
Open77.players.getModel(playerId)
```

`NETWORK`: Uses the network backend.

Read the authoritative model selection and owner's readiness.

Requires players.model.read and a connected positive integer player ID. Returns nil without an error for the original model, or a table with player, record, appearance, revision, ready, resetOnDeath and optional remainingMs. Ready is the subject's presentation acknowledgement, not proof that every observer has streamed the body. Cosmetic NPC presentation only: the real player retains identity, controls, health, equipment and saved appearance. Character.* records resolve against each client's installed game data, not a virtual allowlist. See player-models.md for ownership, events, cleanup and the known special-rig/vehicle limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `table? state`, `string? reason`

```lua
local state, reason = GetPlayerModel(playerId)
if state then print(state.record, state.revision, state.ready) end
```

[Player morphs and model lifecycle guide](../../01%20Documentation/player-models.md)

Registered in `LuaResourceRuntime.cs` (line 8704) as `LuaResourceRuntime.Bootstrap`.

## getStats

```lua
Open77.players.getStats(playerId)
```

`NETWORK`: Uses the network backend.

A player's combined health and stamina snapshot.

Requires `players.stats.read`. `GetPlayerStats` is the same function, and `Open77.stats.get` is the preferred spelling for new resources.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `stats snapshot, or nil`

```lua
-- Show health and stamina together on request
RegisterCommand("stats", function(source)
    local stats = Open77.players.getStats(source)
    if stats then
        Open77.chat.send(source, ("HP %d / %d  Stamina %d / %d"):format(
            stats.health, stats.maxHealth, stats.stamina, stats.maxStamina))
    end
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 8872) as `LuaResourceRuntime.Bootstrap`.

## getVehicleSeat

```lua
Open77.players.getVehicleSeat(playerId)
```

`NETWORK`: Uses the network backend.

A player's canonical vehicle seat assignment.

Requires `world.vehicles`. Alias of `Open77.vehicles.getPlayerSeat`. Canonical means the server's reservation, which exists before the client has mounted and survives a stream-out.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `seat assignment, or nil`

```lua
-- Passengers cannot pay the toll: find the driver of the same car
local seat = Open77.players.getVehicleSeat(playerId)
if seat and seat.seat ~= "seat_front_left" then
    local driver = Open77.vehicles.getDriver(seat.vehicleId)
    if driver then chargeToll(driver) end
end
```

Registered in `LuaResourceRuntime.cs` (line 8761) as `LuaResourceRuntime.Bootstrap`.

## getWanted

```lua
Open77.players.getWanted(playerId)
```

`NETWORK`: Uses the network backend.

The wanted policy this server has decided for a player.

Returns `{ level, maxLevel, enabled }`, where `enabled` is the dispatch switch of the bucket the player is in. It is what the SERVER decided and pushed, never a reading of the game: the server has no way to learn the engine's real heat stage. A player the server has never set answers the defaults - level 0, ceiling 5, and their bucket's dispatch setting. Requires `players.wanted.read`, which is separate from `players.wanted` because what a server has decided about a player is state a resource has no business enumerating just because it can call a function.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `table { level, maxLevel, enabled }, or nil`, `reason: permission_denied:players.wanted.read, invalid_player`

```lua
-- /wanted: show a player their current heat
RegisterCommand("wanted", function(source)
    local wanted = Open77.players.getWanted(source)
    if wanted then
        Open77.chat.send(source, ("Wanted level %d of %d"):format(wanted.level, wanted.maxLevel))
    end
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 8722) as `LuaResourceRuntime.Bootstrap`.

## heal

```lua
Open77.players.heal(playerId, amount)
```

`NETWORK`: Uses the network backend.

Heals a player by an amount.

Requires `players.stats.apply` (the older spelling `players.damage.apply` is still accepted by the runtime but is not the catalogued name). `HealPlayer` is the same function. Clamped at the canonical maximum; it does not revive a dead player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `amount` | `number` | required | — |

Returns: `true, or false`, `reason`

```lua
-- A medkit pickup heals 50 points
RegisterNetEvent("shop:useMedkit", function()
    Open77.players.heal(source, 50)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8880) as `LuaResourceRuntime.Bootstrap`.

## identifier

```lua
Open77.players.identifier(playerId)
```

`NETWORK`: Uses the network backend.

A player's durable authenticated identifier.

No permission. `GetPlayerIdentifier` is the same function. Durable across sessions, unlike the player id, which is only valid for this connection -- key persistent data on this. **The id must be an integer of 1 or more, checked before the call.** `0` or a negative id is not answered `nil`: the native throws `id must be positive`, which no `pcall` catches -- the resource is stopped with `runtime_error` and its dependants with `dependency_stopped`. A command run at the dedicated console has `source == 0`, so guard every id that may come from the console (see [A player id is an integer from 1 up](https://open2077.net/docs/api/server/server-api.md#a-player-id-is-an-integer-from-1-up----validate-it-before-every-lookup)).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer >= 1` | required | — |

Returns: `identifier, or nil`

```lua
-- Key the player's money on the durable identifier, never on the session id
AddEventHandler("onPlayerReady", function(playerId)
    playerId = tonumber(playerId)
    if not playerId or playerId < 1 then return end            -- never pass 0: it stops the resource
    local key = "money:" .. Open77.players.identifier(playerId)
    local balance = Open77.kvp.get(key, 500)
    Open77.chat.send(playerId, ("Welcome back. Balance: %d eddies"):format(balance))
end)
```

Registered in `LuaResourceRuntime.cs` (line 8731) as `LuaResourceRuntime.Bootstrap`.

## identifiers

```lua
Open77.players.identifiers(playerId)
```

`NETWORK`: Uses the network backend.

Every identifier Open77 holds for a player, as a table.

The table form of `GetPlayerIdentifiers`. Carries `open77` and `userId` (the same installation identity GUID), `name`, `fingerprint` and `joinedAt`. Since `2.31.13+op77.101`, also carries `license`, `steam` and `gog` from the verified Master ticket when available. `license` is the permanent Open77 account GUID without dashes, `steam` a lowercase hexadecimal SteamID and `gog` a decimal GOG ID. All values are strings without type prefixes; unavailable fields are omitted and read as `nil`. Account/store IDs need no permission; only `endpoint` requires `players.identity.sensitive`. Call in `onPlayerConnected` or later. Invalid IDs return `nil, "invalid_player_id"`; unknown sessions return `nil, "player_not_found"`. One qualifying store is sufficient, and a linked store ID alone is not a fresh ownership check. Use `license` for account-level persistence. No Discord identifier is exposed. See [account and store identifiers](../../01%20Documentation/connection-control.md#steam-gog-and-permanent-account-identifiers).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `table, or nil`, `reason`

```lua
-- Log what Open77 knows about a player when they join
AddEventHandler("onPlayerConnected", function(playerId)
    local ids = Open77.players.identifiers(playerId)
    if ids then print(("%s joined (account %s)"):format(ids.name, ids.userId)) end
end)
```

Registered in `LuaResourceRuntime.PlayerConnection.cs` (line 155) as `LuaResourceRuntime.Bootstrap`.

## identity

```lua
Open77.players.identity(playerId)
```

`NETWORK`: Uses the network backend.

An admitted player's authenticated identity.

No permission required. `GetPlayerIdentity` is the same function. Returns `{ userId, name, publicKey, fingerprint, joinedAt }` for an admitted player, or `nil`; the fingerprint matches the client identity dump. Since `2.31.13+op77.101`, also includes `license`, `steam` and `gog` when supplied by the verified Master ticket. `license` is the permanent Open77 account GUID without dashes, `steam` a lowercase hexadecimal SteamID and `gog` a decimal GOG ID. Keep these bare values as strings; unavailable fields are omitted. `userId` remains the installation identity GUID, while `license` is the account key across linked devices. See [account and store identifiers](../../01%20Documentation/connection-control.md#steam-gog-and-permanent-account-identifiers).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `table, or nil`

```lua
-- Look up the account behind a session
RegisterCommand("whois", function(source, args)
    local identity = Open77.players.identity(tonumber(args[1]))
    if not identity then return Open77.chat.send(source, "No such player.") end
    Open77.chat.send(source, ("%s, account %s, joined %s"):format(identity.name, identity.userId, identity.joinedAt))
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8760) as `LuaResourceRuntime.Bootstrap`.

## inBucket

```lua
Open77.players.inBucket(bucket)
```

`NETWORK`: Uses the network backend.

The authenticated players in one routing bucket, ascending.

No permission. `GetPlayersInBucket` is the same function. Answers `false, invalid_bucket` for a negative or oversized bucket.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | required | — |

Returns: `array of player ids, or false`, `invalid_bucket`

```lua
-- Start the race only when the lobby instance has at least two players
local racers = Open77.players.inBucket(RACE_BUCKET)
if #racers >= 2 then
    startRace(racers)
end
```

Registered in `LuaResourceRuntime.cs` (line 8734) as `LuaResourceRuntime.Bootstrap`.

## inScope

```lua
Open77.players.inScope(viewerId)
```

`NETWORK`: Uses the network backend.

Return the players currently in a viewer's replication scope.

Returns a table of active player IDs using the same directed relation as isInScope, including the owner. Unknown IDs return an empty table. Non-positive IDs return false, invalid_player. No visibility is granted. Querying in native code avoids a Lua call for every player in the server; revalidate delayed publications after yielding.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `viewerId` | `integer` | required | — |

Returns: `table player IDs`, `false, reason on invalid input`

```lua
for _, id in ipairs(Open77.players.inScope(viewerId)) do
    print(id)
end
```

Registered in `LuaResourceRuntime.cs` (line 8736) as `LuaResourceRuntime.Bootstrap`.

## isDead

```lua
Open77.players.isDead(playerId)
```

`NETWORK`: Uses the network backend.

Whether a player is dead, revive-pending or respawn-pending.

Requires `players.life.read`. `IsPlayerDead` is the same function. Resolved from the enum rather than the string, so it is immune to the client/server spelling split -- this is the safe way to ask.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `boolean, or nil`

```lua
-- A dead player cannot use the shop
RegisterCommand("shop", function(source)
    if Open77.players.isDead(source) then
        return Open77.chat.send(source, "You are in no state to shop.")
    end
    openShop(source)
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 8772) as `LuaResourceRuntime.Bootstrap`.

## isDriveByEnabled

```lua
Open77.players.isDriveByEnabled(playerId)
```

`NETWORK`: Uses the network backend.

Reads combined server policy for vehicle drive-by.

Server; requires players.life.read and a positive connected player ID. True means no server resource denies vehicle drive-by. It does not mean the player is in a vehicle or actively fighting, and cannot reveal client-only restrictions. Returns nil, reason for an unavailable player or denied permission. Use matching Unstable 2.31.21-unstable+op77.118 client and server (protocol 1.43), including the animation assets. Passenger support was introduced in .117; car and motorcycle drivers require .118. See [Vehicle drive-by](../../01%20Documentation/drive-by.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `boolean enabled or nil`, `reason on failure`

```lua
RegisterCommand("drivebypolicy", function(playerId)
    local enabled, reason = Open77.players.isDriveByEnabled(playerId)
    if enabled == nil then print(reason) else print("Allowed:", enabled) end
end, true)
```

[Passenger drive-by guide](../../01%20Documentation/drive-by.md)

Registered in `LuaResourceRuntime.DriveBy.cs` (line 72) as `LuaResourceRuntime.Bootstrap`.

## isFallDamageEnabled

```lua
Open77.players.isFallDamageEnabled(playerId)
```

`NETWORK`: Uses the network backend.

Whether landings still hurt this player.

Requires `players.life.read`. Answered from the server-owned life bit; unknown players, and a resource without the read permission, answer `true` because vanilla fall damage is the safe default. `Open77.players.get(id).fallDamage` carries the same bit on the rich read.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `boolean`

```lua
-- Refuse the rooftop teleport unless the course already shielded them
if Open77.players.isFallDamageEnabled(target) then
    return Open77.chat.send(source, "Arm the course first.")
end
```

Registered in `LuaResourceRuntime.Players6.cs` (line 234) as `LuaResourceRuntime.Bootstrap`.

## isFrozen

```lua
Open77.players.isFrozen(playerId)
```

`NETWORK`: Uses the network backend.

Whether a player is currently held still.

Requires `players.life.read`. `IsPlayerFrozen` is the same function. Reads the canonical life state rather than a resource's own bookkeeping, so it answers true while ANY resource holds the player -- not only this one. `Open77.players.getLifeState(id).frozen` carries the same bit.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `boolean, or nil`

```lua
-- Refuse a second /cuff on a player somebody already holds
if Open77.players.isFrozen(target) then
    return Open77.chat.send(source, "They are already restrained.")
end
```

[Freezing a player guide](../../01%20Documentation/player-freeze.md)

Registered in `LuaResourceRuntime.cs` (line 8821) as `LuaResourceRuntime.Bootstrap`.

## isGhosted

```lua
Open77.players.isGhosted(playerId)
```

`NETWORK`: Uses the network backend.

Whether a player is currently non-solid.

Requires `players.life.read`. `IsPlayerGhosted` is the same function. Reads the canonical life state rather than a resource's own bookkeeping.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `boolean, or nil`

```lua
-- Spawn protection is still on: skip the damage report
if Open77.players.isGhosted(victimId) then return end
```

Registered in `LuaResourceRuntime.cs` (line 8816) as `LuaResourceRuntime.Bootstrap`.

## isInDriveBy

```lua
Open77.players.isInDriveBy(playerId)
```

`NETWORK`: Uses the network backend.

Reports a fresh server-validated vehicle combat action.

Server; requires players.life.read and a positive connected player ID. True for entering, active or exiting. False when no fresh admitted action is available, including after death or unmount. Use getDriveByState for the exact phase, canonical vehicle and seat. Returns nil, reason for an unknown player or denied permission. Use matching Unstable 2.31.21-unstable+op77.118 client and server (protocol 1.43), including the animation assets. Passenger support was introduced in .117; car and motorcycle drivers require .118. See [Vehicle drive-by](../../01%20Documentation/drive-by.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `boolean active or nil`, `reason on failure`

```lua
RegisterCommand("indriveby", function(playerId)
    local active, reason = Open77.players.isInDriveBy(playerId)
    if active == nil then print(reason) else print("Window action:", active) end
end, true)
```

[Passenger drive-by guide](../../01%20Documentation/drive-by.md)

Registered in `LuaResourceRuntime.DriveBy.cs` (line 77) as `LuaResourceRuntime.Bootstrap`.

## isInScope

```lua
Open77.players.isInScope(viewerId, subjectId)
```

`NETWORK`: Uses the network backend.

Check whether a player is in another player's native replication scope.

Read-only directed visibility relation used by player roster and motion replication. Returns true for an active player viewing itself. Distinct players must be active, in the same routing bucket, and in the current area-of-interest graph. Unknown or disconnected players return false; invalid non-positive IDs return false, invalid_player. With deferred bootstrap, new peers remain outside scope until placement permits reconciliation. This does not grant interest or measure render visibility. Use it to filter presentation events and replays so they follow the native roster.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `viewerId` | `integer` | required | — |
| `subjectId` | `integer` | required | — |

Returns: `boolean visible`, `string reason on invalid input`

```lua
if Open77.players.isInScope(viewer, subject) then
    TriggerClientEvent("my:appearance", viewer, subject, appearance)
end
```

Registered in `LuaResourceRuntime.cs` (line 8735) as `LuaResourceRuntime.Bootstrap`.

## isModelReady

```lua
Open77.players.isModelReady(playerId)
```

`NETWORK`: Uses the network backend.

Read the model owner's native presentation acknowledgement.

Requires players.model.read and a connected positive integer player ID. Returns false for the original character or a model still preparing; nil, reason is an error. Observe onPlayerModelReady(player, revision) for asynchronous readiness and match the requested revision. This does not certify observer rendering or every animation. Cosmetic NPC presentation only: the real player retains identity, controls, health, equipment and saved appearance. Character.* records resolve against each client's installed game data, not a virtual allowlist. See player-models.md for ownership, events, cleanup and the known special-rig/vehicle limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `boolean? ready`, `string? reason`

```lua
if IsPlayerModelReady(playerId) then print('Owner model ready') end
```

[Player morphs and model lifecycle guide](../../01%20Documentation/player-models.md)

Registered in `LuaResourceRuntime.cs` (line 8706) as `LuaResourceRuntime.Bootstrap`.

## isVisible

```lua
Open77.players.isVisible(playerId)
```

`NETWORK`: Uses the network backend.

Whether a player's body is rendered by other clients.

Requires `players.life.read`. `IsPlayerVisible` is the same function. **Answered from the server-owned life bit and never probed from a client**: Cyberpunk 2.31 exposes no readable hidden flag on `entIVisualComponent`, so there is nothing in the engine to ask. Unknown players answer `true` -- absence of a life state is not invisibility. The same bit is on the life snapshot as `getLifeState(id).visible`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `boolean, or nil`

```lua
-- A hidden admin is skipped by the scoreboard
for _, playerId in ipairs(Open77.players.all()) do
    if Open77.players.isVisible(playerId) then
        table.insert(board, Open77.players.name(playerId))
    end
end
```

Registered in `LuaResourceRuntime.cs` (line 8826) as `LuaResourceRuntime.Bootstrap`.

## kick

```lua
Open77.players.kick(playerId, [reason])
```

`NETWORK`: Uses the network backend.

Alias of `Open77.players.disconnect`.

Requires `players.disconnect`. The same function under the name an admin command would use; it is not a separate, harsher action.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `reason` | `string` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- Kick anyone who is AFK for more than 15 minutes
CreateThread(function()
    while true do
        Wait(60000)
        for _, playerId in ipairs(Open77.players.all()) do
            local read = Open77.players.get(playerId)
            if read and read.speed == 0 and (idleSince[playerId] or 0) > 15 * 60 then
                Open77.players.kick(playerId, "Kicked for inactivity.")
            end
        end
    end
end)
```

Registered in `LuaResourceRuntime.cs` (line 8758) as `LuaResourceRuntime.Bootstrap`.

## kill

```lua
Open77.players.kill(playerId, [options])
```

`NETWORK`: Uses the network backend.

Kills a player authoritatively.

Requires `players.life.kill`. Options are `killer`, `cause` (default `script`), `weapon` and `impulse = { x, y, z }`. Do not call it on a player whose readiness gate has not opened -- see `Open77.ready`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `options` | `table { killer?, cause?, weapon?, impulse? }` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- /slay <id>: an admin kills a player
RegisterCommand("slay", function(source, args)
    local target = tonumber(args[1])
    if not target then return end
    local ok, reason = Open77.players.kill(target, { killer = source, cause = "script" })
    if not ok then Open77.chat.send(source, "Refused: " .. reason) end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8773) as `LuaResourceRuntime.Bootstrap`.

## lastSeen

```lua
Open77.players.lastSeen(identifier)
```

`NETWORK`: Uses the network backend.

When an identity was last on this server, online or not.

Requires `players.identity.history`. Takes the durable identifier `identifiers()` reports (the user id, not the session's player id) and answers `{ userId, name, online, lastSeenUtc, firstSeenUtc, previousSeenUtc?, joinCount, totalPlaySeconds }` -- with `playerId`, `connectedAtUtc` and `sessionSeconds` added while the identity is connected, in which case `lastSeenUtc` is now. Reads the identity directory the server persists for every admitted player; a server without one answers `history_unavailable`. Fails with `invalid_identifier`, `identity_unknown` (never admitted here), `permission_denied:players.identity.history`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `identifier` | `string (the durable user id)` | required | — |

Returns: `table, or nil`, `reason on failure`

```lua
-- /lastseen <userId>: an admin asks when somebody was last around
RegisterCommand('lastseen', function(source, args)
    local seen, reason = Open77.players.lastSeen(args[1] or '')
    if not seen then return Open77.chat.send(source, 'unknown: ' .. tostring(reason)) end
    Open77.chat.send(source, ('%s: %s (%d visits, %d h)'):format(seen.name,
        seen.online and 'online now' or seen.lastSeenUtc, seen.joinCount, math.floor(seen.totalPlaySeconds / 3600)))
end, true)
```

Registered in `LuaResourceRuntime.Identity6.cs` (line 188) as `LuaResourceRuntime.Bootstrap`.

## listAttachments

```lua
Open77.players.listAttachments()
```

`NETWORK`: Uses the network backend.

Lists this resource's persistent player bindings.

Requires `players.attachments.read`. Returns only bindings owned by the calling resource, not all bindings on the server. See [persistent player attachments](../../01%20Documentation/player-interactions.md#persistent-player-attachments-experimental). Experimental: alive-player root/exported-slot following only; arbitrary skeleton bones, corpse transport and custom-bank poses are not reliable supported workflows. Requires matching protocol 1.50 and `open77_player_interactions`.

Returns: `array of state tables, or nil`, `reason`

```lua
for _, state in ipairs(Open77.players.listAttachments() or {}) do
    print(state.playerId, state.phase)
end
```

Registered in `LuaResourceRuntime.cs` (line 8900) as `PlayerAttachmentOp`.

## locale

```lua
Open77.players.locale(playerId)
```

`NETWORK`: Uses the network backend.

The language a player's own game runs in, as their client reported it at world-ready.

Requires `players.locale.read` -- a language says where a person is from more often than not, and every read of that kind here is declared in the manifest where the operator sees it. Returns `{ code, gameLanguage, voiceLanguage, subtitles, chromium, ageMs, revision }`: `code` is a BCP-47 tag derived from the engine's text language (`en-US`, `fr-FR`, `pt-BR`, `zh-CN`); `gameLanguage`, `voiceLanguage` and `subtitles` are the three raw engine codes (`en-us`, `jp-jp`, `kr-kr`) for a script that wants the audio language or meets a code the mapping has not seen; `chromium` is the client's OS UI language (what its WebUI pages see as `navigator.language`), kept for contrast and never used for `code`; `ageMs` dates the report and `revision` counts them -- a player who re-enters a world after changing language at the main menu re-reports, and the revision moves. Pushed by the client HOST on the reserved `open77:session:locale` net event, consumed before any resource route and refused to resources on both sides, so it is what the player's own game said and nothing else -- a report, not a ledger: a modified client can lie about it. Four refusals, distinct on purpose: `invalid_player_id`, `player_not_found` for a player who is not connected, `not_reported` for one whose client has not pushed yet (a client that never reached world-ready, or an older build), and `locale_unavailable` in an embedding with no transport.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `{ code, gameLanguage, voiceLanguage, subtitles, chromium, ageMs, revision }, or nil`, `reason`

```lua
-- Greet in the player's own language once they are in the world
AddEventHandler("onPlayerReady", function(playerId)
    CreateThread(function()
        Wait(5000) -- the report follows world-ready by a few seconds
        local locale = Open77.players.locale(playerId)
        local language = locale and locale.code:sub(1, 2) or "en"
        local hello = ({ fr = "Bienvenue !", de = "Willkommen!", pt = "Bem-vindo!" })[language] or "Welcome!"
        Open77.chat.send(tonumber(playerId), hello)
    end)
end)
```

Registered in `LuaResourceRuntime.Locale6.cs` (line 93) as `LuaResourceRuntime.Bootstrap`.

## name

```lua
Open77.players.name(playerId)
```

`NETWORK`: Uses the network backend.

A player's display name.

No permission. `GetPlayerName` is the same function. Answers `nil` for an unknown or departed session. **The id must be an integer of 1 or more, checked before the call.** `0` or a negative id is not answered `nil`: the native throws `id must be positive`, which no `pcall` catches -- the resource is stopped with `runtime_error` and its dependants with `dependency_stopped`. A command run at the dedicated console has `source == 0`, so guard every id that may come from the console (see [A player id is an integer from 1 up](https://open2077.net/docs/api/server/server-api.md#a-player-id-is-an-integer-from-1-up----validate-it-before-every-lookup)).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer >= 1` | required | — |

Returns: `name, or nil`

```lua
-- /whois: works from the console only with an explicit id (a console command has source == 0)
RegisterCommand("whois", function(source, args)
    local target = tonumber(args[1]) or tonumber(source)
    if not target or target < 1 then print("whois: needs a player id"); return end
    print(Open77.players.name(target) or "unknown session")
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8730) as `LuaResourceRuntime.Bootstrap`.

## nearby

```lua
Open77.players.nearby(anchor, [radius], [options])
```

`NETWORK`: Uses the network backend.

The players within a radius of a point or of another player, nearest first.

No permission. `anchor` is a player id (an integer) or a point (a table with numeric `x`, `y`, `z`) -- never ambiguous, and a `vector3` is a point. An omitted or nil `radius` means no radius limit. Options: `bucket` (an integer narrows to it; `false` means every bucket; the default is the anchor player's own bucket, or every bucket for a point anchor), `includeSelf` (default false, ignored for a point anchor), `limit`, and `maxAgeMs`. The default keeps stale readings, because a player whose snapshots stalled is still somewhere; `maxAgeMs` is how you ask for a threshold. Entries carry `playerId`, `distance`, `bucket`, `fresh`, and `ageMs`/`name`/`position`/`heading`/`speed` where available -- deliberately not the whole of `get`, which would ask three services once per player per call. Sorted nearest first, ties broken by player id so two identical calls answer in the same order.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `anchor` | `integer (player id) | table { x, y, z }` | required | — |
| `radius` | `number` | optional | — |
| `options` | `table { bucket?, includeSelf?, limit?, maxAgeMs? }` | optional | — |

Returns: `array of entries sorted by distance, or nil`, `reason: invalid_argument, invalid_position, invalid_radius, invalid_options, invalid_bucket, invalid_limit, invalid_max_age, player_not_found, position_unknown, sessions_unavailable`

```lua
for _, witness in ipairs(Open77.players.nearby(robber, 40)) do
  Open77.notifications.send(witness.playerId, { text = ("%.0f m away"):format(witness.distance) })
end
```

Registered in `LuaResourceRuntime.cs` (line 8740) as `LuaResourceRuntime.Bootstrap`.

## observers

```lua
Open77.players.observers(subjectId)
```

`NETWORK`: Uses the network backend.

Return the viewers currently observing a player.

Returns a table of active player IDs using the same directed relation as isInScope, including the owner. Unknown IDs return an empty table. Non-positive IDs return false, invalid_player. No visibility is granted. Querying in native code avoids a Lua call for every player in the server; revalidate delayed publications after yielding.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `subjectId` | `integer` | required | — |

Returns: `table player IDs`, `false, reason on invalid input`

```lua
for _, id in ipairs(Open77.players.observers(subjectId)) do
    print(id)
end
```

Registered in `LuaResourceRuntime.cs` (line 8737) as `LuaResourceRuntime.Bootstrap`.

## ping

```lua
Open77.players.ping(playerId)
```

`NETWORK`: Uses the network backend.

The transport's measured round-trip time for one player, in milliseconds.

No capability: this is latency, not identity. Every scoreboard in the genre shows it, the player already sees their own, and a resource that could not read it would measure it with a round trip of its own. GameNetworkingSockets' smoothed estimate, refreshed roughly twice a second. Three failures, deliberately distinct: `player_not_found` for a player who is not connected, `ping_unavailable` when the transport cannot measure at all (the phantom transport that drives bots never can) or has not sampled yet, and `invalid_player_id`. A bare nil would collapse all three into "no".

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `milliseconds, or nil`, `reason`

```lua
-- /ping: show your own latency
RegisterCommand("ping", function(source)
    local ms, reason = Open77.players.ping(source)
    Open77.chat.send(source, ms and ("Ping: %d ms"):format(ms) or ("Ping unavailable: " .. reason))
end, false)
```

Registered in `LuaResourceRuntime.PlayerConnection.cs` (line 153) as `LuaResourceRuntime.Bootstrap`.

## position

```lua
Open77.players.position(playerId)
```

`NETWORK`: Uses the network backend.

A player's last known position and routing bucket.

No permission. `GetPlayerPosition` is the same function. Returns `{ x, y, z, bucket }` from the server's snapshot, so it can lag a fast-moving player by a tick. **The id must be an integer of 1 or more, checked before the call.** `0` or a negative id is not answered `nil`: the native throws `id must be positive`, which no `pcall` catches -- the resource is stopped with `runtime_error` and its dependants with `dependency_stopped`. A command run at the dedicated console has `source == 0`, so guard every id that may come from the console (see [A player id is an integer from 1 up](https://open2077.net/docs/api/server/server-api.md#a-player-id-is-an-integer-from-1-up----validate-it-before-every-lookup)).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer >= 1` | required | — |

Returns: `table { x, y, z, bucket }, or nil`

```lua
-- /where: print where a player is standing
RegisterCommand("where", function(source, args)
    local pos = Open77.players.position(tonumber(args[1]) or source)
    if not pos then return Open77.chat.send(source, "Unknown position.") end
    Open77.chat.send(source, ("%.1f, %.1f, %.1f (bucket %d)"):format(pos.x, pos.y, pos.z, pos.bucket))
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8732) as `LuaResourceRuntime.Bootstrap`.

## positions

```lua
Open77.players.positions()
```

`NETWORK`: Uses the network backend.

Every player's position in one call.

No permission. `GetPlayersPositions` is the same function. Returns `{ [playerId] = { x, y, z, bucket } }` for every player with a snapshot younger than two seconds; a player without one is **absent** rather than a `nil` entry. One call replaces a loop of `position` over the roster.

Returns: `table keyed by player id`

```lua
-- Count who is inside the event zone, in one pass
local inside = 0
for playerId, pos in pairs(Open77.players.positions()) do
    if #(vector3(pos.x, pos.y, pos.z) - zoneCenter) < zoneRadius then
        inside = inside + 1
    end
end
```

Registered in `LuaResourceRuntime.cs` (line 8738) as `LuaResourceRuntime.Bootstrap`.

## ragdoll

```lua
Open77.players.ragdoll(playerId, options)
```

`NETWORK`: Uses the network backend.

Puts a player's body on the ground for a bounded time; it gets up on its own.

Requires `players.motion.control`. FiveM's `SetPedToRagdoll`, composed on `Open77.motion.knockdown`: the engine's own knockdown status, a physical fall the observers replicate as a hit reaction. `options.direction` defaults to the body's own facing (from the last accepted transform; a player who never reported one falls north) and accepts a heading in degrees (0 = north) or `{ x, y }`; `options.distance` (0..6 m, default 0) turns the fall into a shove; `options.durationMs` counts from the moment the owning client reports the body down and is **clamped to the native 3000 ms** -- the effective number comes back as `durationMs`. `Open77.motion.cancel(id, result.id)` stands the body up early; `onPlayerMotionChanged` reports `ended` with `reason = "duration_elapsed"` when the clock fires. Refusals are the knockdown's: `motion_busy` (a motion is on the body, or the 1.5 s recovery after one), `body_unavailable` (not alive, on foot and ready -- a player mid-revive or respawn can never be knocked down, which is what keeps this away from the respawn ragdoll release), `invalid_direction`, `invalid_duration`; and `motion_unavailable` if the motion backend is unavailable. Motion leases do not require an implant database; canonical body readiness and matching life, incarnation and bucket remain required. The true physics ragdoll stays deliberately unbound; see the freeze page.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `options` | `table { durationMs?, direction?, distance? }` | required | — |

Returns: `{ ok = true, id = ..., durationMs = ... }, or nil`, `reason`

```lua
-- A stun grenade: everybody within 6 m goes down for two seconds.
for _, entry in ipairs(Open77.players.nearby({ x = x, y = y, z = z }, 6.0) or {}) do
    local read = Open77.players.get(entry.playerId)
    if read and read.alive and not read.inVehicle then
        Open77.players.ragdoll(entry.playerId, { durationMs = 2000 })
    end
end
```

Registered in `LuaResourceRuntime.Players6.cs` (line 255) as `LuaResourceRuntime.Bootstrap`.

## requestLifeResync

```lua
Open77.players.requestLifeResync(playerId)
```

`NETWORK`: Uses the network backend.

Asks a client to re-apply the canonical life state.

Requires `players.life.resync`. `RequestPlayerLifeResync` is the same function. The repair path for a client whose local view has drifted from the server's record; it does not change the record itself.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `true, or false`, `reason`

```lua
-- A player reports "I look dead but I am alive": push the canonical state again
RegisterCommand("resync", function(source)
    Open77.players.requestLifeResync(source)
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 8806) as `LuaResourceRuntime.Bootstrap`.

## requestScreenshot

```lua
Open77.players.requestScreenshot(playerId, options)
```

`NETWORK`: Uses the network backend.

Asks one client for a screenshot of its game view, delivered by that client to a URL the server names; resolves with the upload's receipt.

Requires `players.screenshot`. The question goes to the player's client HOST over the callback frames (the reserved name `open77:screen:capture`, answered without any client resource or client permission, like `Open77.world.groundZ`); the client captures its own swap chain, POSTs the bytes to `options.url` itself and answers with the receipt. **The picture never travels over the game transport** -- a net event carries 48 KiB and there is no client-to-server latent path -- so where it goes is the server owner's decision: an image host, a Discord webhook, or this server's own `Open77.http.listen` route (which decodes text and caps bodies at 16 KiB, so a thumbnail goes there as `encoding = "json"`). Returns an `Open77.Promise` (`:await()`, `:status()`) that resolves with `{ status, body, bodyTruncated, sentBytes, elapsedMs, width, height, sizeBytes, format }` -- `status` is the target's HTTP answer -- or `nil, reason`. Options are the client's `Open77.screen.capture` plus `Open77.screen.upload` vocabulary: `url` (required, absolute http/https, no credentials), `format` (`jpeg` | `png`), `quality` (1..100), `region = { x, y, w, h }`, `includeOverlay` (default false: the game alone, without Open77's own UI), `settleFrames`, `maxBytes`, `scale` (0.05..1, the output shrunk to that fraction of the region -- `0.25` makes a thumbnail this server's own 16 KiB route can take), `encoding` (`multipart` default | `json`), `field` (default `files[]`), `filename`, `headers`, `fields`, `timeoutMs` (the client's HTTP timeout, 1000..60000), and `timeout` (this promise's own deadline in ms, default 20000; the client is handed the same budget as `deadlineMs`). `await` must run from a scheduler coroutine; elsewhere it answers `await_requires_scheduler_coroutine`. What is captured is the client's own rendered frame -- never the desktop, never another window. Reasons before the wire: `permission_denied:players.screenshot`, `invalid_player_id`, `player_not_found`, `invalid_options`, `invalid_url`, `invalid_capture_option:<key>`, `invalid_quality`, `region_invalid`, `format_unsupported`, `invalid_settle_frames`, `invalid_max_bytes`, `invalid_scale`, `invalid_upload_option:<key>`, `invalid_header`, `invalid_field`, `invalid_callback_timeout`, `network_unavailable`, `resource_stopping`, `callback_request_limit`, `callback_payload_too_large`. Rejections carry the client's own names: `capture_in_flight`, `device_unavailable`, `capture_too_large`, `encode_failed:<stage>`, `request_failed:<stage>:<code>`, `timeout`, `capture_timeout`, `callback_timeout`, `callback_resource_unavailable`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `options` | `table` | required | — |

Returns: `promise, or nil`, `reason`

```lua
-- Evidence for a report: the reported player's frame, posted to the MDT backend
RegisterCommand("report", function(source, args)
    local accused = tonumber(args[1])
    local shot, reason = Open77.players.requestScreenshot(accused, {
        url = "https://mdt.example.net/api/evidence", quality = 80,
        fields = { reporter = tostring(source), accused = tostring(accused) },
        headers = { Authorization = "Bearer " .. GetConvar("mdt_token", "") },
    })
    if not shot then return print("screenshot refused: " .. tostring(reason)) end
    local receipt, why = shot:await()
    if not receipt then return print("screenshot failed: " .. tostring(why)) end
    print(("evidence uploaded: HTTP %d, %dx%d, %d bytes"):format(receipt.status, receipt.width, receipt.height, receipt.sizeBytes))
end, true)
```

Registered in `LuaResourceRuntime.Screenshots.cs` (line 273) as `LuaResourceRuntime.Bootstrap`.

## resetModel

```lua
Open77.players.resetModel(playerId)
```

`NETWORK`: Uses the network backend.

Release this resource's model override and restore the original character.

Requires players.model.control and a connected positive integer player ID. Returns true when already original. Refuses model_owned_by_other_resource if another resource generation owns the override. Broadcasts restoration in the player's bucket and emits onPlayerModelChanged with a nil record and reason reset. Cosmetic NPC presentation only: the real player retains identity, controls, health, equipment and saved appearance. Character.* records resolve against each client's installed game data, not a virtual allowlist. See player-models.md for ownership, events, cleanup and the known special-rig/vehicle limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `boolean`, `string? reason`

```lua
local ok, reason = ResetPlayerModel(playerId)
if not ok then print(reason) end
```

[Player morphs and model lifecycle guide](../../01%20Documentation/player-models.md)

Registered in `LuaResourceRuntime.cs` (line 8705) as `LuaResourceRuntime.Bootstrap`.

## respawn

```lua
Open77.players.respawn(playerId, options)
```

`NETWORK`: Uses the network backend.

Brings a DEAD player back to life at a chosen point.

Requires `players.life.respawn`. **Not a general move**: it refuses with `invalid_state` unless the player's phase is already `Dead`, so it cannot relocate a living player at all -- use `Open77.players.teleport` for that. `options.position` is required; `heading` (default 0), `bucket` (default 0), `health` (a fraction, default 1.0) and `graceMs` (default 5000) are optional. Wait for the readiness gate before calling it on a joining player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `options` | `table { position, heading?, bucket?, health?, graceMs? }` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Respawn at the hospital 10 seconds after death
AddEventHandler("onPlayerLifeStateChanged", function(playerId, revision, phase)
    if phase ~= "dead" then return end
    SetTimeout(10000, function()
        Open77.players.respawn(playerId, {
            position = { x = -1442.2, y = 127.4, z = 18.0 },
            heading = 90.0,
            health = 0.5,
        })
    end)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8798) as `LuaResourceRuntime.Bootstrap`.

## restoreHealth

```lua
Open77.players.restoreHealth(playerId)
```

`NETWORK`: Uses the network backend.

Fills a player's health pool.

Requires `players.stats.apply`. `RestorePlayerHealth` is the same function, and `Open77.stats.restore(playerId, "health")` is the pool-generic spelling. Fills to the canonical maximum without reviving a dead player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Round start: everyone at full health
for _, playerId in ipairs(Open77.players.inBucket(ARENA_BUCKET)) do
    Open77.players.restoreHealth(playerId)
end
```

Registered in `LuaResourceRuntime.cs` (line 8883) as `LuaResourceRuntime.Bootstrap`.

## restoreStamina

```lua
Open77.players.restoreStamina(playerId)
```

`NETWORK`: Uses the network backend.

Fills a player's stamina pool.

Requires `players.stats.apply`. `RestorePlayerStamina` is the same function. Useful at a round start, where waiting for regeneration would be visible.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Round start: full stamina so nobody starts the sprint tired
for _, playerId in ipairs(Open77.players.inBucket(ARENA_BUCKET)) do
    Open77.players.restoreStamina(playerId)
end
```

Registered in `LuaResourceRuntime.cs` (line 8891) as `LuaResourceRuntime.Bootstrap`.

## revive

```lua
Open77.players.revive(playerId, [options])
```

`NETWORK`: Uses the network backend.

Revives a player where they fell.

Requires `players.life.revive`. Options are `health` (a fraction, default 0.20) and `graceMs` (default 3000), the window during which the revived player cannot be damaged again.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `options` | `table { health?, graceMs? }` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- A medic revives a downed player nearby
RegisterCommand("revive", function(source)
    local patient = Open77.players.closest(source, { radius = 3 })
    if not patient or not Open77.players.isDead(patient.playerId) then
        return Open77.chat.send(source, "Nobody to revive here.")
    end
    Open77.players.revive(patient.playerId, { health = 0.3, graceMs = 5000 })
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 8783) as `LuaResourceRuntime.Bootstrap`.

## sessionStats

```lua
Open77.players.sessionStats(playerId)
```

`NETWORK`: Uses the network backend.

The live session of a connected player, and its history when the resource holds `players.identity.history`.

The live half is ungated: `{ playerId, userId, connectedAtUtc, sessionSeconds, lastSeenUtc }` for a connected player, with `connectedAtUtc` the same `JoinedAtUtc` the session manager stamped at admission (so it can never disagree with `identifiers().joinedAt`). With `players.identity.history` the read also carries `firstSeenUtc`, `previousSeenUtc?`, `joinCount` and `totalPlaySeconds` (past sessions plus the live one) from the identity directory the server already persists -- no master round trip. Without the capability those fields are absent, never nil-filled. Fails with `invalid_player_id` or `player_not_found`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `table, or nil`, `reason on failure`

```lua
-- A playtime greeting on join
AddEventHandler('onPlayerReady', function(playerId)
    local stats = Open77.players.sessionStats(playerId)
    if not stats then return end
    local hours = math.floor((stats.totalPlaySeconds or stats.sessionSeconds) / 3600)
    Open77.chat.send(tonumber(playerId), ('Welcome back -- %d h played, visit #%d'):format(hours, stats.joinCount or 1))
end)
```

Registered in `LuaResourceRuntime.Identity6.cs` (line 187) as `LuaResourceRuntime.Bootstrap`.

## setArmor

```lua
Open77.players.setArmor(playerId, armor)
```

`NETWORK`: Uses the network backend.

Sets a player's canonical armor.

Requires `players.stats.apply` (the older spelling `players.damage.apply` is still accepted by the runtime but is not the catalogued name). `SetPlayerArmor` is the same function. Armor is a separate pool from health and is consumed first by incoming damage.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `armor` | `number` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Buying a vest at the shop
RegisterNetEvent("shop:buyArmor", function()
    Open77.players.setArmor(source, 100)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8884) as `LuaResourceRuntime.Bootstrap`.

## setDownedDamageable

```lua
Open77.players.setDownedDamageable(playerId, enabled)
```

`NETWORK`: Uses the network backend.

Whether a downed player can still be damaged.

Requires `players.stats.apply` (the older spelling `players.damage.apply` is still accepted by the runtime but is not the catalogued name). `SetPlayerDownedDamageable` is the same function. Turn it on for a mode with finishing blows, off for one where a downed player is safe until revived.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Battle-royale rule: a downed player can still be finished
AddEventHandler("onPlayerReady", function(playerId)
    Open77.players.setDownedDamageable(playerId, true)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8886) as `LuaResourceRuntime.Bootstrap`.

## setDriveByEnabled

```lua
Open77.players.setDriveByEnabled(playerId, enabled)
```

`NETWORK`: Uses the network backend.

Allows or denies a player's vehicle drive-by on the server.

Server; requires players.driveby. False holds this resource's denial; true releases only its own denial. Other resources' restrictions remain effective. Stopping the owner resource releases its claims. Denials survive death, respawn and routing-bucket changes; disconnect removes them. Pending native respawn/revive can return transition_in_progress; retry after completion. Disabling requests native exit and rejects passenger ranged damage. Enabling does not force a pose or equip a weapon. Returns false, reason on refusal. Use matching Unstable 2.31.21-unstable+op77.118 client and server (protocol 1.43), including the animation assets. Passenger support was introduced in .117; car and motorcycle drivers require .118. See [Vehicle drive-by](../../01%20Documentation/drive-by.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `boolean accepted`, `reason on failure`

```lua
RegisterCommand("drivebyoff", function(playerId)
    local ok, reason = Open77.players.setDriveByEnabled(playerId, false)
    if not ok then print(reason) end
end, true)
```

[Passenger drive-by guide](../../01%20Documentation/drive-by.md)

Registered in `LuaResourceRuntime.DriveBy.cs` (line 65) as `LuaResourceRuntime.Bootstrap`.

## setFallDamage

```lua
Open77.players.setFallDamage(playerId, enabled)
```

`NETWORK`: Uses the network backend.

Switches one player's fall damage off or back on. Fall damage only -- not god mode.

Requires `players.life.falldamage`. `false` makes landings harmless for this player; `true` restores the vanilla behaviour. **Fall damage only:** a shielded player is still shot, stabbed, burned, blown up and run over exactly as before -- this is FiveM's fall-damage toggle and not `setGodMode`, nor the grav-chute's blanket `Invulnerable` shield. On 2.31 a hard landing is a self-targeted attack the player's own locomotion machine fires through the damage pipeline (`Attacks.HardLanding` / `VeryHardLanding` / `DeathLanding`), so the bit rides the life state and the player's own client drops exactly those three records before they are priced; the stumble, camera hit and grunt still play. Survives a proxy re-stream and a bucket change; dropped by death and by a reconnect; switched back on when the calling resource stops or reloads. Refused with `transition_in_progress` during a revive or respawn, allowed while dead so a gamemode can pre-arm it before the respawn it issues (the respawn keeps it).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `true, or false`, `reason`

```lua
-- A parkour round: nobody dies to the course, everybody still dies to each other.
for _, id in ipairs(Open77.players.all()) do
    Open77.players.setFallDamage(id, false)
end
```

Registered in `LuaResourceRuntime.Players6.cs` (line 227) as `LuaResourceRuntime.Bootstrap`.

## setFrozen

```lua
Open77.players.setFrozen(playerId, frozen)
```

`NETWORK`: Uses the network backend.

Holds a player's body still, server-authoritative.

Requires `players.life.freeze`. `SetPlayerFrozen` is the lower-level spelling. The body is held where it stands with the engine's own `GameplayRestriction.NoMovement` -- the same record vanilla death applies -- so nothing fights the hold and nothing slides on an observer's screen.\n\n**What it stops:** walking, running, sprinting, crouch-moving, jumping, dodging, sliding and vaulting. Every locomotion input, pressed or held.\n\n**What it does NOT stop, deliberately:** looking around, chat, voice, menus and the inventory, interaction prompts, and damage -- a frozen player can still be shot and still dies. It does not holster a weapon, block firing, or eject anyone from a vehicle. A freeze is a primitive, not a policy: combine it with the control-blocking surface if you want more.\n\nThe claim is keyed by the calling resource. Two resources may hold the same player, and `setFrozen(id, false)` drops only your own claim -- he stays frozen while anybody else still holds him. A resource's claims are dropped for it when it stops, reloads or crashes, and a freeze is also lifted by death, respawn, a bucket change and disconnect, so a script that went away can never strand a player. Re-apply after a respawn if the gamemode still wants him held. Refused with `transition_in_progress` while a revive or respawn is in flight, exactly like `setGhosted`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `frozen` | `boolean` | required | — |

Returns: `true, or false`, `reason`

```lua
-- /freeze <id> toggles a hold on a player
RegisterCommand("freeze", function(source, args)
    local target = tonumber(args[1])
    if not target then return end
    local frozen = Open77.players.isFrozen(target)
    Open77.players.setFrozen(target, not frozen)
end, true)
```

[Freezing a player guide](../../01%20Documentation/player-freeze.md)

Registered in `LuaResourceRuntime.cs` (line 8817) as `LuaResourceRuntime.Bootstrap`.

## setGhosted

```lua
Open77.players.setGhosted(playerId, ghosted, [options])
```

`NETWORK`: Uses the network backend.

Makes a player non-solid, with an optional automatic clear.

Requires `players.life.ghost`. `SetPlayerGhosted` is the lower-level spelling. `options.autoClear` defaults to **true**, with `clearDistance` and `clearMs` deciding when the ghost lifts -- that is what stops a spawn-protected player staying non-solid indefinitely. The server owns the state, so every observer agrees about it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `ghosted` | `boolean` | required | — |
| `options` | `table { autoClear?, clearDistance?, clearMs? }` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- Spawn protection: non-solid for 10 seconds or until they walk 15 m away
AddEventHandler("onPlayerReady", function(playerId)
    Open77.players.setGhosted(playerId, true, { clearMs = 10000, clearDistance = 15.0 })
end)
```

Registered in `LuaResourceRuntime.cs` (line 8807) as `LuaResourceRuntime.Bootstrap`.

## setGodMode

```lua
Open77.players.setGodMode(playerId, enabled)
```

`NETWORK`: Uses the network backend.

Toggles a player's canonical damage immunity.

Requires `players.stats.apply` (the older spelling `players.damage.apply` is still accepted by the runtime but is not the catalogued name). `SetPlayerGodMode` is the same function. God mode is checked by the server, not the client, so it also refuses damage a Lua arbiter would have allowed.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `true, or false`, `reason`

```lua
-- /god toggles invulnerability for an admin
RegisterCommand("god", function(source)
    godMode[source] = not godMode[source]
    Open77.players.setGodMode(source, godMode[source])
    Open77.chat.send(source, godMode[source] and "God mode on." or "God mode off.")
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8885) as `LuaResourceRuntime.Bootstrap`.

## setHeading

```lua
Open77.players.setHeading(playerId, yaw)
```

`NETWORK`: Uses the network backend.

Turns a living player to face a yaw.

Requires `players.teleport`. The cheap half of the placement pair: a heading streams nothing and cannot fall through a floor, so this dispatches and returns instead of confirming. `Open77.players.teleport` is the one that confirms. Refuses with `player_not_ready`, `player_not_alive` or `invalid_heading`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `yaw` | `number` | required | — |

Returns: `true, or false`, `reason`

```lua
-- After the teleport, face the player toward the shop counter
Open77.players.setHeading(playerId, 270.0)
```

Registered in `LuaResourceRuntime.cs` (line 8797) as `LuaResourceRuntime.Bootstrap`.

## setHealth

```lua
Open77.players.setHealth(playerId, health)
```

`NETWORK`: Uses the network backend.

Sets a player's health.

Requires `players.stats.apply` (the older spelling `players.damage.apply` is still accepted by the runtime but is not the catalogued name). `SetPlayerHealth` is the same function. Setting zero passes through life authority rather than leaving a player alive at no health, so it is a real death with the usual replication.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `health` | `number` | required | — |

Returns: `true, or false`, `reason`

```lua
-- /sethp <id> <value>
RegisterCommand("sethp", function(source, args)
    local target, value = tonumber(args[1]), tonumber(args[2])
    if not target or not value then return end
    Open77.players.setHealth(target, value)
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8881) as `LuaResourceRuntime.Bootstrap`.

## setHoloCallEyes

```lua
Open77.players.setHoloCallEyes(playerId, enabled, [options])
```

`NETWORK`: Uses the network backend.

Synchronize a player's native blue holocall eye glow.

Requires players.holocall.control. Uses a network player ID, not an entity handle. Enabled must be a boolean; options may only contain durationMs (integer 0..600000, default 0). True acquires/renews this resource VM's lease; false releases only its lease. The glow remains enabled while any resource holds a lease. It clears on death, disconnect, resource stop/reload/failure or expiry. Reliable bucket snapshots cover late joins and re-stream; clients project onto native and F7 bodies. Does not open the phone UI or start a call. See holocall-eyes.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `enabled` | `boolean` | required | — |
| `options` | `table` | optional | — |

Returns: `true, or false, reason`

```lua
-- Blue eyes for 30 seconds while the holocall runs
RegisterNetEvent("phone:callStarted", function()
    Open77.players.setHoloCallEyes(source, true, { durationMs = 30000 })
end)
```

[Blue holocall eyes guide](../../01%20Documentation/holocall-eyes.md)

Registered in `LuaResourceRuntime.cs` (line 8723) as `LuaResourceRuntime.Bootstrap`.

## setMaxHealth

```lua
Open77.players.setMaxHealth(playerId, maxHealth)
```

`NETWORK`: Uses the network backend.

Sets a player's canonical maximum health.

Requires `players.stats.apply` (the older spelling `players.damage.apply` is still accepted by the runtime but is not the catalogued name). `SetPlayerMaxHealth` is the same function. Lowering the maximum clamps the current value.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `maxHealth` | `number` | required | — |

Returns: `true, or false`, `reason`

```lua
-- A perk raises the maximum health to 150
Open77.players.setMaxHealth(playerId, 150)
```

Registered in `LuaResourceRuntime.cs` (line 8882) as `LuaResourceRuntime.Bootstrap`.

## setMaxStamina

```lua
Open77.players.setMaxStamina(playerId, maximum)
```

`NETWORK`: Uses the network backend.

Sets a player's canonical maximum stamina.

Requires `players.stats.apply`. `SetPlayerMaxStamina` is the same function. Lowering the maximum clamps the current value.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `maximum` | `number` | required | — |

Returns: `true, or false`, `reason`

```lua
-- A "runner" job class gets more stamina
Open77.players.setMaxStamina(playerId, 150)
```

Registered in `LuaResourceRuntime.cs` (line 8890) as `LuaResourceRuntime.Bootstrap`.

## setMaxWanted

```lua
Open77.players.setMaxWanted(playerId, level)
```

`NETWORK`: Uses the network backend.

Sets the ceiling one player's wanted level may reach, 0 to 5.

**The engine has no per-player wanted level.** `PreventionSystem` holds ONE heat stage, for the one player its client is running: `ChangeHeatStage(stage, reason)`, `GetHeatStage()` and `SetMinMaxResetHeatLevels(min, max, isDefault)` take no target, the state is scalar on a singleton, and every consumer reads one global `UI_WantedBar`. A per-player number works only because every Open77 player has their own game and the server tells each client its own value. The police that follow are LOCAL to that client - not Open77 entities, not replicated - so two wanted players standing together are two private pursuits, each invisible to the other. Do not build a shared chase on this. The value is pushed to the client immediately and re-pushed on a bucket change and on world entry; the server never learns the engine's real heat, because there is no value channel back from REDscript. Requires `players.wanted`. The engine call behind it is `SetMinMaxResetHeatLevels(0, level, false)`; the floor stays 0 because a nonzero minimum would strand a session at a heat it could never clear. Lowering the ceiling below a standing level lowers the level with it and reports that through `onPlayerWantedChanged` - leaving a player above their own ceiling is a state the engine's reset rules would undo at the next opportunity anyway.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `level` | `integer (0..5)` | required | — |

Returns: `table { level, maxLevel }, or nil`, `reason: permission_denied:players.wanted, invalid_player, invalid_wanted_level, player_unavailable`

```lua
-- A VIP zone where the police never go past two stars
AddEventHandler("onPlayerBucketChange", function(playerId, bucket, previous)
    Open77.players.setMaxWanted(playerId, bucket == VIP_BUCKET and 2 or 5)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8719) as `LuaResourceRuntime.Bootstrap`.

## setModel

```lua
Open77.players.setModel(playerId, record, [options])
```

`NETWORK`: Uses the network backend.

Select a server-owned NPC body for a connected player.

Requires players.model.control. The player must be connected, alive and gameplay-ready. This resource generation exclusively owns the override; another owner fails with model_owned_by_other_resource. Options: appearance (ASCII CName up to 128 characters, default empty), durationMs (integer 0–86400000, default 0), resetOnDeath (boolean, default false); unknown fields are rejected. True, revision means accepted, not visible. Observe onPlayerModelReady/onPlayerModelFailed; a missing record fails asynchronously and restores the original body. Identical options extend the lease without respawn. Stopping the owner, timeout or disconnect releases state. Cosmetic NPC presentation only: the real player retains identity, controls, health, equipment and saved appearance. Character.* records resolve against each client's installed game data, not a virtual allowlist. See player-models.md for ownership, events, cleanup and the known special-rig/vehicle limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `record` | `string` | required | — |
| `options` | `table` | optional | — |

Returns: `boolean accepted`, `integer revision or string reason`

```lua
local ok, revision = SetPlayerModel(playerId, 'Character.Rogue', { durationMs = 60000, resetOnDeath = true })
```

[Player morphs and model lifecycle guide](../../01%20Documentation/player-models.md)

Registered in `LuaResourceRuntime.cs` (line 8703) as `LuaResourceRuntime.Bootstrap`.

## setRegen

```lua
Open77.players.setRegen(playerId, pointsPerSecond)
```

`NETWORK`: Uses the network backend.

Sets a player's health regeneration rate.

Requires `players.stats.apply` (the older spelling `players.damage.apply` is still accepted by the runtime but is not the catalogued name). `SetPlayerRegen` is the same function, and `Open77.stats.setRegenRate(playerId, "health", rate)` is the pool-generic spelling. Zero disables regeneration.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `pointsPerSecond` | `number` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Hardcore mode: no passive health regeneration
AddEventHandler("onPlayerReady", function(playerId)
    Open77.players.setRegen(playerId, 0)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8887) as `LuaResourceRuntime.Bootstrap`.

## setRegenEnabled

```lua
Open77.players.setRegenEnabled(playerId, enabled)
```

`NETWORK`: Uses the network backend.

Turns a player's health regeneration on or off.

Requires `players.stats.apply`. `SetPlayerRegenEnabled` is the same function. Distinct from setting the rate to zero: the configured rate is remembered and comes back when regeneration is re-enabled.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Regeneration pauses while the player is bleeding, without losing the configured rate
Open77.players.setRegenEnabled(playerId, false)
SetTimeout(30000, function() Open77.players.setRegenEnabled(playerId, true) end)
```

Registered in `LuaResourceRuntime.cs` (line 8888) as `LuaResourceRuntime.Bootstrap`.

## setStamina

```lua
Open77.players.setStamina(playerId, value)
```

`NETWORK`: Uses the network backend.

Sets a player's stamina.

Requires `players.stats.apply`. `SetPlayerStamina` is the same function, and `Open77.stats.set(playerId, "stamina", value)` is the pool-generic spelling. Stamina is authoritative on the server: the client reads it and never sets it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `value` | `number` | required | — |

Returns: `true, or false`, `reason`

```lua
-- An energy drink refills stamina to 80
RegisterNetEvent("shop:useEnergyDrink", function()
    Open77.players.setStamina(source, 80)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8889) as `LuaResourceRuntime.Bootstrap`.

## setStaminaRegen

```lua
Open77.players.setStaminaRegen(playerId, pointsPerSecond)
```

`NETWORK`: Uses the network backend.

Sets a player's stamina regeneration rate.

Requires `players.stats.apply`. `SetPlayerStaminaRegen` is the same function. Zero disables regeneration; use `setStaminaRegenEnabled` to pause it while keeping the configured rate.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `pointsPerSecond` | `number` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Athletic perk: stamina comes back twice as fast
Open77.players.setStaminaRegen(playerId, 20)
```

Registered in `LuaResourceRuntime.cs` (line 8892) as `LuaResourceRuntime.Bootstrap`.

## setStaminaRegenEnabled

```lua
Open77.players.setStaminaRegenEnabled(playerId, enabled)
```

`NETWORK`: Uses the network backend.

Turns a player's stamina regeneration on or off.

Requires `players.stats.apply`. `SetPlayerStaminaRegenEnabled` is the same function. Turning it off is how a mode makes a sprint budget matter without rewriting the rate.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Exhaustion effect: no stamina regeneration for 20 seconds
Open77.players.setStaminaRegenEnabled(playerId, false)
SetTimeout(20000, function() Open77.players.setStaminaRegenEnabled(playerId, true) end)
```

Registered in `LuaResourceRuntime.cs` (line 8893) as `LuaResourceRuntime.Bootstrap`.

## setVehicleExitLocked

```lua
Open77.players.setVehicleExitLocked(playerId, locked, [vehicleId])
```

`NETWORK`: Uses the network backend.

Sets or clears a player's durable no-exit policy.

Requires `world.vehicles`. Alias of `Open77.vehicles.setPlayerExitLocked`. Blocks ordinary unmount and manual seat switching; a server-forced exit still overrides it. Distinct from the vehicle's own `locked` entry flag, which governs getting *in*.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `locked` | `boolean` | required | — |
| `vehicleId` | `vehicle id` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- The prisoner transport: nobody leaves the van until it arrives
Open77.players.warpIntoVehicle(prisonerId, vanId, Open77.vehicles.seats.rearLeft)
Open77.players.setVehicleExitLocked(prisonerId, true, vanId)
```

Registered in `LuaResourceRuntime.cs` (line 8768) as `LuaResourceRuntime.Bootstrap`.

## setVisible

```lua
Open77.players.setVisible(playerId, visible)
```

`NETWORK`: Uses the network backend.

Hides a player's body from every other client, or shows it again.

Requires `players.life.visibility`. `SetPlayerVisible` is the lower-level spelling. **Presentation only, and deliberately orthogonal to `setGhosted`**: an invisible player is still solid, still shootable, still heard in proximity voice and still listed in `Open77.players.all()` -- which is what makes "invisible but still simulated" the default rather than a special mode. His nameplate is withheld with his body, because a plate over empty air would leak the exact position the hide exists to conceal, and a resource cannot opt out of that. The state is server-owned and rides the reliable life state, so it survives a proxy re-stream; clients re-assert the hide on a 1 Hz beat because 2.31 exposes no readable hidden flag to detect a render-proxy rebuild dropping it. On a remote body that is simultaneously being dressed, the body stays hidden while either owner holds it and appears only when both release, in either order. Released automatically on death, respawn, bucket change, disconnect, reconnect, and when the calling resource stops or reloads -- invisibility has no proximity auto-clear the way a ghost does, so nothing else would ever recover a stranded body.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `visible` | `boolean` | required | — |

Returns: `true, or false`, `reason`

```lua
-- /vanish toggles an invisible admin
RegisterCommand("vanish", function(source)
    hidden[source] = not hidden[source]
    Open77.players.setVisible(source, not hidden[source])
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8822) as `LuaResourceRuntime.Bootstrap`.

## setWanted

```lua
Open77.players.setWanted(playerId, level)
```

`NETWORK`: Uses the network backend.

Sets one player's NCPD wanted level, 0 to 5.

**The engine has no per-player wanted level.** `PreventionSystem` holds ONE heat stage, for the one player its client is running: `ChangeHeatStage(stage, reason)`, `GetHeatStage()` and `SetMinMaxResetHeatLevels(min, max, isDefault)` take no target, the state is scalar on a singleton, and every consumer reads one global `UI_WantedBar`. A per-player number works only because every Open77 player has their own game and the server tells each client its own value. The police that follow are LOCAL to that client - not Open77 entities, not replicated - so two wanted players standing together are two private pursuits, each invisible to the other. Do not build a shared chase on this. The value is pushed to the client immediately and re-pushed on a bucket change and on world entry; the server never learns the engine's real heat, because there is no value channel back from REDscript. Requires `players.wanted`. `setWanted(id, 0)` is `ClearPlayerWantedLevel`. The player's ceiling is applied here, not on the client: asking for 5 under a ceiling of 2 succeeds and reports `level = 2` beside the `requested` value, so a caller is told about the clamp instead of discovering it. Fires the host event `onPlayerWantedChanged(playerId, level, previous)` when the value actually changes - a decision this server made, never an observation of the game.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `level` | `integer (0..5)` | required | — |

Returns: `table { level, requested, maxLevel }, or nil`, `reason: permission_denied:players.wanted, invalid_player, invalid_wanted_level, player_unavailable`

```lua
-- Robbing the store puts three stars on the robber
RegisterNetEvent("store:robbed", function()
    local result = Open77.players.setWanted(source, 3)
    if result then print("wanted level now", result.level) end
end)
```

Registered in `LuaResourceRuntime.cs` (line 8716) as `LuaResourceRuntime.Bootstrap`.

## spectate

```lua
Open77.players.spectate(playerId, targetId, options)
```

`GAME`: Requires a live game instance.

Puts one player behind another player's shoulder, body and all.

Requires `players.spectate`. Ghosts the spectator, hides his body and puts a follow camera on the target, as one transaction: if the body cannot be taken away nothing is applied and nothing is sent. `targetId` false or nil ends the session. `options`: `blendMs` (0..10000, default 400), `distance` (0..100, default 4.0), `height` (-50..50, default 1.5). Both halves come back on every way it can end -- either side dying, disconnecting, reconnecting or changing bucket, and this resource stopping -- and the body is restored to what it was, so a ghost another script owns survives. Refusals: `cannot_spectate_self`, `target_not_found`, `spectator_not_alive`, `target_not_alive`, `different_bucket`, `target_is_spectating`, `not_spectating`. The world streams around the SPECTATOR's body, so teleport him to the target for anything further than a couple of hundred metres. See the Scripted cameras guide.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `number` | required | — |
| `targetId` | `number` | required | — |
| `options` | `table` | required | — |

Returns: `true, or false and a reason`

```lua
-- /spectate <id>, and /spectate alone to stop
RegisterCommand("spectate", function(source, args)
    local target = tonumber(args[1])
    local ok, reason = Open77.players.spectate(source, target, { distance = 5.0 })
    if not ok then Open77.chat.send(source, "Cannot spectate: " .. reason) end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8843) as `LuaResourceRuntime.Bootstrap`.

## spectating

```lua
Open77.players.spectating(playerId)
```

`GAME`: Requires a live game instance.

Who this player is watching, or 0.

Requires `players.life.read`. Answered from the server-owned session roster, never probed from a client. See the Scripted cameras guide.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `number` | required | — |

Returns: `target player id, or 0`

```lua
-- A spectator cannot be damaged by what they are watching
local watching = Open77.players.spectating(playerId)
if watching ~= 0 then return end
```

Registered in `LuaResourceRuntime.cs` (line 8850) as `LuaResourceRuntime.Bootstrap`.

## spectators

```lua
Open77.players.spectators(playerId)
```

`GAME`: Requires a live game instance.

Everyone watching this player, ascending.

Requires `players.life.read`. Returns an array of player ids. See the Scripted cameras guide.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `number` | required | — |

Returns: `array of player ids`

```lua
-- Tell a streamer how many admins are watching them
local count = #Open77.players.spectators(playerId)
if count > 0 then
    Open77.chat.send(playerId, ("%d admin(s) are spectating you."):format(count))
end
```

Registered in `LuaResourceRuntime.cs` (line 8851) as `LuaResourceRuntime.Bootstrap`.

## taskEnterVehicle

```lua
Open77.players.taskEnterVehicle(playerId, vehicleId, seat, [options])
```

`NETWORK`: Uses the network backend.

Seats a player through the vanilla entry animation.

Requires `world.vehicles`. Alias of `Open77.vehicles.taskPlayerEnter`, with the same `options` as `warpIntoVehicle`: `moveBucket` (default true) and `exitLocked`/`lockExit` (default false). The reservation, the bucket move and every refusal reason are the warp's; only the way the affected client reaches the seat changes, and it falls back to the warp when the animation does not land within three seconds.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `vehicleId` | `integer` | required | — |
| `seat` | `integer|string` | required | — |
| `options` | `table` | optional | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- The taxi arrived: the customer climbs in with the animation
Open77.players.taskEnterVehicle(customerId, taxiId, Open77.vehicles.seats.rearRight)
```

Registered in `LuaResourceRuntime.Vehicles.cs` (line 1307) as `LuaResourceRuntime.Bootstrap`.

## taskLeaveVehicle

```lua
Open77.players.taskLeaveVehicle(playerId, [vehicleId])
```

`NETWORK`: Uses the network backend.

Ejects a player from their current vehicle.

Requires `world.vehicles`. Alias of `Open77.vehicles.taskPlayerLeave`, which is the same durable forced exit as `forceOutOfVehicle`. The leaving player's own exit is instant on this build -- the local authored exit workspot crashed the engine's release dispatch when it was measured, so the client does not attempt it -- while every observer still sees the exit animated on this player's proxy.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `vehicleId` | `integer` | optional | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- The taxi reached its destination
Open77.players.taskLeaveVehicle(customerId, taxiId)
```

Registered in `LuaResourceRuntime.Vehicles.cs` (line 1310) as `LuaResourceRuntime.Bootstrap`.

## teleport

```lua
Open77.players.teleport(playerId, position, [options])
```

`NETWORK`: Uses the network backend.

Moves a living player, with no life transition.

Requires `players.teleport`. Returns a **Promise** that resolves when the client reports the body settled at the destination, or `nil, reason` when the request was refused outright. Health, inventory, weapons and vehicle occupancy are untouched -- this is not `respawn`, which refuses unless the player is already dead and costs a death to work around. `options`: `heading`, `bucket` (changed before the move, so no observer left in the origin bucket ever sees the destination; restored if the move fails), `fade` (default true, with `fadeOutMs`/`fadeInMs` default 400), `dismount` (default false) and `timeoutMs` (1000..30000, default 12000). Resolves with `{ x, y, z, state }` where `state` is `settled` or `near`. Rejects with `player_not_ready` (no life record -- the continue screen, where acting on a client crashes it), `player_not_alive`, `player_in_vehicle`, `invalid_position` or `settle_timeout`. A player in a vehicle is refused rather than ejected: pass `dismount = true` to accept losing the vehicle, which unmounts behind the fade and refuses with `dismount_failed` if the body is still seated after 750 ms.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `position` | `table { x, y, z }` | required | — |
| `options` | `table { heading?, bucket?, fade?, fadeOutMs?, fadeInMs?, dismount?, timeoutMs? }` | optional | — |

Returns: `Promise, or nil`, `reason`

```lua
-- /tp <x> <y> <z>, waiting until the body has really landed
RegisterCommand("tp", function(source, args)
    local x, y, z = tonumber(args[1]), tonumber(args[2]), tonumber(args[3])
    if not x or not y or not z then return end
    local pending, reason = Open77.players.teleport(source, { x = x, y = y, z = z }, { dismount = true })
    if not pending then return Open77.chat.send(source, "Teleport refused: " .. reason) end
    local landed, err = pending:await()
    Open77.chat.send(source, landed and ("Arrived (%s)"):format(landed.state) or ("Failed: " .. err))
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8796) as `LuaResourceRuntime.Bootstrap`.

## warpIntoVehicle

```lua
Open77.players.warpIntoVehicle(playerId, vehicleId, seat, [options])
```

`NETWORK`: Uses the network backend.

Puts a player in a vehicle seat authoritatively.

Requires `world.vehicles`. Alias of `Open77.vehicles.warpPlayerIntoVehicle`, with the same options: `moveBucket` (default true; false makes a bucket mismatch answer `wrong_bucket`) and `exitLocked`/`lockExit` (default false). Seats accept `-1..2`, the `Open77.vehicles.seats` constants, aliases like `driver`, and canonical `seat_*` names.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `vehicleId` | `vehicle id` | required | — |
| `seat` | `integer|string` | required | — |
| `options` | `table { moveBucket?, exitLocked? }` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- /seat <vehicleId>: put the caller in the driver seat instantly
RegisterCommand("seat", function(source, args)
    local vehicleId = tonumber(args[1])
    if not vehicleId then return end
    local ok, reason = Open77.players.warpIntoVehicle(source, vehicleId, "driver")
    if not ok then Open77.chat.send(source, "Refused: " .. reason) end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8762) as `LuaResourceRuntime.Bootstrap`.
