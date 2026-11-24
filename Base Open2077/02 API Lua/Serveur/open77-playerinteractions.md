# Open77.playerInteractions (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-playerinteractions.md) · [Page web](https://open2077.net/docs/api/server/open77-playerinteractions)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

6 functions.

## cancel

```lua
Open77.playerInteractions.cancel(interactionId, [reason])
```

`NETWORK`: Uses the network backend.

Cancel an interaction owned by this resource.

Requires `players.interactions.control` -- the handler chooses the name per operation, and `players.interactions.read` covers only the inspection calls (`get`, `current`, `list`, `isReserved`); without it the answer is `nil, permission_denied:players.interactions.control`. Uses canonical player IDs. The coordinator reserves both players and cancels on death, disconnect, vehicle entry, bucket change, resource stop, timeout or presentation failure. Gameplay effects remain the resource's responsibility. See player-interactions.md. Custom reason accepts 1–64 ASCII alphanumeric/underscore characters; invalid reasons normalize to cancelled. Delayed callbacks should retain the interaction ID, never cancel whatever action happens to occupy the player later.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `interactionId` | `string` | required | — |
| `reason` | `string` | optional | — |

Returns: `terminal state or nil, reason`

```lua
-- The item vanished mid-handover: cancel the interaction with a reason
Open77.playerInteractions.cancel(interactionId, "item_removed")
```

