# Open77.tunables (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-tunables.md) · [Page web](https://open2077.net/docs/api/server/open77-tunables)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

6 functions.

## capture

```lua
Open77.tunables.capture()
```

`NETWORK`: Uses the network backend.

A frozen plain table of every current value.

No permission. Unlike the `declare` proxy, this does not read through: it is a snapshot to pin onto one match, which is the only correct form for a mode running several rounds at once.

Returns: `table of key to value`

```lua
-- Pin the rules for this match so a mid-match panel write cannot change them
local rules = Open77.tunables.capture()
startMatch(rules)
```

Registered in `LuaResourceRuntime.cs` (line 8119) as `LuaResourceRuntime.Bootstrap`.

## declare

```lua
Open77.tunables.declare(declaration)
```

`NETWORK`: Uses the network backend.

Declares this resource's operator-tunable settings and returns a live proxy.

No permission. Declaration fields are `value` (the required default), `type` (`number`, `integer`, `boolean`, `string`, `enum`), `min`, `max`, `step`, `choices`, `unit`, `apply` (`live`, `next_round`, `next_match`), `label`, `description`, `group` and `order`. `min`, `max`, `type` and `choices` are enforced on every write; `step` and `unit` are presentation only. Limits: 128 tunables per resource, 64 KiB of declaration, 32 choices, 256-character strings. **The proxy reads through to the host on every access** -- read it at the point of use, because a value hoisted into a file-scope local is captured at load time and never updates while the admin panel goes on reporting the new number. A malformed declaration raises; an undeclared key read through the proxy raises rather than answering nil.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `declaration` | `table` | required | — |

Returns: `live proxy table`

```lua
-- Operator-tunable settings, editable live from the Warden panel
local settings = Open77.tunables.declare({
    roundSeconds = { value = 300, type = "integer", min = 60, max = 1800, unit = "s", apply = "next_round" },
    friendlyFire = { value = false, type = "boolean", apply = "live" },
})
print("round length", settings.roundSeconds) -- reads through to the host every time
```

Registered in `LuaResourceRuntime.cs` (line 8116) as `LuaResourceRuntime.Bootstrap`.

## get

```lua
Open77.tunables.get(key)
```

`NETWORK`: Uses the network backend.

The current value of one tunable.

No permission. Answers `nil, reason` for an undeclared key. A key declared `next_round` or `next_match` does not reach this until the resource calls `promote()`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string` | required | — |

Returns: `value, or nil`, `reason`

```lua
-- Read one value at the point of use
local seconds = Open77.tunables.get("roundSeconds")
```

Registered in `LuaResourceRuntime.cs` (line 8117) as `LuaResourceRuntime.Bootstrap`.

## pending

```lua
Open77.tunables.pending()
```

`NETWORK`: Uses the network backend.

The changes waiting on a round or match boundary.

No permission. Key to value for everything an operator has written that `promote()` has not yet adopted.

Returns: `table of key to value`

```lua
-- Tell the admin what will change at the next round
for key, value in pairs(Open77.tunables.pending()) do
    print(("next round: %s = %s"):format(key, tostring(value)))
end
```

Registered in `LuaResourceRuntime.cs` (line 8120) as `LuaResourceRuntime.Bootstrap`.

## promote

```lua
Open77.tunables.promote()
```

`NETWORK`: Uses the network backend.

Adopts every pending change and returns the keys that moved.

No permission. Call it at the boundary the resource declared -- the start of a round or a match -- so a mid-match write cannot change the rules under the players.

Returns: `array of promoted keys`

```lua
-- Adopt operator changes at the round boundary
AddEventHandler("arena:roundStarting", function()
    for _, key in ipairs(Open77.tunables.promote()) do print("applied", key) end
end)
```

Registered in `LuaResourceRuntime.cs` (line 8121) as `LuaResourceRuntime.Bootstrap`.

## set

```lua
Open77.tunables.set(key, value)
```

`NETWORK`: Uses the network backend.

Writes one tunable, with the same validation as the admin panel.

No permission. Returns `ok, message, pending`; `pending` is true when the key's `apply` policy holds the value at a boundary. Values are owned by the host, so they survive a reload **and** a stop and come back after a restart from `tunables.json`. The owning resource -- and no other -- receives `onTunableChanged(key, value, pending)` after every accepted write.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string` | required | — |
| `value` | `number|integer|boolean|string` | required | — |

Returns: `ok (boolean)`, `message`, `pending (boolean)`

```lua
-- /roundtime <seconds> writes the tunable with the same validation as the panel
RegisterCommand("roundtime", function(source, args)
    local ok, message, pending = Open77.tunables.set("roundSeconds", tonumber(args[1]))
    if not ok then Open77.chat.send(source, message) elseif pending then Open77.chat.send(source, "Applies next round.") end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8118) as `LuaResourceRuntime.Bootstrap`.
