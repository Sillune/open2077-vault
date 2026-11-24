# Open77.npcs.tasks (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-npcs-tasks.md) · [Page web](https://open2077.net/docs/api/server/open77-npcs-tasks)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

28 functions.

## all

```lua
Open77.npcs.tasks.all(id)
```

`NETWORK`: Uses the network backend.

Every task currently queued or running on an NPC.

Requires `world.npcs`. Ordered by channel and priority, which is the order the scheduler will take them in.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |

Returns: `array of task snapshots`

```lua
-- Debug: what is the NPC doing?
for _, task in ipairs(Open77.npcs.tasks.all(npcId)) do
    print(task.id, task.type, task.status)
end
```

Registered in `LuaResourceRuntime.cs` (line 9354) as `LuaResourceRuntime.Bootstrap`.

## attack

```lua
Open77.npcs.tasks.attack(id, target, [options])
```

`NETWORK`: Uses the network backend.

Orders an NPC to engage a target.

Requires `world.npcs`. **Action channel**, so it composes with a standing movement task instead of replacing one: the engine suspends `ignoreInCombat` movement for the fight and resumes it afterwards. The target is a player id or `{ npcId = n }`, and an NPC target must belong to this resource. The task forces the pair hostile and seeds the target into the puppet's own target tracker, re-seeding every `reacquireMs` (default 4,000; 500-60,000); the engine's combat AI drives the body from there, so Open77 never issues a shoot command and never cancels the behaviour tree. It reports `success` when the target dies, streams out or is removed, and `failure` with `attack_target_unavailable` when the target cannot be resolved at all, or with `invocation_failed` when the engine refused the attitude or threat native the seed is made of (`SetAttitudeTowards` / `AddThreat` returned false). A player **seated in a vehicle** can answer `invocation_failed`: treat a mounted target as unsupported and order the attack once `Open77.vehicles.getPlayerSeat(target)` is nil. Cancelling drops the seeded threat; the attitude is left to whatever `setAttitude` states.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `target` | `player id, or table { playerId } | { npcId }` | required | — |
| `options` | `table { reacquireMs?, priority?, timeoutMs? }` | optional | — |

Returns: `task id, or nil`, `reason`

```lua
-- The bodyguard engages whoever attacked the client
AddEventHandler("vip:attacked", function(attackerId)
    Open77.npcs.tasks.attack(bodyguardId, attackerId)
end)
```

Registered in `LuaResourceRuntime.cs` (line 9409) as `LuaResourceRuntime.Bootstrap`.

## cancel

```lua
Open77.npcs.tasks.cancel(id, taskId)
```

`NETWORK`: Uses the network backend.

Cancels one queued or running task.

Requires `world.npcs`. Leaves the rest of the queue in place; `clear` is the way to empty a channel.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `taskId` | `task id` | required | — |

Returns: `boolean`

```lua
-- The player paid: the guard stops attacking
local task = Open77.npcs.tasks.attack(guardId, playerId)
AddEventHandler("debt:paid", function()
    Open77.npcs.tasks.cancel(guardId, task)
end)
```

Registered in `LuaResourceRuntime.cs` (line 9351) as `LuaResourceRuntime.Bootstrap`.

## chase

```lua
Open77.npcs.tasks.chase(id, target, [options])
```

`NETWORK`: Uses the network backend.

Drives an NPC's vehicle after a target.

Requires `world.npcs` and `world.vehicles`. This is a ped-addressed view of `Open77.vehicles.ai`, which owns the vanilla `AIVehicleDriveToPointAutonomousCommand` pipeline -- there is no second driving stack and no second set of limits. The NPC must already be the attached driver of a vehicle (`Open77.vehicles.ai.attachDriver(vehicleId, { npcId = id })`), otherwise every one of these returns `nil, "npc_not_driving"`. The return value is the driving state the vehicle API returns.

