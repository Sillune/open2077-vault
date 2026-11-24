# Open77.environment (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-environment.md) · [Page web](https://open2077.net/docs/api/server/open77-environment)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

8 functions.

## clearBucket

```lua
Open77.environment.clearBucket(bucket)
```

`NETWORK`: Uses the network backend.

Retires one routing bucket's environment override.

Requires `world.environment`. The players still standing in that bucket are re-synced onto the default environment with a revision newer than the one they last applied, so nobody is left on the sky of a world that no longer exists. Once the last override is gone the authority returns to a single broadcast per mutation instead of one send per connected player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `number` | required | — |

Returns: `state table, or nil`, `reason: permission_denied:world.environment, environment_unavailable, invalid_bucket, unknown_bucket`

```lua
-- The arena match is over: the instance goes back to the default sky
Open77.environment.clearBucket(ARENA_BUCKET)
```

Registered in `LuaResourceRuntime.Environment.cs` (line 178) as `LuaResourceRuntime.Bootstrap`.

## getState

```lua
Open77.environment.getState(bucket)
```

`NETWORK`: Uses the network backend.

The canonical clock and weather state, right now.

Requires `world.environment`. Carries `scope` and `bucket`, the clock as both `secondsOfDay` and `hour`/`minute`/`second`, `rate`, `frozen`/`timeFrozen`, `weather`/`weatherPreset`/`weatherPriority`, `transitionSeconds` and `weatherTransitionRemainingMs`, `weatherFrozen`/`randomWeather`/`nextWeatherInMs`, the `revision`/`weatherRevision`/`authorityEpoch` triple clients use to reject a stale snapshot, and `buckets` — every routing bucket currently holding an override. It is the same table every setter returns and the same payload `onEnvironmentChanged` carries.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `number?` | required | — |

Returns: `state table, or nil`, `reason: permission_denied:world.environment, environment_unavailable, invalid_bucket, unknown_bucket`

```lua
-- /time: tell the player the in-game time
RegisterCommand("time", function(source)
    local env = Open77.environment.getState()
    if env then Open77.chat.send(source, ("It is %02d:%02d, %s"):format(env.hour, env.minute, env.weather)) end
end, false)
```

Registered in `LuaResourceRuntime.Environment.cs` (line 175) as `LuaResourceRuntime.Bootstrap`.

## publishChange

```lua
Open77.environment.publishChange(state)
```

`NETWORK`: Uses the network backend.

Publishes the reserved `onEnvironmentChanged` event host-wide.

Requires `world.environment`. The authority's own door: `onEnvironmentChanged` is a platform name reserved on the event bus, so `TriggerEvent` refuses it with `reserved_event` and this is the only way it can be published. Same queue, same by-value marshalling and same tick boundary as `TriggerEvent`. No resource other than the one holding the environment state needs to call it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `state` | `table` | required | — |

Returns: `true, or false`, `reason: permission_denied:world.environment, resource_stopping, resource_preparing, event_payload_not_serializable, event_queue_limit`

```lua
-- Only the environment authority calls this; a gamemode listens instead
AddEventHandler("onEnvironmentChanged", function(state)
    print("weather is now", state.weather)
end)
```

Registered in `LuaResourceRuntime.Environment.cs` (line 184) as `LuaResourceRuntime.Bootstrap`.

## setTime

```lua
Open77.environment.setTime(hour, minute, second, bucket)
```

`NETWORK`: Uses the network backend.

Sets the authoritative time of day every player projects.

