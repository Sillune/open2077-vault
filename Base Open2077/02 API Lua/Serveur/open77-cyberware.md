# Open77.cyberware (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-cyberware.md) · [Page web](https://open2077.net/docs/api/server/open77-cyberware)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

14 functions.

## activity

```lua
Open77.cyberware.activity(playerId)
```

`NETWORK`: Uses the network backend.

Read fresh authoritative Gorilla charge activity.

Requires players.cyberware.read. Returns {player,incarnation,sequence,armed,holding,charged,chargeExpired,holdAt,elapsedMs,remainingMs,maxChargeMs,definition,grade}. Null for unavailable/stale (over1500ms) body/implant activity. Values drive presentation, not an extra permission to commit damage. Hold expiry latches until release/new swing.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `activity table, or nil`, `nil, reason on failure`

```lua
local activity = Open77.cyberware.activity(playerId)
if activity and activity.holding then print(activity.remainingMs) end
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `LuaResourceRuntime.cs` (line 8072) as `LuaResourceRuntime.Bootstrap`.

## activityPlayers

```lua
Open77.cyberware.activityPlayers()
```

`NETWORK`: Uses the network backend.

List players with fresh, armed, unexpired Gorilla charge holds.

Requires players.cyberware.read. Takes no arguments and synchronously returns an array of player IDs whose current authoritative activity is holding with remainingMs greater than zero. Excludes stale activity (over1500ms), mismatched implant/body incarnations, unarmed players and expired holds. Use these candidates to query activity(playerId) for presentation instead of polling every connected player; the list is not permission to commit damage. Returns an empty array when no players qualify or the action service is unavailable. Returns nil, reason for resource_stopping, permission_denied:players.cyberware.read or cyberware_storage_unavailable.

Returns: `array of player IDs`, `nil, reason on failure`

```lua
local players, reason = Open77.cyberware.activityPlayers()
if not players then print(reason); return end
for _, playerId in ipairs(players) do
    local activity = Open77.cyberware.activity(playerId)
    if activity and activity.holding then print(playerId, activity.remainingMs) end
end
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `LuaResourceRuntime.cs` (line 8073) as `LuaResourceRuntime.Bootstrap`.

## bind

```lua
Open77.cyberware.bind(playerId, character)
```

`NETWORK`: Uses the network backend.

Bind the authenticated player to a server-selected durable character.

Requires players.cyberware.identity. The user UUID comes from the connection; only the character key is supplied. Identity ownership is exclusive to a resource. Starts asynchronous database/native restoration; success is not readiness. Prefer the shipped appearance identity adapter and never accept an unvalidated client-selected character key.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `character` | `string` | required | — |

Returns: `{ok=true,...}`, `nil, reason on rejection`

```lua
-- In a trusted identity adapter after server-side character selection:
local result, err = Open77.cyberware.bind(playerId, "default")
if not result then print(err) end
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `LuaResourceRuntime.cs` (line 8068) as `LuaResourceRuntime.Bootstrap`.

## cancel

```lua
Open77.cyberware.cancel(playerId, ticket)
```

`NETWORK`: Uses the network backend.

Cancel an owned installation/removal before durable commit.

Requires players.cyberware.manage. Supply the ticket from the same resource's operation. Native staging is rolled back; an already submitted storage transaction returns operation_committing and completes normally. Never refund irrevocably until the correlated operation outcome is known.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `ticket` | `string` | required | — |

Returns: `{ok=true,...}`, `nil, reason on rejection`

```lua
local result, err = Open77.cyberware.cancel(playerId, ticket)
if not result then print(err) end
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `LuaResourceRuntime.cs` (line 8083) as `LuaResourceRuntime.Bootstrap`.

## current

```lua
Open77.cyberware.current(playerId)
```

`NETWORK`: Uses the network backend.

Read the ready durable implant record.

Requires players.cyberware.read. Returned synchronously, not a Promise. Record fields are revision, arms and operationId; arms includes instanceId, definition, definitionVersion, profile, slot and the committed grade snapshot. Returns nil while loading/projecting/failed. An empty arms value means no installed implant. Temporary arena overlays do not overwrite this record. A ready record does not imply its definition provider is running.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `record table, or nil when not ready`, `nil, reason on failure`

