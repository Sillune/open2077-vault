# Open77.runtime (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-runtime.md) · [Page web](https://open2077.net/docs/api/client/open77-runtime)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

6 functions.

## commands

```lua
Open77.runtime.commands()
```

`CLIENT`: Client-only surface.

Every client command registered across every running resource.

Client-only. Returns an array of `{ name, resource, restricted, help?, parameters? }`. Only serialisable metadata crosses; handlers stay in their owner's state, exactly as `Open77.input.mappings` does for key binds. This is what the chat composer reads to merge client commands into its completion list -- it skips the restricted ones, which cannot run. `GetRegisteredCommands` is the same function under the FiveM name.

Returns: `array of tables`

```lua
-- Build a "/help" list from every client command
for _, command in ipairs(Open77.runtime.commands()) do
    print("/" .. command.name, command.help or "")
end
```

Registered in `ResourceHost.cpp` (line 24633) as `LuaGetRegisteredCommands`.

## executeCommand

```lua
Open77.runtime.executeCommand(line)
```

`CLIENT`: Client-only surface.

Runs a command line against the CLIENT command registry only.

Client-only. Accepts a raw line with or without a leading `/`, tokenised exactly as the chat composer tokenises it: `\` escapes, `"` and `'` quote, 32 arguments, 4096 bytes. The handler is scheduled, never called inline, so one resource's command cannot run inside another's C frame. `false, unknown_command` is the signal to forward the line to the server and is not an error; `command_restricted`, `empty_command`, `unterminated_quote`, `trailing_escape`, `too_many_arguments`, `command_line_too_long` and `command_task_limit` are. First match wins in resource-name order. Unlike FiveM's `ExecuteCommand` this never forwards to the server by itself -- the caller decides.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `line` | `string` | required | — |

Returns: `true when a client command claimed the line, otherwise false`, `reason`

```lua
-- Run a client command from a key; forward to the server when no client command matched
RegisterKeyMapping("quick_cmd", "Quick command", "F4", function()
    local ok, reason = Open77.runtime.executeCommand("hud toggle")
    if not ok and reason == "unknown_command" then TriggerServerEvent("chat:command", "hud toggle") end
end)
```

Registered in `ResourceHost.cpp` (line 24634) as `LuaExecuteCommand`.

## luaVersion

```lua
Open77.runtime.luaVersion()
```

`SHARED`: Available without a live game instance.

Embedded Lua version.

Returns the embedded Lua interpreter version used by this resource VM. Use it for diagnostics or compatibility checks, not to infer the Open77 build version.

Returns: `string`

```lua
-- Log the interpreter version once
print("Lua", Open77.runtime.luaVersion())
```

Registered in `ResourceHost.cpp` (line 24626) as `LuaLanguageVersion`.

## registerCommand

```lua
Open77.runtime.registerCommand(name, handler, [restricted], [suggestion])
```

`CLIENT`: Client-only surface.

Registers a client-side slash command in this resource's VM.

Client-only; no permission. The handler is `(source, args, raw)` with `source` always 0, because on the client there is no other party to attribute a command to. The optional third argument `restricted` is honoured by REFUSING: such a command is registered and listed but declines to run with `command_restricted`, because the client has no ACL and cannot answer an authority question -- an admin-only action belongs on the server. The optional fourth argument `{ help, parameters }` is merged into the chat composer's completions (8 parameters, 240 bytes each). Names accept `[A-Za-z0-9_.:-]` up to 64 bytes, the same set the server's `RegisterCommand` accepts; a leading `/` is stripped. Re-registering a name inside one VM replaces the handler. 128 commands per resource. `RegisterCommand` is the same function under the FiveM name.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `handler` | `function` | required | — |
| `restricted` | `boolean` | optional | — |
| `suggestion` | `table` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- A local command with chat completion metadata
Open77.runtime.registerCommand("hudscale", function(source, args)
    hud.scale = tonumber(args[1]) or 1.0
end, false, { help = "Scale the HUD", parameters = { { name = "factor", help = "0.5 to 2" } } })
```

Registered in `ResourceHost.cpp` (line 24631) as `LuaRegisterCommand`.

## unregisterCommand

```lua
Open77.runtime.unregisterCommand(name)
```

`CLIENT`: Client-only surface.

Withdraws a client command this resource registered.

Client-only. The registry is per VM: a resource can only withdraw its own command, and asking for another's answers `false, command_not_found`. Stopping a resource withdraws all of its commands.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Withdraw the command when the feature is disabled
Open77.runtime.unregisterCommand("hudscale")
```

Registered in `ResourceHost.cpp` (line 24632) as `LuaUnregisterCommand`.

## version

```lua
Open77.runtime.version()
```

`SHARED`: Available without a live game instance.

Open77 version.

Returns the Open77 scripting runtime version exposed to resources. This describes the host API and is distinct from both the resource manifest version and game build.

Returns: `string`

```lua
-- Refuse to run on an older runtime
print("Open77 runtime", Open77.runtime.version())
```

Registered in `ResourceHost.cpp` (line 24625) as `LuaRuntimeVersion`.