Requires `world.environment`. The host installs this facade in every VM; the authority itself is the bundled `open77_weather` resource, reached through a synchronous export call, because the only code able to move a sky is that resource's CLIENT projection. `bucket` is optional: omitted it moves the default environment, given it creates or updates that routing bucket's override, seeded from the default as it reads at that moment. Answers `environment_unavailable` when the authority is not running — which `resources.load` can cause silently. See [the guide](https://open2077.net/docs/api/server/weather.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `hour` | `number` | required | — |
| `minute` | `number` | required | — |
| `second` | `number` | required | — |
| `bucket` | `number?` | required | — |

Returns: `state table, or nil`, `reason: permission_denied:world.environment, environment_unavailable, invalid_time, invalid_bucket, too_many_environment_overrides`

```lua
-- /settime <hour> <minute> for admins
RegisterCommand("settime", function(source, args)
    Open77.environment.setTime(tonumber(args[1]) or 12, tonumber(args[2]) or 0, 0)
end, true)
```

Registered in `LuaResourceRuntime.Environment.cs` (line 160) as `LuaResourceRuntime.Bootstrap`.

## setTimeFrozen

```lua
Open77.environment.setTimeFrozen(frozen, bucket)
```

`NETWORK`: Uses the network backend.

Stops or resumes the authoritative clock where it stands.

Requires `world.environment`. Freezing rebases first, so the clock stops at the reading it had rather than at its last anchor, and resuming continues from there. This is the server-side clock only: the client projection never holds REDengine's `SetPausedState`, which two-client runtime testing showed also slows gameplay and vehicle physics. `bucket` scopes the change to one routing bucket.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `frozen` | `boolean` | required | — |
| `bucket` | `number?` | required | — |

Returns: `state table, or nil`, `reason: permission_denied:world.environment, environment_unavailable, invalid_argument, invalid_bucket`

```lua
-- Freeze the clock at night for the horror event
Open77.environment.setTime(2, 0, 0)
Open77.environment.setTimeFrozen(true)
```

Registered in `LuaResourceRuntime.Environment.cs` (line 163) as `LuaResourceRuntime.Bootstrap`.

## setTimeRate

```lua
Open77.environment.setTimeRate(rate, bucket)
```

`NETWORK`: Uses the network backend.

Sets how many game seconds pass per real second.

Requires `world.environment`. Accepts `0` to `120`; the shipped default is `8.0`, which MUST match the engine's own rate or the projection has to correct the clock continuously — and every correction is a `SetGameTimeByHMS` world time jump that makes the streamer re-resolve nodes and resurrect destroyed props (measured 25 Aug). Changing day length properly needs a native rate control, not this.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `rate` | `number` | required | — |
| `bucket` | `number?` | required | — |

Returns: `state table, or nil`, `reason: permission_denied:world.environment, environment_unavailable, rate_must_be_between_0_and_120, invalid_bucket`

```lua
-- Slow days: one game hour per 15 real minutes
Open77.environment.setTimeRate(4)
```

Registered in `LuaResourceRuntime.Environment.cs` (line 166) as `LuaResourceRuntime.Bootstrap`.

## setWeather

```lua
Open77.environment.setWeather(preset, transitionSeconds, bucket)
```

`NETWORK`: Uses the network backend.

Applies a synchronized weather preset over a transition.

Requires `world.environment`. `preset` is an Open77 name (`sunny`, `lightclouds`, `cloudy`, `rain`, `heavyclouds`, `fog`, `pollution`, `sandstorm`), one of the configured aliases, or the REDengine `24h_weather_*` value. `transitionSeconds` defaults to the preset's own and must be `0` to `300`. A player who joins mid-transition picks it up at its current point rather than restarting it. `bucket` scopes the change to one routing bucket.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `preset` | `string` | required | — |
| `transitionSeconds` | `number?` | required | — |
| `bucket` | `number?` | required | — |

Returns: `state table, or nil`, `reason: permission_denied:world.environment, environment_unavailable, unknown_weather, invalid_transition, transition_must_be_between_0_and_300, invalid_bucket`

```lua
-- Rain in the arena instance only, over 20 seconds
Open77.environment.setWeather("rain", 20, ARENA_BUCKET)
```

Registered in `LuaResourceRuntime.Environment.cs` (line 169) as `LuaResourceRuntime.Bootstrap`.

## setWeatherFrozen

```lua
Open77.environment.setWeatherFrozen(frozen, bucket)
```

`NETWORK`: Uses the network backend.

Pins the current preset by stopping the random scheduler.

Requires `world.environment`. The name means something different on each runtime, deliberately: on the CLIENT it is an unconditional lock against a vanilla controller submitting its own preset, and is not an operator choice; on the SERVER it means pin the preset — the weighted random draw stops, so the sky stays where it was put. It is the same switch `weather.random off` throws, and it is the half a competitive gamemode wants, because lighting is a fairness variable.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `frozen` | `boolean` | required | — |
| `bucket` | `number?` | required | — |

Returns: `state table, or nil`, `reason: permission_denied:world.environment, environment_unavailable, invalid_argument, invalid_bucket`

```lua
-- Competitive mode: pin the weather so lighting is fair
Open77.environment.setWeather("sunny", 0)
Open77.environment.setWeatherFrozen(true)
```

Registered in `LuaResourceRuntime.Environment.cs` (line 172) as `LuaResourceRuntime.Bootstrap`.
