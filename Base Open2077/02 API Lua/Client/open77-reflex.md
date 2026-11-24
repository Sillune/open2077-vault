# Open77.reflex (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-reflex.md) · [Page web](https://open2077.net/docs/api/client/open77-reflex)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

3 functions.

## clear

```lua
Open77.reflex.clear()
```

`GAME`: Requires a live game instance.

Releases the reflex overdrive this resource holds on the local player.

Client support primitive; requires player.reflex.overdrive. Idempotent; the boost also ends on its own deadline, on death, a vehicle, a workspot, a session change and the owning resource stopping.

Returns: `true, or nil, reason`

```lua
-- Give the boost back when the authority ends it early, and on the way out
RegisterNetEvent("open77:reflex:phase", function(value)
    if value.phase ~= "cancelled" then return end
    Open77.reflex.clear()   -- idempotent: harmless if the deadline beat us to it
    activeRequest = nil
end)

AddEventHandler("onClientResourceStop", function(name)
    if name == GetCurrentResourceName() then Open77.reflex.clear() end
end)
```

Registered in `ResourceHost.cpp` (line 24914) as `LuaReflexClear`.

## overdrive

```lua
Open77.reflex.overdrive(kind, durationMs)
```

`GAME`: Requires a live game instance.

Applies the local reflex overdrive: a bounded real-time speed/handling buff.

Client support primitive; requires player.reflex.overdrive. Kind is reflex or reflex_heavy; duration is clamped to 15 s. It applies stat modifiers through gameStatsSystem and touches no clock: the shared world stays real time, other players are never slowed and bullets are never slowed. Server creators use the separate server Open77.reflex define/grant/revoke API documented in reflex-overdrive.md. A returned request number is admission, not rendered speed. Arbitrary stat plans are unavailable. Multiplayer acceptance is pending.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `kind` | `string` | required | — |
| `durationMs` | `integer` | required | — |

Returns: `request number, or nil`, `reason on failure`

```lua
-- Apply the boost only once the server has admitted it (open77_reflex does
-- this for you; a gamemode grants the tier with the server API instead).
RegisterNetEvent("open77:reflex:permit", function(permit)
    if permit.ok ~= true then return end          -- refused: no charge was spent
    local request, reason = Open77.reflex.overdrive(permit.tier, permit.durationMs)
    if not request then return print("overdrive refused: " .. tostring(reason)) end
    activeRequest = request
end)
```

Registered in `ResourceHost.cpp` (line 24913) as `LuaReflexOverdrive`.

## state

```lua
Open77.reflex.state()
```

`GAME`: Requires a live game instance.

Reads the local reflex-overdrive machine.

Client support primitive; requires player.reflex.overdrive. Returns phase, kind, owner, reason, request, grantedMs, remainingMs, elapsedMs, activations, refusals, releases, watchdogRemovals, releaseOverdue and active. It is admission and lifecycle state, not proof that the engine honoured a modifier.

Returns: `table, or nil, reason`

```lua
-- An overdrive bar on the HUD, read from the lifecycle state
SetTick(function()
    local state = Open77.reflex.state()
    if not state or not state.active then overdriveBar.visible = false; return end
    overdriveBar.visible = true
    overdriveBar.fill = state.remainingMs / math.max(state.grantedMs, 1)
    overdriveBar.label = state.kind == "reflex_heavy" and "OVERDRIVE+" or "OVERDRIVE"
end)
```

Registered in `ResourceHost.cpp` (line 24915) as `LuaReflexOverdriveState`.
