# Open77.stats (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-stats.md) · [Page web](https://open2077.net/docs/api/client/open77-stats)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

5 functions.

## get

```lua
Open77.stats.get([playerId])
```

`GAME`: Requires a live game instance.

Reads the canonical health and stamina state for one player.

Requires `players.stats.read` (or the legacy `players.life.read`). Omit `playerId` for the local player. Returns the same combined shape as the server API, including both point pools, maximums, fractions, regeneration flags/rates, armor, god mode, damage attribution and revisions. Client resources cannot mutate these server-owned values.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `combined stats table, or nil`

```lua
-- A custom HUD reads both pools of the local player
SetTick(function()
    local stats = Open77.stats.get()
    if stats then hud.set(stats.health.fraction, stats.stamina.fraction) end
end)
```

Registered in `ResourceHost.cpp` (line 24804) as `LuaStatsGet`.

## getHealth

```lua
Open77.stats.getHealth([playerId])
```

`GAME`: Requires a live game instance.

Reads one player's canonical health pool.

Requires `players.stats.read` (or `players.life.read`). Omit `playerId` for the local player. The table contains `value/current`, `maximum/max`, `fraction`, `percentage`, `regenEnabled`, `regenPerSecond` and `regenerating`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `health pool table, or nil`

```lua
-- Flash the screen when health drops under 25 %
local hp = Open77.stats.getHealth()
if hp and hp.fraction < 0.25 then flashRed() end
```

Registered in `ResourceHost.cpp` (line 24806) as `LuaStatsPool`.

## getStamina

```lua
Open77.stats.getStamina([playerId])
```

`GAME`: Requires a live game instance.

Reads one player's canonical stamina pool.

Requires `players.stats.read` (or `players.life.read`). Omit `playerId` for the local player. The table contains `value/current`, `maximum/max`, `fraction`, `percentage`, `regenEnabled`, `regenPerSecond` and `regenerating`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `stamina pool table, or nil`

```lua
-- Grey out the sprint icon when stamina is empty
local stamina = Open77.stats.getStamina()
sprintIcon.enabled = stamina and stamina.value > 0
```

Registered in `ResourceHost.cpp` (line 24810) as `LuaStatsPool`.

## health

```lua
Open77.stats.health([playerId])
```

`GAME`: Requires a live game instance.

Alias of `Open77.stats.getHealth`.

Reads the replicated server-owned health pool. This alias has exactly the same permission, optional player id and return shape as `getHealth`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `health pool table, or nil`

```lua
-- Short spelling: another player's health on the scoreboard
local hp = Open77.stats.health(playerId)
if hp then print(("%d %%"):format(hp.percentage)) end
```

Registered in `ResourceHost.cpp` (line 24808) as `LuaStatsPool`.

## stamina

```lua
Open77.stats.stamina([playerId])
```

`GAME`: Requires a live game instance.

Alias of `Open77.stats.getStamina`.

Reads the replicated server-owned stamina pool. This alias has exactly the same permission, optional player id and return shape as `getStamina`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `stamina pool table, or nil`

```lua
-- Short spelling: is the player too tired to climb?
local stamina = Open77.stats.stamina()
if stamina and stamina.fraction < 0.1 then Open77.hud.notify("Too tired to climb") end
```

Registered in `ResourceHost.cpp` (line 24812) as `LuaStatsPool`.
