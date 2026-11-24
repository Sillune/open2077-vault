# Open77.log (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-log.md) · [Page web](https://open2077.net/docs/api/server/open77-log)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

4 functions.

## debug

```lua
Open77.log.debug(...)
```

`NETWORK`: Uses the network backend.

Writes a resource-prefixed debug line to the server log.

No permission. Writes to the resource-prefixed server logger at the `DBG` level, which a production log filter usually drops -- the place for per-tick chatter. `Citizen.Trace` is this function. Values are joined with a tab like `print`; control sequences (ANSI colour codes, cursor moves) are stripped from the line before it reaches the log, so a ported script's coloured output cannot corrupt the terminal or the log file.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `...` | `values` | required | — |

```lua
-- Verbose line in the server log
Open77.log.debug("purchase", source, itemId, price)
```

Registered in `LuaResourceRuntime.cs` (line 9856) as `LuaResourceRuntime.Bootstrap`.

## error

```lua
Open77.log.error(...)
```

`NETWORK`: Uses the network backend.

Writes a resource-prefixed error line to the server log.

No permission. Writes to the resource-prefixed server logger at the `ERR` level. It does not raise, abort the handler or mark the resource unhealthy: it is a log line, not an exception. Values are joined with a tab like `print`; control sequences (ANSI colour codes, cursor moves) are stripped from the line before it reaches the log, so a ported script's coloured output cannot corrupt the terminal or the log file.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `...` | `values` | required | — |

```lua
-- Log a refused call without raising
local ok, reason = Open77.vehicles.create(definition)
if not ok then Open77.log.error("spawn refused:", reason) end
```

Registered in `LuaResourceRuntime.cs` (line 9859) as `LuaResourceRuntime.Bootstrap`.

## info

```lua
Open77.log.info(...)
```

`NETWORK`: Uses the network backend.

Writes a resource-prefixed informational line to the server log.

No permission. Writes to the resource-prefixed server logger at the `INF` level, the same one `print` writes at. Values are joined with a tab like `print`; control sequences (ANSI colour codes, cursor moves) are stripped from the line before it reaches the log, so a ported script's coloured output cannot corrupt the terminal or the log file.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `...` | `values` | required | — |

```lua
-- Lifecycle line
Open77.log.info("shop resource started with", #items, "items")
```

Registered in `LuaResourceRuntime.cs` (line 9857) as `LuaResourceRuntime.Bootstrap`.

## warn

```lua
Open77.log.warn(...)
```

`NETWORK`: Uses the network backend.

Writes a resource-prefixed warning line to the server log.

No permission. Writes to the resource-prefixed server logger at the `WRN` level, so an operator filtering a resource's warnings from its chatter sees only these. Values are joined with a tab like `print`; control sequences (ANSI colour codes, cursor moves) are stripped from the line before it reaches the log, so a ported script's coloured output cannot corrupt the terminal or the log file.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `...` | `values` | required | — |

```lua
-- Survivable but worth a look
Open77.log.warn("no database configured, prices will not persist")
```

Registered in `LuaResourceRuntime.cs` (line 9858) as `LuaResourceRuntime.Bootstrap`.