```lua
local record, err = Open77.cyberware.current(playerId)
if record then print(record.revision, record.arms and record.arms.definition) elseif err then print(err) end
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `LuaResourceRuntime.cs` (line 8070) as `LuaResourceRuntime.Bootstrap`.

## define

```lua
Open77.cyberware.define(definition)
```

`NETWORK`: Uses the network backend.

Register resource-owned Gorilla Arms grades.

Requires players.cyberware.define. definition={id,version,slot="arms",profile="gorilla_arms",grades={...}}. IDs use1..96 ASCII alphanumeric/_.:- characters; version positive;1..32 unique grades. Required grade fields: id,normalDamage/chargedDamage(0..300),knockbackMeters(0..6),cooldownMs(100..600000),chargeMs(100..10000). Optional normalKnockbackMeters defaults to35% of charged, maxChargeMs defaults10000 and must be>=chargeMs and<=60000; nullable normalStaminaCost/chargedStaminaCost0..300 retain native price when omitted. nonlethal/cosmetic defaultfalse; blockDamageMultiplier0..1 default0 and blockAngleDegrees0..180 default45. Stop unregisters the provider and disables its grants without deleting paid records.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `definition` | `table` | required | — |

Returns: `{ok=true,error=nil,ticket=nil,lease=nil}`, `nil, reason on rejection`

```lua
local result, err = Open77.cyberware.define({
 id="myrp.gorilla",version=1,slot="arms",profile="gorilla_arms",
 grades={{id="training",normalDamage=15,chargedDamage=35,
   knockbackMeters=2,cooldownMs=650,chargeMs=700,nonlethal=true,
   normalStaminaCost=10,chargedStaminaCost=20,maxChargeMs=3000}}
})
if not result then print(err) end
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `LuaResourceRuntime.cs` (line 8067) as `LuaResourceRuntime.Bootstrap`.

## effective

```lua
Open77.cyberware.effective(playerId)
```

`NETWORK`: Uses the network backend.

Read the ready implant including an active temporary loadout.

Requires players.cyberware.read. Returned synchronously, not a Promise. Record fields are revision, arms and operationId; arms includes instanceId, definition, definitionVersion, profile, slot and the committed grade snapshot. An active lease overlays arms; durable revision/operationId retain their base meaning. Pending/restoring projections return nil. Use current() for paid storage identity and effective() for ready presentation/gameplay loadout.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `effective record table, or nil`, `nil, reason on failure`

```lua
local record = Open77.cyberware.effective(playerId)
if record and record.arms then print(record.arms.grade.id) end
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `LuaResourceRuntime.cs` (line 8071) as `LuaResourceRuntime.Bootstrap`.

## install

```lua
Open77.cyberware.install(playerId, definition, grade, options)
```

`NETWORK`: Uses the network backend.

Stage a durable Gorilla Arms installation.

Requires players.cyberware.manage. options requires expectedRevision and operationId. Compare-and-swap and receipt storage are atomic; reuse the same ID/contents on retries. Native projection occurs before commit. Completion is asynchronous via onCyberwareOperationCompleted(playerId,ticket,encodedResult), correlated by ticket; retain the pending operation until final outcome. Rejected/failed work restores the prior paid record. Server makers own consent, costs and progression.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `definition` | `string` | required | — |
| `grade` | `string` | required | — |
| `options` | `table` | required | — |

Returns: `{ok=true,ticket=string,...}; ticket may be absent on a completed idempotent replay`, `nil, reason on rejection`

```lua
local record = Open77.cyberware.current(playerId)
if not record then return end
local operationId, err = Open77.cyberware.newOperationId()
if not operationId then print(err); return end
local result, rejected = Open77.cyberware.install(playerId, "myrp.gorilla", "training", {
 expectedRevision=record.revision,operationId=operationId
})
if not result then print(rejected) else print(result.ticket) end
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `LuaResourceRuntime.cs` (line 8079) as `LuaResourceRuntime.Bootstrap`.

## lease

```lua
Open77.cyberware.lease(playerId, definition, grade, [options])
```

`NETWORK`: Uses the network backend.

Stage a temporary implant without overwriting paid state.

Requires players.cyberware.temporary. options.durationMs is1000..300000, default300000. Requires a ready bound character and registered definition/grade. pending→active only after native acknowledgement. Wait for the matching onCyberwareLeaseChanged state before admitting PvP. Owner stop, body/bucket change, expiry or disconnect releases the overlay; native restoration remains asynchronous.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `definition` | `string` | required | — |
| `grade` | `string` | required | — |
| `options` | `table` | optional | — |

Returns: `{ok=true,lease=string,ticket=string,...}`, `nil, reason on rejection`

