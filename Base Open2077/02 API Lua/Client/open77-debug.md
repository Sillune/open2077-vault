# Open77.debug (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-debug.md) · [Page web](https://open2077.net/docs/api/client/open77-debug)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

3 functions.

## command

```lua
Open77.debug.command(commandLine)
```

`GAME`: Requires a live game instance.

Runs one registered native laboratory command directly.

Requires `debug.runtime` and the trusted `open77_debug` owner. Input is limited to 4096 bytes and unknown commands are not forwarded to the server.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `commandLine` | `string` | required | — |

Returns: `true and command result on success`, `false and reason`

```lua
-- The trusted debug resource runs a lab command
local ok, result = Open77.debug.command("population.restore")
print(ok, result)
```

Registered in `ResourceHost.cpp` (line 24639) as `LuaDebugCommand`.

## eval

```lua
Open77.debug.eval(source, [label])
```

`GAME`: Requires a live game instance.

Compiles and executes a bounded text-only Lua chunk in the privileged debug VM.

Requires `debug.runtime` and the trusted `open77_debug` owner. Source is limited to 32 KiB, the optional chunk label to 64 bytes, returned values to 16 and result text to 8 KiB. The ordinary memory, instruction and frame-time quotas remain active.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `source` | `string` | required | — |
| `label` | `string` | optional | `exec` |

Returns: `true and bounded result text on success`, `false and compile/runtime reason`

```lua
-- The trusted debug resource evaluates a snippet in the privileged VM
local ok, text = Open77.debug.eval("return Open77.character.state().position.x", "probe")
print(ok, text)
```

Registered in `ResourceHost.cpp` (line 24638) as `LuaDebugEval`.

## redscript

```lua
Open77.debug.redscript(command)
```

`GAME`: Requires a live game instance.

Queues one bounded command through the REDscript debug bridge.

Requires `debug.runtime` and the trusted `open77_debug` owner. The opaque command is limited to 4096 bytes and executes from a genuine REDscript frame.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `command` | `string` | required | — |

Returns: `true and bridge result on success`, `false and reason`

```lua
-- The trusted debug resource queues a REDscript bridge command
Open77.debug.redscript("perspective.state")
```

Registered in `ResourceHost.cpp` (line 24640) as `LuaDebugRedscript`.
