# Open77.cyberware (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-cyberware.md) · [Page web](https://open2077.net/docs/api/client/open77-cyberware)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

20 functions.

## approveLegJump

```lua
Open77.cyberware.approveLegJump(request, sequence, allowed)
```

`GAME`: Requires a live game instance.

Answer the owned native second-jump intent with the server's decision.

Requires player.cyberware.project and a running resource. request is the owned native request; sequence is the positive uint32 of the pending action reported by legsActivity; allowed is a boolean. true means the matching pending native intent accepted the decision, not that a jump was consumed, displaced or rendered. Open77 buffers an eligible press for at most 750 ms, so a late grant still has to pass the native timeout and the original native predicates. Stamina is charged on server admission and is not refunded when native execution later expires. This is a trusted projection primitive, not a public server-side jump command or a security boundary against a modified client.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `request` | `integer` | required | — |
| `sequence` | `integer` | required | — |
| `allowed` | `boolean` | required | — |

Returns: `true on acceptance`, `nil, reason on rejection`

```lua
local ok, reason = Open77.cyberware.approveLegJump(request, activity.sequence, true)
if not ok then print(reason) end
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `ResourceHost.cpp` (line 24842) as `LuaCyberwareApproveLegJump`.

## attackState

```lua
Open77.cyberware.attackState()
```

`GAME`: Requires a live game instance.

Read local native Gorilla melee activity.

Requires player.cyberware.read. Trusted native projection primitive used by open77_cyberware; it does not install a durable implant or authorize damage. Returns {sequence,holding,variant,record,chargeExpired} when a usable native Gorilla weapon is observed; unavailable/unarmed states return nil, reason. This is client observation, not authoritative charge, damage or cooldown. The platform forwards bounded intent and the server validates it.

Returns: `activity table`, `nil, reason on rejection`

```lua
local state = Open77.cyberware.attackState()
if state then print(state.sequence, state.holding, state.chargeExpired) end
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `ResourceHost.cpp` (line 24838) as `LuaCyberwareAttackState`.

## captureArms

```lua
Open77.cyberware.captureArms()
```

`GAME`: Requires a live game instance.

Capture the local native Gorilla Arms posture profiles.

Requires player.cyberware.read. Trusted native projection primitive used by open77_cyberware; it does not install a durable implant or authorize damage. Returns {family,groups}; groups contain part/name hashes and key pairs. The validated adapter supplies matching holstered and drawn alternatives for male or female, not arbitrary replacement assets. Native readiness and equipment readback do not prove rendered geometry.

Returns: `body profile table`, `nil, reason on rejection`

```lua
local profile, err = Open77.cyberware.captureArms()
if not profile then print(err); return end
print(profile.family, #profile.groups)
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `ResourceHost.cpp` (line 24833) as `LuaCyberwareCaptureArms`.

## configureLegs

```lua
Open77.cyberware.configureLegs(request, staminaManaged, maxAirborneMs, maxFallSpeed)
```

`GAME`: Requires a live game instance.

Configure the owned ready legs adapter's admission limits.

Requires player.cyberware.project and a running resource. request must be a ready owned request. staminaManaged=true selects suppression of the native second-jump stamina debit and requires the server pricing path; it is not a stamina cost argument. maxAirborneMs is an integer 100-10000 and maxFallSpeed a finite 0.1-30 m/s, matching the grade table. These limits only narrow native eligibility: they never change jump height, trajectory, gravity, fall damage or native collision response. Support sets them from the installed grade snapshot.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `request` | `integer` | required | — |
| `staminaManaged` | `boolean` | required | — |
| `maxAirborneMs` | `integer` | required | — |
| `maxFallSpeed` | `number` | required | — |

Returns: `true on acceptance`, `nil, reason on rejection`

```lua
local ok, reason = Open77.cyberware.configureLegs(request, true, 10000, 30.0)
if not ok then print(reason) end
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `ResourceHost.cpp` (line 24841) as `LuaCyberwareConfigureLegs`.

