# Open77.time (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-time.md) · [Page web](https://open2077.net/docs/api/server/open77-time)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

3 functions.

## monotonic

```lua
Open77.time.monotonic()
```

`NETWORK`: Uses the network backend.

Server-monotonic seconds since the process started.

No permission. The same unit and meaning as the client `Open77.time.monotonic`. Use it for durations: it never jumps backwards, unlike wall-clock time.

Returns: `seconds (number)`

```lua
-- Measure how long a round lasted
local startedAt = Open77.time.monotonic()
AddEventHandler("arena:roundOver", function()
    print(("round lasted %.0f s"):format(Open77.time.monotonic() - startedAt))
end)
```

Registered in `LuaResourceRuntime.cs` (line 8130) as `LuaResourceRuntime.Bootstrap`.

## unix

```lua
Open77.time.unix()
```

`NETWORK`: Uses the network backend.

Wall-clock seconds since 1970 UTC, fractional.

No permission. `GetUnixTime` is the same function. The sandbox has no `os` library, so this is how an expiring ban or a scheduled event gets the real date.

Returns: `seconds (number)`

```lua
-- A ban that expires in 24 hours
bans[userId] = { until_ = Open77.time.unix() + 86400 }
```

Registered in `LuaResourceRuntime.cs` (line 8130) as `LuaResourceRuntime.Bootstrap`.

## utc

```lua
Open77.time.utc()
```

`NETWORK`: Uses the network backend.

The current instant as an ISO 8601 string.

No permission. `GetUtcTimestamp` is the same function. The same instant as `unix`, formatted for a log line or a stored record.

Returns: `ISO 8601 string`

```lua
-- Timestamp a log line
print(Open77.time.utc(), "player", source, "bought", itemId)
```

Registered in `LuaResourceRuntime.cs` (line 8130) as `LuaResourceRuntime.Bootstrap`.
