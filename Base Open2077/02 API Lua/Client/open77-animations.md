# Open77.animations (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-animations.md) · [Page web](https://open2077.net/docs/api/client/open77-animations)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

16 functions.

## cancel

```lua
Open77.animations.cancel([playbackId])
```

`NETWORK`: Uses the network backend.

Cancel the local player's RP action and pending requests.

Stops local presentation and queues a reliable self-cancellation request. Omitting playbackId also abandons pending starts; late acceptances are cancelled. Supply the ID from request() or sequence() in delayed callbacks so they cannot stop a replacement action. A mismatched active ID rejects with stale_playback. Resolving true means cancellation was queued (or nothing was active), not that the server has acknowledged the stop; observe onPlayerAnimationChanged for authoritative state. A player may cancel their own server-started action, never another player's action. Declare dependency 'open77_animations' in your resource manifest. This asynchronous facade requires the service in the same downloaded resource host, not a separate bootstrap host. Await from a managed coroutine/event handler. No animations.presentation capability is required by the caller; underscore hooks are internal.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playbackId` | `string` | optional | — |

Returns: `Open77.Promise resolving to true when cancellation is queued or already inactive`, `nil, reason if dispatch fails; await() returns nil, reason if rejected`

```lua
CreateThread(function()
    local pending, err = Open77.animations.cancel()
    if not pending then print(err); return end
    local cancelled, rejected = pending:await()
    if not cancelled then print(rejected) end
end)
```

