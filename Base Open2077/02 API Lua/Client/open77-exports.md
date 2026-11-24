# Open77.exports (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-exports.md) · [Page web](https://open2077.net/docs/api/client/open77-exports)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

2 functions.

## call

```lua
Open77.exports.call(resource, export, […])
```

`SHARED`: Available without a live game instance.

Calls a function exported by another resource, asynchronously.

Schedules another running client resource's exported function and returns a generation-bound Promise. Arguments and results must be serializable; unavailable resources, missing exports, stale generations and scheduler saturation are returned immediately as `nil, reason`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `resource` | `string` | required | — |
| `export` | `string` | required | — |
| `…` | `any` | optional | — |

Returns: `promise, or nil`, `reason`

```lua
-- Show a toast through the notifications package, asynchronously
CreateThread(function()
    local pending, reason = Open77.exports.call("open77_notifications", "show", {
        type = "success", message = "Vehicle stored.",
    })
    if not pending then return print(reason) end
    local result = pending:await()
    if result and not result.ok then print(result.error) end
end)
```

Registered in `ResourceHost.cpp` (line 24689) as `LuaCallExport`.

## callSync

```lua
Open77.exports.callSync(resource, export, […])
```

`SHARED`: Available without a live game instance.

Calls a function exported by another resource, inline on this thread.

The non-sugar form of exports.resource:name(...). Runs the callee in its own VM before returning and yields its values directly. RAISES on failure instead of returning nil, reason — the one place Open77 departs from its own convention, so that ported FiveM code behaves the same; use pcall to handle failure. The callee must not yield (export_yielded), gets its own instruction budget (export_budget_exhausted fails only that call), and chains are capped at 8 frames (export_recursion_limit). Arguments and results are copied, never shared. A resource may call its own export.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `resource` | `string` | required | — |
| `export` | `string` | required | — |
| `…` | `any` | optional | — |

Returns: `whatever the export returns`

```lua
-- The export name is in a variable: the non-sugar synchronous form
local ok, fuel = pcall(Open77.exports.callSync, "open77_fuel", exportName, vehicleId)
if ok then print("fuel", fuel) end
```

Registered in `ResourceHost.cpp` (line 24690) as `LuaCallExportSync`.
