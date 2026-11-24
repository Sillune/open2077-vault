# Open77.runtime (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-runtime.md) · [Page web](https://open2077.net/docs/api/server/open77-runtime)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

6 functions.

## cancelRestart

```lua
Open77.runtime.cancelRestart()
```

`NETWORK`: Uses the network backend.

Disarms a pending server restart.

Requires `runtime.restart`. The reason `open77:admin:scheduledRestart` is a plain notification rather than a cancellable event: stopping a shutdown is an authority, not a veto every listener gets for free. Broadcasts the cancellation to players and raises `open77:admin:restartCancelled` when a restart was actually armed; a no-op otherwise. Refusals: `permission_denied:runtime.restart`, `restart_control_unavailable`.

Returns: `{ scheduled = false, secondsRemaining = 0 }, or nil`, `reason`

```lua
-- /cancelrestart for admins
RegisterCommand("cancelrestart", function(source)
    Open77.runtime.cancelRestart()
end, true)
```

Registered in `LuaResourceRuntime.RuntimeControl.cs` (line 343) as `LuaResourceRuntime.Bootstrap`.

## commands

```lua
Open77.runtime.commands()
```

`NETWORK`: Uses the network backend.

Every command registered across every running resource, plus the built-in verbs.

No capability; it is a read of the operator's own installation. Entries are `{ name, resource, restricted, source }`, `source` being `resource` for a Lua registration and `console` for a built-in verb. The built-ins listed are the resource-lifecycle set the coordinator owns; the ACL, ban, routing-bucket and phantom verbs are **not** listed, because they live in a hand-written dispatcher with no registry to read. The `source` field is what tells a reader this list has two origins.

Returns: `array of tables, or nil`, `reason`

```lua
-- /commands lists everything, built-ins included
RegisterCommand("commands", function(source)
    for _, entry in ipairs(Open77.runtime.commands() or {}) do
        Open77.chat.send(source, ("/%s (%s)"):format(entry.name, entry.source))
    end
end, true)
```

Registered in `LuaResourceRuntime.RuntimeControl.cs` (line 328) as `LuaResourceRuntime.Bootstrap`.

## executeCommand

```lua
Open77.runtime.executeCommand(line)
```

`NETWORK`: Uses the network backend.

Queues one console line to run at the next tick boundary.

Requires `runtime.commands`, which grants a PLAIN resource's authority: it may invoke another resource's ordinary command and nothing else. A command declared `restricted` and every resource-lifecycle verb are refused. A resource that also declares `resources.control` runs the line with the authority the in-game operator escalation uses. FiveM's `ExecuteCommand` runs as the console with no declaration anywhere that a resource holds rcon; here the manifest says it. **Queued, never inline**: Lua runs inside the host's tick, so dispatching a line synchronously would re-enter a VM and reset an instruction budget mid-flight. The call reports acceptance, not completion, and the reply goes to the server log naming the resource that asked. Refusals: `permission_denied:runtime.commands`, `empty_command`, `command_line_too_long`, `command_too_many_tokens` (more than 64 tokens; every token is a Lua stack push at drain time), `command_queue_full`, `invalid_command_line`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `line` | `string` | required | — |

Returns: `true when queued, otherwise false`, `reason`

```lua
-- Run another resource's ordinary command from a script (needs runtime.commands)
Open77.runtime.executeCommand("weather rain")
```

Registered in `LuaResourceRuntime.RuntimeControl.cs` (line 324) as `LuaResourceRuntime.Bootstrap`.

## luaVersion

```lua
Open77.runtime.luaVersion()
```

`NETWORK`: Uses the network backend.

The Lua version this server VM runs.

No permission. Returns the interpreter's `_VERSION` string, for a resource that wants to record what it ran on.

Returns: `version string`

```lua
-- Record the interpreter in the startup log
print("running on", Open77.runtime.luaVersion())
```

Registered in `LuaResourceRuntime.cs` (line 8113) as `LuaResourceRuntime.Bootstrap`.

## restartStatus

```lua
Open77.runtime.restartStatus()
```

`NETWORK`: Uses the network backend.

Whether a restart is armed, and how long is left.

Requires `runtime.restart`. The read `open77:admin:scheduledRestart` cannot replace: an event announces a CHANGE, and a resource that started after the restart was scheduled never saw one. Ask this once at start, then subscribe. `secondsRemaining` is measured at the moment of the read and is 0 when nothing is armed. Refusals: `permission_denied:runtime.restart`, `restart_control_unavailable`.

Returns: `{ scheduled, secondsRemaining, reason? }, or nil`, `reason`

```lua
-- Warn a joining player about a pending restart
AddEventHandler("onPlayerReady", function(playerId)
    local status = Open77.runtime.restartStatus()
    if status and status.scheduled then
        Open77.chat.send(tonumber(playerId), ("Server restart in %d s"):format(status.secondsRemaining))
    end
end)
```

Registered in `LuaResourceRuntime.RuntimeControl.cs` (line 346) as `LuaResourceRuntime.Bootstrap`.

## scheduleRestart

```lua
Open77.runtime.scheduleRestart(seconds, [reason])
```

`NETWORK`: Uses the network backend.

Schedules a server restart, with the same warnings an operator's would raise.

Requires `runtime.restart` -- its own capability, not `resources.control`: stopping one resource and stopping the whole server are different powers. Takes exactly the path a Warden-scheduled restart takes, so players get the same broadcast now and the same warning ladder (30m, 15m, 10m, 5m, 2m, 60s, 30s, 10s) as the deadline approaches, and every running resource gets `open77:admin:scheduledRestart(secondsRemaining, reason)` on each rung. Calling it again re-arms the schedule rather than adding a second one. `seconds` must be an integer in [1, 604800] and is clamped by the scheduler to at least 5 s; `reason` is broadcast to every player, so it is capped at 200 bytes and must hold no control characters. Refusals: `permission_denied:runtime.restart`, `invalid_delay`, `invalid_reason`, `restart_control_unavailable` (a host with no game loop).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `seconds` | `integer` | required | — |
| `reason` | `string` | optional | — |

Returns: `{ scheduled, secondsRemaining, reason? }, or nil`, `reason`

```lua
-- /restart <minutes> <reason> with the standard warning ladder
RegisterCommand("restart", function(source, args)
    local minutes = tonumber(args[1]) or 15
    Open77.runtime.scheduleRestart(minutes * 60, table.concat(args, " ", 2))
end, true)
```

Registered in `LuaResourceRuntime.RuntimeControl.cs` (line 334) as `LuaResourceRuntime.Bootstrap`.
