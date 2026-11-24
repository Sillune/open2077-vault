# Open77.dash (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-dash.md) · [Page web](https://open2077.net/docs/api/client/open77-dash)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

1 function.

## call

```lua
Open77.dash.call(operation, [arguments])
```

`GAME`: Requires a live game instance.

Runs the resource-owned native Dash projector operation.

Client support adapter; requires player.dash.project. Operations configure, state, request, approve, cancel, activity, release and present are bounded and generation-scoped. Server creators use the separate server Open77.dash define/grant/revoke API documented in dash.md. Native permits and correlated phases are not proof of rendered movement. Arbitrary speed, distance, effect IDs and stamina settings are unavailable. Multiplayer acceptance is pending.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `operation` | `string` | required | — |
| `arguments` | `operation-specific values` | optional | — |

Returns: `operation result or nil`, `reason on failure`

```lua
-- The support resource forwards a server-accepted dash phase to the native projector
RegisterNetEvent("dash:phase", function(...)
    Open77.dash.call("present", ...)
end)
```

Registered in `ResourceHost.cpp` (line 24852) as `LuaDashCall`.