The target is a player id, or a table naming `playerId`, `vehicleId` or `npcId`. The destination is refreshed twice a second from the target's position for as long as it stays in the same bucket.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `target` | `player id | table { playerId | vehicleId | npcId }` | required | — |
| `options` | `table { distance?, speed?, style?, timeoutMs? }` | optional | — |

Returns: `driving state, or nil`, `reason`

```lua
-- A police NPC already driving a cruiser chases the suspect
Open77.npcs.tasks.chase(copId, suspectId, { distance = 8 })
```

Registered in `LuaResourceRuntime.cs` (line 9571) as `LuaResourceRuntime.Bootstrap`.

## clear

```lua
Open77.npcs.tasks.clear(id, [channel])
```

`NETWORK`: Uses the network backend.

Clears queued and current tasks, optionally on one channel.

Requires `world.npcs`. Omitting the channel clears them all. Clearing the movement channel stops the NPC where it is rather than returning it anywhere.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `channel` | `integer` | optional | — |

Returns: `boolean`

```lua
-- Stop everything the NPC was doing
Open77.npcs.tasks.clear(npcId)
```

Registered in `LuaResourceRuntime.cs` (line 9352) as `LuaResourceRuntime.Bootstrap`.

## driveTo

```lua
Open77.npcs.tasks.driveTo(id, position, [options])
```

`NETWORK`: Uses the network backend.

Drives an NPC's vehicle to a position.

Requires `world.npcs` and `world.vehicles`. This is a ped-addressed view of `Open77.vehicles.ai`, which owns the vanilla `AIVehicleDriveToPointAutonomousCommand` pipeline -- there is no second driving stack and no second set of limits. The NPC must already be the attached driver of a vehicle (`Open77.vehicles.ai.attachDriver(vehicleId, { npcId = id })`), otherwise every one of these returns `nil, "npc_not_driving"`. The return value is the driving state the vehicle API returns.

`options.style` is the driving behaviour (`normal`, `cautious`, `aggressive`) and is the same field `Open77.vehicles.ai.setBehavior` writes.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `position` | `vector3` | required | — |
| `options` | `table { speed?, style?, arrivalRadius?, timeoutMs? }` | optional | — |

Returns: `driving state, or nil`, `reason`

```lua
-- The taxi driver takes the car to the drop-off point
Open77.npcs.tasks.driveTo(driverId, vector3(-1442.2, 127.4, 18.0), { speed = 15 })
```

Registered in `LuaResourceRuntime.cs` (line 9535) as `LuaResourceRuntime.Bootstrap`.

## driveWander

```lua
Open77.npcs.tasks.driveWander(id, [options])
```

`NETWORK`: Uses the network backend.

Drives an NPC's vehicle along the road network indefinitely.

Requires `world.npcs` and `world.vehicles`. This is a ped-addressed view of `Open77.vehicles.ai`, which owns the driving commands -- there is no second driving stack and no second set of limits. The NPC must already be the attached driver of a vehicle (`Open77.vehicles.ai.attachDriver(vehicleId, { npcId = id })`), otherwise every one of these returns `nil, "npc_not_driving"`. The return value is the driving state the vehicle API returns.

This is `joinTraffic`, the engine's own traffic enrollment: REDengine chooses the lane, the path and the speed. It is **not** a random-destination wander, because Cyberpunk traffic AI drives authored lanes and there is no free-roam equivalent to GTA's `TaskVehicleDriveWander`. Because the engine owns the motion, this verb reads no `speed` and no `style`/`behavior`; only `{ timeoutMs }` applies, and a request naming either of the other two is refused with `traffic_speed_unsupported` / `traffic_behavior_unsupported`. One task is submitted per vehicle and the engine advances its own path, so a car can hold still in traffic legitimately. The native move pattern is the one option the enrollment does take: `{ pattern = "default" | "stop" | "panic" }` rides the join exactly as it does on `Open77.vehicles.ai.joinTraffic`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `options` | `table { timeoutMs?, pattern? }` | optional | — |

Returns: `driving state, or nil`, `reason`