[RP animation guide](../../01%20Documentation/rp-animations.md#client-lua-api)

Registered in `ResourceHost.cpp` (line 24717) as `LuaAnimationServiceExport`.

## clip

```lua
Open77.animations.clip(clip)
```

`NETWORK`: Uses the network backend.

Look up which RP profile owns a clip name.

Clip-first discovery: the counterpart of get(), for a caller that knows the animation name but not the action that carries it. Resolves to a copy of the owning profile, or nil when no shipped device's authored tree contains that clip. Clip names are case-sensitive. A name can appear in the game's 23,044-entry discovery inventory and still resolve to nil here: only the 70 clips of the twelve shipped devices are addressable. Declare dependency 'open77_animations' in your resource manifest. This asynchronous facade requires the service in the same downloaded resource host, not a separate bootstrap host. Await from a managed coroutine/event handler. No animations.presentation capability is required by the caller; underscore hooks are internal.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `clip` | `string` | required | — |

Returns: `Open77.Promise resolving to the owning profile table, or nil when no shipped device carries the clip`, `nil, reason if dispatch fails; await() returns nil, reason if rejected`

```lua
CreateThread(function()
    local pending = Open77.animations.clip('stand__dance__02__dancing__03')
    local profile = pending and pending:await()
    if profile then print(profile.id) end -- dance
end)
```

[RP animation guide](../../01%20Documentation/rp-animations.md#client-lua-api)

Registered in `ResourceHost.cpp` (line 24717) as `LuaAnimationServiceExport`.

## clips

```lua
Open77.animations.clips([query])
```

`NETWORK`: Uses the network backend.

List every addressable clip with the profile that owns it.

The closest thing this engine has to enumerating an animation dictionary. Returns one row per clip, { clip = string, profile = string }, sorted by clip name. The optional query is a case-insensitive substring matched against the clip name and the profile ID; it must be at most 128 characters. This is the complete addressable set, not a catalogue of what exists: a clip plays only because a shipped device's authored workspot tree carries it. Declare dependency 'open77_animations' in your resource manifest. This asynchronous facade requires the service in the same downloaded resource host, not a separate bootstrap host. Await from a managed coroutine/event handler. No animations.presentation capability is required by the caller; underscore hooks are internal.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `query` | `string` | optional | `''` |

Returns: `Open77.Promise resolving to an array of { clip, profile } rows`, `nil, reason if dispatch fails; await() returns nil, invalid_query if the query is too long`

```lua
CreateThread(function()
    local rows = Open77.animations.clips('smoke'):await()
    for _, row in ipairs(rows or {}) do print(row.profile, row.clip) end
end)
```

[RP animation guide](../../01%20Documentation/rp-animations.md#client-lua-api)

Registered in `ResourceHost.cpp` (line 24717) as `LuaAnimationServiceExport`.

## current

```lua
Open77.animations.current(entity)
```

`GAME`: Requires a live game instance.

The clip a body is currently playing.

Returns false when it is playing nothing — which is a state, not an error.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity` | required | — |

Returns: `clip name, or false`, `reason, when applicable`

```lua
-- Is the remote player's body playing an emote?
local entity = Open77.players.entity(playerId)
local clip = entity and Open77.animations.current(entity)
if clip then print("playing", clip) end
```

[RP animation guide](../../01%20Documentation/rp-animations.md#client-lua-api)

Registered in `ResourceHost.cpp` (line 24730) as `LuaAnimationCurrent`.

## get

```lua
Open77.animations.get(profileId)
```

`NETWORK`: Uses the network backend.

Look up one RP profile without starting playback.

Resolves to a copy of the profile, including its default clip and exact allowed variants. Profile IDs are case-sensitive. An unknown ID resolves to nil without an error; check the second await result to distinguish a failed request from a missing profile. Declare dependency 'open77_animations' in your resource manifest. This asynchronous facade requires the service in the same downloaded resource host, not a separate bootstrap host. Await from a managed coroutine/event handler. No animations.presentation capability is required by the caller; underscore hooks are internal.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `profileId` | `string` | required | — |

Returns: `Open77.Promise resolving to a profile table, or nil when absent`, `nil, reason if dispatch fails; await() returns nil, reason if rejected`

```lua
CreateThread(function()
    local pending, err = Open77.animations.get('smoke')
    if not pending then print(err); return end
    local profile, rejected = pending:await()
    if rejected then print(rejected); return end
    if profile then print(profile.clip, #profile.clips) end
end)
```

[RP animation guide](../../01%20Documentation/rp-animations.md#client-lua-api)

Registered in `ResourceHost.cpp` (line 24717) as `LuaAnimationServiceExport`.

## isPointing

```lua
Open77.animations.isPointing([playerId])
```

`NETWORK`: Uses the network backend.

Check whether a player is pointing in their camera direction.

Defaults to the local player, or accepts a visible player's network ID. Resolves to true while that player's active synchronized action is the looping camera_point profile and this client has not failed to present it, and to false otherwise, including when the player has no active action. It reads the state this client last received and does not ask the server. Do not use a client observation to grant authoritative rewards. Declare dependency 'open77_animations' in your resource manifest. This asynchronous facade requires the service in the same downloaded resource host, not a separate bootstrap host. Await from a managed coroutine/event handler. No animations.presentation capability is required by the caller; underscore hooks are internal.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | `local player` |

Returns: `Open77.Promise resolving to true while the player is pointing, otherwise false`, `nil, reason if dispatch fails; await() returns nil, reason if rejected`

```lua
CreateThread(function()
    local pending, err = Open77.animations.isPointing()
    if not pending then print(err); return end
    local active, rejected = pending:await()
    if rejected then print(rejected); return end
    print('pointing: ' .. tostring(active))
end)
```

[RP animation guide](../../01%20Documentation/rp-animations.md#client-lua-api)

Registered in `ResourceHost.cpp` (line 25627) as `LuaAnimationServiceExport`.

## list

```lua
Open77.animations.list([query])
```

`NETWORK`: Uses the network backend.

Find synchronized RP animation profiles in the local catalogue.

Returns all supported profiles when the query is empty, or a case-insensitive substring match across ID, label and category (up to 128 characters). Definitions include kind, clip variants and rig metadata. Choose a kind=layer profile such as drink_walk to preserve locomotion; catalogue membership does not prove native rendering. Resolves asynchronously to copied definitions. Declare dependency 'open77_animations' in your resource manifest. This asynchronous facade requires the service in the same downloaded resource host, not a separate bootstrap host. Await from a managed coroutine/event handler. No animations.presentation capability is required by the caller; underscore hooks are internal.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `query` | `string` | optional | `""` |

Returns: `Open77.Promise resolving to a profile array`, `nil, reason if dispatch fails; await() returns nil, reason if rejected`

```lua
CreateThread(function()
    local pending, err = Open77.animations.list('social')
    if not pending then print(err); return end
    local profiles, rejected = pending:await()
    if not profiles then print(rejected); return end
    for _, profile in ipairs(profiles) do print(profile.id, profile.label) end
end)
```

[RP animation guide](../../01%20Documentation/rp-animations.md#client-lua-api)

Registered in `ResourceHost.cpp` (line 24717) as `LuaAnimationServiceExport`.

## play

```lua
Open77.animations.play(entity, animation)
```

`GAME`: Requires a live game instance.

Plays a named animation on a body.

Does not work on the local player: their body cannot hold a workspot. Use `Open77.animations.playSelf` for that, which stands a double in for them.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity` | required | — |
| `animation` | `string` | required | — |

Returns: `true on success, otherwise false`, `reason for the refusal`

```lua
local ok, reason = Open77.animations.play(entityId, "emote_smoke")
if not ok then print("refused: " .. reason) end
```

[RP animation guide](../../01%20Documentation/rp-animations.md#client-lua-api)

Registered in `ResourceHost.cpp` (line 24726) as `LuaAnimationPlay`.

## playSelf

```lua
Open77.animations.playSelf(animation, [thirdPerson])
```

`GAME`: Requires a live game instance.

Plays an emote on the local player, through a stand-in body.

The player's own body cannot hold a workspot, so an invisible double takes their place for the length of the clip. This is a separate call rather than `play` with the player's id because the mechanism is genuinely different.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `animation` | `string` | required | — |
| `thirdPerson` | `boolean` | optional | — |

Returns: `true on success, otherwise false`, `reason for the refusal`

```lua
-- /sit as a purely local emote, through the stand-in body
RegisterCommand("sit", function()
    local ok, reason = Open77.animations.playSelf("emote_sit", true)
    if not ok then Open77.hud.notify("Cannot sit here: " .. reason) end
end, false)
```

[RP animation guide](../../01%20Documentation/rp-animations.md#client-lua-api)

Registered in `ResourceHost.cpp` (line 24727) as `LuaSelfEmotePlay`.

## request

```lua
Open77.animations.request(profileId, [options])
```

`NETWORK`: Uses the network backend.

Request a synchronized RP action for the local player.

Requests an action for the authenticated local player. Use cancel() for this synchronized action, not the legacy local stop(entity). Returns authoritative acceptance and playbackId, not proof of native presentation. Options are clip, durationMs, loop and item. The clip must belong to the selected profile. loop defaults to true. Looping profiles may use omitted/zero duration for indefinite playback; once profiles and loop=false use the measured clip duration when available, otherwise 5000 ms. Explicit nonzero durations must be integers from 1000 to 600000 ms. For a layer, item omitted uses its default native item; item=false suppresses the animation-owned item. A server may select an Items.* record with world.props. Custom records requested by clients return item_requires_server; workspots reject the item option with item_requires_layer. The native attachment supplies the grip; item does not accept offset or rotation. The runtime owns temporary-item cleanup; inventory, needs and job rewards remain resource-owned. Upper-body layers preserve walking and running; workspots cancel on movement. Requires matching client, server and animation archives. The item option requires native animation-item support; the native function's since field does not establish the first build supporting every option. First-person support is experimental and profile-dependent. See [RP animations](../../01%20Documentation/rp-animations.md#upper-body-actions-for-jobs-and-inventory). Declare dependency 'open77_animations' in your resource manifest. This asynchronous facade requires the service in the same downloaded resource host, not a separate bootstrap host. Await from a managed coroutine/event handler. No animations.presentation capability is required by the caller; underscore hooks are internal. itemContact is server-only, including false; client requests return item_requires_server. See [item size and mouth contact](../../01%20Documentation/rp-animations.md#item-size-and-mouth-contact) for the experimental server adapter.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `profileId` | `string` | required | — |
| `options` | `table` | optional | `{}` |

Returns: `Open77.Promise resolving to the server-accepted playback state`, `nil, reason if dispatch fails; await() returns nil, reason if rejected or timed out`

```lua
CreateThread(function()
    local pending, err = Open77.animations.request('smoke', { loop = false, durationMs = 15000 })
    if not pending then print(err); return end
    local playback, rejected = pending:await()
    if not playback then print(rejected); return end
    print('Accepted action: ' .. playback.playbackId)
end)
```

[RP animation guide](../../01%20Documentation/rp-animations.md#client-lua-api)

Registered in `ResourceHost.cpp` (line 24717) as `LuaAnimationServiceExport`.

## requestClip

```lua
Open77.animations.requestClip(clip, [options])
```

`NETWORK`: Uses the network backend.

Request a synchronized RP action for the local player by clip name.

Resolves an exact clip through the shipped profile catalogue; a missing clip returns unknown_clip. Discovery inventories are not allowlists. A clip shared with a workspot resolves to the stationary profile: use play/request with a layer profile ID to select walking behavior. Options are durationMs, loop and item; the clip is the argument. Unsupported upperBody, blendIn, blendOut, blendMs, holdLastFrame, flags, dict and playbackRate options return unsupported_option:<key>; upperBody does not convert arbitrary clips. For a layer, item omitted uses its default native item; item=false suppresses the animation-owned item. A server may select an Items.* record with world.props. Custom records requested by clients return item_requires_server; workspots reject the item option with item_requires_layer. The native attachment supplies the grip; item does not accept offset or rotation. The runtime owns temporary-item cleanup; inventory, needs and job rewards remain resource-owned. Upper-body layers preserve walking and running; workspots cancel on movement. Requires matching client, server and animation archives. The item option requires native animation-item support; the native function's since field does not establish the first build supporting every option. First-person support is experimental and profile-dependent. See [RP animations](../../01%20Documentation/rp-animations.md#upper-body-actions-for-jobs-and-inventory). Declare dependency 'open77_animations' in your resource manifest. This asynchronous facade requires the service in the same downloaded resource host, not a separate bootstrap host. Await from a managed coroutine/event handler. No animations.presentation capability is required by the caller; underscore hooks are internal. itemContact is server-only, including false; client requests return item_requires_server. See [item size and mouth contact](../../01%20Documentation/rp-animations.md#item-size-and-mouth-contact) for the experimental server adapter.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `clip` | `string` | required | — |
| `options` | `table` | optional | `{}` |

Returns: `Open77.Promise resolving to the accepted playback state including playbackId`, `nil, unknown_clip when no shipped device carries the clip; nil, reason on rejection`

```lua
CreateThread(function()
    local playback, err = Open77.animations.requestClip(
        'stand__dance__02__dancing__03', { loop = true }):await()
    if not playback then print(err); return end
    print(playback.playbackId)
end)
```

[RP animation guide](../../01%20Documentation/rp-animations.md#client-lua-api)

Registered in `ResourceHost.cpp` (line 24717) as `LuaAnimationServiceExport`.

## sequence

```lua
Open77.animations.sequence(steps, [options])
```

`NETWORK`: Uses the network backend.

Request an ordered RP sequence for the local player.

Takes no playerId; requests a sequence for the local player. Accepts 1–16 steps with profile, optional clip, durationMs and item. Each omitted duration uses the measured clip cycle when available, otherwise 5000 ms; steps must last 1000–600000 ms and the total cannot exceed 600000 ms. The sequence option loop defaults to false. A hold_item_walk → drink_walk → hold_item_walk chain preserves the prop ID while the item record stays the same. Each new play call gets a new playbackId; a sequence coordinates one player. Workspot transitions may require asynchronous exit/re-entry. For a layer, item omitted uses its default native item; item=false suppresses the animation-owned item. A server may select an Items.* record with world.props. Custom records requested by clients return item_requires_server; workspots reject the item option with item_requires_layer. The native attachment supplies the grip; item does not accept offset or rotation. The runtime owns temporary-item cleanup; inventory, needs and job rewards remain resource-owned. Upper-body layers preserve walking and running; workspots cancel on movement. Requires matching client, server and animation archives. The item option requires native animation-item support; the native function's since field does not establish the first build supporting every option. First-person support is experimental and profile-dependent. See [RP animations](../../01%20Documentation/rp-animations.md#upper-body-actions-for-jobs-and-inventory). Declare dependency 'open77_animations' in your resource manifest. This asynchronous facade requires the service in the same downloaded resource host, not a separate bootstrap host. Await from a managed coroutine/event handler. No animations.presentation capability is required by the caller; underscore hooks are internal. itemContact is server-only, including false; client requests return item_requires_server. See [item size and mouth contact](../../01%20Documentation/rp-animations.md#item-size-and-mouth-contact) for the experimental server adapter.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `steps` | `table[]` | required | — |
| `options` | `table` | optional | `{ loop = false }` |

Returns: `Open77.Promise resolving to the server-accepted playback state`, `nil, reason if dispatch fails; await() returns nil, reason if rejected`

```lua
CreateThread(function()
    local pending, err = Open77.animations.sequence({
        { profile = 'handsup', durationMs = 4000 },
        { profile = 'clap', durationMs = 6000 },
    }, { loop = false })
    if not pending then print(err); return end
    local playback, rejected = pending:await()
    if not playback then print(rejected); return end
    print(playback.playbackId)
end)
```

[RP animation guide](../../01%20Documentation/rp-animations.md#client-lua-api)

Registered in `ResourceHost.cpp` (line 24717) as `LuaAnimationServiceExport`.

## setPointing

```lua
Open77.animations.setPointing(enabled)
```

`NETWORK`: Uses the network backend.

Start or stop camera-directed finger pointing for the local player.

true requests the looping camera_point profile for the local player on behalf of the calling resource. It is an upper-body layer on the right arm, so walking continues, and every client aims that arm along the player's replicated camera direction. It resolves to the server-accepted playback state. Repeating true while this resource's pointing action is active resolves to that state without restarting it. Any other active animation, including a pointing action another resource started, rejects with animation_busy; a pending animation request rejects with request_pending; a call before the local player is ready rejects with player_not_ready; and a request the server has not answered within 10 seconds rejects with request_timeout. false stops pointing: it abandons this resource's pending pointing request, whose await then returns nil, cancelled, and cancels the active pointing action this resource started. It resolves to true when cancellation is queued or nothing is pointing, and never stops a different animation; a pointing action owned by another resource, or by an earlier generation of this one, rejects with not_owner. A non-boolean argument rejects with invalid_enabled. The local player's pointing stops when they sprint, slide, vault, swim, leave the ground, are knocked down, draw a weapon, attack or block in melee, or mount a vehicle; that failure reaches onAnimationPlaybackFailed with the reason interrupted. When the owning resource stops or restarts, the service cancels its pointing action. Call true and false from the press and release handlers of an input rather than back to back. Requires matching client, server and animation archives that include the camera_point layer. Declare dependency 'open77_animations' in your resource manifest. This asynchronous facade requires the service in the same downloaded resource host, not a separate bootstrap host. Await from a managed coroutine/event handler. No animations.presentation capability is required by the caller; underscore hooks are internal.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `Open77.Promise resolving to the server-accepted playback state when starting, or true when stopping`, `nil, reason if dispatch fails; await() returns nil, reason if rejected or timed out`

```lua
-- Hold B to point; release to stop
RegisterKeyMapping('point', 'Point with your finger', 'B',
    function()
        local pending, err = Open77.animations.setPointing(true)
        if not pending then print(err); return end
        CreateThread(function()
            local playback, rejected = pending:await()
            if not playback then print(rejected) end
        end)
    end,
    function()
        local pending, err = Open77.animations.setPointing(false)
        if not pending then print(err) end
    end)
```

[RP animation guide](../../01%20Documentation/rp-animations.md#client-lua-api)

Registered in `ResourceHost.cpp` (line 25627) as `LuaAnimationServiceExport`.

## state

```lua
Open77.animations.state([playerId])
```

`NETWORK`: Uses the network backend.

Read the last active RP state visible to this client.

Defaults to the local player, or accepts a visible player's network ID. Resolves to a copied canonical state, or nil when inactive or unavailable. It does not query arbitrary players outside the routing bucket or verify native rendering. Fields include playerId, playbackId, epoch, revision, active, steps, loop and step timing. On the client, onPlayerAnimationChanged receives this state as a Lua table; the server event instead passes JSON. Do not use a client observation to grant authoritative rewards. Declare dependency 'open77_animations' in your resource manifest. This asynchronous facade requires the service in the same downloaded resource host, not a separate bootstrap host. Await from a managed coroutine/event handler. No animations.presentation capability is required by the caller; underscore hooks are internal.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | `local player` |

Returns: `Open77.Promise resolving to active playback state, or nil when unavailable`, `nil, reason if dispatch fails; await() returns nil, reason if rejected`

```lua
CreateThread(function()
    local pending, err = Open77.animations.state()
    if not pending then print(err); return end
    local state, rejected = pending:await()
    if rejected then print(rejected); return end
    if state then print(state.playbackId, state.active) end
end)
```

[RP animation guide](../../01%20Documentation/rp-animations.md#client-lua-api)

Registered in `ResourceHost.cpp` (line 24717) as `LuaAnimationServiceExport`.

## stop

```lua
Open77.animations.stop(entity)
```

`GAME`: Requires a live game instance.

Stops whatever a body is playing and releases what was set up for it.

Stops the workspot animation currently managed by Open77 for the selected body and releases its native animation state. It does not claim or stop arbitrary game animations that Open77 does not own.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity` | required | — |

Returns: `true on success, otherwise false`, `reason for the refusal`

```lua
-- Stop the emote this resource started on another body
local entity = Open77.players.entity(playerId)
if entity then Open77.animations.stop(entity) end
```

[RP animation guide](../../01%20Documentation/rp-animations.md#client-lua-api)

Registered in `ResourceHost.cpp` (line 24729) as `LuaAnimationStop`.

## stopSelf

```lua
Open77.animations.stopSelf()
```

`GAME`: Requires a live game instance.

Stops the local player's emote and removes the stand-in.

Ends the local-player emote created by `playSelf` and removes its stand-in body. Cleanup is idempotent and is also performed when the owning resource generation stops.

Returns: `true on success, otherwise false`, `reason for the refusal`

```lua
-- Any movement key ends the local emote
RegisterKeyMapping("emote_stop", "Stop emote", "X", function()
    Open77.animations.stopSelf()
end)
```

[RP animation guide](../../01%20Documentation/rp-animations.md#client-lua-api)

Registered in `ResourceHost.cpp` (line 24728) as `LuaSelfEmoteStop`.
