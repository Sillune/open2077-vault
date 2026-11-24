# Open77.voice (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-voice.md) · [Page web](https://open2077.net/docs/api/client/open77-voice)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

18 functions.

## devices

```lua
Open77.voice.devices([flow])
```

`CLIENT`: Client-only surface.

Enumerates active native voice endpoints.

Requires `voice.client`. `flow` is `input`, `output`, or `all`; the empty endpoint id represents the current Windows communications default.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `flow` | `string` | optional | `all` |

Returns: `array of { id, name, flow, default, selected, available }, or nil`, `reason`

```lua
-- Fill the settings dropdown with the microphones this machine has
for _, device in ipairs(Open77.voice.devices("input") or {}) do
    print(device.id, device.name, device.selected and "(selected)" or "")
end
```

[Client voice API guide](../../01%20Documentation/voice-client.md)

Registered in `ResourceHost.cpp` (line 24668) as `LuaVoiceDevices`.

## getLipSyncStatus

```lua
Open77.voice.getLipSyncStatus()
```

`CLIENT`: Client-only surface.

Reads native mouth-presentation policy and tracked-player counts.

Requires voice.client. Fields: enabled, sessionActive, mode=audio_envelope, trackedPlayers, availablePlayers, activePlayers. Not phoneme recognition.

Returns: `status table, or nil, reason`

```lua
local state, reason = Open77.voice.getLipSyncStatus()
if state then print(state.mode, state.activePlayers) else print(reason) end
```

[Voice lipsync guide](../../01%20Documentation/voice-lipsync.md)

Registered in `ResourceHost.cpp` (line 24684) as `LuaVoiceLipSyncStatus`.

## getPlayerLipSyncState

```lua
Open77.voice.getPlayerLipSyncState(playerId)
```

`CLIENT`: Client-only surface.

Reads a player's current facial binding and smoothed voice envelope.

Requires voice.client. Fields: playerId, entity, isLocal, enabled, available, active, level (0..1), reason. Unstreamed/custom unsupported heads report unavailable without changing the body animation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `state table, or nil, reason`

```lua
local state, reason = Open77.voice.getPlayerLipSyncState(42)
if state then print(state.available, state.level, state.reason) else print(reason) end
```

[Voice lipsync guide](../../01%20Documentation/voice-lipsync.md)

Registered in `ResourceHost.cpp` (line 24685) as `LuaVoicePlayerLipSyncState`.

## selectDevice

```lua
Open77.voice.selectDevice(flow, [endpointId])
```

`CLIENT`: Client-only surface.

Selects a microphone or voice output endpoint.

Requires `voice.client`. `flow` is `input` or `output`; an empty id follows the Windows communications default.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `flow` | `string` | required | — |
| `endpointId` | `string` | optional | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- The player picked a microphone in the settings page
RegisterNetEvent("settings:microphone", function(endpointId)
    Open77.voice.selectDevice("input", endpointId)
end)
```

[Client voice API guide](../../01%20Documentation/voice-client.md)

Registered in `ResourceHost.cpp` (line 24671) as `LuaVoiceSelectDevice`.

## setCaptureEnabled

```lua
Open77.voice.setCaptureEnabled(enabled)
```

`CLIENT`: Client-only surface.

Enables or disables native microphone capture.

Requires `voice.client`. Capture alone does not select recipients or bypass PTT/server policy.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- A "mute microphone" toggle key
local muted = false
RegisterKeyMapping("voice_mute", "Mute microphone", "M", function()
    muted = not muted
    Open77.voice.setCaptureEnabled(not muted)
    Open77.hud.notify(muted and "Microphone muted" or "Microphone on")
end)
```

[Client voice API guide](../../01%20Documentation/voice-client.md)

Registered in `ResourceHost.cpp` (line 24672) as `LuaVoiceBoolean<&VoiceBackend::setCaptureEnabled>`.

## setChannelVolume

```lua
Open77.voice.setChannelVolume(channelId, gain)
```

`CLIENT`: Client-only surface.

Sets local gain for a voice route/channel.

Requires `voice.client`. The native engine retains it for current and future streams using that route id.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `channelId` | `integer|string` | required | — |
| `gain` | `number 0..2` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Turn the radio channel down while driving
Open77.voice.setChannelVolume("ncpd_dispatch", 0.5)
```

[Client voice API guide](../../01%20Documentation/voice-client.md)

Registered in `ResourceHost.cpp` (line 24681) as `LuaVoiceChannelVolume`.

## setInputVolume

```lua
Open77.voice.setInputVolume(gain)
```

`CLIENT`: Client-only surface.

Sets microphone gain.

Requires `voice.client`. Gain is clamped before the native capture frame is metered and encoded; the pause package persists it per server.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `gain` | `number 0..2` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- A microphone gain slider (0..2)
RegisterNetEvent("settings:micGain", function(gain)
    Open77.voice.setInputVolume(gain)
end)
```

