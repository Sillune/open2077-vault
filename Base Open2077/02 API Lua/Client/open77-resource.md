# Open77.resource (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-resource.md) · [Page web](https://open2077.net/docs/api/client/open77-resource)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

7 functions.

## generation

```lua
Open77.resource.generation([resource])
```

`SHARED`: Available without a live game instance.

Generation number, incremented on every reload.

With no argument, returns the current resource generation. Pass a resource name to inspect that resource, or receive nil when it is missing.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `resource` | `string` | optional | — |

Returns: `integer, or nil`

```lua
-- Tag a deferred callback with the generation so a reload can ignore it
local generation = Open77.resource.generation()
SetTimeout(5000, function()
    if Open77.resource.generation() ~= generation then return end
    print("still the same VM")
end)
```

Registered in `ResourceHost.cpp` (line 24582) as `LuaResourceGeneration`.

## hasPermission

```lua
Open77.resource.hasPermission(permission)
```

`SHARED`: Available without a live game instance.

Whether the resource holds this permission.

Checks whether the current resource manifest grants an exact Open77 capability. It is a read-only convenience check; native APIs still enforce their own permission and must handle refusal results.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `permission` | `string` | required | — |

Returns: `boolean`

```lua
-- Only offer the map feature if the manifest granted it
if Open77.resource.hasPermission("ui.vanilla.map") then
    Open77.blips.create({ position = shop, label = "Shop" })
end
```

Registered in `ResourceHost.cpp` (line 24590) as `LuaHasPermission`.

## name

```lua
Open77.resource.name()
```

`SHARED`: Available without a live game instance.

Name of the current resource.

Returns the current resource's immutable manifest name. This is equivalent to `GetCurrentResourceName()` and remains stable for the lifetime of the resource definition.

Returns: `string`

```lua
-- Prefix log lines with the resource name
print(("[%s] loaded"):format(Open77.resource.name()))
```

Registered in `ResourceHost.cpp` (line 24581) as `LuaResourceName`.

## readFile

```lua
Open77.resource.readFile(path)
```

`SHARED`: Available without a live game instance.

Reads a file from the resource, within its declared allowlist.

Reads a file declared by the current resource manifest through a safe relative path. Undeclared files, traversal, oversized content and host filesystem paths are rejected.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `path` | `string` | required | — |

Returns: `contents, or nil`, `reason`

```lua
-- Load a JSON price list declared in the manifest's files
local text = Open77.resource.readFile("data/prices.json")
local prices = text and json.decode(text) or {}
```

Registered in `ResourceHost.cpp` (line 24585) as `LuaReadFile`.

## readPackedFile

```lua
Open77.resource.readPackedFile(resource, path)
```

`SHARED`: Available without a live game instance.

Reads a file another resource declared, as base64.

Requires `resources.files.read`, reserved for the bundled audio service. Reads only files declared in the target resource's `files` list; scripts, manifests and undeclared disk files are inaccessible. Other resources use `Open77.resource.readFile` for their own files. Returns Base64 for binary-safe transport, up to 3 MiB raw data. Base64 expansion can exceed the separate 2 MiB WebUI JSON message cap; see [runtime quotas](../../01%20Documentation/resource-runtime.md#sandbox-and-quotas). Errors: `permission_denied:resources.files.read`, `invalid_resource_path`, `resource_not_found`, `file_not_declared`, `file_too_large`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `resource` | `string` | required | — |
| `path` | `string` | required | — |

Returns: `base64 string`, `or nil, reason`

```lua
-- The audio service reads a declared sound file from another resource, as base64
local base64, reason = Open77.resource.readPackedFile("myradio", "sounds/jingle.mp3")
```

Registered in `ResourceHost.cpp` (line 24589) as `LuaReadPackedFile`.

## state

```lua
Open77.resource.state(resource)
```

`SHARED`: Available without a live game instance.

State of a resource on this client: `discovered`, `starting`, `running`, `stopping`, `stopped`, `failed`, `blocked`, or `missing` for a name this host does not know. There is no `started`: a resource that is up reads `running`, so `== "started"` is never true.

Returns the lifecycle state known for a resource name (the argument is required; `GetResourceState` is the same function): `discovered`, `starting`, `running`, `stopping`, `stopped`, `failed`, `blocked`, or `missing` for a name this host does not know. There is no `started`: a resource that is up reads `running`, so `== "started"` is never true. An immediate snapshot; it does not wait for dependencies or mutate lifecycle state.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `resource` | `string` | required | — |

Returns: `one of the eight state names`

```lua
-- Only talk to the notifications package if it is running (the state is "running", never "started")
if Open77.resource.state("open77_notifications") == "running" then
    Open77.exports.call("open77_notifications", "show", { message = "Ready" })
end
```

Registered in `ResourceHost.cpp` (line 24584) as `LuaGetResourceState`.

## version

```lua
Open77.resource.version()
```

`SHARED`: Available without a live game instance.

Version declared in `open77.lua`.

Returns the version string declared by the current resource manifest. It is package metadata and is unrelated to the Open77 runtime or Lua interpreter version.

Returns: `string`

```lua
-- Show the version in a debug line
print(("running %s %s"):format(Open77.resource.name(), Open77.resource.version()))
```

Registered in `ResourceHost.cpp` (line 24583) as `LuaResourceVersion`.
