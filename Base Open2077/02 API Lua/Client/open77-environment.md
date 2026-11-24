# Open77.environment (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-environment.md) · [Page web](https://open2077.net/docs/api/client/open77-environment)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

6 functions.

## getTime

```lua
Open77.environment.getTime()
```

`GAME`: Requires a live game instance.

Lit l'horloge REDengine locale.

Primitive interne protégée par `world.environment`. Retourne un snapshot atomique de l'heure appliquée et de l'état de pause. Dans une session normale, utilisez l'export `open77_weather.getState` plutôt que cette valeur de projection.

Returns: `table { day, hour, minute, second, totalSeconds, frozen }, ou nil`, `raison si indisponible`

```lua
-- Show the game clock on the HUD (the client projection of the server's time)
SetTick(function()
    local t = Open77.environment.getTime()
    if t then hud.clock = ("%02d:%02d"):format(t.hour, t.minute) end
end)
```

Registered in `ResourceHost.cpp` (line 25169) as `LuaEnvironmentTime`.

## isWeatherFrozen

```lua
Open77.environment.isWeatherFrozen()
```

`GAME`: Requires a live game instance.

Indique si la météo automatique vanilla est neutralisée.

Demande `world.environment`.

Returns: `boolean, ou nil`, `raison si indisponible`

```lua
-- Debug: is vanilla weather neutralised on this client?
print("weather frozen:", Open77.environment.isWeatherFrozen())
```

Registered in `ResourceHost.cpp` (line 25174) as `LuaEnvironmentWeatherFrozen`.

## setTime

```lua
Open77.environment.setTime(hour, minute, [second])
```

`GAME`: Requires a live game instance.

Projette une heure serveur dans REDengine.

Demande `world.environment`. `open77_weather` est normalement le seul propriétaire de cette permission. REDengine choisit la prochaine occurrence de l'heure demandée ; le package officiel filtre donc les petits reculs réseau pour éviter un saut d'un jour.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `hour` | `integer 0..23` | required | — |
| `minute` | `integer 0..59` | required | — |
| `second` | `integer 0..59` | optional | `0` |

Returns: `true en cas de succès, sinon false`, `raison du refus`

```lua
-- A local projection primitive (normally only open77_weather holds this permission)
Open77.environment.setTime(22, 30)
```

Registered in `ResourceHost.cpp` (line 25170) as `LuaEnvironmentSetTime`.

## setTimeFrozen

```lua
Open77.environment.setTimeFrozen(frozen)
```

`GAME`: Requires a live game instance.

Fige ou libère uniquement l'horloge du jeu.

Ne fige pas la simulation. Demande `world.environment`; utilisé par la projection serveur pour neutraliser la vitesse vanilla.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `frozen` | `boolean` | required | — |

Returns: `true en cas de succès, sinon false`, `raison du refus`

```lua
-- Stop only the clock on this client (the projection layer's job, not a gamemode's)
Open77.environment.setTimeFrozen(true)
```

Registered in `ResourceHost.cpp` (line 25171) as `LuaEnvironmentFreezeTime`.

## setWeather

```lua
Open77.environment.setWeather(preset, [transitionSeconds], [priority])
```

`GAME`: Requires a live game instance.

Applique un preset météo REDengine.

Primitive interne demandant `world.environment`. Le second retour `applied=false` est aussi possible en cas de succès si le preset était déjà actif.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `preset` | `string` | required | — |
| `transitionSeconds` | `number` | optional | `0` |
| `priority` | `integer` | optional | `5` |

Returns: `true en cas de succès, sinon false`, `applied si succès, sinon raison du refus`

```lua
-- Local projection of a preset over 20 seconds (normally only open77_weather calls it)
local ok, applied = Open77.environment.setWeather("rain", 20)
```

Registered in `ResourceHost.cpp` (line 25172) as `LuaEnvironmentSetWeather`.

## setWeatherFrozen

```lua
Open77.environment.setWeatherFrozen(frozen)
```

`GAME`: Requires a live game instance.

Désactive ou réactive le contrôleur météo automatique vanilla.

Demande `world.environment`. Figer conserve le preset courant et laisse le serveur décider du prochain événement.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `frozen` | `boolean` | required | — |

Returns: `true en cas de succès, sinon false`, `raison du refus`

```lua
-- Keep the vanilla controller from overriding the server's sky
Open77.environment.setWeatherFrozen(true)
```

Registered in `ResourceHost.cpp` (line 25173) as `LuaEnvironmentFreezeWeather`.