```lua
-- Ambient traffic: this driver cruises on the engine's own lanes
Open77.npcs.tasks.driveWander(driverId)
```

Registered in `LuaResourceRuntime.cs` (line 9558) as `LuaResourceRuntime.Bootstrap`.

## driving

```lua
Open77.npcs.tasks.driving(id)
```

`NETWORK`: Uses the network backend.

The driving job an NPC is the attached driver of.

Requires `world.npcs` and `world.vehicles`. This is a ped-addressed view of `Open77.vehicles.ai`, which owns the vanilla `AIVehicleDriveToPointAutonomousCommand` pipeline -- there is no second driving stack and no second set of limits. The NPC must already be the attached driver of a vehicle (`Open77.vehicles.ai.attachDriver(vehicleId, { npcId = id })`), otherwise every one of these returns `nil, "npc_not_driving"`. The return value is the driving state the vehicle API returns.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |

Returns: `driving state, or nil`, `reason`

```lua
-- Is the taxi still on its way?
local state = Open77.npcs.tasks.driving(driverId)
if state then print("driving job:", state.status) end
```

Registered in `LuaResourceRuntime.cs` (line 9534) as `LuaResourceRuntime.Bootstrap`.

## enqueue

```lua
Open77.npcs.tasks.enqueue(id, taskType, [parameters], [options])
```

`NETWORK`: Uses the network backend.

Enqueues any supported task on an NPC.

Requires `world.npcs`. The escape hatch under the typed helpers: `taskType` names the task, `parameters` is JSON-encoded for the host, and `options` carries `channel` (default 0), `priority` (default 0) and `timeoutMs` (default 0, meaning no timeout). Channels are in `Open77.npcs.channels`: `movement` (0), `look` (1), `action` (2) and `fullBody` (3) -- tasks on different channels run at the same time.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `taskType` | `string` | required | — |
| `parameters` | `table` | optional | — |
| `options` | `table { channel?, priority?, timeoutMs? }` | optional | — |

Returns: `task id, or nil`, `reason`

```lua
-- The raw form of moveTo, with an explicit channel and priority
Open77.npcs.tasks.enqueue(npcId, "moveTo", { x = -1442.2, y = 127.4, z = 18.0, speed = "run" }, {
    channel = Open77.npcs.channels.movement,
    priority = 5,
    timeoutMs = 20000,
})
```

Registered in `LuaResourceRuntime.cs` (line 9345) as `LuaResourceRuntime.Bootstrap`.

## enterVehicle

```lua
Open77.npcs.tasks.enterVehicle(id, vehicleId, [seat], [options])
```

`NETWORK`: Uses the network backend.

Sends an NPC to a vehicle seat.

Requires `world.npcs`. Movement channel -- walking to a car is locomotion, so it replaces a standing `moveTo` rather than composing with it.

Seats use `Open77.vehicles.seats` numbering (`-1` driver, `0` front passenger, `1` rear left, `2` rear right) and also accept the names `driver`, `frontPassenger`, `rearLeft`, `rearRight`. The body walks to the car and mounts when it is within `approachDistance`; `warp = true` skips the walk and mounts on the spot.

The task reports `Executing` while walking and succeeds only once the mounting relation is readable back off the body, because that is the only honest proof it is seated. It fails with `vehicle_not_streamed` when no client has the car streamed in, and `vehicle_mount_refused` when the engine refuses the seat.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `vehicleId` | `vehicle id` | required | — |
| `seat` | `integer | string` | optional | — |
| `options` | `table { warp?, approachDistance?, speed?, priority?, timeoutMs? }` | optional | — |

Returns: `task id, or nil`, `reason (`npc_task_invalid_target`, `npc_task_invalid_seat`)`

```lua
-- The getaway driver walks to the car and takes the wheel
Open77.npcs.tasks.enterVehicle(driverId, getawayCarId, Open77.vehicles.seats.driver)
```

Registered in `LuaResourceRuntime.cs` (line 9497) as `LuaResourceRuntime.Bootstrap`.

## exitVehicle

