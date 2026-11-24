# Open77.motion (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-motion.md) · [Page web](https://open2077.net/docs/api/server/open77-motion)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

4 functions.

## cancel

```lua
Open77.motion.cancel(playerId, id)
```

`NETWORK`: Uses the network backend.

Cancel this resource's authoritative motion lease.

Requires players.motion.control. Requires the exact player and motion ID owned by the calling resource. Ends network ownership and requests native cleanup; collision/braking and get-up remain native, so cancellation need not instantly stop velocity or animation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `id` | `string` | required | — |

Returns: `{ok=true,id=string}`, `nil, reason on rejection`

```lua
local result, err = Open77.motion.cancel(playerId, motionId)
if not result then print(err) end
```

[Cyberware authority & motion guide](../../01%20Documentation/cyberware.md)

Registered in `LuaResourceRuntime.cs` (line 8007) as `LuaResourceRuntime.Bootstrap`.

## current

```lua
Open77.motion.current(playerId)
```

`NETWORK`: Uses the network backend.

Read an authoritative living-motion lease.

Requires players.motion.read. Returns the authoritative lease {id,player,incarnation,bucket,directionX,directionY,distance,createdAt,expiresAt,phase,reason,kind,push,lift}, or nil. Launch leases have kind="launch" and bounded push/lift values; ordinary knockdown retains its existing distance contract. The optional lifeRevision identifies the authoritative life generation when the server uses a generic body incarnation. Times are server-monotonic milliseconds. pending lasts until the controlling client acknowledges; active is an acknowledged motion lease, not proof of observer rendering. Use the exact id with cancel to end this resource's lease. No generic teleport or map collision solution is provided.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `motion table, or nil`, `nil, reason on rejection`

```lua
local motion = Open77.motion.current(playerId)
if motion then print(motion.id, motion.phase) end
```

[Cyberware authority & motion guide](../../01%20Documentation/cyberware.md)

Registered in `LuaResourceRuntime.cs` (line 7995) as `LuaResourceRuntime.Bootstrap`.

## knockdown

```lua
Open77.motion.knockdown(playerId, options)
```

`NETWORK`: Uses the network backend.

Request bounded native living-player motion.

Requires players.motion.control. options={x,y,distance}; direction nonzero and normalized, distance0..6 m. Requires ready/alive/unmounted exact body and no existing lease; native owner additionally refuses unsuitable states. Returns admission, not damage or physical completion. onPlayerMotionChanged reports pending/active/ended; server pending expires after2s. Active snapshot checks are ACK-rooted, not exact native launch-origin enforcement.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `options` | `table` | required | — |

Returns: `{ok=true,id=string}`, `nil, reason on rejection`

```lua
local result, err = Open77.motion.knockdown(playerId, {x=0,y=1,distance=0})
if not result then print(err) else print(result.id) end
```

[Cyberware authority & motion guide](../../01%20Documentation/cyberware.md)

Registered in `LuaResourceRuntime.cs` (line 8000) as `LuaResourceRuntime.Bootstrap`.

## launch

```lua
Open77.motion.launch(playerId, options)
```

`NETWORK`: Uses the network backend.

Request a bounded server-authorized character launch.

Requires players.motion.control and a matching development server. options={x,y,push,lift}; XY must be finite and nonzero and is normalized. push is 0..20 m/s, lift is 0..14 m/s, and at least one must be positive. Requires a ready, alive, unmounted exact incarnation with no active motion or recovery protection. Returns {ok=true,id=string} for admission, or nil/reason; admission is not completed movement. onPlayerMotionChanged reports pending/active/ended, and the pending lease expires after two seconds without the owning client's acknowledgement. Real owners apply one native 3D impulse; updated headless bots publish a bounded ballistic fixture trajectory without REDengine terrain or collision simulation. Observers present the pose and follow owner snapshots. current(playerId) reads the lease, including kind, push and lift; cancel(playerId,id) ends this resource's lease. No damage or generic teleport authority is granted. Motion does not require an implant database; the server still requires canonical body readiness and matching life, incarnation and bucket.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `options` | `table` | required | — |

Returns: `{ok=true,id=string}`, `nil, reason on rejection`

```lua
-- Server: target was selected and authorized by gameplay policy.
local result, reason = Open77.motion.launch(target, {x=1,y=0,push=12,lift=10})
if result then print(result.id) else print(reason) end
```

[Character launch & weapon blasts](../../01%20Documentation/weapon-customization.md#launch-nearby-characters)

Registered in `LuaResourceRuntime.cs` (line 7988) as `LuaResourceRuntime.Bootstrap`.
