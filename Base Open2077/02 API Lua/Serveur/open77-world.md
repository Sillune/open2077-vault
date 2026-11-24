# Open77.world (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-world.md) · [Page web](https://open2077.net/docs/api/server/open77-world)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

7 functions.

## clearArea

```lua
Open77.world.clearArea(position, radius, [options])
```

`NETWORK`: Uses the network backend.

Removes the server-owned entities inside a radius and can suppress vanilla population there.

The server half of `Open77.world`, and the only function in it: the client's `world` table (raycast, groundZ, nearby, nearest, district) needs a physics world and a streamed player, and a dedicated server has neither. **Ownership is the contract.** It removes what the CALLING resource owns and nothing else; entities inside the radius belonging to another resource are counted in `refused` and left standing. Without that rule a gamemode clearing a street would delete the parked cars a roleplay resource spawned and still holds ids for, and that resource would discover it only when its own ids began answering `not_found`. `foreign = true` lifts the rule, and only for VEHICLES, and only for a resource whose manifest carries `world.clearArea.foreign`; without the permission the WHOLE call is refused rather than quietly downgraded, because a silent downgrade would report success on a street that still has somebody else's cars in it. The restriction to vehicles is structural, not a policy choice: NPC and prop removal is owner-checked inside the authority services themselves, and the server's NPC read is owner-scoped, so a foreign NPC is never even a candidate. Options: `npcs`, `vehicles`, `props` (booleans, all default true), `population` (default false), `bucket`, `foreign` (default false), `grace` (metres of outward slack). A `nil` bucket means every bucket, the same reading `Open77.zones.playersIn` takes, since a place is not a player and has no bucket of its own to borrow. `population = true` REQUIRES a bucket and drives `Open77.routingBuckets.setPopulationEnabled`; the suppression that follows is **bucket-wide, not confined to the radius**, because that is the only shape the switch has. There is no radius-scoped vanilla-population control on 2.31. The containment test is `Open77.zones.contains` from the shared zone module, so the radius cap and the geometry rules are the ones every other Open77 zone obeys. A missing `world.npcs`, `world.vehicles` or `world.props` skips that kind and still succeeds: a teardown that failed outright over one of three permissions would leave the other two kinds behind.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `position` | `table { x, y, z }` | required | — |
| `radius` | `number (metres, 0 < r <= 2000)` | required | — |
| `options` | `table { npcs?, vehicles?, props?, population?, bucket?, foreign?, grace? }` | optional | — |

Returns: `table { npcs, vehicles, props, refused, foreignVehicles, population } where population is "unchanged", "suppressed" or "unavailable", or nil`, `reason: permission_denied:world.clearArea.foreign, invalid_radius, invalid_position, invalid_options, invalid_bucket, bucket_required_for_population, or a shared zone-module token (invalid_grace)`

```lua
-- Clear this resource's leftovers around the arena and suppress vanilla population there
local report = Open77.world.clearArea({ x = -1442.2, y = 127.4, z = 18.0 }, 150, {
    npcs = true, vehicles = true, props = true, population = true, bucket = ARENA_BUCKET,
})
if report then print(report.npcs, "npcs,", report.vehicles, "vehicles removed;", report.refused, "refused") end
```

Registered in `LuaResourceRuntime.cs` (line 8592) as `LuaResourceRuntime.Bootstrap`.

## getPopulation

```lua
Open77.world.getPopulation(bucket)
```

`NETWORK`: Uses the network backend.

Reads the ambient population policy of one routing bucket.

Ungated. `enabled` is derived (`crowd > 0 or traffic > 0 or police`), so it can never say "on" while every density is zero. `crowdGranularity` is `continuous` and `trafficGranularity` is `binary`, which is the honest statement of what 2.31 can apply.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | required | — |

Returns: `table { bucket, crowd, traffic, police, enabled, crowdGranularity, trafficGranularity }, or nil`, `reason`

```lua
-- Is the arena instance empty of vanilla crowd?
local policy = Open77.world.getPopulation(ARENA_BUCKET)
if policy and policy.enabled then print("population still on in the arena") end
```

Registered in `LuaResourceRuntime.cs` (line 8555) as `LuaResourceRuntime.Bootstrap`.

## getTimeScale

```lua
Open77.world.getTimeScale(bucket)
```

`NETWORK`: Uses the network backend.

Reads the time scale one routing bucket is held at.

