# Open77.gizmos (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-gizmos.md) · [Page web](https://open2077.net/docs/api/client/open77-gizmos)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

14 functions.

## activate

```lua
Open77.gizmos.activate(handle)
```

`GAME`: Requires a live game instance.

Acquires native input and begins editing an owned gizmo.

Client-only; requires gizmos.edit. Rechecks the streamed target and incarnation, captures its current transform as the cancellation baseline and resets history. Refuses another input owner, a background game, dead player, vehicle/workspot or unsupported target. The tool must be visible. Re-activating an already active handle is idempotent. Emits activated; focus loss, target loss, timeout and owner stop cancel it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `handle` | `string` | required | — |

Returns: `boolean success`, `string reason on failure`

[Entity gizmos guide](../../01%20Documentation/gizmos.md)

Registered in `ResourceHost.cpp` (line 25294) as `LuaGizmoActivate`.

## active

```lua
Open77.gizmos.active()
```

`GAME`: Requires a live game instance.

Reads this resource's active gizmo, if any.

Client-only; requires gizmos.edit. Returns the active snapshot only when the calling resource generation owns it. Returns nil without an error when no tool is active or another resource owns the input lease. Only one gizmo can capture native cursor/keyboard input across all resources.

Returns: `table active snapshot or nil`, `string reason on failure`

[Entity gizmos guide](../../01%20Documentation/gizmos.md)

Registered in `ResourceHost.cpp` (line 25292) as `LuaGizmoActive`.

## cancel

```lua
Open77.gizmos.cancel(handle)
```

`GAME`: Requires a live game instance.

Restores the activation baseline and ends editing.

Client-only; requires gizmos.edit. Attempts to restore the original native transform, ends an active drag, releases only the editor's own holds and emits cancel/deactivated. Own-player/vehicle cursor input returns immediately but movement remains briefly fenced while restoration is observed; a newer normal travel/server placement wins. Those asynchronous adapters later emit settled with nativeTransform or error. A missing or replaced target cannot be recreated; cleanup still runs. The owned handle remains until destroy or resource teardown. Escape during a drag cancels that drag first; a second Escape cancels the session. See gizmos.md for asynchronous restoration failures.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `handle` | `string` | required | — |

Returns: `boolean success`, `string reason on failure`

[Entity gizmos guide](../../01%20Documentation/gizmos.md)

Registered in `ResourceHost.cpp` (line 25297) as `LuaGizmoCancel`.

## capabilities

```lua
Open77.gizmos.capabilities([target])
```

`GAME`: Requires a live game instance.

Queries which transforms a target adapter supports.

Client-only; requires gizmos.edit. Omit target for the virtual transform capabilities, or supply a typed target table with kind and id. Returns per-axis translate, rotate and scale booleans plus preview, commit and networked. The target must be attached. Native characters and props are yaw-only and native scaling is currently unsupported; virtual transforms support all axes. Never infer server authority from this query: commit=false on a network target means acceptance is a proposal, not a canonical mutation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `table` | optional | — |

Returns: `table capabilities or nil`, `string reason on failure`

[Entity gizmos guide](../../01%20Documentation/gizmos.md)

Registered in `ResourceHost.cpp` (line 25293) as `LuaGizmoCapabilities`.

## commit

```lua
Open77.gizmos.commit(handle)
```

`GAME`: Requires a live game instance.

Accepts the current transform and ends editing.

Client-only; requires gizmos.edit. Emits commit with the accepted transform and returns cursor input. Virtual targets retain data; supported local entities retain their pose. A network target requests restoration and emits a proposal only: the server must validate an edit ticket, ACL, identity, bucket, revision and bounds before applying its own authoritative mutation API. Local puppet placement and own-player/vehicle restoration report their asynchronous result through settled (with nativeTransform) or error. commit is not that completion acknowledgement, and settled never acknowledges server acceptance. This call neither sends a commit to the server nor grants authority. See gizmos.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `handle` | `string` | required | — |

Returns: `boolean success`, `string reason on failure`

[Entity gizmos guide](../../01%20Documentation/gizmos.md)

Registered in `ResourceHost.cpp` (line 25296) as `LuaGizmoCommit`.

## create

```lua
Open77.gizmos.create(options)
```

`GAME`: Requires a live game instance.

Creates a resource-owned native 3D gizmo.