[Client voice API guide](../../01%20Documentation/voice-client.md)

Registered in `ResourceHost.cpp` (line 24675) as `LuaVoiceGain<&VoiceBackend::setInputVolume>`.

## setLipSyncEnabled

```lua
Open77.voice.setLipSyncEnabled(enabled)
```

`CLIENT`: Client-only surface.

Enables or suppresses local voice-driven mouth presentation.

Requires voice.client. False adds this resource's global suppression; true removes only its own suppression. Resource stop/reload/failure releases it. Does not mute audio. See the Voice lipsync guide for setup and client-build requirements.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
-- The helmet hides the face: no mouth animation while it is on
AddEventHandler("helmet:on", function() Open77.voice.setLipSyncEnabled(false) end)
AddEventHandler("helmet:off", function() Open77.voice.setLipSyncEnabled(true) end)
```

[Voice lipsync guide](../../01%20Documentation/voice-lipsync.md)

Registered in `ResourceHost.cpp` (line 24682) as `LuaVoiceLipSyncEnabled<false>`.

## setMicrophoneTest

```lua
Open77.voice.setMicrophoneTest(enabled)
```

`CLIENT`: Client-only surface.

Enables local microphone loopback.

Requires `voice.client`. This is a bounded local monitor and does not transmit by itself.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- The "test your microphone" button in the settings page
Open77.voice.setMicrophoneTest(true)
SetTimeout(5000, function() Open77.voice.setMicrophoneTest(false) end)
```

[Client voice API guide](../../01%20Documentation/voice-client.md)

Registered in `ResourceHost.cpp` (line 24674) as `LuaVoiceBoolean<&VoiceBackend::setMicrophoneTest>`.

## setOutputVolume

```lua
Open77.voice.setOutputVolume(gain)
```

`CLIENT`: Client-only surface.

Sets the global remote-voice gain.

Requires `voice.client`. This is local rendering policy and does not affect what any other listener hears.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `gain` | `number 0..2` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Halve everybody's voice during the cutscene
Open77.voice.setOutputVolume(0.5)
AddEventHandler("cutscene:ended", function() Open77.voice.setOutputVolume(1.0) end)
```

[Client voice API guide](../../01%20Documentation/voice-client.md)

Registered in `ResourceHost.cpp` (line 24676) as `LuaVoiceGain<&VoiceBackend::setOutputVolume>`.

## setPlayerBlocked

```lua
Open77.voice.setPlayerBlocked(playerId, blocked)
```

`CLIENT`: Client-only surface.

Locally blocks or unblocks one remote talker.

This affects only local playback. Use the server mute/deaf APIs for authoritative policy.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `blocked` | `boolean` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- A local block from the scoreboard's "mute" button
RegisterNetEvent("scoreboard:blockVoice", function(playerId, blocked)
    Open77.voice.setPlayerBlocked(playerId, blocked)
end)
```

[Client voice API guide](../../01%20Documentation/voice-client.md)

Registered in `ResourceHost.cpp` (line 24680) as `LuaVoicePlayerBlocked`.

## setPlayerLipSyncEnabled

```lua
Open77.voice.setPlayerLipSyncEnabled(playerId, enabled)
```

`CLIENT`: Client-only surface.

Enables or suppresses voice-driven lipsync for one network player.

Requires voice.client. Positive network player ID, not entity ID. True cannot override another resource's suppression or a global suppression. Per-player policies reset across sessions.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
local ok, reason = Open77.voice.setPlayerLipSyncEnabled(42, false)
if not ok then print(reason) end
```

[Voice lipsync guide](../../01%20Documentation/voice-lipsync.md)

Registered in `ResourceHost.cpp` (line 24683) as `LuaVoiceLipSyncEnabled<true>`.

## setPlayerVolume

```lua
Open77.voice.setPlayerVolume(playerId, gain)
```

`CLIENT`: Client-only surface.

Sets one remote player's local playback gain.

Requires `voice.client`. The preference is applied to the native jitter/render path only; it cannot change server reachability.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `gain` | `number 0..2` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Turn one loud player down locally
Open77.voice.setPlayerVolume(playerId, 0.4)
```