## configureLocal

```lua
Open77.cyberware.configureLocal(request, options)
```

`GAME`: Requires a live game instance.

Configure native charge limits and stamina-override presence.

Requires player.cyberware.project. Trusted native projection primitive used by open77_cyberware; it does not install a durable implant or authorize damage. Requires the same owned ready request. options accepts nullable normalStaminaCost/chargedStaminaCost (0..300) and maxChargeMs (100..60000, default10000). Native code uses cost presence to suppress supported native attack prices; the server owns the actual costs and debits. It does not grant stamina or suppress unrelated movement costs.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `request` | `integer` | required | — |
| `options` | `table` | required | — |

Returns: `true`, `nil, reason on rejection`

```lua
local ok, err = Open77.cyberware.configureLocal(request, {
    normalStaminaCost=10, chargedStaminaCost=20, maxChargeMs=3000
})
if not ok then print(err) end
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `ResourceHost.cpp` (line 24837) as `LuaCyberwareConfigureLocal`.

## customizationCatalog

```lua
Open77.cyberware.customizationCatalog()
```

`GAME`: Requires a live game instance.

List Cyberpunk 2077 2.5 cosmetic cyberware styles and palettes.

Requires player.cyberware.read. Reads usable UIGlamdocCustomizationItem records from the installed TweakDB, including modded records. Entries contain record, type, brand, index, longSleevesIndex, defaultPrimary, defaultSecondary and five RGB colors. Lua palette array positions 1–5 correspond to setter indices 0–4. No menu is opened and no implant is granted. Requires Stable 152 or later on Cyberpunk 2077 2.5. See [Cyberware styles and colors](https://open2077.net/docs/api/client/cyberware-customization.md).

Returns: `array of style tables`, `nil, reason on failure`

```lua
local styles, reason = Open77.cyberware.customizationCatalog()
if not styles then print(reason); return end
for _, style in ipairs(styles) do print(style.record, style.type) end
```

[Cyberware styles & colors](../../01%20Documentation/cyberware-customization.md)

Registered in `ResourceHost.cpp` (line 25778) as `LuaCyberwareCustomizationCatalog`.

## getCustomization

```lua
Open77.cyberware.getCustomization()
```

`GAME`: Requires a live game instance.

Read the local cosmetic cyberware style as a restorable snapshot.

Requires player.cyberware.read and a loaded character. Returns schemaVersion=1, enabled, record, primary and secondary; colors are explicit sRGB {r,g,b} bytes. Pass the table to setCustomization to restore it. Persistence belongs to the server's character system. This is local native state, not a remote-player replication API. Requires Stable 152 or later on Cyberpunk 2077 2.5. See [Cyberware styles and colors](https://open2077.net/docs/api/client/cyberware-customization.md).

Returns: `snapshot table`, `nil, reason on failure`

```lua
local style, reason = Open77.cyberware.getCustomization()
if style then print(style.record, style.enabled) else print(reason) end
```

[Cyberware styles & colors](../../01%20Documentation/cyberware-customization.md)

Registered in `ResourceHost.cpp` (line 25779) as `LuaCyberwareGetCustomization`.

## grantLegLease

```lua
Open77.cyberware.grantLegLease(lease, epoch, budget, ttlMs)
```

`GAME`: Requires a live game instance.

Hold the server's double-jump activation lease on the local body.

Requires player.cyberware.project and a running resource. Trusted projection primitive used by the shipped open77_cyberware client: with a lease held, a second-jump press is spent locally and the jump happens on the frame the player pressed; the server is told afterwards and reconciles at the stamp. lease is the server's lease id, epoch its uint32 epoch, budget the activations it allows (1-64) and ttlMs how much longer it is valid (1-30000) -- a remaining lifetime, never an absolute deadline, so no clock domain is mixed. Returns true when the lease is held; nil, reason otherwise (invalid_lease, cyberware_backend_unavailable). Without a lease the ask-and-wait path still runs.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `lease` | `string (lease id)` | required | — |
| `epoch` | `integer` | required | — |
| `budget` | `integer 1-64` | required | — |
| `ttlMs` | `integer 1-30000` | required | — |

Returns: `true`, `nil, reason on rejection`

```lua
-- Mirror the server's lease grant onto the body (open77_cyberware does this for you)
RegisterNetEvent("open77:leases:grant", function(value)
    if value.kind ~= "double_jump" then return end
    local ttl = value.deadlineMs - serverNowMs()
    local ok, err = Open77.cyberware.grantLegLease(value.leaseId, value.epoch or 0, value.budget or 1, ttl)
    if not ok then print(err) end
end)
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `ResourceHost.cpp` (line 24843) as `LuaCyberwareGrantLegLease`.

