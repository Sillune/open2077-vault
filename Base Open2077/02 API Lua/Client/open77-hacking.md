# Open77.hacking (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-hacking.md) · [Page web](https://open2077.net/docs/api/client/open77-hacking)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

6 functions.

## clear

```lua
Open77.hacking.clear()
```

`GAME`: Requires a live game instance.

Requests cleanup of the calling resource's hacking presentation.

Client-only; requires player.hacking.project. Clears only presentation owned by the calling resource and is harmless when it owns none. Resource stop invokes the same owned cleanup automatically. true acknowledges the cleanup request, not native completion; the native script retains its bounded expiry fallback if the bridge cannot accept a clear command. This does not cancel an authoritative upload, purge a status, refund costs or undo committed damage. Use the server hacking APIs for gameplay state changes.

Returns: `true, or nil, reason`

```lua
-- The hack minigame closes: drop this resource's HUD presentation
AddEventHandler("hackui:closed", function()
    Open77.hacking.clear()
end)
```

Registered in `ResourceHost.cpp` (line 24869) as `LuaHackingClear`.

## clearNativeStatus

```lua
Open77.hacking.clearNativeStatus([kind])
```

`GAME`: Requires a live game instance.

Releases this resource's owned native hacking statuses.

Client-only; requires player.hacking.status. An omitted kind clears every status the calling resource holds; passing "blind" or "weapon_jam" clears only that slot, and any other value is refused with invalid_kind. It never clears another resource's slot and is harmless when the caller owns none. true acknowledges the clear request, not native completion; the adapter retains its bounded expiry, death, vehicle, workspot and session guards. This does not cancel an authoritative upload or undo committed damage.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `kind` | `string` | optional | — |

Returns: `true, or nil, reason`

```lua
-- The server cured the blindness early: release just that slot
RegisterNetEvent("hack:cured", function(kind)
    Open77.hacking.clearNativeStatus(kind) -- "blind" or "weapon_jam"
end)
```

Registered in `ResourceHost.cpp` (line 24871) as `LuaHackingClearNativeStatus`.

## nativeStatus

```lua
Open77.hacking.nativeStatus(kind, durationMs)
```

`GAME`: Requires a live game instance.

Applies a bounded native hacking status -- blind or weapon jam -- to the local player.

Client-only; requires `player.hacking.status`, a server-driven projection primitive a player resource should not hold by default (distinct from the HUD-only `player.hacking.project`). `kind` is `blind` (Reboot Optics) or `weapon_jam` (Weapon Glitch); `durationMs` is 1..3600000 and the client clamps it to the kind's cap (15 s). One independent slot per kind, owned by the calling resource: a second call on the same kind replaces this resource's own slot and cannot touch another resource's. The adapter releases on the deadline, on death, vehicle entry, workspot, session change and resource stop; the server's deadline stays the authority and this only presents it. Returns the request id, not proof of a rendered effect.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `kind` | `string (blind | weapon_jam)` | required | — |
| `durationMs` | `integer` | required | — |

Returns: `request id, or nil`, `reason: permission_denied:player.hacking.status, resource_not_running, hacking_status_backend_unavailable, invalid_kind, invalid_duration`

```lua
-- The server says we were hit by Reboot Optics: blind the local player for 5 s
RegisterNetEvent("hack:blinded", function()
    Open77.hacking.nativeStatus("blind", 5000)
end)
```

Registered in `ResourceHost.cpp` (line 24870) as `LuaHackingNativeStatus`.

## nativeStatusState

```lua
Open77.hacking.nativeStatusState()
```

`GAME`: Requires a live game instance.

Reads the bounded native hacking status slots and their counters.

Client-only; requires player.hacking.status. Returns one sub-table per held kind, keyed by kind (blind, weapon_jam), each carrying kind, phase, owner, reason, request, grantedMs, remainingMs, elapsedMs, releaseOverdue and applied, plus the totals applications, refusals, releases and watchdogRemovals. Slots are independent, one per kind, and the client clamps each duration to that kind's cap. applied is adapter state, not proof of a rendered effect; the server's deadline is the authority and the client only presents it.

Returns: `state table, or nil, reason`

```lua
local state, reason = Open77.hacking.nativeStatusState()
if state and state.blind then print(state.blind.phase, state.blind.remainingMs) end
```

Registered in `ResourceHost.cpp` (line 24872) as `LuaHackingNativeStatusState`.

## observe

```lua
Open77.hacking.observe(playerId)
```

`GAME`: Requires a live game instance.

Reads a streamed multiplayer target's position, camera angle and local visibility for real-time selection.

Client-only; requires world.query. playerId is a positive integer network player ID. Returns streamed=true, visible, distance in metres, angle in radians and position={x,y,z}; no engine pointer or entity handle is exposed. The target must have an attached, presentation-ready remote body. Native selection checks camera-to-chest distance from 0.8 through 80 metres; peers outside the 0.35-radian selection cone return visible=false without a physics query. Within the cone, a failed native raycast returns nil, reason. This is local selection evidence only: it does not authorize a target, start an upload or prove server visibility. The server separately validates identity, life, range, policy and challenged visibility. It does not enter the vanilla scanner or slow game time. See the Hacking and counterplay guide for the public server workflow.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `observation, or nil, reason`

```lua
local target, reason = Open77.hacking.observe(2)
if target and target.visible then
  print(('Target %.1f m away, angle %.3f rad'):format(target.distance, target.angle))
end
```

Registered in `ResourceHost.cpp` (line 24867) as `LuaHackingObserve`.

## present

```lua
Open77.hacking.present(options)
```

`GAME`: Requires a live game instance.

Queues this resource's native hacking warning, progress or outcome presentation.

Client-only; requires player.hacking.project and a running resource. options contains actionId (integer 1..9007199254740991), phase (targeting, incoming, upload, block, purge, impact or cancel) and finite progress (0..1). actionId correlates presentation; it is not a damage authorization. Ownership comes from the calling resource. A different resource's active 1.5-second presentation lease refuses admission; refresh a continuing presentation periodically. The native adapter uses the vanilla progress HUD and sound cues while preserving other native HUD ownership. true means queued, not rendered or acoustically proven: the native script may refuse or defer a busy HUD. The official hacking resource supplies an incoming-warning fallback. This API does not execute a vanilla quickhack, apply native damage or change movement/camera controls. Resource stop requests owned cleanup; native presentation also has body/session and expiry guards.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table { actionId, phase, progress }` | required | — |

Returns: `true when queued, or nil, reason`

```lua
local queued, reason = Open77.hacking.present({actionId=123, phase='incoming', progress=.25})
-- Queued admission is not proof that the HUD or audio was rendered.
```

Registered in `ResourceHost.cpp` (line 24868) as `LuaHackingPresent`.
