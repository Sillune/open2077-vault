# Open77.voice (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-voice.md) · [Page web](https://open2077.net/docs/api/server/open77-voice)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

18 functions.

## addPlayer

```lua
Open77.voice.addPlayer(channelId, playerId, [options])
```

`NETWORK`: Uses the network backend.

Adds a player to a voice channel.

Requires `voice.manage`. `options.canSpeak` and `options.canListen` both default to **true**; pass `false` explicitly for a listen-only or speak-only membership. This is the function name -- `addChannelPlayer` does not exist, and calling it inside a `pcall` is how a squad channel silently ends up with no members.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `channelId` | `string` | required | — |
| `playerId` | `integer|string` | required | — |
| `options` | `table { canSpeak?, canListen? }` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- An officer joins the dispatch channel when they clock in
RegisterNetEvent("police:onDuty", function()
    Open77.voice.addPlayer(dispatchChannel.id, source, { canSpeak = true, canListen = true })
end)
```

[Voice channels guide](../../01%20Documentation/voice-channels.md)

Registered in `LuaResourceRuntime.cs` (line 8185) as `LuaResourceRuntime.Bootstrap`.

## channels

```lua
Open77.voice.channels()
```

`NETWORK`: Uses the network backend.

Every canonical voice channel.

Requires `voice.manage`. Covers radio, phone, party, admin and spatial channels alike -- they differ by configuration, not by kind.

Returns: `array of channels, or nil`, `reason`

```lua
-- /voicechannels: list every channel for an admin
RegisterCommand("voicechannels", function(source)
    for _, channel in ipairs(Open77.voice.channels() or {}) do
        Open77.chat.send(source, ("%s (%s)"):format(channel.name, channel.mode))
    end
end, true)
```

[Voice channels guide](../../01%20Documentation/voice-channels.md)

Registered in `LuaResourceRuntime.cs` (line 8168) as `LuaResourceRuntime.Bootstrap`.

## createChannel

```lua
Open77.voice.createChannel(options)
```

`NETWORK`: Uses the network backend.

Creates a resource-owned voice channel.

Requires `voice.manage`. `options` is required and must be a table. Effects include `gain`, `lowPassHz`, `highPassHz`, `distortion`, `radioNoise`, `spatialBlend` and the reverb controls `reverbWet`, `reverbRoomSize`, `reverbDecay`, `reverbDamping` and `reverbPreDelayMs`. A non-persistent channel is released automatically with its owning resource. See [Integrated voice chat](https://open2077.net/docs/api/server/voice.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table` | required | — |

Returns: `channel, or nil`, `reason: options_required among others`

```lua
-- A police radio channel with a radio-style effect
local dispatch, reason = Open77.voice.createChannel({
    name = "NCPD dispatch",
    mode = "radio",
    effect = { highPassHz = 220.0, lowPassHz = 4800.0, radioNoise = 0.04 },
})
if not dispatch then print("no radio: " .. reason) end
```

[Voice channels guide](../../01%20Documentation/voice-channels.md)

Registered in `LuaResourceRuntime.cs` (line 8172) as `LuaResourceRuntime.Bootstrap`.

## getChannel

```lua
Open77.voice.getChannel(channelId)
```

`NETWORK`: Uses the network backend.

Reads one voice channel.

Requires `voice.manage`. Answers `nil` for an unknown channel id.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `channelId` | `string` | required | — |

Returns: `channel, or nil`, `reason`

```lua
-- Is the party channel still there after a reload?
local party = Open77.voice.getChannel(partyChannelId)
if not party then party = Open77.voice.createChannel({ name = "Party", mode = "party" }) end
```

[Voice channels guide](../../01%20Documentation/voice-channels.md)

Registered in `LuaResourceRuntime.cs` (line 8169) as `LuaResourceRuntime.Bootstrap`.

## getParticipant

```lua
Open77.voice.getParticipant(playerId)
```

`NETWORK`: Uses the network backend.

Reads one player's voice participation.

Requires `voice.manage`. Carries proximity settings, server mute and deaf policy, and the channels they belong to.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `participant, or nil`, `reason`

```lua
-- Tell an admin what a player's voice state is
RegisterCommand("voiceinfo", function(source, args)
    local participant = Open77.voice.getParticipant(tonumber(args[1]))
    if participant then Open77.chat.send(source, ("muted=%s deaf=%s"):format(tostring(participant.muted), tostring(participant.deaf))) end
end, true)
```

[Voice channels guide](../../01%20Documentation/voice-channels.md)

Registered in `LuaResourceRuntime.cs` (line 8171) as `LuaResourceRuntime.Bootstrap`.

## participants

```lua
Open77.voice.participants()
```