[Client voice API guide](../../01%20Documentation/voice-client.md)

Registered in `ResourceHost.cpp` (line 24679) as `LuaVoicePlayerVolume`.

## setProximityDistance

```lua
Open77.voice.setProximityDistance(distance)
```

`CLIENT`: Client-only surface.

Requests a personal proximity-voice reach.

Requires `voice.client`. Queues a reliable request only; the authenticated server clamps it to the advertised minimum/maximum and republishes the canonical value in `status().proximityDistance`. Proximity remains permanently enabled unless server policy disables it; this method cannot change enabled, mute, deaf, buckets, memberships, or recipients.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `distance` | `number 0..1000` | required | — |

Returns: `true when queued or already canonical, otherwise false`, `reason`

```lua
-- Whisper and shout keys change how far the player is heard
RegisterKeyMapping("voice_whisper", "Whisper", "PAGEDOWN", function() Open77.voice.setProximityDistance(3) end)
RegisterKeyMapping("voice_shout", "Shout", "PAGEUP", function() Open77.voice.setProximityDistance(40) end)
```

[Client voice API guide](../../01%20Documentation/voice-client.md)

Registered in `ResourceHost.cpp` (line 24678) as `LuaVoiceSetProximityDistance`.

## setTransmitting

```lua
Open77.voice.setTransmitting(enabled, [intent])
```

`CLIENT`: Client-only surface.

Sets PTT state and requested route intent.

Requires `voice.client`. Intent may be `proximity`, `channels`, `channel:<id>`, or `all`. `channels` selects up to four authorized speakable memberships and `all` adds proximity. The server still validates every recipient and membership.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |
| `intent` | `string` | optional | `proximity` |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Push-to-talk into the radio channel while the key is held
Open77.input.registerKeyMapping({
    id = "radio_ptt", name = "Radio", key = "CAPSLOCK", hold = true,
    onPressed = function() Open77.voice.setTransmitting(true, "channels") end,
    onReleased = function() Open77.voice.setTransmitting(false) end,
})
```

[Client voice API guide](../../01%20Documentation/voice-client.md)

Registered in `ResourceHost.cpp` (line 24673) as `LuaVoiceSetTransmitting`.

## setVoiceActivation

```lua
Open77.voice.setVoiceActivation(enabled, [threshold])
```

`CLIENT`: Client-only surface.

Enables VAD and sets its normalized threshold.

Requires `voice.client`. Threshold `0` is most sensitive and `1` least sensitive. The native gate uses a short hangover, emits an explicit talkspurt end, and sends no silent talkspurt frames; held PTT may still use bounded Opus DTX.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |
| `threshold` | `number 0..1` | optional | `0.25` |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Voice activation instead of push-to-talk, medium sensitivity
Open77.voice.setVoiceActivation(true, 0.4)
```

[Client voice API guide](../../01%20Documentation/voice-client.md)

Registered in `ResourceHost.cpp` (line 24677) as `LuaVoiceActivation`.

## status

```lua
Open77.voice.status()
```

`CLIENT`: Client-only surface.

Returns one native voice capture/render snapshot.

Requires `voice.client`. Includes device ids, gains, input/output levels, PTT/VAD state, packet counters, active talker count, backend states, and the local canonical proximity reach. `proximityDistance` is server-confirmed; `requestedProximityDistance` and `proximityRequestPending` describe an in-flight request. The server-advertised default/minimum/maximum bounds are read-only. It never includes audio payloads.

Returns: `table, or nil`, `reason`

```lua
-- A microphone level meter on the HUD
SetTick(function()
    local voice = Open77.voice.status()
    if voice then micMeter.set(voice.inputLevel) end
end)
```

[Client voice API guide](../../01%20Documentation/voice-client.md)

Registered in `ResourceHost.cpp` (line 24669) as `LuaVoiceStatus`.

## talkers

```lua
Open77.voice.talkers()
```

`CLIENT`: Client-only surface.

Lists native remote-talker playback state.

Requires `voice.client`. Each entry contains player id, speaking/level state, local volume/block preference, and authorized active routes.

Returns: `array of talker snapshots, or nil`, `reason`

```lua
-- Show a speaker icon over everyone who is talking
for _, talker in ipairs(Open77.voice.talkers() or {}) do
    if talker.speaking then showSpeakerIcon(talker.playerId) end
end
```

[Client voice API guide](../../01%20Documentation/voice-client.md)

Registered in `ResourceHost.cpp` (line 24670) as `LuaVoiceTalkers`.
