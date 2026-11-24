# Open77.props (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-props.md) · [Page web](https://open2077.net/docs/api/client/open77-props)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

14 functions.

## attach

```lua
Open77.props.attach(propId, binding)
```

`GAME`: Requires a live game instance.

Attach a resource-owned local prop to a rendered bone or vehicle.

Binding fields: parentType ('player' or 'vehicle'), parentId (canonical network ID, not an engine handle), bone (named slot, empty for root), offset {x,y,z} in metres (each ±20), rotation {x,y,z} in degrees (each ±360, local Z-Y-X composition). Unknown fields and non-finite numbers reject. Missing streamed parents are hidden, then rebound when available. See attachments.md. Requires world.props. This changes only this client's prop, not server authority. Physics uses a visual-only attachment host; arbitrary mesh paths without that host are refused.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `propId` | `integer` | required | — |
| `binding` | `table` | required | — |

Returns: `boolean success, reason on failure`

```lua
-- A local placement ghost held in the player's hand
Open77.props.attach(localPropId, { parentType = "player", parentId = Open77.players.localId(), bone = "RightHand" })
```

[Synchronized attachments guide](../../01%20Documentation/attachments.md#client-api-and-diagnostics)

Registered in `ResourceHost.cpp` (line 25501) as `LuaPropAttach`.

## bones

```lua
Open77.props.bones(parentType, parentId)
```

`GAME`: Requires a live game instance.

List available named attachment slots on a streamed parent.

Requires world.props. Returns sorted names from the current native slot components, not all skeleton bones. Parent IDs are canonical player/vehicle IDs. Empty array means unavailable or no slots; it does not establish that a different body/model supports a requested slot.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `parentType` | `string` | required | — |
| `parentId` | `integer` | required | — |

Returns: `string[] or nil, reason`

```lua
local slots, err = Open77.props.bones('player', playerId)
if slots then for _, slot in ipairs(slots) do print(slot) end end
```

[Synchronized attachments guide](../../01%20Documentation/attachments.md#client-api-and-diagnostics)

Registered in `ResourceHost.cpp` (line 25503) as `LuaPropAttachmentBones`.

## clear

```lua
Open77.props.clear()
```

`GAME`: Requires a live game instance.

Removes every local prop this resource created.

Called automatically when the resource stops or reloads, so a reloading gamemode never leaves orphaned decorations behind. It never touches another resource's props, nor server-owned ones.

Returns: `boolean`

```lua
-- Leaving build mode removes every placement ghost
AddEventHandler("build:exit", function()
    Open77.props.clear()
end)
```

Registered in `ResourceHost.cpp` (line 25507) as `LuaPropClear`.

## create

```lua
Open77.props.create(options)
```

`GAME`: Requires a live game instance.

Creates a local, decorative prop owned by this resource.

Requires `world.props`. This is the **client** meaning of the name: the prop is never replicated and never visible to anyone else -- a placement ghost, a build cursor, a preview being dragged. It is charged against this resource's quota and released when the resource stops. The identically-named *server* function mints an authoritative, replicated prop instead; the two `Open77` tables are different objects on different machines. Stable 156 accepts raw .mesh and .ent depot paths without an alias whitelist. Every observer needs the assets and dependencies mounted. Mesh loading is asynchronous; rendered stays false until ready. Raw .ent requires kind="prop" and retains its authored graph/appearance. Visual scale does not rescale collision. Attached props are non-blocking; collision=false keeps raw decorations non-solid after detach. Arbitrary gameplay/physics template families remain experimental. Existing alias and registered-model routing is preserved. See [model limits](../../01%20Documentation/props.md#model-and-collision-limits).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table { model, position, yaw?, scale?, appearance?, kind?, physics?, collision?, visible?, light? }` | required | — |

Returns: `decimal-string prop id, or nil`, `reason`

```lua
local ghost = assert(Open77.props.create({
  model = "crate.small",
  position = { x = -1442.2, y = 127.4, z = 18.0 },
  collision = false
}))
```

Registered in `ResourceHost.cpp` (line 25500) as `LuaPropCreate`.

## detach

```lua
Open77.props.detach(propId)
```

`GAME`: Requires a live game instance.

Detach a resource-owned local prop.

Requires world.props. Restores an independent host at the last locally rendered pose. Does not detach a server-owned network prop. Calling the server API is required for a replicated release.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `propId` | `integer` | required | — |

Returns: `boolean success, reason on failure`

```lua
-- Drop the local ghost prop where it is
Open77.props.detach(localPropId)
```

[Synchronized attachments guide](../../01%20Documentation/attachments.md#client-api-and-diagnostics)

Registered in `ResourceHost.cpp` (line 25502) as `LuaPropDetach`.

## list

```lua
Open77.props.list()
```

`GAME`: Requires a live game instance.

Local props this resource owns, with what the renderer did with each.

Each entry carries the model, kind, physics, transform, scale, the back-end that drew it, and whether it rendered or failed. Scoped to the caller: it deliberately excludes server-owned props, which belong to the session rather than to any resource, so asking a resource-scoped question about them would return an empty table and read as "no props" rather than "wrong question". Use the `open77_props` resource's `all` export for those.

Returns: `array of prop snapshots`

```lua
-- Did every local prop actually render?
for _, prop in ipairs(Open77.props.list()) do
    if not prop.rendered then print("prop failed to render:", prop.model) end
end
```

Registered in `ResourceHost.cpp` (line 25508) as `LuaPropList`.

## nearest

```lua
Open77.props.nearest(position, [radius])
```

`GAME`: Requires a live game instance.

Finds the nearest local prop owned by this resource.

Requires `world.props`. Searches only the calling resource's local prop set, using a 10 metre default radius. A non-positive or non-finite radius is rejected; use `list` when an unbounded set is intended.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `position` | `table { x, y, z }` | required | — |
| `radius` | `number` | optional | `10` |

Returns: `prop snapshot and distance, or nil`, `reason on validation failure`

```lua
-- Press E on the nearest of this resource's ghosts to confirm the placement
RegisterKeyMapping("build_confirm", "Confirm placement", "E", function()
    local prop, distance = Open77.props.nearest(Open77.character.state().position, 3)
    if prop then TriggerServerEvent("build:confirm", prop.model, prop.position) end
end)
```

Registered in `ResourceHost.cpp` (line 25510) as `LuaPropNearest`.

## project

```lua
Open77.props.project(serverId, options)
```

`GAME`: Requires a live game instance.

Upserts a server-owned prop into the world, keyed by the server id.

What the bundled `open77_props` projection resource drives; not the way a gamemode places objects. Because the native layer is keyed by the server's own id, a prop that only moved is transformed in place rather than destroyed and rebuilt, which is what stops it visibly popping. Projected props are owned by the session, survive a resource reload, and are not charged against any resource's quota.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `serverId` | `decimal string` | required | — |
| `options` | `table, same fields as create` | required | — |

Returns: `boolean`, `reason`

```lua
-- The projection resource shows a server prop under the server's own id
RegisterNetEvent("props:sync", function(serverId, definition)
    Open77.props.project(serverId, definition)
end)
```

Registered in `ResourceHost.cpp` (line 25511) as `LuaPropProject`.

## projected

```lua
Open77.props.projected(serverPropId)
```

`GAME`: Requires a live game instance.

Inspect a replicated prop by its server ID.

Requires world.props. Returns the current local projection snapshot, or nil if absent. Includes backend, rendered, failed and attachmentStatus. Native item projection reports rendered only after native attachment and visual spawning. hidden_first_person is explicitly not rendered. Read-only; does not create or take ownership of a server prop.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `serverPropId` | `integer|string` | required | — |

Returns: `table snapshot, or nil`, `reason on failure`

```lua
local prop = Open77.props.projected(serverPropId)
if prop then print(prop.backend, prop.attachmentStatus, prop.rendered) end
```

Registered in `ResourceHost.cpp` (line 25509) as `LuaPropProjected`.

## remove

```lua
Open77.props.remove(id)
```

`GAME`: Requires a live game instance.

Removes a local prop this resource owns.

Removing a prop that is already gone succeeds: the caller asked for it not to be there, and it is not there. Another resource's prop answers `owned_by_another_resource`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |

Returns: `boolean`, `reason`

```lua
-- The player cancelled the placement: remove the ghost
Open77.props.remove(ghostId)
```

Registered in `ResourceHost.cpp` (line 25506) as `LuaPropRemove`.

## setTransform

```lua
Open77.props.setTransform(id, transform)
```

`GAME`: Requires a live game instance.

Moves a local prop to a position and yaw.

The one mutation the engine accepts on a live entity, so it never respawns and never pops. Scale is deliberately not part of it: scale is read when the render proxy is built, so it goes through `update` and costs a respawn.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |
| `transform` | `table { position, yaw? }` | required | — |

Returns: `boolean`, `reason`

```lua
-- Drag the placement ghost to wherever the reticle points
SetTick(function()
    local ray = Open77.camera.aimRay(20)
    if ray and ray.hit then Open77.props.setTransform(ghostId, { position = ray.position }) end
end)
```

Registered in `ResourceHost.cpp` (line 25505) as `LuaPropSetTransform`.

## unproject

```lua
Open77.props.unproject(serverId)
```

`GAME`: Requires a live game instance.

Stops projecting one server-owned prop.

Succeeds when the prop is already absent: the server is saying stop showing something, and it is not being shown.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `serverId` | `decimal string` | required | — |

Returns: `boolean`, `reason`

```lua
-- The server removed the prop: stop showing it
RegisterNetEvent("props:removed", function(serverId)
    Open77.props.unproject(serverId)
end)
```

Registered in `ResourceHost.cpp` (line 25512) as `LuaPropUnproject`.

## unprojectAll

```lua
Open77.props.unprojectAll()
```

`GAME`: Requires a live game instance.

Stops projecting every server-owned prop.

Used on disconnect and when the projection resource restarts, so a stale world never survives into a new session.

Returns: `boolean`

```lua
-- Leaving the server: nothing stale survives into the next session
AddEventHandler("session:disconnected", function()
    Open77.props.unprojectAll()
end)
```

Registered in `ResourceHost.cpp` (line 25513) as `LuaPropUnprojectAll`.

## update

```lua
Open77.props.update(id, patch)
```

`GAME`: Requires a live game instance.

Patches a local prop this resource owns.

Only the fields present in the patch are taken. Changing anything the render proxy reads when it is built -- model, scale, kind, physics, collision, visibility or the light parameters -- respawns the underlying entity rather than mutating it, because the engine offers no in-place path for those. Use `setTransform` to move a prop without that cost.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |
| `patch` | `table, same fields as create` | required | — |

Returns: `boolean`, `reason`

```lua
-- The player chose a bigger crate: scale respawns the ghost
Open77.props.update(ghostId, { scale = 1.5 })
```

Registered in `ResourceHost.cpp` (line 25504) as `LuaPropUpdate`.