## legReportSent

```lua
Open77.cyberware.legReportSent(reportSequence)
```

`GAME`: Requires a live game instance.

Mark the pending leased jump report as sent.

Requires player.cyberware.project. After a leased press the activity reported by legsActivity carries reportSequence; once the resource has sent that report to the server, this returns the activation to idle. The lease keeps whatever budget it has left.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `reportSequence` | `integer` | required | — |

```lua
local activity = Open77.cyberware.legsActivity()
if activity and activity.reportSequence > 0 then
    TriggerServerEvent("open77:cyberware:legReport", { sequence = activity.reportSequence, ageMs = activity.ageMs })
    Open77.cyberware.legReportSent(activity.reportSequence)
end
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `ResourceHost.cpp` (line 24846) as `LuaCyberwareLegReportSent`.

## legsActivity

```lua
Open77.cyberware.legsActivity()
```

`GAME`: Requires a live game instance.

Read fresh native legs action and airborne state.

Requires player.cyberware.read. Returns {request,sequence,phase,grounded,airborneMs,verticalSpeed}. Phases are idle, pending, granted, consumed and rejected; sequence is a positive uint32 while an action is pending and can report zero when idle. This is local native observation, not an authorization and not rendered proof. A resource that does not own the native adapter should use open77_cyberware's read-only legsActivity export instead: the core method inspects only the calling VM's own owner.

Returns: `activity table`, `nil, reason on rejection`

```lua
local activity, reason = Open77.cyberware.legsActivity()
if activity then print(activity.phase, activity.airborneMs) else print(reason) end
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `ResourceHost.cpp` (line 24847) as `LuaCyberwareLegsActivity`.

## legsState

```lua
Open77.cyberware.legsState(request)
```

`GAME`: Requires a live game instance.

Read the owned native legs projection phase.

Requires player.cyberware.project. request is the positive ID returned by projectLegs. Returns "pending", "ready" or "failed" plus a reason string for a validly shaped query; a malformed request returns nil, reason. "ready" means the native equipment readback matched across three bridge polls, not that a second jump was performed or rendered. Another resource cannot read this owner's private request by guessing its numeric ID.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `request` | `integer` | required | — |

Returns: `phase string, reason string`, `nil, reason on a malformed query`

```lua
local phase, reason = Open77.cyberware.legsState(request)
if phase ~= 'ready' then print(phase, reason) end
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `ResourceHost.cpp` (line 24840) as `LuaCyberwareLegsState`.

## localState

```lua
Open77.cyberware.localState(request)
```

`GAME`: Requires a live game instance.

Read the native equipment projection state for an owned request.

Requires player.cyberware.project. Trusted native projection primitive used by open77_cyberware; it does not install a durable implant or authorize damage. Returns a phase string and a reason string: pending, ready or failed for the matching native request. Invalid/foreign native requests may return an empty phase plus reason; Lua permission/argument failures return nil, reason. ready describes native equipment readback, not SQL commit or visible confirmation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `request` | `integer` | required | — |

Returns: `phase string, reason string`, `nil, reason on Lua validation failure`

```lua
local phase, reason = Open77.cyberware.localState(request)
if phase == "failed" or not phase or phase == "" then print(reason) end
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `ResourceHost.cpp` (line 24836) as `LuaCyberwareLocalState`.