`scale` is 1 and nothing else for a bucket at real time (a timed beat past its deadline included). Otherwise `remainingMs` is what is left of a timed hold (absent for an open-ended one), `easeMs` the ramp clients run, `owner` the resource that set it and `revision` the monotonic counter clients order replays by. Ungated: a resource in the bucket can feel it anyway.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | required | — |

Returns: `table { bucket, scale, remainingMs?, easeMs?, owner?, revision? }, or nil`, `reason (`invalid_bucket`, `timescale_unavailable_on_this_host`)`

```lua
-- Refuse to start a round while another resource is still holding a beat.
local held = Open77.world.getTimeScale(ARENA_BUCKET)
if held and held.scale < 1 then
    print(("round start deferred: %s holds the clock at %.2f"):format(tostring(held.owner), held.scale))
end
```

Registered in `LuaResourceRuntime.World6.cs` (line 481) as `LuaResourceRuntime.Bootstrap`.

## groundZ

```lua
Open77.world.groundZ(position, [options])
```

`NETWORK`: Uses the network backend.

The height of the static ground under a point, observed by the nearest connected client.

A dedicated server has no physics world, so this is an OBSERVATION: the nearest connected client standing within `options.radius` of the point (150 m by default, 300 m at most) is asked over the callback frames, and its own `Open77.world.groundZ` ray -- static geometry, a 400 m start, one kilometre down -- answers. The client host answers the reserved name itself, so no client resource has to exist or hold a permission. It WAITS for the answer, so call it from a scheduler coroutine (a command, an event handler, `CreateThread`); at file scope it refuses with `await_requires_scheduler_coroutine` rather than blocking the server.

**Never a default height.** Nobody near the point is `nil, "no_observer"`, because a spawn placed on an invented height kills; the radius is conservative under every measured streaming window (entities stream in at 225 m, cameras refuse past 250 m; the static-collision radius on 2.31 is unmeasured) and a caller who has measured better may widen it. `no_ground` means the observer's ray found nothing -- open water, a void, or a sector that client had not streamed yet -- and is never cached; ask again from closer. The ground is the same in every routing bucket, so observers are searched across buckets unless `options.bucket` restricts them; `options.playerId` names the observer outright (any distance, no cache). A measured metre cell answers from cache for a minute with `ageMs` reported; `cache = false` forces a fresh ray. `options.maxAgeMs` (30 s) bounds how old the observer's own position reading may be; `options.timeout` (3 s) how long to wait; `options.fromZ` (400) the ray's start height.

An observation, not a fact: the number comes from a client, so it is fit for placing a spawn, a prop or a marker and not for adjudicating anti-cheat -- the detail names the observer so a server that cares can weigh it. Requires `world.query`. Reasons: `permission_denied:world.query`, `invalid_position`, `invalid_options`, `invalid_radius`, `invalid_player_id`, `invalid_bucket`, `invalid_callback_timeout`, `no_observer`, `player_not_found`, `no_ground`, `callback_timeout`, `network_unavailable`, `resource_stopping`, `callback_request_limit`, `await_requires_scheduler_coroutine`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `position` | `table { x, y } or vector3` | required | — |
| `options` | `table { playerId?, radius? (<= 300), maxAgeMs?, timeout?, bucket?, fromZ?, cache? }` | optional | — |

Returns: `z (number), detail { playerId, distance, observerAgeMs, ageMs, cached }`, `nil, reason, detail (when an observer answered)`

```lua
-- Drop a supply crate on the ground under a map point, or say why not.
RegisterCommand("crate", function(source, args)
    local x, y = tonumber(args[1]), tonumber(args[2])
    local z, detail = Open77.world.groundZ({ x = x, y = y })
    if z == nil then
        print(("no ground for the crate: %s"):format(tostring(detail)))
        return
    end
    Open77.props.create({ model = "crate.small", position = { x = x, y = y, z = z + 0.5 } })
    print(("placed at z=%.2f, seen by player %d %.0f m away"):format(z, detail.playerId, detail.distance or 0))
end, true)
```

Registered in `LuaResourceRuntime.World6.cs` (line 411) as `LuaResourceRuntime.Bootstrap`.

## setPopulation

```lua
Open77.world.setPopulation(bucket, options)
```

`NETWORK`: Uses the network backend.

Sets the ambient population density of one routing bucket.

Requires `world.population`. Server-owned and replicated to every client in the bucket; the client half (`Open77.world.population`) is read-only, so there is nothing to race.