```lua
local result, err = Open77.cyberware.lease(playerId, "myrp.gorilla", "training", {durationMs=60000})
if not result then print(err); return end
-- Keep result.lease; wait for its matching active event before PvP.
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `LuaResourceRuntime.cs` (line 8075) as `LuaResourceRuntime.Bootstrap`.

## leaseState

```lua
Open77.cyberware.leaseState(playerId)
```

`NETWORK`: Uses the network backend.

Read a temporary loadout lease and restoration phase.

Requires players.cyberware.read. Returns {id,player,phase,definition,grade,expiresAt,ticket,reason}, or nil when none exists. Phases pending/active/restoring/failed; ended is an event state, then the read becomes nil. expiresAt is server-monotonic milliseconds. Wait for nil AND current() ready before another lease.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `lease-state table, or nil`, `nil, reason on failure`

```lua
local state = Open77.cyberware.leaseState(playerId)
if state then print(state.id, state.phase) end
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `LuaResourceRuntime.cs` (line 8074) as `LuaResourceRuntime.Bootstrap`.

## newOperationId

```lua
Open77.cyberware.newOperationId()
```

`NETWORK`: Uses the network backend.

Create an operation ID to retain across durable retries.

Requires players.cyberware.manage. Returns a32-character GUID string. Store it with the pending operation and reuse the same ID, expectedRevision and contents when retrying; generating a new ID per retry loses receipt idempotency. This does not start a transaction.

Returns: `operation ID string`, `nil, reason on rejection`

```lua
local operationId, err = Open77.cyberware.newOperationId()
if not operationId then print(err); return end
-- Persist operationId with the intended mutation before submitting it.
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `LuaResourceRuntime.cs` (line 8066) as `LuaResourceRuntime.Bootstrap`.

## releaseLease

```lua
Open77.cyberware.releaseLease(playerId, lease)
```

`NETWORK`: Uses the network backend.

Release this resource's temporary loadout and restore paid arms.

Requires players.cyberware.temporary. Works during pending/active; repeated restoration requests return the same ticket. Lease id and resource owner must match. An ended event due to body loss alone is not proof of native paid restoration: wait for no lease and a ready current record.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `lease` | `string` | required | — |

Returns: `{ok=true,ticket=string,...}`, `nil, reason on rejection`

```lua
local result, err = Open77.cyberware.releaseLease(playerId, leaseId)
if not result then print(err) end
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `LuaResourceRuntime.cs` (line 8078) as `LuaResourceRuntime.Bootstrap`.

## remove

```lua
Open77.cyberware.remove(playerId, options)
```

`NETWORK`: Uses the network backend.

Stage a durable Gorilla Arms removal.

Requires players.cyberware.manage. options requires expectedRevision and operationId. Compare-and-swap and receipt storage are atomic; reuse the same ID/contents on retries. Native projection occurs before commit. Completion is asynchronous via onCyberwareOperationCompleted(playerId,ticket,encodedResult), correlated by ticket; retain the pending operation until final outcome. Rejected/failed work restores the prior paid record. Server makers own consent, costs and progression.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `options` | `table` | required | — |

Returns: `{ok=true,ticket=string,...}; ticket may be absent on a completed idempotent replay`, `nil, reason on rejection`

```lua
local record = Open77.cyberware.current(playerId)
if not record then return end
local operationId, err = Open77.cyberware.newOperationId()
if not operationId then print(err); return end
local result, rejected = Open77.cyberware.remove(playerId, {
 expectedRevision=record.revision,operationId=operationId
})
if not result then print(rejected) else print(result.ticket) end
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `LuaResourceRuntime.cs` (line 8082) as `LuaResourceRuntime.Bootstrap`.

## unbind

```lua
Open77.cyberware.unbind(playerId)
```

`NETWORK`: Uses the network backend.

Release this resource's character binding and runtime grants.

Requires players.cyberware.identity. Does not delete durable implants. Stops current projection/temporary ownership and prevents stale acknowledgements from restoring the old grant. Another owner cannot unbind the character.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `{ok=true,...}`, `nil, reason on rejection`

```lua
-- The character logged out: release the binding and its runtime grants
AddEventHandler("onPlayerDisconnected", function(playerId)
    Open77.cyberware.unbind(playerId)
end)
```

[Gorilla Arms implementation guide](../../01%20Documentation/gorilla-arms.md)

Registered in `LuaResourceRuntime.cs` (line 8069) as `LuaResourceRuntime.Bootstrap`.
