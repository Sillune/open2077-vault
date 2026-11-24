# Open77.log (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-log.md) · [Page web](https://open2077.net/docs/api/client/open77-log)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

4 functions.

## debug

```lua
Open77.log.debug(message)
```

`SHARED`: Available without a live game instance.

Writes a debug line to this resource's client log.

No permission. The line is written with the calling resource's name, so `red4ext/logs/open77-*.log` always says which resource spoke. Unlike its server namesake, the client honours the level. It takes one value, converted to text.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `message` | `value` | required | — |

```lua
-- Verbose tracing that only shows at debug level
Open77.log.debug(("menu opened at %.1f"):format(Open77.time.monotonic()))
```

Registered in `ResourceHost.cpp` (line 24570) as `LuaLog`.

## error

```lua
Open77.log.error(message)
```

`SHARED`: Available without a live game instance.

Writes an error line to this resource's client log.

No permission. Same resource-prefixed client log as `Open77.log.debug`, at the error level. It only logs: it does not raise, abort the handler or stop the resource.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `message` | `value` | required | — |

```lua
-- A refused call is logged, not raised
local ok, reason = Open77.blips.create({ position = spot })
if not ok then Open77.log.error("blip refused: " .. tostring(reason)) end
```

Registered in `ResourceHost.cpp` (line 24573) as `LuaLog`.

## info

```lua
Open77.log.info(message)
```

`SHARED`: Available without a live game instance.

Writes an informational line to this resource's client log.

No permission. Same resource-prefixed client log as `Open77.log.debug`, at the informational level.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `message` | `value` | required | — |

```lua
-- Normal lifecycle lines
Open77.log.info("HUD started")
```

Registered in `ResourceHost.cpp` (line 24571) as `LuaLog`.

## warn

```lua
Open77.log.warn(message)
```

`SHARED`: Available without a live game instance.

Writes a warning line to this resource's client log.

No permission. Same resource-prefixed client log as `Open77.log.debug`, at the warning level.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `message` | `value` | required | — |

```lua
-- Something unexpected but survivable
Open77.log.warn("notifications package missing, falling back to the vanilla toast")
```

Registered in `ResourceHost.cpp` (line 24572) as `LuaLog`.