## presentArms

```lua
Open77.cyberware.presentArms(playerId, profile)
```

`GAME`: Requires a live game instance.

Project or clear an owned Gorilla Arms appearance overlay.

Requires player.cyberware.project. Trusted native projection primitive used by open77_cyberware; it does not install a durable implant or authorize damage. playerId is the network player ID, including the local player for its self-view. Pass a validated captured {family,groups} profile, or false to clear. Overlay ownership survives identical replays; conflicting owners fail. Native attachment/posture work is asynchronous; true is acceptance, not visible completion.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `profile` | `table|false` | required | — |

Returns: `boolean accepted`, `reason on failure`

```lua
local profile, err = Open77.cyberware.captureArms()
if profile then
    local ok, reason = Open77.cyberware.presentArms(playerId, profile)
    if not ok then print(reason) end
end
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `ResourceHost.cpp` (line 24834) as `LuaCyberwarePresentArms`.

## projectLegs

```lua
Open77.cyberware.projectLegs(enabled)
```

`GAME`: Requires a live game instance.

Queue native local double-jump legs installation or removal.

Requires player.cyberware.project and a running resource. Trusted native projection primitive used by open77_cyberware; it does not install a durable implant, grant a jump or authorize damage. enabled=true equips the supported native legs cyberware (Items.BoostedTendonsRare through the native LegsCW area); false releases it. Returns an owned positive integer request ID; poll legsState(request). Readiness requires matching owned item and HasDoubleJump readback and is not rendered proof. Arms and legs have distinct native owners, requests and equipment areas, so a legs request never disturbs an installed Gorilla Arms projection. Resource stop invokes owned cleanup, which unequips only the item this adapter created.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `positive request id`, `nil, reason on rejection`

```lua
local request, reason = Open77.cyberware.projectLegs(true)
if not request then print(reason); return end
print(Open77.cyberware.legsState(request))
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `ResourceHost.cpp` (line 24839) as `LuaCyberwareProjectLegs`.

## projectLocal

```lua
Open77.cyberware.projectLocal(enabled)
```

`GAME`: Requires a live game instance.

Queue native local Gorilla equipment installation or removal.

Requires player.cyberware.project. Trusted native projection primitive used by open77_cyberware; it does not install a durable implant or authorize damage. enabled=true equips the supported native arms; false restores bare arms. Returns an owned positive integer request ID. Poll localState(request); queued work is neither a committed purchase nor visual proof. Resource stop releases the native owner.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `request ID`, `nil, reason on rejection`

```lua
local request, err = Open77.cyberware.projectLocal(true)
if not request then print(err); return end
-- The platform projector correlates native readiness with the server ticket.
print(Open77.cyberware.localState(request))
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `ResourceHost.cpp` (line 24835) as `LuaCyberwareProjectLocal`.

## refreshLegLease

```lua
Open77.cyberware.refreshLegLease(epoch, remaining, ttlMs)
```

`GAME`: Requires a live game instance.

Refresh the held double-jump lease with the server's latest budget and lifetime.

Requires player.cyberware.project. Applies the 20 Hz refresh the server sends on its snapshot stream: epoch must match the held lease, remaining is the activations still allowed, ttlMs the remaining lifetime. A refresh for another epoch is ignored. Trusted projection primitive; not a public API.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `epoch` | `integer` | required | — |
| `remaining` | `integer` | required | — |
| `ttlMs` | `integer 1-30000` | required | — |

```lua
RegisterNetEvent("open77:leases:refresh", function(value)
    if value.kind ~= "double_jump" then return end
    Open77.cyberware.refreshLegLease(value.epoch, value.remaining or 0, value.deadlineMs - serverNowMs())
end)
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `ResourceHost.cpp` (line 24844) as `LuaCyberwareRefreshLegLease`.

