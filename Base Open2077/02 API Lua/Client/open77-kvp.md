# Open77.kvp (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-kvp.md) · [Page web](https://open2077.net/docs/api/client/open77-kvp)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

11 functions.

## clear

```lua
Open77.kvp.clear([prefix])
```

`CLIENT`: Client-only surface.

Clears this resource's keys matching a prefix.

Atomically removes every key matching an optional prefix from the calling resource's current-server store and returns the removal count. An empty prefix clears only that resource namespace.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `prefix` | `string` | optional | — |

Returns: `removed count, or nil`, `reason`

```lua
-- Reset every saved setting of this resource on this server
RegisterCommand("resetsettings", function()
    local removed = Open77.kvp.clear()
    Open77.hud.notify(("Cleared %d settings"):format(removed or 0))
end, false)
```

Registered in `ResourceHost.cpp` (line 24600) as `LuaKvpClear`.

## compareAndSet

```lua
Open77.kvp.compareAndSet(key, expected, replacement)
```

`CLIENT`: Client-only surface.

Atomically changes a key when its typed value matches.

Atomically replaces a key only when its typed value exactly matches the expected value. Nil expected means the key must be absent; nil replacement deletes after a successful comparison.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string` | required | — |
| `expected` | `any` | required | — |
| `replacement` | `any` | required | — |

Returns: `boolean`, `reason on storage failure`

```lua
-- Only mark the tutorial done if it was still pending
Open77.kvp.compareAndSet("tutorial", "pending", "done")
```

Registered in `ResourceHost.cpp` (line 24603) as `LuaKvpCas`.

## delete

```lua
Open77.kvp.delete(key)
```

`CLIENT`: Client-only surface.

Deletes one persistent key.

Atomically deletes one key from the current resource store and returns whether it existed. Deleting a missing key succeeds with false.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string` | required | — |

Returns: `true when removed, false when absent`, `reason on storage failure`

```lua
-- Forget the saved HUD layout
Open77.kvp.delete("hud_layout")
```

Registered in `ResourceHost.cpp` (line 24597) as `LuaKvpDelete`.

## find

```lua
Open77.kvp.find([prefix], [limit])
```

`CLIENT`: Client-only surface.

Finds typed entries by sorted key prefix.

Performs a sorted prefix search in the current resource store and returns bounded `{key, value, type}` records. The default result limit is 256 and the hard limit is 4096.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `prefix` | `string` | optional | — |
| `limit` | `integer` | optional | `256` |

Returns: `array of { key, value, type }, or nil`, `reason`

```lua
-- List every saved radio favourite with its value
for _, entry in ipairs(Open77.kvp.find("radio:") or {}) do
    print(entry.key, entry.value, entry.type)
end
```

Registered in `ResourceHost.cpp` (line 24598) as `LuaKvpFind`.

## get

```lua
Open77.kvp.get(key, [default])
```

`CLIENT`: Client-only surface.

Reads one typed persistent value.

Reads a typed value from the current connection-address and resource namespace. Missing keys return the optional default (or nil) and are distinct from storage errors, which return `nil, reason`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string` | required | — |
| `default` | `any` | optional | — |

Returns: `stored value, default or nil`, `reason on storage failure`

```lua
-- Remember the last radio station across sessions on this server
local station = Open77.kvp.get("radio:last", "night_fm")
```

Registered in `ResourceHost.cpp` (line 24595) as `LuaKvpGet`.

## has

```lua
Open77.kvp.has(key)
```

`CLIENT`: Client-only surface.

Checks whether a persistent key exists.

Checks whether a key exists in the calling resource's current-server store without exposing its value or any other resource namespace.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string` | required | — |

Returns: `boolean`, `reason on storage failure`

```lua
-- Show the tutorial only the first time
if not Open77.kvp.has("tutorial_seen") then
    showTutorial()
    Open77.kvp.set("tutorial_seen", true)
end
```

Registered in `ResourceHost.cpp` (line 24596) as `LuaKvpHas`.

## increment

```lua
Open77.kvp.increment(key, [delta])
```

`CLIENT`: Client-only surface.

Atomically increments a numeric key.

Atomically creates or increments a numeric key. Integer arithmetic remains 64-bit and rejects overflow; mixed/floating arithmetic rejects non-finite results.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string` | required | — |
| `delta` | `integer|number` | optional | `1` |

Returns: `new numeric value, or nil`, `reason`

```lua
-- Count how many times the player opened the map
local opens = Open77.kvp.increment("map_opens", 1)
```

Registered in `ResourceHost.cpp` (line 24601) as `LuaKvpIncrement`.

## keys

```lua
Open77.kvp.keys([prefix], [limit])
```

`CLIENT`: Client-only surface.

Lists sorted keys matching a prefix.

Returns only the sorted keys matching an optional prefix in the current resource store. It is a bounded index query and cannot enumerate another package or connection address.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `prefix` | `string` | optional | — |
| `limit` | `integer` | optional | `256` |

Returns: `string array, or nil`, `reason`

```lua
-- Which settings has the player customised?
for _, key in ipairs(Open77.kvp.keys("setting:") or {}) do print(key) end
```

Registered in `ResourceHost.cpp` (line 24599) as `LuaKvpFind`.

## set

```lua
Open77.kvp.set(key, value)
```

`CLIENT`: Client-only surface.

Persists one typed value for this server address and resource.

Atomically persists a string, signed integer, finite number or boolean under the current connection address and resource. Keys/values and the 1 MiB resource quota are validated before the old file is replaced.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string` | required | — |
| `value` | `string|integer|number|boolean` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Save the chosen HUD position on this machine, for this server
Open77.kvp.set("hud_position", "bottom_right")
```

Registered in `ResourceHost.cpp` (line 24594) as `LuaKvpSet`.

## setIfAbsent

```lua
Open77.kvp.setIfAbsent(key, value)
```

`CLIENT`: Client-only surface.

Stores a value only when its key is missing.

Redis-style SETNX for the current resource namespace: atomically stores the typed value only when the key is missing and reports whether insertion occurred.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string` | required | — |
| `value` | `string|integer|number|boolean` | required | — |

Returns: `boolean`, `reason on storage failure`

```lua
-- Default settings on first run only
Open77.kvp.setIfAbsent("hud_scale", 1.0)
```

Registered in `ResourceHost.cpp` (line 24602) as `LuaKvpSetNx`.

## stats

```lua
Open77.kvp.stats()
```

`CLIENT`: Client-only surface.

Returns this resource store's usage and quotas.

Returns this resource's entry/byte usage and enforced quotas together with the active address/resource scope. It exposes no keys or usage belonging to other resources.

Returns: `table, or nil`, `reason`

```lua
-- Warn before the resource's store fills up
local stats = Open77.kvp.stats()
if stats and stats.bytes > stats.maximumBytes * 0.9 then print("kvp store nearly full") end
```

Registered in `ResourceHost.cpp` (line 24604) as `LuaKvpStats`.
