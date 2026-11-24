# Open77.http (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-http.md) · [Page web](https://open2077.net/docs/api/server/open77-http)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

15 functions.

## base64Decode

```lua
Open77.http.base64Decode(…)
```

`SERVER`: Available in server resources only.

Outbound HTTP: base64Decode.

Open77.http calls web APIs, webhooks and file hosts. It requires the http.request
permission and a server that enabled outbound HTTP with an allow-list. The request leaves the
tick thread at once, and the callback (or .await) resumes on the owning resource's own tick.

Documented in [server-api.md](../../01%20Documentation/server-api.md#outbound-http) (line 3029).

## base64Encode

```lua
Open77.http.base64Encode(…)
```

`SERVER`: Available in server resources only.

Outbound HTTP: base64Encode.

Open77.http calls web APIs, webhooks and file hosts. It requires the http.request
permission and a server that enabled outbound HTTP with an allow-list. The request leaves the
tick thread at once, and the callback (or .await) resumes on the owning resource's own tick.

Documented in [server-api.md](../../01%20Documentation/server-api.md#outbound-http) (line 3029).

## cancel

```lua
Open77.http.cancel(id)
```

`SERVER`: Available in server resources only.

Outbound HTTP: cancel.

Open77.http calls web APIs, webhooks and file hosts. It requires the http.request
permission and a server that enabled outbound HTTP with an allow-list. The request leaves the
tick thread at once, and the callback (or .await) resumes on the owning resource's own tick.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `not specified` | required | — |

Documented in [server-api.md](../../01%20Documentation/server-api.md#outbound-http) (line 3028).

## download

```lua
Open77.http.download(url, path, options?, cb?)
```

`SERVER`: Available in server resources only.

Outbound HTTP: download.

Open77.http calls web APIs, webhooks and file hosts. It requires the http.request
permission and a server that enabled outbound HTTP with an allow-list. The request leaves the
tick thread at once, and the callback (or .await) resumes on the owning resource's own tick.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `url` | `not specified` | required | — |
| `path` | `not specified` | required | — |
| `options` | `not specified` | optional | — |
| `cb` | `not specified` | optional | — |

Documented in [server-api.md](../../01%20Documentation/server-api.md#outbound-http) (line 3027).

## encodeQuery

```lua
Open77.http.encodeQuery(…)
```

`SERVER`: Available in server resources only.

Outbound HTTP: encodeQuery.

Open77.http calls web APIs, webhooks and file hosts. It requires the http.request
permission and a server that enabled outbound HTTP with an allow-list. The request leaves the
tick thread at once, and the callback (or .await) resumes on the owning resource's own tick.

Documented in [server-api.md](../../01%20Documentation/server-api.md#outbound-http) (line 3029).

## isAllowed

```lua
Open77.http.isAllowed(urlOrHost)
```

`SERVER`: Available in server resources only.

Outbound HTTP: isAllowed.

Open77.http calls web APIs, webhooks and file hosts. It requires the http.request
permission and a server that enabled outbound HTTP with an allow-list. The request leaves the
tick thread at once, and the callback (or .await) resumes on the owning resource's own tick.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `urlOrHost` | `not specified` | required | — |

Documented in [server-api.md](../../01%20Documentation/server-api.md#outbound-http) (line 3030).

## limits

```lua
Open77.http.limits()
```

`SERVER`: Available in server resources only.

Outbound HTTP: limits.

Open77.http calls web APIs, webhooks and file hosts. It requires the http.request
permission and a server that enabled outbound HTTP with an allow-list. The request leaves the
tick thread at once, and the callback (or .await) resumes on the owning resource's own tick.

Documented in [server-api.md](../../01%20Documentation/server-api.md#outbound-http) (line 3030).

## listen

```lua
Open77.http.listen(prefix, handler)
```

`NETWORK`: Uses the network backend.

Serves inbound HTTP under `/<resource>/<prefix>` on the server's opt-in listener.

Requires `http.serve` -- a different power from `http.request` (the network calling a resource, not the resource calling out), so an operator reading a manifest sees which one a resource wants. The listener is a server-wide opt-in (`httpHandlers.enabled`, `listenUrl` default loopback `127.0.0.1:11781`, `timeoutSeconds`); while it is off every call answers `http_handlers_unavailable`. Routes are namespaced by resource: `listen("/hook", fn)` serves `http://host/<resource>/hook` and everything below it, the longest registered prefix wins, and a resource can never claim another's segment. The handler runs on the resource's own tick (one tick of latency, never cross-thread) with `(req, res)`: `req = { method, path, query, body, route, remoteAddress?, headers }` (header lookup is case-insensitive; the body is capped at 16 KiB and `body_too_large` is answered before Lua sees it) and `res.send(status, body, headers?)` -- a table body is JSON-encoded with a JSON content type -- plus `res.json(status?, value)` and `res.text(status?, text)`; the answer is sent once (`already_sent` after). A handler that throws answers 500 `handler_error`; one that never answers is timed out by the host. Sixteen requests in flight per resource, the rest answered 503. Routes are dropped when the resource stops or reloads. Returns the public route, or `nil, reason`: `invalid_handler`, `invalid_prefix`, `http_handlers_unavailable`, `permission_denied:http.serve`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `prefix` | `string (path under the resource's own segment; `/` for everything)` | required | — |
| `handler` | `function(req, res)` | required | — |

Returns: `public route string, or nil`, `reason on failure`

```lua
-- open77.lua: permissions { 'http.serve' }; server.jsonc: "httpHandlers": { "enabled": true }
-- A webhook that heals a player by id: POST /my_resource/heal
local route, reason = Open77.http.listen('/heal', function(req, res)
    if req.method ~= 'POST' then return res.json(405, { error = 'method_not_allowed' }) end
    local payload = json.decode(req.body or '') or {}
    local ok, why = Open77.players.setHealth(payload.playerId, 100)
    if not ok then return res.json(404, { error = why }) end
    res.json({ healed = payload.playerId })
end)
if not route then print('cannot serve: ' .. tostring(reason)) else print('listening on ' .. route) end
```

Registered in `LuaResourceRuntime.Http.cs` (line 269) as `LuaResourceRuntime.Bootstrap`.

## parseQuery

```lua
Open77.http.parseQuery(…)
```

`SERVER`: Available in server resources only.

Outbound HTTP: parseQuery.

Open77.http calls web APIs, webhooks and file hosts. It requires the http.request
permission and a server that enabled outbound HTTP with an allow-list. The request leaves the
tick thread at once, and the callback (or .await) resumes on the owning resource's own tick.

Documented in [server-api.md](../../01%20Documentation/server-api.md#outbound-http) (line 3029).

## request

```lua
Open77.http.request(url, [callback], [method], [body], [headers])
```

`NETWORK`: Uses the network backend.

Makes an outbound HTTP request; the same function as `PerformHttpRequest`.

Requires `http.request` **and** a server that enabled the bridge with an allow-list -- an empty list reaches nothing and redirects are never followed, so the list is the whole reachable surface. The request leaves the tick thread at once, the exchange runs on the thread pool, and the callback runs on the owning resource's own tick. The callback is `(status, body, headers, err)`, with `status = 0` when the exchange did not complete. A refused request still invokes the callback, so a script never waits on one.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `url` | `string (absolute http/https, host on the allow-list)` | required | — |
| `callback` | `function(status, body, headers, err)` | optional | — |
| `method` | `string (GET default, POST, PUT, PATCH, DELETE, HEAD)` | optional | — |
| `body` | `string|table (JSON-encoded for you; 256 KiB maximum)` | optional | — |
| `headers` | `table (at most 32, no CR/LF)` | optional | — |

Returns: `true, or false`, `reason: permission_denied:http.request, http_unavailable, invalid_url, host_not_allowed, invalid_method, body_too_large, too_many_headers, invalid_header, too_many_requests`

```lua
-- Post to a Discord webhook when the server starts
Open77.http.request("https://discord.com/api/webhooks/...", function(status, body, headers, err)
    if status == 0 then print("webhook failed: " .. tostring(err)) end
end, "POST", { content = "Server is up!" })
```

Registered in `LuaResourceRuntime.cs` (line 8405) as `LuaResourceRuntime.Bootstrap`.

## routes

```lua
Open77.http.routes()
```

`NETWORK`: Uses the network backend.

Lists the public routes this resource is serving.

Ungated read of what `listen` registered for the calling resource, as the public paths a client would request (`/<resource>/<prefix>`), longest first -- the order the host matches them in. Another resource's routes are not listed. Empty while the listener is disabled.

Returns: `array of route strings`

```lua
-- Print what this resource exposes at start, so the operator's log says it
for _, route in ipairs(Open77.http.routes()) do
    print('serving ' .. route)
end
```

Registered in `LuaResourceRuntime.Http.cs` (line 292) as `LuaResourceRuntime.Bootstrap`.

## unlisten

```lua
Open77.http.unlisten(prefix)
```

`NETWORK`: Uses the network backend.

Removes one of this resource's inbound HTTP routes.

Requires `http.serve`. Takes the same prefix `listen` was given (normalised the same way: `/hook/` and `/hook` are one route) and answers `true`, or `false, reason`: `route_not_found` when this resource never registered it (another resource's route is never visible here), `invalid_prefix`, `permission_denied:http.serve`. Not needed on stop or reload: the host drops every route the resource owns.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `prefix` | `string` | required | — |

Returns: `true, or false`, `reason on failure`

```lua
-- Close the maintenance endpoint once the window is over
Open77.http.listen('/maintenance', function(req, res) res.text('maintenance on') end)
SetTimeout(60000, function()
    local ok, reason = Open77.http.unlisten('/maintenance')
    print('maintenance route closed:', ok, reason)
end)
```

Registered in `LuaResourceRuntime.Http.cs` (line 282) as `LuaResourceRuntime.Bootstrap`.

## upload

```lua
Open77.http.upload(url, parts, options?, cb?)
```

`SERVER`: Available in server resources only.

Outbound HTTP: upload.

Open77.http calls web APIs, webhooks and file hosts. It requires the http.request
permission and a server that enabled outbound HTTP with an allow-list. The request leaves the
tick thread at once, and the callback (or .await) resumes on the owning resource's own tick.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `url` | `not specified` | required | — |
| `parts` | `not specified` | required | — |
| `options` | `not specified` | optional | — |
| `cb` | `not specified` | optional | — |

Documented in [server-api.md](../../01%20Documentation/server-api.md#outbound-http) (line 3026).

## urlDecode

```lua
Open77.http.urlDecode(…)
```

`SERVER`: Available in server resources only.

Outbound HTTP: urlDecode.

Open77.http calls web APIs, webhooks and file hosts. It requires the http.request
permission and a server that enabled outbound HTTP with an allow-list. The request leaves the
tick thread at once, and the callback (or .await) resumes on the owning resource's own tick.

Documented in [server-api.md](../../01%20Documentation/server-api.md#outbound-http) (line 3029).

## urlEncode

```lua
Open77.http.urlEncode(…)
```

`SERVER`: Available in server resources only.

Outbound HTTP: urlEncode.

Open77.http calls web APIs, webhooks and file hosts. It requires the http.request
permission and a server that enabled outbound HTTP with an allow-list. The request leaves the
tick thread at once, and the callback (or .await) resumes on the owning resource's own tick.

Documented in [server-api.md](../../01%20Documentation/server-api.md#outbound-http) (line 3029).
