# Open77.resource (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-resource.md) · [Page web](https://open2077.net/docs/api/server/open77-resource)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

8 functions.

## generation

```lua
Open77.resource.generation([resourceName])
```

`NETWORK`: Uses the network backend.

A running server resource's opaque VM generation.

No permission. Omitting the name answers this VM's own generation; naming another running server resource answers its generation, or `0` when it is unavailable. The value changes on every restart and reload, which is how a cached handle or a deferred callback can tell it is talking to the same VM it started with.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `resourceName` | `string` | optional | — |

Returns: `generation, or 0`

```lua
-- Ignore a timer that fired after this resource was reloaded
local generation = Open77.resource.generation()
SetTimeout(60000, function()
    if Open77.resource.generation() ~= generation then return end
    save()
end)
```

Registered in `LuaResourceRuntime.Exports.cs` (line 345) as `LuaResourceRuntime.Bootstrap`.

## list

```lua
Open77.resource.list()
```

`NETWORK`: Uses the network backend.

Every discovered resource and its runtime state.

No capability. Entries are `{ name, version, state, generation, memoryBytes }` plus `error` when the resource failed. `state` is the lowercased runtime state: discovered, starting, running, stopping, stopped, failed or blocked.

Returns: `array of tables, or nil`, `reason`

```lua
-- /resources: state of every resource
RegisterCommand("resources", function(source)
    for _, entry in ipairs(Open77.resource.list() or {}) do
        Open77.chat.send(source, ("%s %s (%s)"):format(entry.name, entry.version, entry.state))
    end
end, true)
```

Registered in `LuaResourceRuntime.RuntimeControl.cs` (line 348) as `LuaResourceRuntime.Bootstrap`.

## metadata

```lua
Open77.resource.metadata(name, [key])
```

`NETWORK`: Uses the network backend.

One resource's parsed manifest, or one field of it.

No capability. With no `key`, returns the whole manifest as `{ name, version, open77Version, autoStart, reloadPolicy, dependencies, permissions, sharedScripts, serverScripts, clientScripts, preloadMods, clientFiles }`. With a key, returns just that field; both the camelCase and the manifest's own snake_case spelling are accepted. `author` and `description` answer `unknown_metadata_key` rather than an empty string: the manifest parser does not read them, so claiming a value would be a claim this server cannot make.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `key` | `string` | optional | — |

Returns: `table or field value, or nil`, `reason`

```lua
-- Read another resource's version from its manifest
print("chat version", Open77.resource.metadata("open77_chat", "version"))
```

Registered in `LuaResourceRuntime.RuntimeControl.cs` (line 349) as `LuaResourceRuntime.Bootstrap`.

## name

```lua
Open77.resource.name()
```

`NETWORK`: Uses the network backend.

This resource's name, as its manifest declares it.

No permission. `GetCurrentResourceName` is the same function.

Returns: `resource name`

```lua
-- Prefix log lines with the resource name
print(("[%s] started"):format(Open77.resource.name()))
```

Registered in `LuaResourceRuntime.cs` (line 8147) as `LuaResourceRuntime.Bootstrap`.

## restart

```lua
Open77.resource.restart(name)
```

`NETWORK`: Uses the network backend.

Queues a resource restart for the next tick boundary.

Requires `resources.control`. See `Open77.resource.start` for why it is queued.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |

Returns: `true when queued, otherwise false`, `reason`

```lua
-- /restart-shop from an admin (needs resources.control)
RegisterCommand("restart-shop", function(source)
    local ok, reason = Open77.resource.restart("shop")
    if not ok then Open77.chat.send(source, reason) end
end, true)
```

Registered in `LuaResourceRuntime.RuntimeControl.cs` (line 361) as `LuaResourceRuntime.Bootstrap`.

## start

```lua
Open77.resource.start(name)
```

`NETWORK`: Uses the network backend.

Queues a resource start for the next tick boundary.

Requires `resources.control`. Queued, never inline: a resource stopping or restarting another while a Lua state is on the stack would free memory that is still executing, so the verb is applied where a console line is applied -- on the game-loop thread, outside every VM. Reports acceptance, not completion; read `Open77.resource.list()` on a later tick for the outcome. An unknown name answers `unknown_resource` immediately rather than as a line in a log nobody is reading.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |

Returns: `true when queued, otherwise false`, `reason`

```lua
-- Start the event resource when an admin opens the event
RegisterCommand("event-open", function()
    Open77.resource.start("summer_event")
end, true)
```

Registered in `LuaResourceRuntime.RuntimeControl.cs` (line 353) as `LuaResourceRuntime.Bootstrap`.

## state

```lua
Open77.resource.state(resourceName)
```

`NETWORK`: Uses the network backend.

The current server-side state of a resource.

No permission. `GetResourceState` is the same function. Worth asking after a lifecycle event: a queued `open77:resource:started` describes a transition that happened, and a stop or start can have followed before the next tick.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `resourceName` | `string` | required | — |

Returns: `state string`

```lua
-- Wait for a dependency to be running before using its export
CreateThread(function()
    while Open77.resource.state("bank") ~= "running" do Wait(500) end
    print("bank is up")
end)
```

Registered in `LuaResourceRuntime.cs` (line 8147) as `LuaResourceRuntime.Bootstrap`.

## stop

```lua
Open77.resource.stop(name)
```

`NETWORK`: Uses the network backend.

Queues a resource stop for the next tick boundary.

Requires `resources.control`. See `Open77.resource.start` for why it is queued.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |

Returns: `true when queued, otherwise false`, `reason`

```lua
-- Stop the event resource when the event closes
RegisterCommand("event-close", function()
    Open77.resource.stop("summer_event")
end, true)
```

Registered in `LuaResourceRuntime.RuntimeControl.cs` (line 357) as `LuaResourceRuntime.Bootstrap`.