```lua
Open77.npcs.tasks.exitVehicle(id, [options])
```

`NETWORK`: Uses the network backend.

Takes an NPC out of whatever seat it is in.

Requires `world.npcs`. Movement channel. Idempotent on purpose: a body already out of the car has done what was asked, and the task succeeds with reason `not_in_vehicle` rather than failing. `options.vehicleId` narrows the task to one vehicle, in which case a body sitting in a different one fails with `vehicle_mismatch` instead of being pulled out of it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `options` | `table { vehicleId?, priority?, timeoutMs? }` | optional | — |

Returns: `task id, or nil`, `reason`

```lua
-- The passenger gets out at the destination
Open77.npcs.tasks.exitVehicle(passengerId, { vehicleId = taxiId })
```

Registered in `LuaResourceRuntime.cs` (line 9519) as `LuaResourceRuntime.Bootstrap`.

## face

```lua
Open77.npcs.tasks.face(id, target, [options])
```

`NETWORK`: Uses the network backend.

Rotates an NPC's body toward a world point.

Requires `world.npcs`. Look channel, so it runs alongside movement. Options are `tolerance` (default 3 degrees), `speed` (rotation, default 180 degrees per second), `priority` and `timeoutMs` (default 5,000).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `target` | `table { x, y, z }` | required | — |
| `options` | `table { tolerance?, speed?, priority?, timeoutMs? }` | optional | — |

Returns: `task id, or nil`, `reason`

```lua
-- The shopkeeper turns to face the customer
local customer = Open77.players.position(playerId)
if customer then Open77.npcs.tasks.face(shopkeeperId, customer) end
```

Registered in `LuaResourceRuntime.cs` (line 9383) as `LuaResourceRuntime.Bootstrap`.

## flee

```lua
Open77.npcs.tasks.flee(id, from, [options])
```

`NETWORK`: Uses the network backend.

Sends an NPC running away from something.

Requires `world.npcs`. Movement channel. `from` is a player id, `{ npcId = n }` or a fixed `{ x, y, z }`; a live body is re-measured every cycle, so the NPC keeps breaking contact as the thing it flees moves. It reports `success` once `distance` metres separate them (5-200, default 50) or the target is gone, and each leg is capped so the body asks for a destination it can path to. The tracked threat on the fled body is dropped once at task start; this is best-effort, because the attitude is untouched and a still-hostile body may re-acquire the same target by perception -- `setAttitude(id, "neutral", { towards = ... })` is the composable answer. Options are `speed` (default `sprint`), `distance`, `priority` and `timeoutMs` (default 30,000).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `from` | `player id, or table { playerId } | { npcId } | { x, y, z }` | required | — |
| `options` | `table { distance?, speed?, priority?, timeoutMs? }` | optional | — |

Returns: `task id, or nil`, `reason`

```lua
-- The civilian runs 60 m away from the shooter
Open77.npcs.tasks.flee(civilianId, shooterId, { distance = 60, speed = "sprint" })
```

Registered in `LuaResourceRuntime.cs` (line 9432) as `LuaResourceRuntime.Bootstrap`.

## follow

```lua
Open77.npcs.tasks.follow(id, target, [options])
```

`NETWORK`: Uses the network backend.

Makes an NPC follow a player, another NPC, or a point.

Requires `world.npcs`. Movement channel. `target` is a player id (a number), `{ type = "npc", id = ... }`, or a position table. Options are `distance` (default 2 metres), `speed` (default `walk`) and `onTargetLost` (default `wait`), plus `priority` and `timeoutMs`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `target` | `integer|table` | required | — |
| `options` | `table { distance?, speed?, onTargetLost?, priority?, timeoutMs? }` | optional | — |

Returns: `task id, or nil`, `reason`

```lua
-- The bodyguard follows the client two metres behind
Open77.npcs.tasks.follow(bodyguardId, clientId, { distance = 2, speed = "walk" })
```

Registered in `LuaResourceRuntime.cs` (line 9361) as `LuaResourceRuntime.Bootstrap`.

