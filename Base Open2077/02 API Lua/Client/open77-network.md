# Open77.network (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-network.md) · [Page web](https://open2077.net/docs/api/client/open77-network)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

7 functions.

## catalog

```lua
Open77.network.catalog()
```

`NETWORK`: Uses the network backend.

The catalog as of the last refresh.

Reads a snapshot: refreshing is asynchronous, so call `refresh`, then `catalog` a beat later.

Returns: `table { phase, generation, servers… }`

```lua
-- The server browser reads the catalog a beat after refreshing it
CreateThread(function()
    Open77.network.refresh()
    Wait(1000)
    local catalog = Open77.network.catalog()
    for _, server in ipairs(catalog.servers or {}) do print(server.name) end
end)
```

Registered in `ResourceHost.cpp` (line 24659) as `LuaNetworkCatalog`.

## connect

```lua
Open77.network.connect(endpoint, [name])
```

`NETWORK`: Uses the network backend.

Joins an Open77 server.

Starts a connection to the supplied Open77 server endpoint using the current identity and resource pipeline. It requires `network.client`; success means the request was accepted, while connection progress and errors are observed through `status()`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `endpoint` | `string` | required | — |
| `name` | `string` | optional | — |

Returns: `true when the request is accepted`, `reason for the refusal`

```lua
-- The trusted server browser joins the selected server
Open77.network.connect("play.example.net:11778", "V")
```

Registered in `ResourceHost.cpp` (line 24660) as `LuaNetworkConnect`.

## disconnect

```lua
Open77.network.disconnect()
```

`NETWORK`: Uses the network backend.

Leaves the current server.

Requests an orderly disconnect with an optional diagnostic reason. It returns after queuing teardown; the session phase in `status()` is the authoritative view of when cleanup has completed.

Returns: `boolean`

```lua
-- A "Leave server" entry in a custom pause menu
RegisterNetEvent("pause:leave", function()
    Open77.network.disconnect()
end)
```

Registered in `ResourceHost.cpp` (line 24661) as `LuaNetworkDisconnect`.

## identity

```lua
Open77.network.identity()
```

`NETWORK`: Uses the network backend.

Reads the local Master-backed identity profile state.

Returns the durable user id, current display name, registration state, and asynchronous update phase. The private key is never exposed. This bridge exists for the trusted local server browser; server-downloaded resources should use the verified player APIs instead.

Returns: `table { phase, generation, userId, displayName, error, registered }, or nil`, `reason`

```lua
-- The trusted browser shows the player's display name
local identity = Open77.network.identity()
if identity then print("playing as", identity.displayName) end
```

Registered in `ResourceHost.cpp` (line 24664) as `LuaNetworkIdentity`.

## identityUpdate

```lua
Open77.network.identityUpdate(catalogUrl, displayName)
```

`NETWORK`: Uses the network backend.

Requests a signed username update from the Master.

Queues the HTTP enrollment on the networking worker and returns immediately. Poll `identity()` until the matching generation becomes `ready` or `failed`. The backend is deliberately available only to the trusted system server-browser resource and refuses updates during a live session.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `catalogUrl` | `string` | required | — |
| `displayName` | `string` | required | — |

Returns: `true when queued, otherwise false`, `reason`

```lua
-- The trusted browser asks the master for a new display name
Open77.network.identityUpdate(masterUrl, "V_from_Watson")
```

Registered in `ResourceHost.cpp` (line 24663) as `LuaNetworkIdentityUpdate`.

## refresh

```lua
Open77.network.refresh([url])
```

`NETWORK`: Uses the network backend.

Asks the directory for the server catalog again.

Starts a master-server catalog refresh for the supplied URL and requires `network.client`. The call only accepts the request; use `catalog()` to inspect generation, phase, HTTP status, body and error.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `url` | `string` | optional | — |

Returns: `boolean`, `reason`

```lua
-- Refresh the server list when the browser opens
Open77.network.refresh()
```

Registered in `ResourceHost.cpp` (line 24658) as `LuaNetworkRefresh`.

## status

```lua
Open77.network.status()
```

`NETWORK`: Uses the network backend.

State of the network session.

Returns the current client session snapshot: phase, endpoint, authenticated player identity, routing bucket, rates, server time, ping and downloaded resource-set metadata. Reading it does not initiate a connection or wait for a phase change.

Returns: `table { phase, endpoint, playerId, routingBucket, ping, error }`

```lua
-- Show the ping on the HUD
SetTick(function()
    local session = Open77.network.status()
    if session then hud.ping = session.ping end
end)
```

Registered in `ResourceHost.cpp` (line 24662) as `LuaNetworkStatus`.
