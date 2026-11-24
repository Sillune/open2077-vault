# Open77.chat (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-chat.md) · [Page web](https://open2077.net/docs/api/server/open77-chat)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

7 functions.

## addSuggestion

```lua
Open77.chat.addSuggestion(target, command, [help], [parameters])
```

`NETWORK`: Uses the network backend.

Publishes one slash-command suggestion to a player's chat.

Facade over `chat:addSuggestion`. `target` is a playerId or -1 for everyone; `command` is the command text (for example "/tow"), `help` a one-line description, `parameters` a list of { name, help } tables. The suggestion list is also what the admin menu harvests. Publishes on the host bus, so it answers `resource_preparing` at the top level of a script: call it from `AddEventHandler("onResourceStart", ...)`, a command or an event handler.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `integer` | required | — |
| `command` | `string` | required | — |
| `help` | `string` | optional | — |
| `parameters` | `table` | optional | — |

Returns: `true on success, otherwise false`, `failure reason`

```lua
-- Show /tow in the chat's autocomplete for everyone
Open77.chat.addSuggestion(-1, "/tow", "Tow the nearest vehicle", {
    { name = "vehicleId", help = "Optional vehicle id" },
})
```

[Chat & slash commands guide](../../01%20Documentation/chat.md)

Registered in `LuaResourceRuntime.Events.cs` (line 303) as `LuaResourceRuntime.Bootstrap`.

## addSuggestions

```lua
Open77.chat.addSuggestions(target, list)
```

`NETWORK`: Uses the network backend.

Publishes a list of slash-command suggestions at once.

Facade over `chat:addSuggestions`. `target` is a player id (a number) or `-1` for everyone; the list is an array of { command, help, parameters } tables, `parameters` being a list of { name, help } tables (FiveM's shape). Usually sent from a `chat:ready` handler (raised by each player's chat UI once it is up; `source` is that player) and once from `onResourceStart` with `-1`, for the players already connected. Publishes on the host bus, so it answers `resource_preparing` at the top level of a script.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `integer|-1` | required | — |
| `list` | `table` | required | — |

Returns: `true on success, otherwise false`, `failure reason`

```lua
-- Publish this resource's commands when a player's chat comes up
RegisterNetEvent("chat:ready", function()
    Open77.chat.addSuggestions(source, {
        { command = "/job", help = "Take a delivery job" },
        { command = "/park", help = "Store your car in the garage" },
    })
end)
```

[Chat & slash commands guide](../../01%20Documentation/chat.md)

Registered in `LuaResourceRuntime.Events.cs` (line 314) as `LuaResourceRuntime.Bootstrap`.

## broadcast

```lua
Open77.chat.broadcast(message)
```

`NETWORK`: Uses the network backend.

Sends a chat message to every player.

Exactly `Open77.chat.send(-1, message)`: the same reasons, the same asynchrony and the same ordering caveat, and the same `resource_preparing` answer at the top level of a script.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `message` | `string|table` | required | — |

Returns: `true on success, otherwise false`, `failure reason`

```lua
-- The round is starting in 30 seconds
Open77.chat.broadcast("Round starts in 30 seconds!")
```

[Chat & slash commands guide](../../01%20Documentation/chat.md)

Registered in `LuaResourceRuntime.Events.cs` (line 291) as `LuaResourceRuntime.Bootstrap`.

## clear

```lua
Open77.chat.clear(target)
```

`NETWORK`: Uses the network backend.

Clears a player's local chat history.

Facade over `chat:clear`. Presentation only -- it empties what that client is showing and deletes nothing on the server.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `integer` | required | — |

Returns: `true on success, otherwise false`, `failure reason`

```lua
-- Clear the chat of everyone in the arena at round start
for _, playerId in ipairs(Open77.players.inBucket(ARENA_BUCKET)) do
    Open77.chat.clear(playerId)
end
```

[Chat & slash commands guide](../../01%20Documentation/chat.md)

Registered in `LuaResourceRuntime.Events.cs` (line 327) as `LuaResourceRuntime.Bootstrap`.

## onMessage

```lua
Open77.chat.onMessage(handler)
```

`NETWORK`: Uses the network backend.

Observes every chat message the authority accepted.

Subscribes to `chat:message`, so the handler receives (playerId, playerName, text) AFTER the cancellable `chatMessage` veto has settled. It is therefore the wrong place to refuse a message: to do that, AddEventHandler('chatMessage', ...) and call CancelEvent(). Returns the handler id for RemoveEventHandler.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `handler` | `function` | required | — |

Returns: `handler id, or nil`, `reason`

```lua
-- Log every accepted chat line to the server console
Open77.chat.onMessage(function(playerId, playerName, text)
    print(("[chat] %s (%d): %s"):format(playerName, playerId, text))
end)
```

[Chat & slash commands guide](../../01%20Documentation/chat.md)

Registered in `LuaResourceRuntime.Events.cs` (line 299) as `LuaResourceRuntime.Bootstrap`.

## removeSuggestion

```lua
Open77.chat.removeSuggestion(target, command)
```

`NETWORK`: Uses the network backend.

Withdraws one slash-command suggestion.

Facade over `chat:removeSuggestion`. Use it when a command stops being available to that player rather than leaving a suggestion that will be refused.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `integer` | required | — |
| `command` | `string` | required | — |

Returns: `true on success, otherwise false`, `failure reason`

```lua
-- The player left the police job: /arrest disappears from their autocomplete
Open77.chat.removeSuggestion(playerId, "/arrest")
```

[Chat & slash commands guide](../../01%20Documentation/chat.md)

Registered in `LuaResourceRuntime.Events.cs` (line 320) as `LuaResourceRuntime.Bootstrap`.

## send

```lua
Open77.chat.send(playerId, message)
```

`NETWORK`: Uses the network backend.

Sends a chat message to one player, or to everyone with -1.

A thin facade over `chat:addMessage` on the host-wide event bus; `open77_chat` remains the only chat authority and does the actual push to clients. `playerId` must be a **number**: an integer id, `-1` or `nil` for everyone. A player id delivered by a host event (`onPlayerReady`, `onPlayerDisconnected`, `onPlayerEnteredVehicle`...) is a string and is refused with `invalid_chat_target` -- pass `tonumber(playerId)`; `source` inside a net-event or command handler is already a number. `message` is a plain string (1 to 4,096 bytes) or the full table the chat UI renders ({ type, author, text, playerId, color = { r, g, b } as a positional array of three 0-255 numbers, e.g. `{ 0, 229, 255 }` -- a keyed `{ r = ..., g = ..., b = ... }` table is silently ignored by the chat UI }). Grants nothing: the facade never reaches a client itself, so it needs no `network.events`. Because it publishes on the host bus it answers `resource_preparing` at the top level of a script, before the chunk has loaded: send from `onResourceStart`, a command or an event handler. Delivery is asynchronous -- it lands on the next tick boundary -- and two sends in the same tick arrive in the opposite order, because the resource scheduler collects ready tasks back to front. Put a `Wait(0)` between lines that must read in order. With no chat authority running the call still returns true, since the bus cannot tell a publisher that nobody listened.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|-1|nil` | required | — |
| `message` | `string|table` | required | — |

Returns: `true on success, otherwise false`, `failure reason`

```lua
-- A private message from the server, in a colour
Open77.chat.send(playerId, {
    author = "Dispatch",
    text = "A robbery is in progress downtown.",
    color = { 255, 120, 0 },
})
```

[Chat & slash commands guide](../../01%20Documentation/chat.md)

Registered in `LuaResourceRuntime.Events.cs` (line 281) as `LuaResourceRuntime.Bootstrap`.