`crowd` is a fraction of the vanilla pedestrian density, 0..1, and is applied as one through the engine's community density modifier. `traffic` is accepted as 0..1 but 2.31 exposes no vehicle density modifier, so it is a **switch**: any value above zero means vanilla traffic. `getPopulation` reports that as `trafficGranularity = "binary"` rather than letting a caller assume the fraction was honoured. `police` allows or forbids vanilla prevention spawns outright.

A bucket nobody has configured is the vanilla world (`1, 1, true`), not an empty one. `Open77.routingBuckets.setPopulationEnabled` is the same state expressed as a switch: off sets all three off, on restores the vanilla figures.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | required | — |
| `options` | `table { crowd? 0..1, traffic? 0..1, police? boolean }` | required | — |

Returns: `true, or nil`, `reason (`permission_denied:world.population`, `invalid_bucket`, `invalid_crowd`, `invalid_traffic`, `invalid_options`)`

```lua
-- A quiet RP district: half the crowd, no traffic, no police
Open77.world.setPopulation(0, { crowd = 0.5, traffic = 0, police = false })
```

Registered in `LuaResourceRuntime.cs` (line 8540) as `LuaResourceRuntime.Bootstrap`.

## setPreventionEnabled

```lua
Open77.world.setPreventionEnabled(bucket, enabled)
```

`NETWORK`: Uses the network backend.

Turns NCPD dispatch on or off for a whole routing bucket.

`TogglePreventionSystem` on every client currently in that bucket, and on any client that enters it afterwards. Per bucket rather than per player because it is a property of the world a set of players share, and because that is the only shape the engine's switch has - it takes no target. A bucket nobody has set runs dispatch, which is vanilla. Turning it off does not change anybody's wanted level: the two are separate, and a player left at level 3 in a bucket with dispatch off keeps the level and stops getting the police. Requires `world.prevention`, which is separate from `players.wanted` on purpose - a resource that should be able to make somebody a fugitive is not automatically one that should be able to abolish the police for everyone. Note that this is NOT the same switch as the police block `Open77.world.clearArea`'s population suppression drives on the client; while that suppression is active the client refuses a wanted-level raise with `population_suppressed`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer (0..65535)` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `table { bucket, enabled }, or nil`, `reason: permission_denied:world.prevention, invalid_bucket, invalid_options`

```lua
-- No NCPD in the arena instance
Open77.world.setPreventionEnabled(ARENA_BUCKET, false)
```

Registered in `LuaResourceRuntime.cs` (line 8670) as `LuaResourceRuntime.Bootstrap`.

## setTimeScale

```lua
Open77.world.setTimeScale(bucket, scale, [options])
```

`NETWORK`: Uses the network backend.

Slows the clock of every client in one routing bucket for a shared beat.

The replicated half of the client `Open77.world.setTimeScale`: every client in `bucket` runs its world at `scale` (0.05..1; 1 releases the bucket) for `options.durationMs` (0 = until released or until this resource stops), easing in and back out over `options.easeMs` on its own clock. A client that enters the bucket mid-beat is told only what remains; one that leaves a slowed bucket is told to release. On each client the beat is a claim under the reserved owner `open77:server`, composing with any claim a client resource holds rather than fighting it.

**The server does not slow down.** It keeps simulating at real time; this changes the rate at which each client in the bucket presents the world. It is a presentation lever for a beat every player in an instance should share -- a countdown, a cinematic, a round end -- and not a combat mechanic: a shot at a slowed client is adjudicated in real time. Requires `world.timescale`. Reasons: `permission_denied:world.timescale`, `invalid_bucket`, `invalid_scale`, `invalid_duration`, `invalid_ease`, `invalid_options`, `timescale_unavailable_on_this_host`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | required | — |
| `scale` | `number (0.05..1; 1 releases)` | required | — |
| `options` | `table { durationMs? 0..3600000, easeMs? 0..10000 }` | optional | — |

Returns: `true, or nil`, `reason`

```lua
-- The whole arena instance sees the final kill at quarter speed for a moment.
AddEventHandler("open77:playerKilled", function(victim, killer)
    local bucket = Open77.routingBuckets.getPlayer(victim)
    if bucket == nil then return end
    Open77.world.setTimeScale(bucket, 0.25, { durationMs = 1500, easeMs = 200 })
end)
```

Registered in `LuaResourceRuntime.World6.cs` (line 466) as `LuaResourceRuntime.Bootstrap`.
