# Open77.playerInteractions (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-playerinteractions.md) · [Page web](https://open2077.net/docs/api/client/open77-playerinteractions)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

9 functions.

## accept

```lua
Open77.playerInteractions.accept(interactionId)
```

`NETWORK`: Uses the network backend.

Queue acceptance of the local player's offered invitation.

Declare dependency 'open77_player_interactions >=1.0.0'. This is an asynchronous resource-export facade: call from a managed coroutine and await the Promise; nil plus reason indicates dispatch/rejection. No presentation capability is required for consumers. Resolving a request ID means queued, not accepted by the server. Observe onPlayerInteractionResult or result(requestId), then lifecycle state events.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `interactionId` | `string` | required | — |

Returns: `Open77.Promise resolving to request ID; nil, reason on rejection`

```lua
CreateThread(function()
    local pending, err = Open77.playerInteractions.accept(interactionId)
    if not pending then print(err); return end
    local value, rejected = pending:await()
    if rejected then print(rejected); return end
    print(value)
end)
```

[Player interactions guide](../../01%20Documentation/player-interactions.md#client-api)

Registered in `ResourceHost.cpp` (line 24704) as `LuaPlayerInteractionExport`.

## cancel

```lua
Open77.playerInteractions.cancel([interactionId])
```

`NETWORK`: Uses the network backend.

Queue cancellation of an action the local player participates in; defaults to their current action.

Declare dependency 'open77_player_interactions >=1.0.0'. This is an asynchronous resource-export facade: call from a managed coroutine and await the Promise; nil plus reason indicates dispatch/rejection. No presentation capability is required for consumers. Resolving a request ID means queued, not accepted by the server. Observe onPlayerInteractionResult or result(requestId), then lifecycle state events.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `interactionId` | `string` | optional | — |

Returns: `Open77.Promise resolving to request ID; nil, reason on rejection`

```lua
CreateThread(function()
    local pending, err = Open77.playerInteractions.cancel(interactionId)
    if not pending then print(err); return end
    local value, rejected = pending:await()
    if rejected then print(rejected); return end
    print(value)
end)
```

[Player interactions guide](../../01%20Documentation/player-interactions.md#client-api)

Registered in `ResourceHost.cpp` (line 24704) as `LuaPlayerInteractionExport`.

## current

```lua
Open77.playerInteractions.current([playerId])
```

`NETWORK`: Uses the network backend.

Read a player's active reservation; defaults to the local player.

Declare dependency 'open77_player_interactions >=1.0.0'. This is an asynchronous resource-export facade: call from a managed coroutine and await the Promise; nil plus reason indicates dispatch/rejection. No presentation capability is required for consumers. Client observations are not authority for inventory, healing or rewards.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `Open77.Promise resolving to state or nil; nil, reason on rejection`

```lua
CreateThread(function()
    local pending, err = Open77.playerInteractions.current()
    if not pending then print(err); return end
    local value, rejected = pending:await()
    if rejected then print(rejected); return end
    print(value)
end)
```

[Player interactions guide](../../01%20Documentation/player-interactions.md#client-api)

Registered in `ResourceHost.cpp` (line 24704) as `LuaPlayerInteractionExport`.

## decline

```lua
Open77.playerInteractions.decline(interactionId)
```

`NETWORK`: Uses the network backend.

Queue refusal of the local player's offered invitation.

Declare dependency 'open77_player_interactions >=1.0.0'. This is an asynchronous resource-export facade: call from a managed coroutine and await the Promise; nil plus reason indicates dispatch/rejection. No presentation capability is required for consumers. Resolving a request ID means queued, not accepted by the server. Observe onPlayerInteractionResult or result(requestId), then lifecycle state events.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `interactionId` | `string` | required | — |

Returns: `Open77.Promise resolving to request ID; nil, reason on rejection`

```lua
CreateThread(function()
    local pending, err = Open77.playerInteractions.decline(interactionId)
    if not pending then print(err); return end
    local value, rejected = pending:await()
    if rejected then print(rejected); return end
    print(value)
end)
```

[Player interactions guide](../../01%20Documentation/player-interactions.md#client-api)

Registered in `ResourceHost.cpp` (line 24704) as `LuaPlayerInteractionExport`.

## get

```lua
Open77.playerInteractions.get(interactionId)
```

`NETWORK`: Uses the network backend.

Read one visible active interaction; missing is not an error.

Declare dependency 'open77_player_interactions >=1.0.0'. This is an asynchronous resource-export facade: call from a managed coroutine and await the Promise; nil plus reason indicates dispatch/rejection. No presentation capability is required for consumers. Client observations are not authority for inventory, healing or rewards.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `interactionId` | `string` | required | — |

Returns: `Open77.Promise resolving to state or nil; nil, reason on rejection`

```lua
CreateThread(function()
    local pending, err = Open77.playerInteractions.get(interactionId)
    if not pending then print(err); return end
    local value, rejected = pending:await()
    if rejected then print(rejected); return end
    print(value)
end)
```

[Player interactions guide](../../01%20Documentation/player-interactions.md#client-api)

Registered in `ResourceHost.cpp` (line 24704) as `LuaPlayerInteractionExport`.

## isReserved

```lua
Open77.playerInteractions.isReserved([playerId])
```

`NETWORK`: Uses the network backend.

Check the visible reservation cache; defaults to the local player.

Declare dependency 'open77_player_interactions >=1.0.0'. This is an asynchronous resource-export facade: call from a managed coroutine and await the Promise; nil plus reason indicates dispatch/rejection. No presentation capability is required for consumers. Client observations are not authority for inventory, healing or rewards.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `Open77.Promise resolving to boolean; nil, reason on rejection`

```lua
CreateThread(function()
    local pending, err = Open77.playerInteractions.isReserved()
    if not pending then print(err); return end
    local value, rejected = pending:await()
    if rejected then print(rejected); return end
    print(value)
end)
```

[Player interactions guide](../../01%20Documentation/player-interactions.md#client-api)

Registered in `ResourceHost.cpp` (line 24704) as `LuaPlayerInteractionExport`.

## list

```lua
Open77.playerInteractions.list()
```

`NETWORK`: Uses the network backend.

Read copied active states visible in this routing bucket.

Declare dependency 'open77_player_interactions >=1.0.0'. This is an asynchronous resource-export facade: call from a managed coroutine and await the Promise; nil plus reason indicates dispatch/rejection. No presentation capability is required for consumers. Client observations are not authority for inventory, healing or rewards.

Returns: `Open77.Promise resolving to state[]; nil, reason on rejection`

```lua
CreateThread(function()
    local pending, err = Open77.playerInteractions.list()
    if not pending then print(err); return end
    local value, rejected = pending:await()
    if rejected then print(rejected); return end
    print(value)
end)
```

[Player interactions guide](../../01%20Documentation/player-interactions.md#client-api)

Registered in `ResourceHost.cpp` (line 24704) as `LuaPlayerInteractionExport`.

## respond

```lua
Open77.playerInteractions.respond(interactionId, accepted)
```

`NETWORK`: Uses the network backend.

Queue acceptance or refusal of the local player's offered invitation. Only the target may respond.

Declare dependency 'open77_player_interactions >=1.0.0'. This is an asynchronous resource-export facade: call from a managed coroutine and await the Promise; nil plus reason indicates dispatch/rejection. No presentation capability is required for consumers. Resolving a request ID means queued, not accepted by the server. Observe onPlayerInteractionResult or result(requestId), then lifecycle state events.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `interactionId` | `string` | required | — |
| `accepted` | `boolean` | required | — |

Returns: `Open77.Promise resolving to request ID; nil, reason on rejection`

```lua
CreateThread(function()
    local pending, err = Open77.playerInteractions.respond(interactionId, true)
    if not pending then print(err); return end
    local value, rejected = pending:await()
    if rejected then print(rejected); return end
    print(value)
end)
```

[Player interactions guide](../../01%20Documentation/player-interactions.md#client-api)

Registered in `ResourceHost.cpp` (line 24704) as `LuaPlayerInteractionExport`.

## result

```lua
Open77.playerInteractions.result(requestId)
```

`NETWORK`: Uses the network backend.

Read a cached server acknowledgement for a queued request; acknowledgements expire after30seconds.

Declare dependency 'open77_player_interactions >=1.0.0'. This is an asynchronous resource-export facade: call from a managed coroutine and await the Promise; nil plus reason indicates dispatch/rejection. No presentation capability is required for consumers. Client observations are not authority for inventory, healing or rewards.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `requestId` | `string` | required | — |

Returns: `Open77.Promise resolving to result table or nil; nil, reason on rejection`

```lua
CreateThread(function()
    local pending, err = Open77.playerInteractions.result(requestId)
    if not pending then print(err); return end
    local value, rejected = pending:await()
    if rejected then print(rejected); return end
    print(value)
end)
```

[Player interactions guide](../../01%20Documentation/player-interactions.md#client-api)

Registered in `ResourceHost.cpp` (line 24704) as `LuaPlayerInteractionExport`.
