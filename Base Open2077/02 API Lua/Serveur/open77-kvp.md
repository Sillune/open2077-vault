# Open77.kvp (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-kvp.md) · [Page web](https://open2077.net/docs/api/server/open77-kvp)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

11 functions.

## clear

```lua
Open77.kvp.clear([prefix])
```

`NETWORK`: Uses the network backend.

Clears this resource's keys matching a prefix.

Server-side; no capability. The store is one JSON file under this resource's own `data/` directory, and no caller-supplied path reaches it, so a resource cannot address another's store -- isolation is structural, which is why this needs no permission where `Open77.io` needs `filesystem.*`. An empty or omitted prefix clears the whole store.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `prefix` | `string` | optional | — |

Returns: `removed count, or nil`, `reason`

```lua
-- Wipe this resource's saved state for a fresh season
RegisterCommand("season-reset", function()
    print("removed", Open77.kvp.clear("season:"), "keys")
end, true)
```

Registered in `LuaResourceRuntime.Kvp.cs` (line 305) as `LuaResourceRuntime.Bootstrap`.

## compareAndSet

```lua
Open77.kvp.compareAndSet(key, expected, replacement)
```

`NETWORK`: Uses the network backend.

Atomically changes a key when its typed value matches.

Server-side; no capability. The store is one JSON file under this resource's own `data/` directory, and no caller-supplied path reaches it, so a resource cannot address another's store -- isolation is structural, which is why this needs no permission where `Open77.io` needs `filesystem.*`. `expected = nil` means "must be absent"; `replacement = nil` deletes. The comparison is TYPED: an integer 1 does not satisfy an expectation written as 1.0, because the store keeps the two apart and a later read would answer differently.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string` | required | — |
| `expected` | `any` | required | — |
| `replacement` | `any` | required | — |

Returns: `boolean`, `reason on storage failure`

```lua
-- Claim the daily bonus only once, atomically
if Open77.kvp.compareAndSet("bonus:" .. identifier, nil, today) then
    giveBonus(playerId)
end
```

Registered in `LuaResourceRuntime.Kvp.cs` (line 308) as `LuaResourceRuntime.Bootstrap`.

## delete

```lua
Open77.kvp.delete(key)
```

`NETWORK`: Uses the network backend.

Deletes one persistent key.

Server-side; no capability. The store is one JSON file under this resource's own `data/` directory, and no caller-supplied path reaches it, so a resource cannot address another's store -- isolation is structural, which is why this needs no permission where `Open77.io` needs `filesystem.*`. Deleting an absent key is `false`, not an error.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string` | required | — |

Returns: `true when removed, false when absent`, `reason on storage failure`

```lua
-- The player sold their house: forget the saved key
Open77.kvp.delete("house:" .. identifier)
```

Registered in `LuaResourceRuntime.Kvp.cs` (line 302) as `LuaResourceRuntime.Bootstrap`.

## find

```lua
Open77.kvp.find([prefix], [limit])
```

`NETWORK`: Uses the network backend.

Finds typed entries by sorted key prefix.

Server-side; no capability. The store is one JSON file under this resource's own `data/` directory, and no caller-supplied path reaches it, so a resource cannot address another's store -- isolation is structural, which is why this needs no permission where `Open77.io` needs `filesystem.*`. Keys are ordinal-sorted. `limit` must be 1..4096; anything else is `invalid_limit` rather than a silent clamp.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `prefix` | `string` | optional | — |
| `limit` | `integer` | optional | `256` |

Returns: `array of { key, value, type }, or nil`, `reason`

```lua
-- A leaderboard from the stored scores
for _, entry in ipairs(Open77.kvp.find("score:", 100) or {}) do
    print(entry.key:sub(7), entry.value)
end
```

Registered in `LuaResourceRuntime.Kvp.cs` (line 303) as `LuaResourceRuntime.Bootstrap`.

## get

```lua
Open77.kvp.get(key, [default])
```

`NETWORK`: Uses the network backend.

Reads one typed persistent value.

Server-side; no capability. The store is one JSON file under this resource's own `data/` directory, and no caller-supplied path reaches it, so a resource cannot address another's store -- isolation is structural, which is why this needs no permission where `Open77.io` needs `filesystem.*`. A missing key answers with the caller's default, or nil, and NO reason -- a reason there would be indistinguishable from a storage failure at the call site.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string` | required | — |
| `default` | `any` | optional | — |

Returns: `stored value, default or nil`, `reason on storage failure`

```lua
-- Money persisted without a database
local balance = Open77.kvp.get("money:" .. identifier, 500)
```

Registered in `LuaResourceRuntime.Kvp.cs` (line 300) as `LuaResourceRuntime.Bootstrap`.

## has

```lua
Open77.kvp.has(key)
```

`NETWORK`: Uses the network backend.

Checks whether a persistent key exists.

Server-side; no capability. The store is one JSON file under this resource's own `data/` directory, and no caller-supplied path reaches it, so a resource cannot address another's store -- isolation is structural, which is why this needs no permission where `Open77.io` needs `filesystem.*`. 

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string` | required | — |

Returns: `boolean`, `reason on storage failure`

```lua
-- First join on this server?
if not Open77.kvp.has("seen:" .. identifier) then
    Open77.kvp.set("seen:" .. identifier, true)
    Open77.chat.send(playerId, "Welcome to the city, first-timer!")
