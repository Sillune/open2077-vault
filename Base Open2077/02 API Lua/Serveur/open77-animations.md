# Open77.animations (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-animations.md) · [Page web](https://open2077.net/docs/api/server/open77-animations)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

11 functions.

## clip

```lua
Open77.animations.clip(clip)
```

`NETWORK`: Uses the network backend.

Look up which RP profile owns a clip name.

Clip-first discovery, the counterpart of get(). Returns the owning profile table, or nil when no shipped device's authored tree carries that clip. Requires no capability, like list() and get(). Clip names are case-sensitive. An absent result is the answer, not an error: a name can appear in the game's discovery inventory and still be unaddressable here. Provided directly by the server runtime; no animation resource export dependency is needed for this server call.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `clip` | `string` | required | — |

Returns: `Profile table, or nil when no shipped device carries the clip`

```lua
local profile = Open77.animations.clip('stand__dance__02__dancing__03')
print(profile and profile.id) -- dance
```

[RP animation guide](../../01%20Documentation/rp-animations.md#server-lua-api)

Registered in `LuaResourceRuntime.cs` (line 8103) as `LuaResourceRuntime.Bootstrap`.

## clips

```lua
Open77.animations.clips([query])
```

`NETWORK`: Uses the network backend.

List every addressable clip with the profile that owns it.

The nearest equivalent this engine offers to enumerating an animation dictionary, and the discovery step before playClip(). Returns one row per clip, { clip = string, profile = string }, ordered by clip name. The optional query is a case-insensitive substring matched against both the clip name and the profile ID, at most 128 characters. Requires no capability. This is the complete addressable set, not a catalogue of what the game contains. Provided directly by the server runtime; no animation resource export dependency is needed for this server call.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `query` | `string` | optional | `''` |

Returns: `Array of { clip, profile } rows`, `nil, invalid_query when the query exceeds 128 characters`

```lua
for _, row in ipairs(Open77.animations.clips('dance') or {}) do
    print(row.profile, row.clip)
end
```

[RP animation guide](../../01%20Documentation/rp-animations.md#server-lua-api)

Registered in `LuaResourceRuntime.cs` (line 8104) as `LuaResourceRuntime.Bootstrap`.

## current

```lua
Open77.animations.current(playerId)
```

`NETWORK`: Uses the network backend.

Read a player's authoritative active RP playback state.

Requires players.animations.read, enforced at call time: without it the answer is nil, permission_denied:players.animations.read. (This card's generated permissions list is empty because the handler chooses the permission name at run time; the requirement is real.) Returns the active state synchronously, or nil without an error when inactive. Fields: epoch, revision, playerId, playbackId, active (always true here), reason (empty here), bucket, steps, loop, startedAtMs, serverTimeMs, cycle, step, elapsedMs, remainingMs, clientRequestId, and anchor ({ x, y, z, yaw }) only for an action started with playAt. steps is an array of { profile, clip, durationMs }; there is no top-level profileId -- read steps[1].profile. remainingMs is -1 for a step with no finite duration. Timestamps are monotonic server time and step/cycle indices are zero-based. clientRequestId is internal correlation, not a cancellation token. State proves scheduling, not that a native clip rendered. Unlike client current(entity), this does not return a raw clip name. Provided directly by the server runtime; no animation resource export dependency is needed for this server call.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `Active playback state, or nil when inactive`, `nil, permission_denied:players.animations.read | invalid_player | animations_unavailable on failure`

```lua
-- open77.lua: permission 'players.animations.read'
local state, err = Open77.animations.current(playerId)
if err then print(err); return end
if state then print(state.playbackId, state.steps[1].profile, state.step, state.elapsedMs) end
```

[RP animation guide](../../01%20Documentation/rp-animations.md#server-lua-api)

Registered in `LuaResourceRuntime.cs` (line 8111) as `LuaResourceRuntime.Bootstrap`.

## get

```lua
Open77.animations.get(profileId)
```

`NETWORK`: Uses the network backend.

Read the definition and allowed clips of one RP profile.

No capability is required. Looks up an exact, case-sensitive profile ID such as smoke or phone. Returns nil without an error for an unknown profile. Use the returned clip and clips fields to construct a valid playback request instead of passing arbitrary game animation names. Provided directly by the server runtime; no animation resource export dependency is needed for this server call.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `profileId` | `string` | required | — |

Returns: `Profile table, or nil when absent`, `nil, error on failure`

```lua
local profile, err = Open77.animations.get('smoke')
if err then print(err); return end
if profile then print(profile.clip, #profile.clips) end
```

[RP animation guide](../../01%20Documentation/rp-animations.md#server-lua-api)

Registered in `LuaResourceRuntime.cs` (line 8087) as `LuaResourceRuntime.Bootstrap`.

## list

```lua
Open77.animations.list([query])
```

`NETWORK`: Uses the network backend.

Search the authoritative RP profile catalogue.

Returns all supported profiles when the query is empty, or a case-insensitive substring match across ID, label and category (up to 128 characters). Definitions include kind, clip variants and rig metadata. Choose a kind=layer profile such as drink_walk to preserve locomotion; catalogue membership does not prove native rendering. No capability is required; returns definitions synchronously. Provided directly by the server runtime; no animation resource export dependency is needed for this server call.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `query` | `string` | optional | `""` |

Returns: `Profile array on success`, `nil, error on failure`

```lua
local profiles, err = Open77.animations.list('social')
if not profiles then print(err); return end
for _, profile in ipairs(profiles) do print(profile.id, profile.label) end
```

[RP animation guide](../../01%20Documentation/rp-animations.md#server-lua-api)

Registered in `LuaResourceRuntime.cs` (line 8086) as `LuaResourceRuntime.Bootstrap`.

## play

```lua
Open77.animations.play(playerId, profileId, [options])
```

`NETWORK`: Uses the network backend.

Start a server-owned, synchronized RP action on a player.

Requires players.animations.control. The calling resource generation owns the action; another resource cannot replace it. The player must be ready, alive, on foot and have a known position. Returns authoritative acceptance and playbackId, not proof of native presentation. Options are clip, durationMs, loop, item and itemContact. The clip must belong to the selected profile. loop defaults to true. Looping profiles may use omitted/zero duration for indefinite playback; once profiles and loop=false use the measured clip duration when available, otherwise 5000 ms. Explicit nonzero durations must be integers from 1000 to 600000 ms. For a layer, item omitted uses its default native item; item=false suppresses the animation-owned item. A server may select an Items.* record with world.props. Custom records requested by clients return item_requires_server; workspots reject the item option with item_requires_layer. The native attachment supplies the grip; item does not accept offset or rotation. The runtime owns temporary-item cleanup; inventory, needs and job rewards remain resource-owned. Upper-body layers preserve walking and running; workspots cancel on movement. Requires matching client, server and animation archives. The item option requires native animation-item support; the native function's since field does not establish the first build supporting every option. First-person support is experimental and profile-dependent. See [RP animations](../../01%20Documentation/rp-animations.md#upper-body-actions-for-jobs-and-inventory). Provided directly by the server runtime; no animation resource export dependency is needed for this server call. Experimental itemContact is server-only. Supplying this option needs the world.props capability, including false. Omit it for automatic geometry-based mouth contact, use false to disable fitting, or {x,y,z} in item-local metres (missing axes are zero; each supplied coordinate finite and within ±2). Accepted on a layer only; invalid points return invalid_item_contact before replacing the action. It adjusts the arm target, preserves the authored grip and reuses the animation-owned item ID when its record is unchanged. Fitting applies to drink_walk, bottle_walk, smoke_walk and cigar_walk; hold_item_walk only holds. Matching runtime/assets are required and visual validation is incomplete. See [item size and mouth contact](../../01%20Documentation/rp-animations.md#item-size-and-mouth-contact). Automatic container contact uses the main mesh end along the authored item-local up direction (the can rim), keeping that end while tilting; smoking uses the end nearest the mouth. In first person, held items stay visible at the lower right between sips and contact follows the camera. Disabling mouth fitting keeps the first-person holding pose.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `profileId` | `string` | required | — |
| `options` | `table` | optional | `{}` |

Returns: `Accepted playback state including playbackId`, `nil, error on rejection`

```lua
-- open77.lua: permission 'players.animations.control'
local playback, err = Open77.animations.play(playerId, 'smoke', {
    durationMs = 15000, loop = false,
})
if not playback then print(err); return end
print(playback.playbackId)
```

[RP animation guide](../../01%20Documentation/rp-animations.md#server-lua-api)

Registered in `LuaResourceRuntime.cs` (line 8088) as `LuaResourceRuntime.Bootstrap`.

## playAt

```lua
Open77.animations.playAt(playerId, profileId, position, [yaw], [options])
```

`NETWORK`: Uses the network backend.

Play a profile at a world pose: the placed form of play (TaskStartScenarioAtPosition).

Requires `players.animations.control`. Moves a player to an anchor, waits for placement, then starts a profile such as `chair`, `lean` or `lie`. Any catalogue profile accepts a pose. `position` is `{x, y, z}` or a three-element array; `yaw` is degrees about Z and defaults to 0. Options match `play`: `clip`, `durationMs`, `loop`. Anchors must be within 5 m; use `Open77.players.teleport` for longer moves. Returns accepted playback state with `anchor`; use `playbackId` with `stopAt`. The caller receives acceptance before clients receive the posture; playback timing starts on arrival. A 5-second anchor watchdog ends an unsuccessful placement with `anchor_unreached`. Observe `onPlayerAnimationChanged`; admission is not proof of rendered animation. Additional errors: `invalid_anchor`, `anchor_too_far`, `anchor_move_refused` (unready, dead or in a vehicle). Provided by the server runtime, without a resource-export dependency.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `profileId` | `string` | required | — |
| `position` | `table { x, y, z }` | required | — |
| `yaw` | `number` | optional | `0` |
| `options` | `table` | optional | `{}` |

Returns: `Accepted playback state including playbackId and anchor { x, y, z, yaw }`, `nil, error on rejection`

```lua
-- open77.lua: permission 'players.animations.control'
-- Sit a player on a chair prop that has no device of its own: the pose is
-- the chair's seat, the yaw is the direction the chair faces.
local chair = { x = -1441.2, y = 129.6, z = 18.05 }
local placed, err = Open77.animations.playAt(playerId, 'chair', chair, 90.0, {
    durationMs = 30000, loop = false,
})
if not placed then print('cannot seat: ' .. tostring(err)); return end
seated[playerId] = placed.playbackId   -- the handle stopAt() takes
```

[RP animation guide](../../01%20Documentation/rp-animations.md#server-lua-api)

Registered in `LuaResourceRuntime.cs` (line 8091) as `LuaResourceRuntime.Bootstrap`.

## playClip

```lua
Open77.animations.playClip(playerId, clip, [options])
```

`NETWORK`: Uses the network backend.

Start a synchronized RP action on a player, addressed by clip name.

Requires players.animations.control. Resolves an exact clip through the shipped profile catalogue; a missing clip returns unknown_clip. Discovery inventories are not allowlists. A clip shared with a workspot resolves to the stationary profile: use play/request with a layer profile ID to select walking behavior. Options are durationMs, loop, item and itemContact; the clip is the argument. Unsupported upperBody, blendIn, blendOut, blendMs, holdLastFrame, flags, dict and playbackRate options return unsupported_option:<key>; upperBody does not convert arbitrary clips. For a layer, item omitted uses its default native item; item=false suppresses the animation-owned item. A server may select an Items.* record with world.props. Custom records requested by clients return item_requires_server; workspots reject the item option with item_requires_layer. The native attachment supplies the grip; item does not accept offset or rotation. The runtime owns temporary-item cleanup; inventory, needs and job rewards remain resource-owned. Upper-body layers preserve walking and running; workspots cancel on movement. Requires matching client, server and animation archives. The item option requires native animation-item support; the native function's since field does not establish the first build supporting every option. First-person support is experimental and profile-dependent. See [RP animations](../../01%20Documentation/rp-animations.md#upper-body-actions-for-jobs-and-inventory). Provided directly by the server runtime; no animation resource export dependency is needed for this server call. Experimental itemContact is server-only. Supplying this option needs the world.props capability, including false. Omit it for automatic geometry-based mouth contact, use false to disable fitting, or {x,y,z} in item-local metres (missing axes are zero; each supplied coordinate finite and within ±2). Accepted on a layer only; invalid points return invalid_item_contact before replacing the action. It adjusts the arm target, preserves the authored grip and reuses the animation-owned item ID when its record is unchanged. Fitting applies to drink_walk, bottle_walk, smoke_walk and cigar_walk; hold_item_walk only holds. Matching runtime/assets are required and visual validation is incomplete. See [item size and mouth contact](../../01%20Documentation/rp-animations.md#item-size-and-mouth-contact). Automatic container contact uses the main mesh end along the authored item-local up direction (the can rim), keeping that end while tilting; smoking uses the end nearest the mouth. In first person, held items stay visible at the lower right between sips and contact follows the camera. Disabling mouth fitting keeps the first-person holding pose.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `clip` | `string` | required | — |
| `options` | `table` | optional | `{}` |

Returns: `Accepted playback state including playbackId`, `nil, unknown_clip | unsupported_option:<key> | invalid_duration | any play() rejection`

```lua
-- open77.lua: permission 'players.animations.control'
local playback, err = Open77.animations.playClip(
    playerId, 'stand__dance__02__dancing__03', { durationMs = 15000, loop = false })
if not playback then print(err); return end
print(playback.playbackId)
```

[RP animation guide](../../01%20Documentation/rp-animations.md#server-lua-api)

Registered in `LuaResourceRuntime.cs` (line 8100) as `LuaResourceRuntime.Bootstrap`.

## sequence

```lua
Open77.animations.sequence(playerId, steps, [options])
```

`NETWORK`: Uses the network backend.

Schedule a server-owned sequence of RP profiles for a player.

Requires players.animations.control and the same ownership/readiness checks as play(). Accepts 1–16 steps with profile, optional clip, durationMs, item and itemContact. Each omitted duration uses the measured clip cycle when available, otherwise 5000 ms; steps must last 1000–600000 ms and the total cannot exceed 600000 ms. The sequence option loop defaults to false. A hold_item_walk → drink_walk → hold_item_walk chain preserves the prop ID while the item record stays the same. Each new play call gets a new playbackId; a sequence coordinates one player. Workspot transitions may require asynchronous exit/re-entry. For a layer, item omitted uses its default native item; item=false suppresses the animation-owned item. A server may select an Items.* record with world.props. Custom records requested by clients return item_requires_server; workspots reject the item option with item_requires_layer. The native attachment supplies the grip; item does not accept offset or rotation. The runtime owns temporary-item cleanup; inventory, needs and job rewards remain resource-owned. Upper-body layers preserve walking and running; workspots cancel on movement. Requires matching client, server and animation archives. The item option requires native animation-item support; the native function's since field does not establish the first build supporting every option. First-person support is experimental and profile-dependent. See [RP animations](../../01%20Documentation/rp-animations.md#upper-body-actions-for-jobs-and-inventory). Provided directly by the server runtime; no animation resource export dependency is needed for this server call. Experimental itemContact is server-only. Supplying this option needs the world.props capability, including false. Omit it for automatic geometry-based mouth contact, use false to disable fitting, or {x,y,z} in item-local metres (missing axes are zero; each supplied coordinate finite and within ±2). Accepted on a layer only; invalid points return invalid_item_contact before replacing the action. It adjusts the arm target, preserves the authored grip and reuses the animation-owned item ID when its record is unchanged. Fitting applies to drink_walk, bottle_walk, smoke_walk and cigar_walk; hold_item_walk only holds. Matching runtime/assets are required and visual validation is incomplete. See [item size and mouth contact](../../01%20Documentation/rp-animations.md#item-size-and-mouth-contact). Automatic container contact uses the main mesh end along the authored item-local up direction (the can rim), keeping that end while tilting; smoking uses the end nearest the mouth. In first person, held items stay visible at the lower right between sips and contact follows the camera. Disabling mouth fitting keeps the first-person holding pose.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `steps` | `table[]` | required | — |
| `options` | `table` | optional | `{ loop = false }` |

Returns: `Accepted playback state including playbackId`, `nil, error on rejection`

```lua
local playback, err = Open77.animations.sequence(playerId, {
    { profile = 'handsup', durationMs = 4000 },
    { profile = 'clap', durationMs = 6000 },
}, { loop = false })
if not playback then print(err); return end
print(playback.playbackId)
```

[RP animation guide](../../01%20Documentation/rp-animations.md#server-lua-api)

Registered in `LuaResourceRuntime.cs` (line 8105) as `LuaResourceRuntime.Bootstrap`.

## stop

```lua
Open77.animations.stop(playerId, [playbackId])
```

`NETWORK`: Uses the network backend.

Stop an RP action owned by the calling server resource.

Requires players.animations.control. Omitting playbackId stops the current action owned by this VM generation; an action owned by another resource is refused. Pass the recorded ID in delayed callbacks so an old timer cannot stop a newer action: stale IDs return stale_playback. Stopping an already inactive player succeeds. The authoritative onPlayerAnimationChanged event carries the player ID string and JSON state string; native detachment may finish asynchronously on clients. This is not the client-local stop(entity) method. Provided directly by the server runtime; no animation resource export dependency is needed for this server call.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `playbackId` | `string` | optional | — |

Returns: `true on success, including already inactive`, `nil, error on rejection`

```lua
local playback, err = Open77.animations.play(playerId, 'smoke')
if not playback then print(err); return end
SetTimeout(10000, function()
    local stopped, reason = Open77.animations.stop(playerId, playback.playbackId)
    if not stopped then print(reason) end
end)
```

[RP animation guide](../../01%20Documentation/rp-animations.md#server-lua-api)

Registered in `LuaResourceRuntime.cs` (line 8108) as `LuaResourceRuntime.Bootstrap`.

## stopAt

```lua
Open77.animations.stopAt(playbackId)
```

`NETWORK`: Uses the network backend.

Stop a placed action by its handle alone.

Requires players.animations.control. Takes the playbackId that playAt (or play) returned and ends that action, tearing down its workspot device on every client; nothing else needs remembering, in particular not which player it belonged to. Refuses with animation_owned when another resource VM started it and invalid_playback for an empty or oversized handle. A handle whose action already ended succeeds, exactly as stop does on an idle player: once the entry is gone, 'already finished' and 'never existed' are the same answer, and the caller's intent -- nothing running under this handle -- holds either way. Resource stop, restart and disposal tear placed actions down without this call, as they do every other action the VM owns. Provided directly by the server runtime; no animation resource export dependency is needed for this server call.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playbackId` | `string` | required | — |

Returns: `true, including when the handle no longer names a running action`, `nil, error on rejection`

```lua
-- Let the player stand up again when they press the key, whoever sat them down
RegisterCommand('standup', function(source)
    local handle = seated[source]
    if not handle then return end
    local ok, err = Open77.animations.stopAt(handle)
    if not ok then print(err) end
    seated[source] = nil
end, false)
```

[RP animation guide](../../01%20Documentation/rp-animations.md#server-lua-api)

Registered in `LuaResourceRuntime.cs` (line 8097) as `LuaResourceRuntime.Bootstrap`.
