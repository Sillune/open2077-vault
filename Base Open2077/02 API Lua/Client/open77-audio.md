# Open77.audio (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-audio.md) · [Page web](https://open2077.net/docs/api/client/open77-audio)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

10 functions.

## destroy

```lua
Open77.audio.destroy(id)
```

`GAME`: Requires a live game instance.

DestroySound: local package audio.

Requires audio.play; network=true is rejected. MP3/PCM WAV decode asynchronously off the game thread. getState reports actual decoder state and cached PCM reuse. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `true, or failure, reason`

```lua
-- Free the engine loop when the player gets out of the car
Open77.audio.destroy(engineLoop)
engineLoop = nil
```

[Package audio: 2D & 3D guide](../../01%20Documentation/package-audio.md)

Registered in `ResourceHost.cpp` (line 12183) as `LuaAudioControl`.

## getState

```lua
Open77.audio.getState(id)
```

`GAME`: Requires a live game instance.

GetSoundState: local package audio.

Requires audio.play; network=true is rejected. MP3/PCM WAV decode asynchronously off the game thread. getState reports actual decoder state and cached PCM reuse. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `table, or nil, reason`

```lua
-- Restart the siren if it stopped
local state = Open77.audio.getState(sirenId)
if state and state.status == "stopped" then Open77.audio.play(sirenId) end
```

[Package audio: 2D & 3D guide](../../01%20Documentation/package-audio.md)

Registered in `ResourceHost.cpp` (line 12287) as `LuaAudioState`.

## pause

```lua
Open77.audio.pause(id)
```

`GAME`: Requires a live game instance.

PauseSound: local package audio.

Requires audio.play; network=true is rejected. MP3/PCM WAV decode asynchronously off the game thread. getState reports actual decoder state and cached PCM reuse. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `true, or failure, reason`

```lua
-- Pause the radio while the phone rings
AddEventHandler("phone:ringing", function()
    Open77.audio.pause(radioId)
end)
```

[Package audio: 2D & 3D guide](../../01%20Documentation/package-audio.md)

Registered in `ResourceHost.cpp` (line 12205) as `LuaAudioControl`.

## play

```lua
Open77.audio.play(id)
```

`GAME`: Requires a live game instance.

PlaySound: local package audio.

Requires audio.play; network=true is rejected. MP3/PCM WAV decode asynchronously off the game thread. getState reports actual decoder state and cached PCM reuse. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `true, or failure, reason`

```lua
-- Resume the radio the phone call paused
AddEventHandler("phone:hungUp", function()
    Open77.audio.play(radioId)
end)
```

[Package audio: 2D & 3D guide](../../01%20Documentation/package-audio.md)

Registered in `ResourceHost.cpp` (line 12205) as `LuaAudioControl`.

## play2D

```lua
Open77.audio.play2D(asset, [options])
```

`GAME`: Requires a live game instance.

Play2DSound: local package audio.

Requires audio.play; network=true is rejected. MP3/PCM WAV decode asynchronously off the game thread. getState reports actual decoder state and cached PCM reuse. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `asset` | `string` | required | — |
| `options` | `table` | optional | — |

Returns: `integer handle, or nil, reason`

```lua
-- A UI click on every menu selection
AddEventHandler("menu:selected", function()
    Open77.audio.play2D("audio/click.wav", { volume = 0.5 })
end)
```

[Package audio: 2D & 3D guide](../../01%20Documentation/package-audio.md)

Registered in `ResourceHost.cpp` (line 12278) as `LuaAudioCreate`.

## play3D

```lua
Open77.audio.play3D(asset, position, [options])
```

`GAME`: Requires a live game instance.

Play3DSound: local package audio.

Requires audio.play; network=true is rejected. MP3/PCM WAV decode asynchronously off the game thread. getState reports actual decoder state and cached PCM reuse. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `asset` | `string` | required | — |
| `position` | `table` | required | — |
| `options` | `table` | optional | — |

Returns: `integer handle, or nil, reason`

```lua
-- A ringing phone at the booth, audible within 15 m
local ring = Open77.audio.play3D("audio/ring.mp3", { x = -1442.2, y = 127.4, z = 18.0 }, { maxDistance = 15, loop = true })
```

[Package audio: 2D & 3D guide](../../01%20Documentation/package-audio.md)

Registered in `ResourceHost.cpp` (line 12278) as `LuaAudioCreate`.

## seek

```lua
Open77.audio.seek(id, seconds)
```

`GAME`: Requires a live game instance.

SeekSound: local package audio.

Requires audio.play; network=true is rejected. MP3/PCM WAV decode asynchronously off the game thread. getState reports actual decoder state and cached PCM reuse. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `seconds` | `number` | required | — |

Returns: `true, or failure, reason`

```lua
-- Skip the intro of the track
Open77.audio.seek(musicId, 12.5)
```

[Package audio: 2D & 3D guide](../../01%20Documentation/package-audio.md)

Registered in `ResourceHost.cpp` (line 12206) as `LuaAudioControl`.

## setPosition

```lua
Open77.audio.setPosition(id, position)
```

`GAME`: Requires a live game instance.

SetSoundPosition: local package audio.

Requires audio.play; network=true is rejected. MP3/PCM WAV decode asynchronously off the game thread. getState reports actual decoder state and cached PCM reuse. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `position` | `table` | required | — |

Returns: `true, or failure, reason`

```lua
-- Keep the boombox sound following the carried prop
SetTick(function()
    Open77.audio.setPosition(boomboxSound, vector3(Open77.character.position()))
end)
```

[Package audio: 2D & 3D guide](../../01%20Documentation/package-audio.md)

Registered in `ResourceHost.cpp` (line 12281) as `LuaAudioControl`.

## setVolume

```lua
Open77.audio.setVolume(id, volume)
```

`GAME`: Requires a live game instance.

SetSoundVolume: local package audio.

Requires audio.play; network=true is rejected. MP3/PCM WAV decode asynchronously off the game thread. getState reports actual decoder state and cached PCM reuse. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `volume` | `number` | required | — |

Returns: `true, or failure, reason`

```lua
-- Fade the music out over two seconds
CreateThread(function()
    for v = 10, 0, -1 do
        Open77.audio.setVolume(musicId, v / 10)
        Wait(200)
    end
    Open77.audio.stop(musicId)
end)
```

[Package audio: 2D & 3D guide](../../01%20Documentation/package-audio.md)

Registered in `ResourceHost.cpp` (line 12281) as `LuaAudioControl`.

## stop

```lua
Open77.audio.stop(id)
```

`GAME`: Requires a live game instance.

StopSound: local package audio.

Requires audio.play; network=true is rejected. MP3/PCM WAV decode asynchronously off the game thread. getState reports actual decoder state and cached PCM reuse. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `true, or failure, reason`

```lua
-- Stop the alarm when the player leaves the building
AddEventHandler("building:left", function()
    Open77.audio.stop(alarmId)
end)
```

[Package audio: 2D & 3D guide](../../01%20Documentation/package-audio.md)

Registered in `ResourceHost.cpp` (line 12280) as `LuaAudioControl`.
