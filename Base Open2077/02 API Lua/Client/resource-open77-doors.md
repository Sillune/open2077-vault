# open77_doors (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/resource-open77-doors.md) · [Page web](https://open2077.net/docs/api/client/resource-open77-doors)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

3 functions.

## get

```lua
Open77.exports.call("open77_doors", "get", id)
```

`NETWORK`: Uses the network backend.

Read a door's latest replicated target state.

This is a server-interest snapshot, not proof that the native object is currently streamed. Native Open77.doors.state(id) reads the local projection instead.

This is an asynchronous client-side resource export. Depend on open77_doors >=1.0.0 and call Open77.exports.call, not a Lua table named open77_doors. The call returns a Promise (or nil, error if it cannot be queued); awaiting it yields: Interested server snapshot or nil. Available with client and server release 2.31.13+op77.58 on the CDN; older clients do not contain its native projection bridge.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |

Returns: `Promise | nil`, `queue error, if any`

```lua
CreateThread(function()
    local pending, error = Open77.exports.call("open77_doors", "get", id)
    if not pending then print(error); return end
    local result, reason = pending:await()
    -- Handle the result before continuing; queued is not accepted.
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Documented in [resources/system/open77_doors/client/main.lua](../../01%20Documentation/doors.md).

## list

```lua
Open77.exports.call("open77_doors", "list", offset, limit)
```

`NETWORK`: Uses the network backend.

List door states within this client's interest.

Offset defaults to 0, limit to 16 (maximum 16). Interest is bounded to 256 nearest doors and follows the player's routing bucket.

This is an asynchronous client-side resource export. Depend on open77_doors >=1.0.0 and call Open77.exports.call, not a Lua table named open77_doors. The call returns a Promise (or nil, error if it cannot be queued); awaiting it yields: { doors, total, nextOffset? }. Available with client and server release 2.31.13+op77.58 on the CDN; older clients do not contain its native projection bridge.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `offset` | `integer` | optional | `0` |
| `limit` | `integer` | optional | `16` |

Returns: `Promise | nil`, `queue error, if any`

```lua
CreateThread(function()
    local pending, error = Open77.exports.call("open77_doors", "list", offset, limit)
    if not pending then print(error); return end
    local result, reason = pending:await()
    -- Handle the result before continuing; queued is not accepted.
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Documented in [resources/system/open77_doors/client/main.lua](../../01%20Documentation/doors.md).

## requestOpen

```lua
Open77.exports.call("open77_doors", "requestOpen", id, open)
```

`NETWORK`: Uses the network backend.

Ask the server to open or close an interactive door.

Queue acceptance is not server approval. Listen to open77:doors:requestResult(id, accepted, reason). Distance, bucket, access and doorway occupancy are checked server-side. Automatic doors and lift doors reject manual requests.

This is an asynchronous client-side resource export. Depend on open77_doors >=1.0.0 and call Open77.exports.call, not a Lua table named open77_doors. The call returns a Promise (or nil, error if it cannot be queued); awaiting it yields: queued boolean, reason?. Available with client and server release 2.31.13+op77.58 on the CDN; older clients do not contain its native projection bridge.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |
| `open` | `boolean` | required | — |

Returns: `Promise | nil`, `queue error, if any`

```lua
CreateThread(function()
    local pending, error = Open77.exports.call("open77_doors", "requestOpen", id, open)
    if not pending then print(error); return end
    local result, reason = pending:await()
    -- Handle the result before continuing; queued is not accepted.
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Documented in [resources/system/open77_doors/client/main.lua](../../01%20Documentation/doors.md).
