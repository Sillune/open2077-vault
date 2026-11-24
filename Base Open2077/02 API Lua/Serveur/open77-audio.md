# Open77.audio (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-audio.md) · [Page web](https://open2077.net/docs/api/server/open77-audio)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

10 functions.

## destroy

```lua
Open77.audio.destroy(id)
```

`NETWORK`: Uses the network backend.

DestroySound: server-authoritative network package audio.

Requires audio.network; network=false is rejected. Server owns the timeline, handles and bucket/optional-target audience. Clients cannot request broadcasts directly. getState reports intended playback, not decoder acknowledgement. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `true, or failure, reason`

```lua
-- Release the network handle
Open77.audio.destroy(concertId)
```

[Package audio: 2D & 3D guide](../../01%20Documentation/package-audio.md)

Registered in `LuaResourceRuntime.cs` (line 9881) as `LuaAudioControl`.

## getState

```lua
Open77.audio.getState(id)
```

`NETWORK`: Uses the network backend.

GetSoundState: server-authoritative network package audio.

Requires audio.network; network=false is rejected. Server owns the timeline, handles and bucket/optional-target audience. Clients cannot request broadcasts directly. getState reports intended playback, not decoder acknowledgement. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `table, or nil, reason`

```lua
-- Where is the shared timeline?
local state = Open77.audio.getState(concertId)
if state then print(state.status, state.position) end
```

[Package audio: 2D & 3D guide](../../01%20Documentation/package-audio.md)

Registered in `LuaResourceRuntime.cs` (line 9882) as `LuaAudioState`.

## pause

```lua
Open77.audio.pause(id)
```

`NETWORK`: Uses the network backend.

PauseSound: server-authoritative network package audio.

Requires audio.network; network=false is rejected. Server owns the timeline, handles and bucket/optional-target audience. Clients cannot request broadcasts directly. getState reports intended playback, not decoder acknowledgement. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `true, or failure, reason`

```lua
-- Intermission
Open77.audio.pause(concertId)
```

[Package audio: 2D & 3D guide](../../01%20Documentation/package-audio.md)

Registered in `LuaResourceRuntime.cs` (line 9876) as `LuaAudioControl`.

## play

```lua
Open77.audio.play(id)
```

`NETWORK`: Uses the network backend.

PlaySound: server-authoritative network package audio.

Requires audio.network; network=false is rejected. Server owns the timeline, handles and bucket/optional-target audience. Clients cannot request broadcasts directly. getState reports intended playback, not decoder acknowledgement. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `true, or failure, reason`

```lua
-- Resume the server-owned concert stream
Open77.audio.play(concertId)
```

[Package audio: 2D & 3D guide](../../01%20Documentation/package-audio.md)

Registered in `LuaResourceRuntime.cs` (line 9875) as `LuaAudioControl`.

## play2D

```lua
Open77.audio.play2D(asset, [options])
```

`NETWORK`: Uses the network backend.

Play2DSound: server-authoritative network package audio.

Requires audio.network; network=false is rejected. Server owns the timeline, handles and bucket/optional-target audience. Clients cannot request broadcasts directly. getState reports intended playback, not decoder acknowledgement. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `asset` | `string` | required | — |
| `options` | `table` | optional | — |

Returns: `integer handle, or nil, reason`

```lua
-- A synchronized announcement for everyone in bucket 0
local id = Open77.audio.play2D("audio/curfew.wav", { bucket = 0, volume = 0.5, duration = 12 })
```

[Package audio: 2D & 3D guide](../../01%20Documentation/package-audio.md)

Registered in `LuaResourceRuntime.cs` (line 9869) as `LuaAudioCreate`.

## play3D

```lua
Open77.audio.play3D(asset, position, [options])
```

`NETWORK`: Uses the network backend.

Play3DSound: server-authoritative network package audio.

Requires audio.network; network=false is rejected. Server owns the timeline, handles and bucket/optional-target audience. Clients cannot request broadcasts directly. getState reports intended playback, not decoder acknowledgement. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `asset` | `string` | required | — |
| `position` | `table` | required | — |
| `options` | `table` | optional | — |

Returns: `integer handle, or nil, reason`

```lua
-- A looping street concert everyone hears at the same point of the track
local concertId = Open77.audio.play3D("audio/concert.mp3", { x = -1590.0, y = 390.0, z = 25.0 }, {
    loop = true, duration = 240, maxDistance = 60,
})
```

[Package audio: 2D & 3D guide](../../01%20Documentation/package-audio.md)

Registered in `LuaResourceRuntime.cs` (line 9872) as `LuaAudioCreate`.

## seek

```lua
Open77.audio.seek(id, seconds)
```

`NETWORK`: Uses the network backend.

SeekSound: server-authoritative network package audio.

Requires audio.network; network=false is rejected. Server owns the timeline, handles and bucket/optional-target audience. Clients cannot request broadcasts directly. getState reports intended playback, not decoder acknowledgement. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `seconds` | `number` | required | — |

Returns: `true, or failure, reason`

```lua
-- Jump the shared timeline to the second track
Open77.audio.seek(concertId, 180)
```

[Package audio: 2D & 3D guide](../../01%20Documentation/package-audio.md)

Registered in `LuaResourceRuntime.cs` (line 9878) as `LuaAudioControl`.

## setPosition

```lua
Open77.audio.setPosition(id, position)
```

`NETWORK`: Uses the network backend.

SetSoundPosition: server-authoritative network package audio.

Requires audio.network; network=false is rejected. Server owns the timeline, handles and bucket/optional-target audience. Clients cannot request broadcasts directly. getState reports intended playback, not decoder acknowledgement. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `position` | `table` | required | — |

Returns: `true, or failure, reason`

```lua
-- Move the shared sound with the parade float
Open77.audio.setPosition(paradeId, Open77.vehicles.getPosition(floatId))
```

[Package audio: 2D & 3D guide](../../01%20Documentation/package-audio.md)

Registered in `LuaResourceRuntime.cs` (line 9880) as `LuaAudioControl`.

## setVolume

```lua
Open77.audio.setVolume(id, volume)
```

`NETWORK`: Uses the network backend.

SetSoundVolume: server-authoritative network package audio.

Requires audio.network; network=false is rejected. Server owns the timeline, handles and bucket/optional-target audience. Clients cannot request broadcasts directly. getState reports intended playback, not decoder acknowledgement. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `volume` | `number` | required | — |

Returns: `true, or failure, reason`

```lua
-- Turn the concert down for everyone
Open77.audio.setVolume(concertId, 0.6)
```

[Package audio: 2D & 3D guide](../../01%20Documentation/package-audio.md)

Registered in `LuaResourceRuntime.cs` (line 9879) as `LuaAudioControl`.

## stop

```lua
Open77.audio.stop(id)
```

`NETWORK`: Uses the network backend.

StopSound: server-authoritative network package audio.

Requires audio.network; network=false is rejected. Server owns the timeline, handles and bucket/optional-target audience. Clients cannot request broadcasts directly. getState reports intended playback, not decoder acknowledgement. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `true, or failure, reason`

```lua
-- Stop the concert for everyone
Open77.audio.stop(concertId)
```

[Package audio: 2D & 3D guide](../../01%20Documentation/package-audio.md)

Registered in `LuaResourceRuntime.cs` (line 9877) as `LuaAudioControl`.
