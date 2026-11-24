# open77_doors (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/resource-open77-doors.md) · [Page web](https://open2077.net/docs/api/server/resource-open77-doors)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

18 functions.

## changes

```lua
Open77.exports.call("open77_doors", "changes", cursor)
```

`SERVER`: Available in server resources only.

Read a bounded feed of authoritative door changes.

At most eight events per page, from a 512-event ring. Continue with the returned cursor; if reset is true, rescan list. Kinds: changed, removed, request, action, npc_request (reason = NPC id, value = the simulating player). Supports cross-resource observation without the host-wide event bus; door actions are also published there as open77:doors:action.

This is an asynchronous server-side resource export. Depend on open77_doors >=1.0.0 and call Open77.exports.call, not a Lua table named open77_doors. The call returns a Promise (or nil, error if it cannot be queued); awaiting it yields: { cursor, reset, events }. Available with client and server release 2.31.13+op77.58 on the CDN; older clients do not contain its native projection bridge.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `cursor` | `integer` | required | — |

Returns: `Promise | nil`, `queue error, if any`

```lua
CreateThread(function()
    local pending, error = Open77.exports.call("open77_doors", "changes", cursor)
    if not pending then print(error); return end
    local result, reason = pending:await()
    -- Handle the result before continuing; queued is not accepted.
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Documented in [resources/system/open77_doors/server/main.lua](../../01%20Documentation/doors.md).

## configure

```lua
Open77.exports.call("open77_doors", "configure", id, bucket, patch)
```

`SERVER`: Available in server resources only.

Configure a door atomically on the server.

Requires ownership. Fields: open, locked, sealed, automatic, autoClose, defaultAccess, openRadius, closeRadius, closeDelay, npcPassage ('public', 'resource', 'always' or 'never'). Radii 1–12m, closeRadius >= openRadius, delay 0.2–60s. Invalid patches change nothing. Locks/seals close the door; lift opening is controlled by the elevator.

This is an asynchronous server-side resource export. Depend on open77_doors >=1.0.0 and call Open77.exports.call, not a Lua table named open77_doors. The call returns a Promise (or nil, error if it cannot be queued); awaiting it yields: true or false, reason. Available with client and server release 2.31.13+op77.58 on the CDN; older clients do not contain its native projection bridge.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |
| `bucket` | `integer` | required | — |
| `patch` | `table` | required | — |

Returns: `Promise | nil`, `queue error, if any`

```lua
CreateThread(function()
    local pending, error = Open77.exports.call("open77_doors", "configure", id, bucket, patch)
    if not pending then print(error); return end
    local result, reason = pending:await()
    -- Handle the result before continuing; queued is not accepted.
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Documented in [resources/system/open77_doors/server/main.lua](../../01%20Documentation/doors.md).

## get

```lua
Open77.exports.call("open77_doors", "get", id, bucket)
```

`SERVER`: Available in server resources only.

Read the authoritative state of a world door.

IDs are opaque hex strings. The bucket defaults to 0. The result is a copy and may exist before the native object streams.

This is an asynchronous server-side resource export. Depend on open77_doors >=1.0.0 and call Open77.exports.call, not a Lua table named open77_doors. The call returns a Promise (or nil, error if it cannot be queued); awaiting it yields: Door snapshot or nil. Available with client and server release 2.31.13+op77.58 on the CDN; older clients do not contain its native projection bridge.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |
| `bucket` | `integer` | optional | `0` |

Returns: `Promise | nil`, `queue error, if any`

```lua
CreateThread(function()
    local pending, error = Open77.exports.call("open77_doors", "get", id, bucket)
    if not pending then print(error); return end
    local result, reason = pending:await()
    -- Handle the result before continuing; queued is not accepted.
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Documented in [resources/system/open77_doors/server/main.lua](../../01%20Documentation/doors.md).

## linkElevator

```lua
Open77.exports.call("open77_doors", "linkElevator", id, bucket, elevatorId, floor)
```

`SERVER`: Available in server resources only.

Link a landing door to its elevator and floor.

Requires ownership of a lift door. The lift must exist in the same bucket, have the zero-based floor, and be horizontally within 40m. Opening requires an idle, powered cabin at that floor with doorsClosed unset.

This is an asynchronous server-side resource export. Depend on open77_doors >=1.0.0 and call Open77.exports.call, not a Lua table named open77_doors. The call returns a Promise (or nil, error if it cannot be queued); awaiting it yields: true or false, reason. Available with client and server release 2.31.13+op77.58 on the CDN; older clients do not contain its native projection bridge.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |
| `bucket` | `integer` | required | — |
| `elevatorId` | `integer` | required | — |
| `floor` | `integer` | required | — |

Returns: `Promise | nil`, `queue error, if any`

```lua
CreateThread(function()
    local pending, error = Open77.exports.call("open77_doors", "linkElevator", id, bucket, elevatorId, floor)
    if not pending then print(error); return end
    local result, reason = pending:await()
    -- Handle the result before continuing; queued is not accepted.
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Documented in [resources/system/open77_doors/server/main.lua](../../01%20Documentation/doors.md).

## list

```lua
Open77.exports.call("open77_doors", "list", bucket, offset, limit)
```

`SERVER`: Available in server resources only.

List registered doors with bounded pagination.

Bucket may be nil for all buckets. Offset defaults to 0, limit to 16 (maximum 16). Pagination is not transactional; rescan if the registry changes.

This is an asynchronous server-side resource export. Depend on open77_doors >=1.0.0 and call Open77.exports.call, not a Lua table named open77_doors. The call returns a Promise (or nil, error if it cannot be queued); awaiting it yields: { doors, total, nextOffset? }. Available with client and server release 2.31.13+op77.58 on the CDN; older clients do not contain its native projection bridge.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | optional | — |
| `offset` | `integer` | optional | `0` |
| `limit` | `integer` | optional | `16` |

Returns: `Promise | nil`, `queue error, if any`

```lua
CreateThread(function()
    local pending, error = Open77.exports.call("open77_doors", "list", bucket, offset, limit)
    if not pending then print(error); return end
    local result, reason = pending:await()
    -- Handle the result before continuing; queued is not accepted.
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Documented in [resources/system/open77_doors/server/main.lua](../../01%20Documentation/doors.md).

## near

```lua
Open77.exports.call("open77_doors", "near", position, bucket, radius)
```

`SERVER`: Available in server resources only.

Find the nearest authoritative doors.

Position is {x,y,z}. The radius must be positive and no more than 120 metres. At most 16 closest doors are returned.

This is an asynchronous server-side resource export. Depend on open77_doors >=1.0.0 and call Open77.exports.call, not a Lua table named open77_doors. The call returns a Promise (or nil, error if it cannot be queued); awaiting it yields: { doors, total, truncated }. Available with client and server release 2.31.13+op77.58 on the CDN; older clients do not contain its native projection bridge.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `position` | `{ x: number, y: number, z: number }` | required | — |
| `bucket` | `integer` | required | — |
| `radius` | `number` | required | — |

Returns: `Promise | nil`, `queue error, if any`

```lua
CreateThread(function()
    local pending, error = Open77.exports.call("open77_doors", "near", position, bucket, radius)
    if not pending then print(error); return end
    local result, reason = pending:await()
    -- Handle the result before continuing; queued is not accepted.
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Documented in [resources/system/open77_doors/server/main.lua](../../01%20Documentation/doors.md).

## npcStats

```lua
Open77.exports.call("open77_doors", "npcStats")
```

`SERVER`: Available in server resources only.

Count NPC door openings admitted and refused.

accepted is the number of NPC openings admitted since the service started; refused maps each refusal reason (not_npc_authority, unknown_npc, npc_not_alive, too_far, wrong_bucket, locked, access_denied, npc_passage_denied, sealed, elevator_controlled, rate_limited, invalid_npc_request, ...) to its count. A refused NPC request is never answered to the client, so this is where it is seen. The result is a copy.

This is an asynchronous server-side resource export. Depend on open77_doors >=1.2.0 and call Open77.exports.call, not a Lua table named open77_doors. The call returns a Promise (or nil, error if it cannot be queued); awaiting it yields: { accepted, refused }. Requires client and server builds that include this open77_doors version; players on an older client never send the matching requests.

Returns: `Promise | nil`, `queue error, if any`

```lua
CreateThread(function()
    local pending, error = Open77.exports.call("open77_doors", "npcStats")
    if not pending then print(error); return end
    local result, reason = pending:await()
    -- Handle the result before continuing; queued is not accepted.
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Documented in [resources/system/open77_doors/server/main.lua](../../01%20Documentation/doors.md).

## register

```lua
Open77.exports.call("open77_doors", "register", descriptor)
```

`SERVER`: Available in server resources only.

Register or claim a world door for the calling resource.

A new descriptor requires id and position; bucket defaults to zero. Optional automatic, autoClose, lift, elevatorId and elevatorFloor describe initial topology. Claims preserve an existing door's state; another resource's ownership cannot be stolen.

This is an asynchronous server-side resource export. Depend on open77_doors >=1.0.0 and call Open77.exports.call, not a Lua table named open77_doors. The call returns a Promise (or nil, error if it cannot be queued); awaiting it yields: Door snapshot or nil, reason. Available with client and server release 2.31.13+op77.58 on the CDN; older clients do not contain its native projection bridge.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `descriptor` | `table` | required | — |

Returns: `Promise | nil`, `queue error, if any`

```lua
CreateThread(function()
    local pending, error = Open77.exports.call("open77_doors", "register", descriptor)
    if not pending then print(error); return end
    local result, reason = pending:await()
    -- Handle the result before continuing; queued is not accepted.
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Documented in [resources/system/open77_doors/server/main.lua](../../01%20Documentation/doors.md).

## remove

```lua
Open77.exports.call("open77_doors", "remove", id, bucket)
```

`SERVER`: Available in server resources only.

Release a door owned by this resource.

Stops projecting this registry record. Discovery may subsequently adopt it again. Owner restarts also release their records; persist rules in your own resource if required.

This is an asynchronous server-side resource export. Depend on open77_doors >=1.0.0 and call Open77.exports.call, not a Lua table named open77_doors. The call returns a Promise (or nil, error if it cannot be queued); awaiting it yields: true or false, reason. Available with client and server release 2.31.13+op77.58 on the CDN; older clients do not contain its native projection bridge.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |
| `bucket` | `integer` | required | — |

Returns: `Promise | nil`, `queue error, if any`

```lua
CreateThread(function()
    local pending, error = Open77.exports.call("open77_doors", "remove", id, bucket)
    if not pending then print(error); return end
    local result, reason = pending:await()
    -- Handle the result before continuing; queued is not accepted.
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Documented in [resources/system/open77_doors/server/main.lua](../../01%20Documentation/doors.md).

## resolveAction

```lua
Open77.exports.call("open77_doors", "resolveAction", ticket, accept, reason)
```

`SERVER`: Available in server resources only.

Approve or refuse a pending owner-approved door action.

Tickets arrive in the host-wide open77:doors:action event (stage 'requested') and in changes. Only the door's owner can resolve; a ticket resolves once and expires after 5 s. accept = true re-checks range, seal and price and answers true only if the door was opened: charge on true, never on false. A refusal reason matching [A-Za-z0-9_.:-]{1,48} is forwarded to the client.

This is an asynchronous server-side resource export. Depend on open77_doors >=1.1.0 and call Open77.exports.call, not a Lua table named open77_doors. The call returns a Promise (or nil, error if it cannot be queued); awaiting it yields: true or false, reason. Requires client and server builds that include this open77_doors version; players on an older client never send the matching requests.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `ticket` | `integer` | required | — |
| `accept` | `boolean` | required | — |
| `reason` | `string` | optional | — |

Returns: `Promise | nil`, `queue error, if any`

```lua
CreateThread(function()
    local pending, error = Open77.exports.call("open77_doors", "resolveAction", ticket, accept, reason)
    if not pending then print(error); return end
    local result, reason = pending:await()
    -- Handle the result before continuing; queued is not accepted.
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Documented in [resources/system/open77_doors/server/main.lua](../../01%20Documentation/doors.md).

## setAccess

```lua
Open77.exports.call("open77_doors", "setAccess", id, bucket, playerId, allow)
```

`SERVER`: Available in server resources only.

Grant or deny a connected player door access.

Requires ownership. A boolean overrides defaultAccess; nil clears the override. Overrides are removed on disconnect. This shared physical door is not an anti-tailgating barrier.

This is an asynchronous server-side resource export. Depend on open77_doors >=1.0.0 and call Open77.exports.call, not a Lua table named open77_doors. The call returns a Promise (or nil, error if it cannot be queued); awaiting it yields: true or false, reason. Available with client and server release 2.31.13+op77.58 on the CDN; older clients do not contain its native projection bridge.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |
| `bucket` | `integer` | required | — |
| `playerId` | `integer` | required | — |
| `allow` | `boolean | nil` | required | — |

Returns: `Promise | nil`, `queue error, if any`

```lua
CreateThread(function()
    local pending, error = Open77.exports.call("open77_doors", "setAccess", id, bucket, playerId, allow)
    if not pending then print(error); return end
    local result, reason = pending:await()
    -- Handle the result before continuing; queued is not accepted.
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Documented in [resources/system/open77_doors/server/main.lua](../../01%20Documentation/doors.md).

## setActions

```lua
Open77.exports.call("open77_doors", "setActions", id, bucket, actions)
```

`SERVER`: Available in server resources only.

Declare which door actions (force, pay, hack) a door accepts.

Requires ownership; nothing is allowed by default and nil or {} withdraws everything. force/hack: true or { approval = 'auto'|'owner', keepUnlocked, range (hack, 1-60 m, default 30) }. pay: { price = 1..1e9 }, always owner-approved. An accepted action unlocks and opens the door; the lock returns when it shuts unless keepUnlocked. Actions bypass the lock and defaultAccess, never an explicit setAccess denial. Lift doors reject actions; invalid declarations change nothing.

This is an asynchronous server-side resource export. Depend on open77_doors >=1.1.0 and call Open77.exports.call, not a Lua table named open77_doors. The call returns a Promise (or nil, error if it cannot be queued); awaiting it yields: true or false, reason. Requires client and server builds that include this open77_doors version; players on an older client never send the matching requests.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |
| `bucket` | `integer` | required | — |
| `actions` | `table | nil` | required | — |

Returns: `Promise | nil`, `queue error, if any`

```lua
CreateThread(function()
    local pending, error = Open77.exports.call("open77_doors", "setActions", id, bucket, actions)
    if not pending then print(error); return end
    local result, reason = pending:await()
    -- Handle the result before continuing; queued is not accepted.
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Documented in [resources/system/open77_doors/server/main.lua](../../01%20Documentation/doors.md).

## setAutomatic

```lua
Open77.exports.call("open77_doors", "setAutomatic", id, bucket, automatic)
```

`SERVER`: Available in server resources only.

Enable or disable authoritative proximity opening.

Requires ownership. Automatic doors use fresh player positions in their bucket; NPC-only local proximity does not drive the server. Lift doors reject this operation.

This is an asynchronous server-side resource export. Depend on open77_doors >=1.0.0 and call Open77.exports.call, not a Lua table named open77_doors. The call returns a Promise (or nil, error if it cannot be queued); awaiting it yields: true or false, reason. Available with client and server release 2.31.13+op77.58 on the CDN; older clients do not contain its native projection bridge.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |
| `bucket` | `integer` | required | — |
| `automatic` | `boolean` | required | — |

Returns: `Promise | nil`, `queue error, if any`

```lua
CreateThread(function()
    local pending, error = Open77.exports.call("open77_doors", "setAutomatic", id, bucket, automatic)
    if not pending then print(error); return end
    local result, reason = pending:await()
    -- Handle the result before continuing; queued is not accepted.
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Documented in [resources/system/open77_doors/server/main.lua](../../01%20Documentation/doors.md).

## setDiscoveryEnabled

```lua
Open77.exports.call("open77_doors", "setDiscoveryEnabled", enabled)
```

`SERVER`: Available in server resources only.

Enable or disable acceptance of new discovered doors.

Existing registered doors remain managed. Use explicit server registration with discovery disabled for security-sensitive worlds; discovery is not a cryptographic attestation of client topology.

This is an asynchronous server-side resource export. Depend on open77_doors >=1.0.0 and call Open77.exports.call, not a Lua table named open77_doors. The call returns a Promise (or nil, error if it cannot be queued); awaiting it yields: true or false, reason. Available with client and server release 2.31.13+op77.58 on the CDN; older clients do not contain its native projection bridge.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `Promise | nil`, `queue error, if any`

```lua
CreateThread(function()
    local pending, error = Open77.exports.call("open77_doors", "setDiscoveryEnabled", enabled)
    if not pending then print(error); return end
    local result, reason = pending:await()
    -- Handle the result before continuing; queued is not accepted.
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Documented in [resources/system/open77_doors/server/main.lua](../../01%20Documentation/doors.md).

## setLocked

```lua
Open77.exports.call("open77_doors", "setLocked", id, bucket, locked)
```

`SERVER`: Available in server resources only.

Lock or unlock a shared door.

Requires ownership and a strict boolean. Locking closes the door and blocks all players, including access-granted players.

This is an asynchronous server-side resource export. Depend on open77_doors >=1.0.0 and call Open77.exports.call, not a Lua table named open77_doors. The call returns a Promise (or nil, error if it cannot be queued); awaiting it yields: true or false, reason. Available with client and server release 2.31.13+op77.58 on the CDN; older clients do not contain its native projection bridge.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |
| `bucket` | `integer` | required | — |
| `locked` | `boolean` | required | — |

Returns: `Promise | nil`, `queue error, if any`

```lua
CreateThread(function()
    local pending, error = Open77.exports.call("open77_doors", "setLocked", id, bucket, locked)
    if not pending then print(error); return end
    local result, reason = pending:await()
    -- Handle the result before continuing; queued is not accepted.
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Documented in [resources/system/open77_doors/server/main.lua](../../01%20Documentation/doors.md).

## setNpcPassage

```lua
Open77.exports.call("open77_doors", "setNpcPassage", id, bucket, mode)
```

`SERVER`: Available in server resources only.

Choose which network NPCs may open a door.

Requires ownership. 'public' (default): any network NPC opens an unlocked door open to everyone. 'resource': the owning resource's own NPCs also pass its locked or private doors. 'always': any network NPC passes locked or private doors. 'never': no NPC. nil restores 'public'. Only the client that simulates an NPC can ask for it; the service checks that client's authority epoch, the NPC's life, bucket and 6 m reach from the server's own NPC state. A lock lifted for an NPC returns when the door shuts. Invalid values (invalid_npc_passage) change nothing.

This is an asynchronous server-side resource export. Depend on open77_doors >=1.2.0 and call Open77.exports.call, not a Lua table named open77_doors. The call returns a Promise (or nil, error if it cannot be queued); awaiting it yields: true or false, reason. Requires client and server builds that include this open77_doors version; players on an older client never send the matching requests.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |
| `bucket` | `integer` | required | — |
| `mode` | `"public" | "resource" | "always" | "never" | nil` | required | — |

Returns: `Promise | nil`, `queue error, if any`

```lua
CreateThread(function()
    local pending, error = Open77.exports.call("open77_doors", "setNpcPassage", id, bucket, mode)
    if not pending then print(error); return end
    local result, reason = pending:await()
    -- Handle the result before continuing; queued is not accepted.
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Documented in [resources/system/open77_doors/server/main.lua](../../01%20Documentation/doors.md).

## setOpen

```lua
Open77.exports.call("open77_doors", "setOpen", id, bucket, open)
```

`SERVER`: Available in server resources only.

Set a non-elevator door's target state.

Requires ownership. Disable automatic mode and autoClose if a scripted open state must persist. This cannot override elevator safety, a lock or a seal.

This is an asynchronous server-side resource export. Depend on open77_doors >=1.0.0 and call Open77.exports.call, not a Lua table named open77_doors. The call returns a Promise (or nil, error if it cannot be queued); awaiting it yields: true or false, reason. Available with client and server release 2.31.13+op77.58 on the CDN; older clients do not contain its native projection bridge.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |
| `bucket` | `integer` | required | — |
| `open` | `boolean` | required | — |

Returns: `Promise | nil`, `queue error, if any`

```lua
CreateThread(function()
    local pending, error = Open77.exports.call("open77_doors", "setOpen", id, bucket, open)
    if not pending then print(error); return end
    local result, reason = pending:await()
    -- Handle the result before continuing; queued is not accepted.
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Documented in [resources/system/open77_doors/server/main.lua](../../01%20Documentation/doors.md).

## setSealed

```lua
Open77.exports.call("open77_doors", "setSealed", id, bucket, sealed)
```

`SERVER`: Available in server resources only.

Seal or unseal a shared door.

Requires ownership and a strict boolean. Sealing closes the door and prevents all player requests.

This is an asynchronous server-side resource export. Depend on open77_doors >=1.0.0 and call Open77.exports.call, not a Lua table named open77_doors. The call returns a Promise (or nil, error if it cannot be queued); awaiting it yields: true or false, reason. Available with client and server release 2.31.13+op77.58 on the CDN; older clients do not contain its native projection bridge.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |
| `bucket` | `integer` | required | — |
| `sealed` | `boolean` | required | — |

Returns: `Promise | nil`, `queue error, if any`

```lua
CreateThread(function()
    local pending, error = Open77.exports.call("open77_doors", "setSealed", id, bucket, sealed)
    if not pending then print(error); return end
    local result, reason = pending:await()
    -- Handle the result before continuing; queued is not accepted.
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Documented in [resources/system/open77_doors/server/main.lua](../../01%20Documentation/doors.md).