end
```

Registered in `LuaResourceRuntime.Kvp.cs` (line 301) as `LuaResourceRuntime.Bootstrap`.

## increment

```lua
Open77.kvp.increment(key, [delta])
```

`NETWORK`: Uses the network backend.

Atomically increments a numeric key.

Server-side; no capability. The store is one JSON file under this resource's own `data/` directory, and no caller-supplied path reaches it, so a resource cannot address another's store -- isolation is structural, which is why this needs no permission where `Open77.io` needs `filesystem.*`. Atomic in the sense that matters: the server Lua runtime is one thread, so a read-modify-write cannot interleave with another resource's. An absent key starts at zero. Integer plus integer stays an integer, so a counter never drifts into float territory. `integer_overflow`, `number_overflow`, `value_not_numeric` and `delta_not_numeric` are named rather than wrapped.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string` | required | — |
| `delta` | `integer|number` | optional | `1` |

Returns: `new numeric value, or nil`, `reason`

```lua
-- One kill counter per player, persisted
AddEventHandler("onPlayerLifeStateChanged", function(playerId, revision, phase, reason)
    if phase ~= "dead" then return end
    local life = Open77.players.getLifeState(playerId)
    -- killer is 0, not nil, when nobody did it; a scripted kill (join spawn, admin move) has none
    if life and life.killer ~= 0 and life.cause ~= "script" then
        Open77.kvp.increment("kills:" .. Open77.players.identifier(life.killer), 1)
    end
end)
```

Registered in `LuaResourceRuntime.Kvp.cs` (line 306) as `LuaResourceRuntime.Bootstrap`.

## keys

```lua
Open77.kvp.keys([prefix], [limit])
```

`NETWORK`: Uses the network backend.

Lists sorted keys matching a prefix.

Server-side; no capability. The store is one JSON file under this resource's own `data/` directory, and no caller-supplied path reaches it, so a resource cannot address another's store -- isolation is structural, which is why this needs no permission where `Open77.io` needs `filesystem.*`. 

| Parameter | Type | Required | Default |
|---|---|---|---|
| `prefix` | `string` | optional | — |
| `limit` | `integer` | optional | `256` |

Returns: `string array, or nil`, `reason`

```lua
-- Every player who has a saved balance
for _, key in ipairs(Open77.kvp.keys("money:") or {}) do print(key) end
```

Registered in `LuaResourceRuntime.Kvp.cs` (line 304) as `LuaResourceRuntime.Bootstrap`.

## set

```lua
Open77.kvp.set(key, value)
```

`NETWORK`: Uses the network backend.

Persists one typed value for this resource.

Server-side; no capability. The store is one JSON file under this resource's own `data/` directory, and no caller-supplied path reaches it, so a resource cannot address another's store -- isolation is structural, which is why this needs no permission where `Open77.io` needs `filesystem.*`. Quotas are the client store's numbers exactly: 256-byte keys, 64 KiB values, 4096 entries, 1 MiB of payload, same reason tokens. Values are string, integer, number or boolean; integer and number stay apart, so a counter read back from disk is still an integer. A refused write leaves the store exactly as it was. Writes are coalesced onto the tick and flushed again when the resource stops, so an orderly stop, reload or restart loses nothing and a hard kill loses at most one tick. A store that cannot be parsed reports `storage_corrupt` and is never overwritten with a fresh one.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string` | required | — |
| `value` | `string|integer|number|boolean` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Save the player's money when they leave
AddEventHandler("onPlayerDisconnected", function(playerId)
    Open77.kvp.set("money:" .. Open77.players.identifier(playerId), wallet[playerId] or 0)
end)
```

Registered in `LuaResourceRuntime.Kvp.cs` (line 299) as `LuaResourceRuntime.Bootstrap`.

## setIfAbsent

```lua
Open77.kvp.setIfAbsent(key, value)
```

`NETWORK`: Uses the network backend.

Stores a value only when its key is missing.

Server-side; no capability. The store is one JSON file under this resource's own `data/` directory, and no caller-supplied path reaches it, so a resource cannot address another's store -- isolation is structural, which is why this needs no permission where `Open77.io` needs `filesystem.*`. The cheap way to claim a lock without a database.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string` | required | — |
| `value` | `string|integer|number|boolean` | required | — |

Returns: `boolean`, `reason on storage failure`

```lua
-- A lock without a database: only one heist at a time
if not Open77.kvp.setIfAbsent("heist:lock", true) then
    return Open77.chat.send(source, "A heist is already running.")
end
```

Registered in `LuaResourceRuntime.Kvp.cs` (line 307) as `LuaResourceRuntime.Bootstrap`.

## stats

```lua
Open77.kvp.stats()
```

`NETWORK`: Uses the network backend.

Returns this resource store's usage and quotas.

Server-side; no capability. The store is one JSON file under this resource's own `data/` directory, and no caller-supplied path reaches it, so a resource cannot address another's store -- isolation is structural, which is why this needs no permission where `Open77.io` needs `filesystem.*`. 

Returns: `table { entries, bytes, maximumEntries, maximumBytes, maximumKeyBytes, maximumValueBytes, resource }, or nil`, `reason`

```lua
-- Log the store usage at startup
local stats = Open77.kvp.stats()
if stats then print(("kvp: %d entries, %d bytes"):format(stats.entries, stats.bytes)) end
```

Registered in `LuaResourceRuntime.Kvp.cs` (line 311) as `LuaResourceRuntime.Bootstrap`.
