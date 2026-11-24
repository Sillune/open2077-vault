# Open77.exports (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-exports.md) · [Page web](https://open2077.net/docs/api/server/open77-exports)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

2 functions.

## call

```lua
Open77.exports.call(target, name, ...)
```

`NETWORK`: Uses the network backend.

Calls another server resource's export and returns a promise.

Asynchronous and cross-resource, unlike `TriggerEvent`, which never leaves this VM. Returns an `Open77.Promise`: `promise:await()` yields the copied return values, `promise:status()` reports pending, resolved, rejected or cancelled. Awaiting a pending call requires a managed coroutine. Call it from a running resource, not during preparation. The provider keeps its own permissions and identity: arguments never supply caller identity, and the exported coroutine does not inherit the player `source`. See [Cross-resource server exports](https://open2077.net/docs/api/server/server-exports.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `string (resource name)` | required | — |
| `name` | `string (export name)` | required | — |
| `...` | `arguments` | required | — |

Returns: `promise, or nil`, `reason when dispatch is refused`

```lua
-- Call the bank resource's export and await the answer
CreateThread(function()
    local pending, reason = Open77.exports.call("bank", "withdraw", playerId, 100)
    if not pending then return print(reason) end
    local ok, err = pending:await()
    if not ok then Open77.chat.send(playerId, "Withdrawal failed: " .. tostring(err)) end
end)
```

Registered in `LuaResourceRuntime.Exports.cs` (line 413) as `LuaResourceRuntime.Bootstrap`.

## callSync

```lua
Open77.exports.callSync(target, name, ...)
```

`SHARED`: Available without a live game instance.

Calls another server resource's export inline on this thread.

The non-sugar form of exports.resource:name(...). Runs the callee in its own VM before returning and yields its values directly. It RAISES on failure instead of returning nil, reason -- the one place Open77 departs from its own convention, so that ported FiveM code behaves the same; use pcall to handle failure. The callee must not yield (export_yielded), gets its own instruction budget (export_budget_exhausted fails only that call), and chains are capped at 8 frames (export_recursion_limit). Arguments and results are copied, never shared, so no Lua value is held across the boundary. Prefer Open77.exports.call when the callee can be slow, because a synchronous call holds the tick.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `string (resource name)` | required | — |
| `name` | `string (export name)` | required | — |
| `...` | `arguments` | required | — |

Returns: `whatever the export returns`

```lua
-- Synchronous export call: the bank answers inline (raises on failure)
local ok, balance = pcall(Open77.exports.callSync, "bank", "balance", source)
if ok then Open77.chat.send(source, ("Balance: %d"):format(balance)) end
```

Registered in `LuaResourceRuntime.Exports.cs` (line 420) as `LuaResourceRuntime.Bootstrap`.