[Player interactions guide](../../01%20Documentation/player-interactions.md#server-api)

Registered in `LuaResourceRuntime.cs` (line 7974) as `LuaResourceRuntime.Bootstrap`.

## current

```lua
Open77.playerInteractions.current(playerId)
```

`NETWORK`: Uses the network backend.

Read the interaction reserving a player, including an unanswered invitation.

Requires `players.interactions.read` -- the handler chooses the name per operation, and `players.interactions.control` is what `request` and `cancel` need instead; without it the answer is `nil, permission_denied:players.interactions.read`. Uses canonical player IDs. The coordinator reserves both players and cancels on death, disconnect, vehicle entry, bucket change, resource stop, timeout or presentation failure. Gameplay effects remain the resource's responsibility. See player-interactions.md. Terminal states are removed from the active registry. Read access never grants cancellation of another resource's action.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `active state or nil; nil, reason on rejection`

```lua
-- Refuse a trade while the player is already in an interaction
local busy = Open77.playerInteractions.current(playerId)
if busy then return Open77.chat.send(playerId, "Finish what you are doing first.") end
```

[Player interactions guide](../../01%20Documentation/player-interactions.md#server-api)

Registered in `LuaResourceRuntime.cs` (line 7971) as `LuaResourceRuntime.Bootstrap`.

## get

```lua
Open77.playerInteractions.get(interactionId)
```

`NETWORK`: Uses the network backend.

Read a copied active state by opaque interaction ID.

Requires `players.interactions.read` -- the handler chooses the name per operation, and `players.interactions.control` is what `request` and `cancel` need instead; without it the answer is `nil, permission_denied:players.interactions.read`. Uses canonical player IDs. The coordinator reserves both players and cancels on death, disconnect, vehicle entry, bucket change, resource stop, timeout or presentation failure. Gameplay effects remain the resource's responsibility. See player-interactions.md. Terminal states are removed from the active registry. Read access never grants cancellation of another resource's action.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `interactionId` | `string` | required | — |

Returns: `active state or nil; nil, reason on rejection`

```lua
-- Read the state of the interaction we started
local state = Open77.playerInteractions.get(interactionId)
if state then print(state.kind, state.phase) end
```

[Player interactions guide](../../01%20Documentation/player-interactions.md#server-api)

Registered in `LuaResourceRuntime.cs` (line 7970) as `LuaResourceRuntime.Bootstrap`.

## isReserved

```lua
Open77.playerInteractions.isReserved(playerId)
```

`NETWORK`: Uses the network backend.

Check whether a player is exclusively reserved by this service.

Requires `players.interactions.read` -- the handler chooses the name per operation, and `players.interactions.control` is what `request` and `cancel` need instead; without it the answer is `nil, permission_denied:players.interactions.read`. Uses canonical player IDs. The coordinator reserves both players and cancels on death, disconnect, vehicle entry, bucket change, resource stop, timeout or presentation failure. Gameplay effects remain the resource's responsibility. See player-interactions.md. Terminal states are removed from the active registry. Read access never grants cancellation of another resource's action.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `boolean; nil, reason on rejection`

```lua
-- A reserved player cannot be teleported by the event script
if Open77.playerInteractions.isReserved(playerId) then return end
```

[Player interactions guide](../../01%20Documentation/player-interactions.md#server-api)

Registered in `LuaResourceRuntime.cs` (line 7972) as `LuaResourceRuntime.Bootstrap`.

## list

```lua
Open77.playerInteractions.list()
```

`NETWORK`: Uses the network backend.

List active interactions owned by this resource only.

Requires `players.interactions.read` -- the handler chooses the name per operation, and `players.interactions.control` is what `request` and `cancel` need instead; without it the answer is `nil, permission_denied:players.interactions.read`. Uses canonical player IDs. The coordinator reserves both players and cancels on death, disconnect, vehicle entry, bucket change, resource stop, timeout or presentation failure. Gameplay effects remain the resource's responsibility. See player-interactions.md. Terminal states are removed from the active registry. Read access never grants cancellation of another resource's action.

Returns: `state[]; nil, reason on rejection`

```lua
-- /interactions: what is this resource coordinating right now?
RegisterCommand("interactions", function(source)
    for _, state in ipairs(Open77.playerInteractions.list() or {}) do
        Open77.chat.send(source, ("%s: %d -> %d (%s)"):format(state.kind, state.actorId, state.targetId, state.phase))
    end
end, true)
```

[Player interactions guide](../../01%20Documentation/player-interactions.md#server-api)

Registered in `LuaResourceRuntime.cs` (line 7973) as `LuaResourceRuntime.Bootstrap`.

## request

```lua
Open77.playerInteractions.request(actorId, targetId, kind, [options])
```

`NETWORK`: Uses the network backend.

Reserve and coordinate a two-player interaction.

Requires `players.interactions.control` -- the handler chooses the name per operation, and `players.interactions.read` covers only the inspection calls (`get`, `current`, `list`, `isReserved`); without it the answer is `nil, permission_denied:players.interactions.control`. Uses canonical player IDs. The coordinator reserves both players and cancels on death, disconnect, vehicle entry, bucket change, resource stop, timeout or presentation failure. Gameplay effects remain the resource's responsibility. See player-interactions.md. Kinds: give, heal, carry, escort, custom. Options: durationMs (500–600000, default4000), startDistance (.25–10, default3), breakDistance (startDistance–20, default5), inviteTimeoutMs (1000–60000, default15000), consent (defaulttrue), optional actorAnimation/targetAnimation RP profiles for stationary kinds. Carry/escort reject profile overrides. Both players must be alive, on foot, near and in the same bucket. Acceptance is not proof of native playback; observe lifecycle events.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `actorId` | `integer` | required | — |
| `targetId` | `integer` | required | — |
| `kind` | `string` | required | — |
| `options` | `table` | optional | — |

Returns: `state table or nil, reason`

```lua
-- /give <id>: hand an item over with a four-second synchronized animation
RegisterCommand("give", function(source, args)
    local target = tonumber(args[1])
    if not target then return end
    local state, reason = Open77.playerInteractions.request(source, target, "give", { durationMs = 4000 })
    if not state then Open77.chat.send(source, "Cannot give: " .. reason) end
end, false)
```

[Player interactions guide](../../01%20Documentation/player-interactions.md#server-api)

Registered in `LuaResourceRuntime.cs` (line 7965) as `LuaResourceRuntime.Bootstrap`.
