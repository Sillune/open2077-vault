# Open77.sound (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-sound.md) · [Page web](https://open2077.net/docs/api/server/open77-sound)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

6 functions.

## broadcast

```lua
Open77.sound.broadcast(file, [options])
```

`NETWORK`: Uses the network backend.

Plays a file for every player on the server.

`play` with a target of `-1`, which leaves as ONE event rather than one per player. Use the `{ near = ..., radius = ... }` target instead for anything positional: a district's shop jingle broadcast to the whole server is on every player's wire in every bucket.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `file` | `string (declared path)` | required | — |
| `options` | `table` | optional | — |

Returns: `the sound id, or false`, `reason`

```lua
-- Curfew siren for everyone, quietly
Open77.sound.broadcast("sfx/curfew.ogg", { volume = 0.4 })
```

[Resource audio guide](../../01%20Documentation/sound.md)

Registered in `LuaResourceRuntime.cs` (line 8246) as `LuaResourceRuntime.Bootstrap`.

## play

```lua
Open77.sound.play(target, file, [options])
```

`NETWORK`: Uses the network backend.

Plays an audio file this resource ships, for one player, for everyone, or for everyone in range.

The server never touches audio: it names a file and an audience, and the bundled `open77_sound` client resource decodes and mixes it. `target` takes three shapes: a player id; `-1` for everyone; or a table `{ near = playerId | {x,y,z}, radius = n, bucket = n | false, limit = n }`, which is resolved HERE through `Open77.players.nearby` and sent to each player found. `radius` is required in that form -- a `near` without one would silently reach the whole bucket. The bucket rule is the one every proximity read uses: the anchor player's own bucket by default, so two players in different routing buckets never hear each other. The file must be declared in THIS resource's `files` manifest entry and must reach the client, so the resource needs a client script too -- a server-only resource is never downloaded and its audio is unreachable, which the client reports on the local `open77:soundFailed` event. `options` is the client option table (`volume`, `loop`, `pitch`, `position`, `entity`, `maxDistance`, `refDistance`, `offset`) plus an optional `id`, which defaults to the file path. See [sound](https://open2077.net/docs/api/server/sound.md) for the spatial model and for the honest list of what this is not.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `integer|table` | required | — |
| `file` | `string (declared path)` | required | — |
| `options` | `table` | optional | — |

Returns: `the sound id, or false`, `reason: invalid_sound_target, invalid_sound_radius, invalid_sound_file, invalid_sound_id, invalid_options, no_audience, permission_denied:network.events, network_unavailable, sound_payload_too_large`

```lua
-- A gunshot everyone within 40 m of the shooter hears
Open77.sound.play({ near = source, radius = 40.0 }, "sfx/shot.ogg")
```

[Resource audio guide](../../01%20Documentation/sound.md)

Registered in `LuaResourceRuntime.cs` (line 8245) as `LuaResourceRuntime.Bootstrap`.

## setPosition

```lua
Open77.sound.setPosition(target, id, position)
```

`NETWORK`: Uses the network backend.

Moves one of this resource's spatial sounds to a new world point.

Refused client-side with `sound_not_spatial` on a sound started without `position` or `entity`. Sending a position at, say, 5 Hz is how a siren follows a car the server owns; for a body the client can see, start the sound with `entity` instead and let the client track it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `integer|table` | required | — |
| `id` | `string` | required | — |
| `position` | `table {x,y,z}` | required | — |

Returns: `true, or false`, `reason`

```lua
-- The siren follows the server-owned cruiser, five times a second
CreateThread(function()
    while chasing do
        Open77.sound.setPosition(-1, "siren", Open77.vehicles.getPosition(cruiserId))
        Wait(200)
    end
end)
```

[Resource audio guide](../../01%20Documentation/sound.md)

Registered in `LuaResourceRuntime.cs` (line 8254) as `LuaResourceRuntime.Bootstrap`.

## setVolume

```lua
Open77.sound.setVolume(target, id, volume)
```

`NETWORK`: Uses the network backend.

Changes a live sound's volume on an audience.

0..1, ramped client-side so it does not click. On a spatial sound it multiplies the distance falloff. It is NOT the player's in-game audio slider, which this system cannot read.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `integer|table` | required | — |
| `id` | `string` | required | — |
| `volume` | `number (0..1)` | required | — |

Returns: `true, or false`, `reason: invalid_volume, ...`

```lua
-- Turn the boombox down for everyone
Open77.sound.setVolume(-1, "boombox", 0.2)
```

[Resource audio guide](../../01%20Documentation/sound.md)

Registered in `LuaResourceRuntime.cs` (line 8248) as `LuaResourceRuntime.Bootstrap`.

## stop

```lua
Open77.sound.stop(target, id)
```

`NETWORK`: Uses the network backend.

Stops one of this resource's sounds on an audience.

Same target shapes as `play`. A client only stops a sound this resource started there: the owner is stamped by the host from the calling resource's name and `open77:sound:` is a reserved client-event prefix, so one resource can never stop another's audio.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `integer|table` | required | — |
| `id` | `string` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Hang up: stop the ringing on that one player
Open77.sound.stop(source, "phone")
```

[Resource audio guide](../../01%20Documentation/sound.md)

Registered in `LuaResourceRuntime.cs` (line 8247) as `LuaResourceRuntime.Bootstrap`.

## stopAll

```lua
Open77.sound.stopAll([target])
```

`NETWORK`: Uses the network backend.

Stops every sound this resource started, for an audience or for everyone.

The target defaults to `-1`. It is also sent automatically when this resource stops, so a looping siren cannot outlive the resource that started it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `integer|table` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- The event ends: silence everything this resource started
Open77.sound.stopAll()
```

[Resource audio guide](../../01%20Documentation/sound.md)

Registered in `LuaResourceRuntime.cs` (line 8258) as `LuaResourceRuntime.Bootstrap`.