Client-only; requires gizmos.edit. Creates an inactive handle for a typed entity target or a virtual transform. It does not capture input until activate succeeds. Options include mode, space, axes, pivotOffset, snapping, bounds, size, timeoutMs and local preview. IDs and events are scoped to the exact resource VM and generation. Quotas are eight handles per owner and 64 globally. See gizmos.md for the full option schema, native capabilities, validation status and server-authority contract.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table` | required | — |

Returns: `string handle or nil`, `string reason on failure`

[Entity gizmos guide](../../01%20Documentation/gizmos.md)

Registered in `ResourceHost.cpp` (line 25289) as `LuaGizmoCreate`.

## deactivate

```lua
Open77.gizmos.deactivate(handle)
```

`GAME`: Requires a live game instance.

Cancels editing and releases the gizmo's input lease.

Client-only; requires gizmos.edit. This is an alias of cancel, not a hidden acceptance operation. Restores the activation baseline when the target still exists, releases owned native input/physics holds and emits cancel/deactivated. The handle remains available for querying or reactivation. Use commit to accept a result or destroy to remove the handle.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `handle` | `string` | required | — |

Returns: `boolean success`, `string reason on failure`

[Entity gizmos guide](../../01%20Documentation/gizmos.md)

Registered in `ResourceHost.cpp` (line 25295) as `LuaGizmoDeactivate`.

## destroy

```lua
Open77.gizmos.destroy(handle)
```

`GAME`: Requires a live game instance.

Removes an owned gizmo, cancelling any active preview.

Client-only; requires gizmos.edit. Cancels and releases an active tool before removing its handle and emitting destroyed. It never deletes the target entity. A previously committed local transform is not rolled back merely by destroying its inactive handle. Resource stop/reload performs equivalent owned-handle cleanup automatically.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `handle` | `string` | required | — |

Returns: `boolean success`, `string reason on failure`

[Entity gizmos guide](../../01%20Documentation/gizmos.md)

Registered in `ResourceHost.cpp` (line 25298) as `LuaGizmoDestroy`.

## get

```lua
Open77.gizmos.get(handle)
```

`GAME`: Requires a live game instance.

Reads one owned gizmo's complete snapshot.

Client-only; requires gizmos.edit. Returns the same typed snapshot shape delivered to open77:gizmo events: current and initial transforms, target, capabilities, phase, active/dragging flags, handle, hover, options, revision and history counts. nativeTransform is false until asynchronous native completion, then contains the observed pose without overwriting the proposed transform. Target IDs are decimal strings. It refuses foreign or stale handles; no native pointer or VM identity is exposed. See gizmos.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `handle` | `string` | required | — |

Returns: `table snapshot or nil`, `string reason on failure`

[Entity gizmos guide](../../01%20Documentation/gizmos.md)

Registered in `ResourceHost.cpp` (line 25290) as `LuaGizmoGet`.

## list

```lua
Open77.gizmos.list()
```

`GAME`: Requires a live game instance.

Lists this resource generation's gizmo snapshots.

Client-only; requires gizmos.edit. Returns an array of owned snapshots, including idle and terminal handles until destroyed. It never lists another resource or VM's tools. Stop/reload destroys all owned handles and restores active previews. Use this query to reconcile state after coalesced events; do not assume an event for every rendered frame.

Returns: `table snapshot array or nil`, `string reason on failure`

[Entity gizmos guide](../../01%20Documentation/gizmos.md)

Registered in `ResourceHost.cpp` (line 25291) as `LuaGizmoList`.

## redo

```lua
Open77.gizmos.redo(handle)
```

`GAME`: Requires a live game instance.

Reapplies an undone gizmo edit.

Client-only; requires gizmos.edit. Restores the next transform in the active tool's history and previews it through the target adapter. Refuses an active drag, inactive handle or empty redo history. Any new accepted edit truncates the redo branch. Emits redo and updates the snapshot's history counts.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `handle` | `string` | required | — |

Returns: `boolean success`, `string reason on failure`

[Entity gizmos guide](../../01%20Documentation/gizmos.md)

Registered in `ResourceHost.cpp` (line 25300) as `LuaGizmoRedo`.

## setTransform

```lua
Open77.gizmos.setTransform(handle, transform)
```

`GAME`: Requires a live game instance.

Applies a numeric transform edit to the active gizmo.

Client-only; requires gizmos.edit. Applies a sparse transform table containing position, rotation and/or scale, retaining unspecified fields. Vectors require x/y/z; rotation requires normalized x/y/z/w. Rejects non-finite values, unsupported target axes, bounds violations, inactive handles and active drags. A changed value is previewed locally, added to history and emitted as change. It is not an authoritative server mutation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `handle` | `string` | required | — |
| `transform` | `table` | required | — |

Returns: `boolean success`, `string reason on failure`

[Entity gizmos guide](../../01%20Documentation/gizmos.md)

Registered in `ResourceHost.cpp` (line 25302) as `LuaGizmoSetTransform`.

## undo

```lua
Open77.gizmos.undo(handle)
```

`GAME`: Requires a live game instance.

Undoes the last completed gizmo edit.

Client-only; requires gizmos.edit. Restores the preceding transform in the active tool's bounded history and previews it through the target adapter. A completed drag is one history entry; numeric setTransform edits also create entries. Refuses an active drag, inactive handle or empty undo history. Emits undo. History stores at most 32 undo steps and resets on activation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `handle` | `string` | required | — |

Returns: `boolean success`, `string reason on failure`

[Entity gizmos guide](../../01%20Documentation/gizmos.md)

Registered in `ResourceHost.cpp` (line 25299) as `LuaGizmoUndo`.

## update

```lua
Open77.gizmos.update(handle, patch)
```

`GAME`: Requires a live game instance.

Reconfigures an owned gizmo without replacing its target.

Client-only; requires gizmos.edit. Applies a validated sparse patch of mode, space, axes, pivotOffset, snap, position/scale bounds, visible and size. Target, transform, preview and timeout are creation-time settings and cannot be changed here. Refuses changes during a drag, unsupported modes and hiding the active gizmo. Emits configured. Axis restrictions control pointer handles; target capabilities and bounds also constrain numeric edits.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `handle` | `string` | required | — |
| `patch` | `table` | required | — |

Returns: `boolean success`, `string reason on failure`

[Entity gizmos guide](../../01%20Documentation/gizmos.md)

Registered in `ResourceHost.cpp` (line 25301) as `LuaGizmoUpdate`.
