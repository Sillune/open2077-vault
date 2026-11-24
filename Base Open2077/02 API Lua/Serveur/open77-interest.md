# Open77.interest (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-interest.md) · [Page web](https://open2077.net/docs/api/server/open77-interest)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

3 functions.

## clear

```lua
Open77.interest.clear(playerId)
```

`NETWORK`: Uses the network backend.

Removes every interest point this resource holds for one player.

Requires `world.interest`. Returns the number of points removed, including `0` when this generation had none, or `false, reason` on a permission or argument failure. Use it when a viewer stops watching all of this resource's remote points; `remove` releases one named point.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `removed count (integer)`, `false, reason on failure`

```lua
-- Every named point this resource holds for the player, in one call
local removed = Open77.interest.clear(playerId)
print("cleared " .. tostring(removed))
```

Registered in `LuaResourceRuntime.cs` (line 7983) as `LuaResourceRuntime.Bootstrap`.

## remove

```lua
Open77.interest.remove(playerId, id)
```

`NETWORK`: Uses the network backend.

Removes one named replication-interest point from one player.

Requires `world.interest`. Only the exact resource generation that created the point can remove it, and only for the player it was created for: a name this generation never set answers `not_found`, which is also what a replacement generation sees after a reload. Removing the last point for a player drops the player's row, and player disconnect or resource stop cleans the rest up. Returns `true`, or `false, reason`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `id` | `string` | required | — |

Returns: `true, or false, reason`

```lua
local ok, reason = Open77.interest.remove(playerId, "camera:loading_bay")
if not ok and reason ~= "not_found" then
    print("interest remove refused: " .. tostring(reason))
end
```

Registered in `LuaResourceRuntime.cs` (line 7980) as `LuaResourceRuntime.Bootstrap`.

## set

```lua
Open77.interest.set(playerId, id, options)
```

`NETWORK`: Uses the network backend.

Creates or replaces this resource's named replication-interest point for one player.

Requires `world.interest`. `options` is `{position={x,y,z}, radius, bucket}`: `radius` is metres (1..500) and `bucket` must be the player's current routing bucket, re-checked against the authoritative routing registry on every call. The point belongs to the calling resource generation and to that player; a second `set` with the same player and id replaces the first, and another resource cannot read, replace or remove it. Returns `true`, or `false, reason`: `permission_denied:world.interest`, `interest_unavailable`, `player_not_found`, `bucket_mismatch`, `player_interest_limit`, `interest_limit` or `stale_generation`. Replication admission is a separate budget from native capture admission, and an accepted point is a streaming request, not a streaming-complete acknowledgement.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `id` | `string` | required | — |
| `options` | `table` | required | — |

Returns: `true, or false, reason`

```lua
-- Keep a viewer's neighbourhood streaming around the camera he is watching
local ok, reason = Open77.interest.set(playerId, "camera:loading_bay", {
    position = { x = -699.094, y = 2164.29, z = 54.7051 },
    radius = 120,
    bucket = 0,
})
if not ok then
    print("interest refused: " .. tostring(reason))
end
```

Registered in `LuaResourceRuntime.cs` (line 7977) as `LuaResourceRuntime.Bootstrap`.