## get

```lua
Open77.npcs.tasks.get(id, taskId)
```

`NETWORK`: Uses the network backend.

Reads one task's state.

Requires `world.npcs`. The task state machine is described in [NPCs](https://open2077.net/docs/api/server/npcs.md); a task that has finished stops being readable.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `taskId` | `task id` | required | — |

Returns: `task snapshot, or nil`

```lua
-- Wait until the walk is finished
CreateThread(function()
    local task = Open77.npcs.tasks.moveTo(npcId, door)
    while Open77.npcs.tasks.get(npcId, task) do Wait(250) end
    print("the NPC has arrived (or gave up)")
end)
```

Registered in `LuaResourceRuntime.cs` (line 9353) as `LuaResourceRuntime.Bootstrap`.

## guard

```lua
Open77.npcs.tasks.guard(id, position, [radius], [options])
```

`NETWORK`: Uses the network backend.

Anchors an NPC to a position and lets it fight from there.

Requires `world.npcs`. Movement channel. A standing order with no natural end: it runs until cancelled or timed out. Whenever the body is further than `radius` metres (2-100, default 10) from the anchor **and is not already in a fight**, it walks back; while it holds a hostile threat it is left entirely alone, so the engine can take cover and manoeuvre. Guarding does not by itself make anything hostile -- pair it with `setAttitude` or a relationship group. Options are `speed` (default `run`), `priority` and `timeoutMs`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `position` | `table { x, y, z }` | required | — |
| `radius` | `number (metres, 2-100)` | optional | — |
| `options` | `table { speed?, priority?, timeoutMs? }` | optional | — |

Returns: `task id, or nil`, `reason`

```lua
-- Two guards hold the gate and fight from there
for _, guardId in ipairs(guards) do
    Open77.npcs.tasks.guard(guardId, gate, 8)
end
```

Registered in `LuaResourceRuntime.cs` (line 9423) as `LuaResourceRuntime.Bootstrap`.

## hold

```lua
Open77.npcs.tasks.hold(id, [options])
```

`NETWORK`: Uses the network backend.

Holds an NPC's movement until cancelled or timed out.

Requires `world.npcs`. Movement channel, no default timeout: it holds indefinitely until `cancel`, `clear`, or an explicit `timeoutMs`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `options` | `table { priority?, timeoutMs? }` | optional | — |

Returns: `task id, or nil`, `reason`

```lua
-- Freeze the NPC in place while the dialogue runs
local hold = Open77.npcs.tasks.hold(npcId)
AddEventHandler("dialogue:closed", function() Open77.npcs.tasks.cancel(npcId, hold) end)
```

Registered in `LuaResourceRuntime.cs` (line 9402) as `LuaResourceRuntime.Bootstrap`.

## lookAt

```lua
Open77.npcs.tasks.lookAt(id, target, [options])
```

`NETWORK`: Uses the network backend.

Drives an NPC's look channel toward a player, NPC or point.

Requires `world.npcs`. Look channel. `target` is a player id (a number), `{ type = "player"|"npc", id = ... }`, or a position table. Unlike `face`, this turns the head rather than the body.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `target` | `integer|table` | required | — |
| `options` | `table { priority?, timeoutMs? }` | optional | — |

Returns: `task id, or nil`, `reason`

```lua
-- The receptionist watches the player who walked in
Open77.npcs.tasks.lookAt(receptionistId, playerId)
```

Registered in `LuaResourceRuntime.cs` (line 9389) as `LuaResourceRuntime.Bootstrap`.

## moveTo

```lua
Open77.npcs.tasks.moveTo(id, position, [options])
```

`NETWORK`: Uses the network backend.

Walks or runs an NPC to a world point.

Requires `world.npcs`. Movement channel. Options are `speed` (default `walk`), `acceptanceRadius` (default 1 metre), `priority` and `timeoutMs` (default 30,000 -- unlike most tasks, this one gives up rather than pathing forever). The walk never arrives by teleport: a destination the AI navigation cannot reach (a locked door, a wall, a point off the walkable mesh) is walked to its closest reachable point, where the NPC asks a managed door in front of it to open (see NPC passage in the doors guide) and re-checks once a second; if the way stays shut for about five seconds the task fails with reason `path_blocked`. Within `acceptanceRadius` + 0.5 m of an unreachable point it succeeds with reason `nearest_reachable`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `position` | `table { x, y, z }` | required | — |
| `options` | `table { speed?, acceptanceRadius?, priority?, timeoutMs? }` | optional | — |

Returns: `task id, or nil`, `reason`

```lua
-- Send the courier running to the drop point
Open77.npcs.tasks.moveTo(courierId, { x = -1442.2, y = 127.4, z = 18.0 }, { speed = "run" })
```

Registered in `LuaResourceRuntime.cs` (line 9355) as `LuaResourceRuntime.Bootstrap`.

## patrol

```lua
Open77.npcs.tasks.patrol(id, points, [options])
```

`NETWORK`: Uses the network backend.

Sends an NPC round a list of points.

Requires `world.npcs`. Movement channel. `loop` defaults to true and `backAndForth` to false, so the default is a closed circuit. Options also take `speed`, `priority` and `timeoutMs`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `points` | `array of { x, y, z }` | required | — |
| `options` | `table { loop?, backAndForth?, speed?, priority?, timeoutMs? }` | optional | — |

Returns: `task id, or nil`, `reason`

```lua
-- A guard walks a loop around the warehouse
Open77.npcs.tasks.patrol(guardId, {
    { x = -1442.2, y = 127.4, z = 18.0 },
    { x = -1448.2, y = 96.1, z = 17.5 },
    { x = -1460.2, y = 99.9, z = 14.8 },
}, { speed = "walk", loop = true })
```

Registered in `LuaResourceRuntime.cs` (line 9373) as `LuaResourceRuntime.Bootstrap`.

## playAnimation

```lua
Open77.npcs.tasks.playAnimation(id, animation, [options])
```

`NETWORK`: Uses the network backend.

Runs a full-body animation on an NPC.

Requires `world.npcs`. Full-body channel, so it takes over movement and look for its duration. `options.loop` defaults to false.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `animation` | `string` | required | — |
| `options` | `table { loop?, priority?, timeoutMs? }` | optional | — |

Returns: `task id, or nil`, `reason`

```lua
-- The bouncer smokes by the door
Open77.npcs.tasks.playAnimation(bouncerId, "emote_smoke", { loop = true })
```

Registered in `LuaResourceRuntime.cs` (line 9445) as `LuaResourceRuntime.Bootstrap`.

## setDriverAbility

```lua
Open77.npcs.tasks.setDriverAbility(id, ability)
```

`NETWORK`: Uses the network backend.

Maps a 0..1 driver ability onto the two levers the engine exposes.

Requires `world.npcs` and `world.vehicles`. This is a ped-addressed view of `Open77.vehicles.ai`, which owns the vanilla `AIVehicleDriveToPointAutonomousCommand` pipeline -- there is no second driving stack and no second set of limits. The NPC must already be the attached driver of a vehicle (`Open77.vehicles.ai.attachDriver(vehicleId, { npcId = id })`), otherwise every one of these returns `nil, "npc_not_driving"`. The return value is the driving state the vehicle API returns.

FiveM's `SetDriverAbility` is an engine skill parameter and 2.31 has none. This is an explicit **mapping**, documented so nobody has to guess it: below 0.34 is `cautious`, above 0.67 is `aggressive`, and in between is `normal`; the speed cap is `8 + ability * 24` metres per second. It returns the resulting driving state so a caller can read exactly what was applied instead of assuming.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `ability` | `number 0..1` | required | — |

Returns: `driving state, or nil`, `reason (`invalid_ability`, `npc_not_driving`)`

```lua
-- A nervous rookie driver
Open77.npcs.tasks.setDriverAbility(driverId, 0.3)
```

Registered in `LuaResourceRuntime.cs` (line 9604) as `LuaResourceRuntime.Bootstrap`.

## stopDriving

```lua
Open77.npcs.tasks.stopDriving(id)
```

`NETWORK`: Uses the network backend.

Stops whatever an NPC's vehicle was doing.

Requires `world.npcs` and `world.vehicles`. This is a ped-addressed view of `Open77.vehicles.ai`, which owns the vanilla `AIVehicleDriveToPointAutonomousCommand` pipeline -- there is no second driving stack and no second set of limits. The NPC must already be the attached driver of a vehicle (`Open77.vehicles.ai.attachDriver(vehicleId, { npcId = id })`), otherwise every one of these returns `nil, "npc_not_driving"`. The return value is the driving state the vehicle API returns.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |

Returns: `driving state, or nil`, `reason`

```lua
-- Pull over: the chase is called off
Open77.npcs.tasks.stopDriving(copId)
```

Registered in `LuaResourceRuntime.cs` (line 9591) as `LuaResourceRuntime.Bootstrap`.

## traffic

```lua
Open77.npcs.tasks.traffic(id, [options])
```

`NETWORK`: Uses the network backend.

Puts an NPC into the game's own pedestrian traffic.

Requires `world.npcs`. Movement channel. The NPC's simulating client puts it into the game's pedestrian traffic (`ReactionSystem.TryAndJoinTraffic`) and keeps it there; the game picks the lanes, the exits and the turns, so there is no destination, route or radius. `speed` is `"walk"` (the default) or `"run"`; `sprint` is refused with `npc_task_invalid_speed`. `threat = { x, y, z }` is the game's own crowd fear: the NPC stays in traffic and runs away from that point, and no workspot visit happens while it is set; a bad point is refused with `npc_task_invalid_point`. `visits` (boolean, default `true`) says whether the game may send the NPC to its crowd workspots; a non-boolean is refused with `npc_task_invalid_parameters`, and it is ignored while `threat` is set. Options also take `priority` and `timeoutMs` (default 0, no timeout). The task never succeeds while the NPC walks or runs: it runs until it is cancelled or replaced, or the client reports `failure` with a reason (`traffic_unavailable` when the body could not be put into traffic). When the NPC enters a crowd workspot the client ends it with `success` and reason `"workspot"`, with the body's placement, and `onNpcTaskState(npcId, taskId, "success", "workspot")` reaches the resources. Once in traffic the NPC is driven by the game's crowd AI and does not execute other movement commands: to change speed, threat or visits, cancel the active traffic task and enqueue the new one in the same call, which keeps the NPC in traffic; enqueued alone it queues behind the old one. With `behavior.observerReplay` the other clients replay the simulator's motion instead of running their own traffic AI.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `options` | `table { speed?, threat?, visits?, priority?, timeoutMs? }` | optional | — |

Returns: `task id, or nil`, `reason`

```lua
-- A walker joins the pedestrian crowd, then runs away from a point while staying in it
local walk = Open77.npcs.tasks.traffic(walkerId, { speed = "walk" })
Open77.npcs.tasks.cancel(walkerId, walk)
Open77.npcs.tasks.traffic(walkerId, { speed = "run", threat = { x = -1442.2, y = 127.4, z = 18.0 } })
```

Registered in `LuaResourceRuntime.cs` (line 9429) as `LuaResourceRuntime.Bootstrap`.

## wait

```lua
Open77.npcs.tasks.wait(id, durationMs, [options])
```

`NETWORK`: Uses the network backend.

Makes an NPC wait for a fixed time.

Requires `world.npcs`. Action channel by default. `timeoutMs` defaults to the duration plus one second, so a wait cannot outlive itself.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `durationMs` | `integer` | required | — |
| `options` | `table { channel?, priority?, timeoutMs? }` | optional | — |

Returns: `task id, or nil`, `reason`

```lua
-- Walk to the counter, wait two seconds, then walk back
CreateThread(function()
    Open77.npcs.tasks.moveTo(npcId, counter)
    Open77.npcs.tasks.wait(npcId, 2000, { channel = Open77.npcs.channels.movement })
    Open77.npcs.tasks.moveTo(npcId, home)
end)
```

Registered in `LuaResourceRuntime.cs` (line 9398) as `LuaResourceRuntime.Bootstrap`.

## wander

```lua
Open77.npcs.tasks.wander(id, [options])
```

`NETWORK`: Uses the network backend.

Lets an NPC wander with the supplied movement parameters.

Requires `world.npcs`. Movement channel. The whole options table is passed to the host as the task parameters, so it carries the wander radius and speed as well as `priority` and `timeoutMs`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `options` | `table` | optional | — |

Returns: `task id, or nil`, `reason`

```lua
-- A civilian roams within 15 m of the plaza
Open77.npcs.tasks.wander(civilianId, {
    x = -1442.2, y = 127.4, z = 18.0, radius = 15, speed = "walk",
})
```

Registered in `LuaResourceRuntime.cs` (line 9379) as `LuaResourceRuntime.Bootstrap`.

## workspot

```lua
Open77.npcs.tasks.workspot(id, reference, [options])
```

`NETWORK`: Uses the network backend.

Puts an NPC into an authored workspot scenario.

Requires `world.npcs`. FullBody channel, the same one `playAnimation` uses, so a workspot and an animation cannot run on one body at once.

`reference` is either a **catalogue profile id** (`smoke`, `sit`, `phone`, `dance`, `meditate`, ... -- see `Open77.npcs.tasks.workspots`), which brings its own shipped device entity and its own validated clip set, or a **raw clip name** played through the generic device. The catalogue profile is the supported form; a raw clip is an escape hatch whose animset the generic device may not carry. Ambient profiles do not appear in `Open77.animations.list()` or `tasks.workspots()` but resolve directly by id.

`options.anchor = { x, y, z, yaw? }` positions a catalogue profile's workspot at world coordinates. Coordinates must be finite; yaw is degrees about Z, defaults to 0, and must be within -360..360. Anchored workspots play on every client, remain `Executing` without an authority and work on Frozen NPCs. An anchor with a raw clip returns `npc_task_invalid_workspot`; invalid anchor values return `npc_task_invalid_anchor`.

`durationMs` ends the scenario by **succeeding**, not by timing out: a scenario that ran its course is not a failure, and `timeoutMs` stays free to mean "no client ever picked the command up". `loop = true` has no end and runs until cancelled.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `reference` | `catalogue profile id or clip name` | required | — |
| `options` | `table { durationMs?, loop?, clip?, anchor?: { x, y, z, yaw? }, priority?, timeoutMs? }` | optional | — |

Returns: `task id, or nil`, `reason (`npc_task_invalid_workspot`, `npc_task_invalid_anchor`, `npc_task_invalid_duration`)`

```lua
-- The mechanic leans on the car for a minute
Open77.npcs.tasks.workspot(mechanicId, "smoke", { durationMs = 60000 })
```

Registered in `LuaResourceRuntime.cs` (line 9526) as `LuaResourceRuntime.Bootstrap`.

## workspots

```lua
Open77.npcs.tasks.workspots([query])
```

`NETWORK`: Uses the network backend.

The vanilla scenario catalogue a workspot task can name.

Ungated. The same twelve shipped RP workspot profiles `Open77.animations.list()` offers for players -- one catalogue, two callers, never two lists that could disagree. Each entry gives `id`, `label`, `category`, the default `clip`, the vanilla `workspot` resource it plays, and the full clip set. `query` filters on id, label or category.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `query` | `string` | optional | — |

Returns: `array of profiles`

```lua
-- List the workspot profiles a task can name
for _, profile in ipairs(Open77.npcs.tasks.workspots()) do
    print(profile.id, profile.label)
end
```

Registered in `LuaResourceRuntime.cs` (line 9455) as `LuaResourceRuntime.Bootstrap`.
