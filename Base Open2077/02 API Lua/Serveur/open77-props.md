# Open77.props (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-props.md) · [Page web](https://open2077.net/docs/api/server/open77-props)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

15 functions.

## all

```lua
Open77.props.all([bucket])
```

`NETWORK`: Uses the network backend.

Every authoritative prop, optionally filtered to one routing bucket.

Requires `world.props`. Returns the whole registry, whichever resource owns each prop.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | optional | — |

Returns: `array of prop snapshots`

```lua
-- Count the props in the event instance
print(#Open77.props.all(EVENT_BUCKET), "props in the event")
```

Registered in `LuaResourceRuntime.cs` (line 9722) as `LuaResourceRuntime.Bootstrap`.

## attach

```lua
Open77.props.attach(propId, binding, [expectedRevision])
```

`NETWORK`: Uses the network backend.

Set a synchronized prop binding.

Binding fields: parentType ('player' or 'vehicle'), parentId (canonical network ID, not an engine handle), bone (named slot, empty for root), offset {x,y,z} in metres (each ±20), rotation {x,y,z} in degrees (each ±360, local Z-Y-X composition). Unknown fields and non-finite numbers reject. Missing streamed parents are hidden, then rebound when available. See attachments.md. Requires world.props and ownership of the prop. Parent must be alive/valid in the same bucket. Up to 32 bindings per parent. Optional expectedRevision enables compare-and-swap; stale updates reject. Idempotent identical attachment does not create a new revision. Experimental contact applies only to native item:Items.* props: omitted selects automatic mouth contact, false disables it, or {x,y,z} specifies item-local metres (finite, each within ±2; missing axes zero). Invalid values return invalid_item_contact; contact on an ordinary mesh returns contact_requires_item. Native items require a player parent, WeaponRight, zero offset/rotation and unit scale. Contact affects the arm during supported consumption profiles, not the grip offset. Requires matching runtime/assets; visual validation is incomplete. See [item size and mouth contact](../../01%20Documentation/rp-animations.md#item-size-and-mouth-contact). Automatic container contact uses the main mesh end along the authored item-local up direction (the can rim), keeping that end while tilting; smoking uses the end nearest the mouth. In first person, held items stay visible at the lower right between sips and contact follows the camera. Disabling mouth fitting keeps the first-person holding pose.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `propId` | `integer` | required | — |
| `binding` | `table` | required | — |
| `expectedRevision` | `integer` | optional | — |

Returns: `true on success; nil, reason on rejection`

```lua
-- The courier picks the crate up: it follows their right hand
Open77.props.attach(crateId, { parentType = "player", parentId = courierId, bone = "RightHand" })
```

[Synchronized attachments guide](../../01%20Documentation/attachments.md#server-api)

Registered in `LuaResourceRuntime.cs` (line 9647) as `LuaResourceRuntime.Bootstrap`.

## attachedTo

```lua
Open77.props.attachedTo(parentType, parentId)
```

`NETWORK`: Uses the network backend.

List replicated props attached to one parent.

Requires world.props. Uses a canonical player/vehicle ID and returns copied prop records, including bindings and revisions. Inspection can include other resources' props, but mutation still checks ownership.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `parentType` | `string` | required | — |
| `parentId` | `integer` | required | — |

Returns: `prop[] or nil, reason`

```lua
-- Every replicated prop currently attached to the player
for _, prop in ipairs(Open77.props.attachedTo("player", playerId) or {}) do
    print(prop.id, prop.model)
end
```

[Synchronized attachments guide](../../01%20Documentation/attachments.md#server-api)

Registered in `LuaResourceRuntime.cs` (line 9622) as `LuaResourceRuntime.Bootstrap`.

## catalog

```lua
Open77.props.catalog()
```

`NETWORK`: Uses the network backend.

Always an empty table on the server.

The dedicated server keeps no curated model catalogue -- the aliases live on the client, next to the records they resolve. It answers an empty list rather than `nil` so the shipped `prop.catalog` console command stays honest instead of failing on a nil call.

Returns: `empty table`

```lua
-- Always empty on the server; the alias list lives on the client
print(#Open77.props.catalog()) -- 0
```

Registered in `LuaResourceRuntime.cs` (line 9728) as `LuaResourceRuntime.Bootstrap`.

## clear

```lua
Open77.props.clear()
```

`NETWORK`: Uses the network backend.

Removes every prop this resource owns and returns how many.

Requires `world.props`. Called for you when the resource stops, so a reloading gamemode never leaves orphaned scenery behind. It never touches another resource's props.

Returns: `number removed`

```lua
-- The event is over: remove every prop this resource placed
AddEventHandler("event:ended", function()
    print("removed", Open77.props.clear(), "props")
end)
```

Registered in `LuaResourceRuntime.cs` (line 9723) as `LuaResourceRuntime.Bootstrap`.

## create

```lua
Open77.props.create(def)
```

`NETWORK`: Uses the network backend.

Creates an authoritative, replicated world prop.

Requires `world.props`. This is the **server** meaning of the name: the prop is owned by the creating resource, streamed per player rather than broadcast to a bucket, and visible to everyone in range. The identically named *client* function makes a local, unreplicated decoration instead. Definitions accept `model`, `position`, `yaw`, `scale` (a number or `{ x, y, z }`), `appearance`, `bucket`, `physics` (`static`, `kinematic`, `dynamic`, `none`), `collision`, `visible`, `kind` (`prop`, `light`, `effect`), `light`, `streamingRadius` (10 to 2000, default 120), `streamingHysteresis` (default 30) and `ttlMs` (0 for no expiry, seven days maximum). The registry holds 8,192 props, of which one resource may own 2,048. Ids are decimal strings; do not pass them through `tonumber`. Stable 156 accepts raw .mesh and .ent depot paths without an alias whitelist. Every observer needs the assets and dependencies mounted. Mesh loading is asynchronous; rendered stays false until ready. Raw .ent requires kind="prop" and retains its authored graph/appearance. Visual scale does not rescale collision. Attached props are non-blocking; collision=false keeps raw decorations non-solid after detach. Arbitrary gameplay/physics template families remain experimental. Existing alias and registered-model routing is preserved. See [model limits](../../01%20Documentation/props.md#model-and-collision-limits).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `def` | `table` | required | — |

Returns: `prop id (decimal string), or nil`, `reason: permission_denied:world.props, quota_exceeded, invalid_position, invalid_model, unknown_alias, record_provisioning_failed, entity_spawn_failed, world_unavailable`

```lua
-- A replicated crate at the drop point, gone after an hour
local crate, reason = Open77.props.create({
    model = "crate.small",
    position = { x = -1442.2, y = 127.4, z = 18.0 },
    yaw = 45.0,
    ttlMs = 60 * 60 * 1000,
})
if not crate then print("no crate: " .. reason) end
```

Registered in `LuaResourceRuntime.cs` (line 9673) as `LuaResourceRuntime.Bootstrap`.

## detach

```lua
Open77.props.detach(propId, [expectedRevision])
```

`NETWORK`: Uses the network backend.

Release a synchronized prop binding.

Requires world.props and prop ownership. Optional expectedRevision prevents detaching a newer binding. Leaves the prop at the server's last coarse parent anchor, which is not an exact client-side animated bone transform. Already detached is idempotent. Death, disconnect, parent removal and bucket changes also detach automatically.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `propId` | `integer` | required | — |
| `expectedRevision` | `integer` | optional | — |

Returns: `true on success; nil, reason on rejection`

```lua
-- The courier put the crate down: it stays where the body last carried it
Open77.props.detach(crateId)
```

[Synchronized attachments guide](../../01%20Documentation/attachments.md#server-api)

Registered in `LuaResourceRuntime.cs` (line 9666) as `LuaResourceRuntime.Bootstrap`.

## get

```lua
Open77.props.get(id)
```

`NETWORK`: Uses the network backend.

Reads one prop's canonical snapshot.

Requires `world.props`. Reads are not restricted to the owner; the snapshot's `resource` field is provenance and cleanup scope, not an authorization boundary.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |

Returns: `prop snapshot, or nil`

```lua
-- Is the crate still there before paying for its pickup?
local crate = Open77.props.get(crateId)
if not crate then return Open77.chat.send(source, "Somebody already took it.") end
```

Registered in `LuaResourceRuntime.cs` (line 9721) as `LuaResourceRuntime.Bootstrap`.

## getAttachment

```lua
Open77.props.getAttachment(propId)
```

`NETWORK`: Uses the network backend.

Read a prop's current replicated binding.

Binding fields: parentType ('player' or 'vehicle'), parentId (canonical network ID, not an engine handle), bone (named slot, empty for root), offset {x,y,z} in metres (each ±20), rotation {x,y,z} in degrees (each ±360, local Z-Y-X composition). Unknown fields and non-finite numbers reject. Missing streamed parents are hidden, then rebound when available. See attachments.md. Requires world.props. Returns a copy, not a mutable reference. Nil can mean detached or absent; inspect the second result for a capability/validation failure.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `propId` | `integer` | required | — |

Returns: `binding table or nil; nil, reason on rejection`

```lua
local binding, err = Open77.props.getAttachment(propId)
if binding then print(binding.parentType, binding.parentId, binding.bone) end
```

[Synchronized attachments guide](../../01%20Documentation/attachments.md#server-api)

Registered in `LuaResourceRuntime.cs` (line 9667) as `LuaResourceRuntime.Bootstrap`.

## isAttached

```lua
Open77.props.isAttached(propId)
```

`NETWORK`: Uses the network backend.

Check whether a prop has a binding.

Requires world.props. Returns false when no binding exists, and nil plus reason for a rejected query. This tests authoritative state, not whether a particular client has streamed or rendered its parent.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `propId` | `integer` | required | — |

Returns: `boolean or nil, reason`

```lua
-- Is the crate still being carried?
if Open77.props.isAttached(crateId) then print("someone is carrying the crate") end
```

[Synchronized attachments guide](../../01%20Documentation/attachments.md#server-api)

Registered in `LuaResourceRuntime.cs` (line 9668) as `LuaResourceRuntime.Bootstrap`.

## remove

```lua
Open77.props.remove(id)
```

`NETWORK`: Uses the network backend.

Removes a prop this resource owns.

Requires `world.props`. Another resource's prop answers `owned_by_another_resource`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |

Returns: `true, or false`, `reason`

```lua
-- The player picked up the crate
RegisterNetEvent("crates:pickup", function(crateId)
    if Open77.props.remove(crateId) then giveReward(source) end
end)
```

Registered in `LuaResourceRuntime.cs` (line 9720) as `LuaResourceRuntime.Bootstrap`.

## setAttachmentTransform

```lua
Open77.props.setAttachmentTransform(propId, [offset], [rotation], [expectedRevision])
```

`NETWORK`: Uses the network backend.

Change a binding's local offset and rotation.

Binding fields: parentType ('player' or 'vehicle'), parentId (canonical network ID, not an engine handle), bone (named slot, empty for root), offset {x,y,z} in metres (each ±20), rotation {x,y,z} in degrees (each ±360, local Z-Y-X composition). Unknown fields and non-finite numbers reject. Missing streamed parents are hidden, then rebound when available. See attachments.md. Requires world.props and prop ownership. Omitted vectors retain their previous values. The current revision is used if expectedRevision is omitted; stale explicit revisions reject. A detached prop rejects with not_attached.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `propId` | `integer` | required | — |
| `offset` | `table` | optional | — |
| `rotation` | `table` | optional | — |
| `expectedRevision` | `integer` | optional | — |

Returns: `true on success; nil, reason`

```lua
-- Nudge the carried crate ten centimetres up and turn it sideways
Open77.props.setAttachmentTransform(crateId, { x = 0, y = 0, z = 0.1 }, { x = 0, y = 0, z = 90 })
```

[Synchronized attachments guide](../../01%20Documentation/attachments.md#server-api)

Registered in `LuaResourceRuntime.cs` (line 9623) as `LuaResourceRuntime.Bootstrap`.

## setBucket

```lua
Open77.props.setBucket(id, bucket)
```

`NETWORK`: Uses the network backend.

Moves a prop to another routing bucket.

Requires `world.props`. Changes which players stream it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |
| `bucket` | `integer` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Move the stage props into the concert instance
for _, propId in ipairs(stageProps) do Open77.props.setBucket(propId, CONCERT_BUCKET) end
```

Registered in `LuaResourceRuntime.cs` (line 9719) as `LuaResourceRuntime.Bootstrap`.

## setTransform

```lua
Open77.props.setTransform(id, def)
```

`NETWORK`: Uses the network backend.

Moves and optionally rescales a prop this resource owns.

Requires `world.props`. Takes `position`, `yaw` and `scale` (a number or `{ x, y, z }`). Unlike the client prop API, scale belongs here rather than in `update`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |
| `def` | `table { position, yaw?, scale? }` | required | — |

Returns: `true, or false`, `reason`

```lua
-- The crane lifts the container two metres
local snapshot = Open77.props.get(containerId)
Open77.props.setTransform(containerId, { position = { x = snapshot.x, y = snapshot.y, z = snapshot.z + 2.0 } })
```

Registered in `LuaResourceRuntime.cs` (line 9706) as `LuaResourceRuntime.Bootstrap`.

## update

```lua
Open77.props.update(id, patch)
```

`NETWORK`: Uses the network backend.

Patches a prop this resource owns.

Requires `world.props`. Sparse, and it deliberately does **not** accept `model` or `kind`: changing either is a remove and a create, because the render proxy is built from them. A resource may read every prop but mutate only its own.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |
| `patch` | `table { appearance?, physics?, collision?, visible?, bucket?, light?, streamingRadius?, streamingHysteresis?, ttlMs? }` | required | — |

Returns: `true, or false`, `reason: not_found, owned_by_another_resource, ...`

```lua
-- Hide the barrier while the road is open, keep it registered
Open77.props.update(barrierId, { visible = false, collision = false })
```

Registered in `LuaResourceRuntime.cs` (line 9696) as `LuaResourceRuntime.Bootstrap`.