## releaseLegLease

```lua
Open77.cyberware.releaseLegLease()
```

`GAME`: Requires a live game instance.

Drop the held double-jump lease; presses go back to ask-and-wait.

Requires player.cyberware.project. Called on a server revoke, on a session change and on resource stop. Idempotent.

```lua
RegisterNetEvent("open77:leases:revoke", function(value)
    if value.kind == "double_jump" then Open77.cyberware.releaseLegLease() end
end)
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `ResourceHost.cpp` (line 24845) as `LuaCyberwareReleaseLegLease`.

## releaseLegs

```lua
Open77.cyberware.releaseLegs()
```

`GAME`: Requires a live game instance.

Request owned native legs cleanup.

Requires player.cyberware.project and a running resource. Releases only the calling resource's own native legs projection and is harmless when it owns none. true acknowledges the request; completion is asynchronous. Cleanup unequips this adapter's owned item and removes an inventory copy only when this adapter created it, so a pre-existing unowned double-jump capability is left alone. Resource teardown invokes the same owned cleanup even after Lua callbacks have ended.

Returns: `true`, `nil, reason on rejection`

```lua
-- The double-jump trial ended: release this resource's legs projection
AddEventHandler("trial:ended", function()
    Open77.cyberware.releaseLegs()
end)
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `ResourceHost.cpp` (line 24848) as `LuaCyberwareReleaseLegs`.

## resetCustomization

```lua
Open77.cyberware.resetCustomization()
```

`GAME`: Requires a live game instance.

Restore the native cyberware style from before this resource's override.

Requires player.cyberware.project. Only the resource that owns the customization can reset it. Restores both the prior model and colors and releases ownership. Resource stop performs the same cleanup. A reconnect or body load requires the character resource to reapply its saved customization. Requires Stable 152 or later on Cyberpunk 2077 2.5. See [Cyberware styles and colors](https://open2077.net/docs/api/client/cyberware-customization.md). Model changes/restoration wait until cyberarms or fists are holstered and their attachment is ready; colors on an existing customized model update immediately.

Returns: `true on restoration`, `nil, reason on failure`

```lua
local ok, reason = Open77.cyberware.resetCustomization()
if not ok then print(reason) end
```

[Cyberware styles & colors](../../01%20Documentation/cyberware-customization.md)

Registered in `ResourceHost.cpp` (line 25781) as `LuaCyberwareResetCustomization`.

## setCustomization

```lua
Open77.cyberware.setCustomization(options)
```

`GAME`: Requires a live game instance.

Apply a local cosmetic cyberware model and two colors without native UI.

Requires player.cyberware.project and a loaded character. options.record is a catalog record ID or TweakDB record name. primary and secondary independently accept a palette index 0–4 or integer sRGB {r,g,b} channels 0–255; omitted colors use the style defaults. enabled defaults to true; {enabled=false} clears the model selector and tint. Cosmetic changes grant no implant powers. The resource owns its override until reset/stop; another owner receives customization_owned. true confirms native dispatch; geometry streams asynchronously. This API does not automatically replicate colors to remote players. Requires Stable 152 or later on Cyberpunk 2077 2.5. See [Cyberware styles and colors](https://open2077.net/docs/api/client/cyberware-customization.md). Model changes/restoration wait until cyberarms or fists are holstered and their attachment is ready; colors on an existing customized model update immediately.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table` | required | — |

Returns: `true on application`, `nil, reason on failure`

```lua
local styles = assert(Open77.cyberware.customizationCatalog())
if styles[1] then
    assert(Open77.cyberware.setCustomization({record=styles[1].record, primary=0, secondary={r=20,g=150,b=240}}))
end
```

[Cyberware styles & colors](../../01%20Documentation/cyberware-customization.md)

Registered in `ResourceHost.cpp` (line 25780) as `LuaCyberwareSetCustomization`.