`NETWORK`: Uses the network backend.

Every voice participant, with proximity, mute/deaf and memberships.

Requires `voice.manage`. The server's own view, so it answers for a player whose client has not reported anything.

Returns: `array of participants, or nil`, `reason`

```lua
-- Count how many players are server-muted
local muted = 0
for _, participant in ipairs(Open77.voice.participants() or {}) do
    if participant.muted then muted = muted + 1 end
end
```

[Voice channels guide](../../01%20Documentation/voice-channels.md)

Registered in `LuaResourceRuntime.cs` (line 8170) as `LuaResourceRuntime.Bootstrap`.

## removeChannel

```lua
Open77.voice.removeChannel(channelId)
```

`NETWORK`: Uses the network backend.

Removes a voice channel.

Requires `voice.manage`. Members are dropped with it; they are not moved to another channel.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `channelId` | `string` | required | — |

Returns: `true, or false`, `reason`

```lua
-- The heist crew disbands: drop its channel
Open77.voice.removeChannel(crewChannelId)
```

[Voice channels guide](../../01%20Documentation/voice-channels.md)

Registered in `LuaResourceRuntime.cs` (line 8182) as `LuaResourceRuntime.Bootstrap`.

## removePlayer

```lua
Open77.voice.removePlayer(channelId, playerId)
```

`NETWORK`: Uses the network backend.

Removes a player from a voice channel.

Requires `voice.manage`. Their other channel memberships and their proximity setting are untouched.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `channelId` | `string` | required | — |
| `playerId` | `integer|string` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Clocking off leaves the dispatch channel
RegisterNetEvent("police:offDuty", function()
    Open77.voice.removePlayer(dispatchChannel.id, source)
end)
```

[Voice channels guide](../../01%20Documentation/voice-channels.md)

Registered in `LuaResourceRuntime.cs` (line 8192) as `LuaResourceRuntime.Bootstrap`.

## setChannelPlayerMuted

```lua
Open77.voice.setChannelPlayerMuted(channelId, playerId, [muted])
```

`NETWORK`: Uses the network backend.

Mutes one player in one channel.

Requires `voice.manage`. `muted` defaults to true when omitted. Scoped to that channel: the player still speaks everywhere else, unlike `setPlayerMuted`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `channelId` | `string` | required | — |
| `playerId` | `integer|string` | required | — |
| `muted` | `boolean` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- The dispatcher silences one officer on the radio only
Open77.voice.setChannelPlayerMuted(dispatchChannel.id, officerId, true)
```

[Voice channels guide](../../01%20Documentation/voice-channels.md)

Registered in `LuaResourceRuntime.cs` (line 8195) as `LuaResourceRuntime.Bootstrap`.

## setChannelPlayerPermissions

```lua
Open77.voice.setChannelPlayerPermissions(channelId, playerId, [options])
```

`NETWORK`: Uses the network backend.

Sets a member's speak and listen rights independently.

Requires `voice.manage`. Both `canSpeak` and `canListen` default to true when omitted, so passing an empty table restores full membership rights.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `channelId` | `string` | required | — |
| `playerId` | `integer|string` | required | — |
| `options` | `table { canSpeak?, canListen? }` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- A trainee may listen to dispatch but not speak
Open77.voice.setChannelPlayerPermissions(dispatchChannel.id, traineeId, { canSpeak = false, canListen = true })
```

[Voice channels guide](../../01%20Documentation/voice-channels.md)

Registered in `LuaResourceRuntime.cs` (line 8200) as `LuaResourceRuntime.Bootstrap`.

## setDefaultProximityDistance

```lua
Open77.voice.setDefaultProximityDistance(distance, [applyToExisting])
```

`NETWORK`: Uses the network backend.

Changes the default proximity reach.

Requires `voice.manage`. `applyToExisting` defaults to **false**, so by default this only affects participants who join afterwards; pass true to retune everyone who is already connected.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `distance` | `number (metres)` | required | — |
| `applyToExisting` | `boolean` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- A quieter RP server: 15 m reach for everyone, including those already online
Open77.voice.setDefaultProximityDistance(15, true)
```

[Voice channels guide](../../01%20Documentation/voice-channels.md)

Registered in `LuaResourceRuntime.cs` (line 8213) as `LuaResourceRuntime.Bootstrap`.

## setEnabled

```lua
Open77.voice.setEnabled([enabled])
```

`NETWORK`: Uses the network backend.

Turns voice on or off for the whole server.

