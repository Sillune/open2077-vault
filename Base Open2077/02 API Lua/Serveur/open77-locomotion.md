# Open77.locomotion (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-locomotion.md) · [Page web](https://open2077.net/docs/api/server/open77-locomotion)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

6 functions.

## get

```lua
Open77.locomotion.get(playerId)
```

`SERVER`: Available in server resources only.

Read a player locomotion assignment.

Returns player, style, packageRevision, revision, resetOnDeath and optional remainingMs. nil without a reason means no custom style is assigned. This reports canonical assignment, not rendered animation readiness. See [Custom locomotion](https://open2077.net/docs/api/server/custom-locomotion.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `assignment table, or nil`, `reason on failure`

```lua
local selected, reason = Open77.locomotion.get(playerId)
```

[Custom locomotion & collections guide](../../01%20Documentation/custom-locomotion.md)

Registered in `LuaResourceRuntime.cs` (line 8853) as `LuaResourceRuntime.Bootstrap`.

## playClip

```lua
Open77.locomotion.playClip(playerId, clipId, [options])
```

`SERVER`: Available in server resources only.

Play a declared clip from the selected locomotion package.

The calling resource must own the selected style. The player must be ready, alive and outside a vehicle. clipId is a declared package clip, not a depot path. options accepts speed (0.25..4, default 1), blendInMs (0..2000, default 80), blendOutMs (0..2000, default 120). Playback is one-shot, without native root motion or game animation events. Late joiners receive elapsed time; finished actions do not restart. See [Custom locomotion](https://open2077.net/docs/api/server/custom-locomotion.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `clipId` | `string` | required | — |
| `options` | `table` | optional | — |

Returns: `true, handle on success`, `false, reason on failure`

```lua
local ok, handle = Open77.locomotion.playClip(playerId, 'wave', { blendInMs = 80, blendOutMs = 120 })
```

[Custom locomotion & collections guide](../../01%20Documentation/custom-locomotion.md)

Registered in `LuaResourceRuntime.cs` (line 8855) as `LuaResourceRuntime.Bootstrap`.

## register

```lua
Open77.locomotion.register(path)
```

`SERVER`: Available in server resources only.

Register a custom locomotion package.

Requires locomotion.register. Reads a version 1 package metadata file relative to this resource. The shared registration script runs on both runtimes. The client validates the graph and mounted native assets; registration does not assign a style. Stable 147 supports 1,024 registered packages across resources/reload generations; existing standalone packages stay compatible. See [Custom locomotion](https://open2077.net/docs/api/server/custom-locomotion.md#registration).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `path` | `string` | required | — |

Returns: `style ID string, or nil`, `reason on failure`

```lua
local style, reason = Open77.locomotion.register('package.json')
```

[Custom locomotion & collections guide](../../01%20Documentation/custom-locomotion.md)

Registered in `LuaResourceRuntime.cs` (line 8851) as `LuaResourceRuntime.Bootstrap`.

## reset

```lua
Open77.locomotion.reset(playerId)
```

`SERVER`: Available in server resources only.

Release this resource's locomotion assignment.

The resource that selected the style owns its reset. Returns true even when there is no assignment. Resource stop, disconnect and package removal also release the selection. See [Custom locomotion](https://open2077.net/docs/api/server/custom-locomotion.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `true on success`, `false, reason on failure`

```lua
local ok, reason = Open77.locomotion.reset(playerId)
```

[Custom locomotion & collections guide](../../01%20Documentation/custom-locomotion.md)

Registered in `LuaResourceRuntime.cs` (line 8854) as `LuaResourceRuntime.Bootstrap`.

## set

```lua
Open77.locomotion.set(playerId, styleId, [options])
```

`SERVER`: Available in server resources only.

Assign a registered movement style to a player.

Requires an authenticated, ready, living player. options accepts durationMs (0..86400000, default 0 for no expiry) and resetOnDeath (default false). The assigning resource generation owns the selection. Repeating the same style/package/death policy renews its duration without respawning the body. Another resource cannot replace it. See [Custom locomotion](https://open2077.net/docs/api/server/custom-locomotion.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `styleId` | `string` | required | — |
| `options` | `table` | optional | — |

Returns: `true, assignment revision on success`, `false, reason on failure`

```lua
local ok, revision = Open77.locomotion.set(playerId, 'relaxed', { resetOnDeath = true })
```

[Custom locomotion & collections guide](../../01%20Documentation/custom-locomotion.md)

Registered in `LuaResourceRuntime.cs` (line 8852) as `LuaResourceRuntime.Bootstrap`.

## stopClip

```lua
Open77.locomotion.stopClip(playerId, [handle], [blendOutMs])
```

`SERVER`: Available in server resources only.

Stop a locomotion clip owned by this resource.

Omit handle to stop the current clip, or pass the returned handle so an old timer cannot cancel a newer action. blendOutMs is 0..2000, default 120. A stale handle returns stale_clip_handle. Returns true, revision (0 for a no-op). See [Custom locomotion](https://open2077.net/docs/api/server/custom-locomotion.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `handle` | `integer` | optional | — |
| `blendOutMs` | `integer` | optional | — |

Returns: `true, revision on success`, `false, reason on failure`

```lua
local ok, revision = Open77.locomotion.stopClip(playerId, handle, 120)
```

[Custom locomotion & collections guide](../../01%20Documentation/custom-locomotion.md)

Registered in `LuaResourceRuntime.cs` (line 8856) as `LuaResourceRuntime.Bootstrap`.