Requires `voice.manage`. `enabled` defaults to true when omitted. Disabling stops relaying without removing channels or memberships, so re-enabling restores the topology.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- /voice off during the event briefing, /voice on after
RegisterCommand("voice", function(source, args)
    Open77.voice.setEnabled(args[1] ~= "off")
end, true)
```

[Voice channels guide](../../01%20Documentation/voice-channels.md)

Registered in `LuaResourceRuntime.cs` (line 8210) as `LuaResourceRuntime.Bootstrap`.

## setPlayerDeaf

```lua
Open77.voice.setPlayerDeaf(playerId, [deaf])
```

`NETWORK`: Uses the network backend.

Stops a player hearing anyone on the server.

Requires `voice.manage`. `deaf` defaults to true. The receive-side counterpart of `setPlayerMuted`, enforced at the relay.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `deaf` | `boolean` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- A "deafened" status effect for 30 seconds
Open77.voice.setPlayerDeaf(playerId, true)
SetTimeout(30000, function() Open77.voice.setPlayerDeaf(playerId, false) end)
```

[Voice channels guide](../../01%20Documentation/voice-channels.md)

Registered in `LuaResourceRuntime.cs` (line 8228) as `LuaResourceRuntime.Bootstrap`.

## setPlayerMuted

```lua
Open77.voice.setPlayerMuted(playerId, [muted])
```

`NETWORK`: Uses the network backend.

Stops a player being heard anywhere on the server.

Requires `voice.manage`. `muted` defaults to true. Server-wide send policy, enforced at the relay -- the client cannot talk its way past it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `muted` | `boolean` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- /vmute <id> for admins
RegisterCommand("vmute", function(source, args)
    local target = tonumber(args[1])
    if target then Open77.voice.setPlayerMuted(target, true) end
end, true)
```

[Voice channels guide](../../01%20Documentation/voice-channels.md)

Registered in `LuaResourceRuntime.cs` (line 8225) as `LuaResourceRuntime.Bootstrap`.

## setProximity

```lua
Open77.voice.setProximity(playerId, [options])
```

`NETWORK`: Uses the network backend.

Enables, disables or tunes one player's proximity reach.

Requires `voice.manage`. `options.enabled` defaults to true; `options.distance` overrides the server default for that player alone. Turning proximity off leaves their channel memberships working.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `options` | `table { enabled?, distance? }` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- A megaphone item: 80 m reach for one player
RegisterNetEvent("items:megaphone", function(on)
    Open77.voice.setProximity(source, { enabled = true, distance = on and 80 or 20 })
end)
```

[Voice channels guide](../../01%20Documentation/voice-channels.md)

Registered in `LuaResourceRuntime.cs` (line 8218) as `LuaResourceRuntime.Bootstrap`.

## setQuality

```lua
Open77.voice.setQuality(quality)
```

`NETWORK`: Uses the network backend.

Sets the global codec quality profile.

Requires `voice.manage`. A server-wide policy, not a per-player one: it trades bandwidth against fidelity for everybody.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `quality` | `string` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Save bandwidth on a full server
Open77.voice.setQuality("standard")
```

[Voice channels guide](../../01%20Documentation/voice-channels.md)

Registered in `LuaResourceRuntime.cs` (line 8207) as `LuaResourceRuntime.Bootstrap`.

## status

```lua
Open77.voice.status()
```

`NETWORK`: Uses the network backend.

Global voice quality, default proximity and relay counters.

Requires `voice.manage`. The dedicated server owns reachability: a client's requested scope never bypasses routing buckets, proximity, membership, mute/deaf, rate or quality checks, and the rejection counters here are where that shows.

Returns: `status table, or nil`, `reason`

```lua
-- Log the relay counters once a minute
CreateThread(function()
    while true do
        Wait(60000)
        local status = Open77.voice.status()
        if status then print("voice quality", status.quality, "default reach", status.proximityDistance) end
    end
end)
```

[Voice channels guide](../../01%20Documentation/voice-channels.md)

Registered in `LuaResourceRuntime.cs` (line 8167) as `LuaResourceRuntime.Bootstrap`.

## updateChannel

```lua
Open77.voice.updateChannel(channelId, [patch])
```

`NETWORK`: Uses the network backend.

Patches a voice channel.

Requires `voice.manage`. Sparse: the patch is merged with the channel id and only the fields present are applied, so one effect can be retuned without restating the rest.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `channelId` | `string` | required | — |
| `patch` | `table` | optional | — |

Returns: `channel, or nil`, `reason: patch_must_be_table among others`

```lua
-- Add more static to the radio without restating the rest
Open77.voice.updateChannel(dispatchChannel.id, { effect = { radioNoise = 0.1 } })
```

[Voice channels guide](../../01%20Documentation/voice-channels.md)

Registered in `LuaResourceRuntime.cs` (line 8176) as `LuaResourceRuntime.Bootstrap`.
