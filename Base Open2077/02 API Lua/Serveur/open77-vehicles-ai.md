# Open77.vehicles.ai (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-vehicles-ai.md) · [Page web](https://open2077.net/docs/api/server/open77-vehicles-ai)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

15 functions.

## attachDriver

```lua
Open77.vehicles.ai.attachDriver(vehicleId, [options])
```

`NETWORK`: Uses the network backend.

Attaches a server-owned vehicle AI controller.

options.npcId optionally mounts an existing NPC owned by this resource; omission means driverless operation. Reserves the driver seat. Requires world.vehicles. Refuses occupied driver seats, destroyed vehicles, unsupported AV/Basilisk records and conflicting controllers. The NPC must be alive, in the same bucket and without active tasks. No NPC is created automatically. options.admissionPlayerId optionally books one of that player's simulator slots before any client simulates the car (see capacity); it is refused with ai_simulator_limit when that player's booked count already equals perSimulatorLimit, and it does not force ownership: election still chooses the simulator and moves the booking to it. A driver whose canonical health reaches zero ends the task as failed with reason driver_dead and detaches the controller with the same reason, releasing the driver seat without removing the body; a driver that was removed or left the car's bucket gives driver_unavailable instead.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |
| `options` | `table` | optional | — |

Returns: `state, or nil, reason`

```lua
-- Put a spawned NPC at the wheel of the taxi
local state, reason = Open77.vehicles.ai.attachDriver(taxiId, { npcId = driverId })
if not state then print("no driver: " .. reason) end
```

[Autonomous vehicles & AI guide](../../01%20Documentation/vehicle-ai.md)

Registered in `LuaResourceRuntime.cs` (line 9001) as `LuaResourceRuntime.Bootstrap`.

## capacity

```lua
Open77.vehicles.ai.capacity()
```

`NETWORK`: Uses the network backend.

Reads the server-wide AI driver admission counts.

Requires world.vehicles; without it returns nil, permission_denied:world.vehicles. Takes no vehicle id and counts every resource's controllers, not only the caller's. globalUsed counts the controllers attached now in any mode or status (an idle or stopped one keeps its slot until removeDriver or vehicle removal); the server sets no total. perSimulatorLimit is the server's simulation.maximumAiVehiclesPerSimulator (1..128, default 128; 128 is the server's own per-simulator ceiling). owners maps decimal player-id strings to the jobs booked against that player: the cars it simulates now plus reservations made with attachDriver's admissionPlayerId for cars no client simulates yet; a player with none is absent. A car whose simulator leaves range, changes bucket or becomes unavailable releases its booking immediately; a nearby simulator whose readiness expires keeps its booking until a fresh acknowledgment or another client is granted it. Election never grants a car to a client at the limit (the task waits with reason simulator_capacity) and a client already simulating a car keeps it. The table is a snapshot: attachDriver re-checks the per-simulator limit and its refusal is authoritative. A configured perSimulatorLimit is an admission bound, not a measured or certified fleet size for any client.

Returns: `capacity, or nil, reason`

```lua
-- Book a car only for a player who still has a free simulator slot
local capacity = Open77.vehicles.ai.capacity()
local booked = capacity and (capacity.owners[tostring(playerId)] or 0)
if capacity and booked < capacity.perSimulatorLimit then
    Open77.vehicles.ai.attachDriver(carId, { npcId = driverId, admissionPlayerId = playerId })
end
```

[Autonomous vehicles & AI guide](../../01%20Documentation/vehicle-ai.md)

Registered in `LuaResourceRuntime.cs` (line 9004) as `LuaResourceRuntime.Bootstrap`.

## chase

```lua
Open77.vehicles.ai.chase(vehicleId, options)
```

`NETWORK`: Uses the network backend.

Pursues a canonical entity using native autonomous pathing.

Same targetKind, targetId, speed, distance, behavior and timeout options as follow. Requires world.vehicles and controller ownership. It does not enable weapon use, teleport catch-up, scripted ramming or police pursuit logic. Targets and simulator must remain in the same routing bucket.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |
| `options` | `table` | required | — |

Returns: `state, or nil, reason`

```lua
-- A police car chases the suspect's vehicle
Open77.vehicles.ai.chase(cruiserId, { targetKind = "vehicle", targetId = suspectCarId, speed = 25, behavior = "aggressive" })
```

[Autonomous vehicles & AI guide](../../01%20Documentation/vehicle-ai.md)

Registered in `LuaResourceRuntime.cs` (line 9008) as `LuaResourceRuntime.Bootstrap`.

## driveTo

```lua
Open77.vehicles.ai.driveTo(vehicleId, options)
```

`NETWORK`: Uses the network backend.

Starts native autonomous driving to a world position.

Requires world.vehicles and this resource's attached controller. options.position={x,y,z} is required. Optional speed (0.5..55 m/s), arrivalRadius (1..20 m), timeoutMilliseconds (1000..3600000) and behavior. Returns a task state, or nil, reason. The native driver plans on the traffic lane graph: the simulating client snaps the point to the nearest lane contact within 21 m and to the road surface before sending the command, and a task that has not moved 3 m after 20 s fails with reason no_route. Acceptance is not arrival: subscribe to arrived, blocked or failed.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |
| `options` | `table` | required | — |

Returns: `state, or nil, reason`

```lua
-- The taxi drives to the customer's destination
Open77.vehicles.ai.driveTo(taxiId, {
    position = { x = -1442.2, y = 127.4, z = 18.0 },
    speed = 15,
    arrivalRadius = 4,
})
```

[Autonomous vehicles & AI guide](../../01%20Documentation/vehicle-ai.md)

Registered in `LuaResourceRuntime.cs` (line 9005) as `LuaResourceRuntime.Bootstrap`.

## follow

```lua
Open77.vehicles.ai.follow(vehicleId, options)
```

`NETWORK`: Uses the network backend.

Continuously follows a canonical player, vehicle or NPC.

Requires world.vehicles and controller ownership. options.targetKind is player, vehicle or npc and targetId is the canonical ID. Optional distance (3..80 m), speed, behavior and timeoutMilliseconds. Uses the native autonomous path follower with server-updated target coordinates, at most twice per second. Stops with target_unavailable if the target leaves the bucket or disappears.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |
| `options` | `table` | required | — |

Returns: `state, or nil, reason`

```lua
-- The bodyguard's car follows the client's car at 10 m
Open77.vehicles.ai.follow(escortId, { targetKind = "player", targetId = clientId, distance = 10 })
```

[Autonomous vehicles & AI guide](../../01%20Documentation/vehicle-ai.md)

Registered in `LuaResourceRuntime.cs` (line 9007) as `LuaResourceRuntime.Bootstrap`.

## followRoute

```lua
Open77.vehicles.ai.followRoute(vehicleId, options)
```

`NETWORK`: Uses the network backend.

Drives an ordered route of world points.

Requires world.vehicles and controller ownership. options.points is an array of 1..64 {x,y,z} positions. Supports loop, speed, behavior, arrivalRadius and timeoutMilliseconds. Emits waypointReached at each point and arrived once at a non-looping route's end. This is a destination sequence, not an exact authored racing spline. As with driveTo, the simulating client snaps each point to the nearest traffic lane contact within 21 m and to the road surface before sending it, and a task that has not moved 3 m after 20 s fails with reason no_route.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |
| `options` | `table` | required | — |

Returns: `state, or nil, reason`

```lua
-- A bus loops its three stops forever
Open77.vehicles.ai.followRoute(busId, {
    points = {
        { x = -1442.2, y = 127.4, z = 18.0 },
        { x = -1448.2, y = 96.1, z = 17.5 },
        { x = -1460.2, y = 99.9, z = 14.8 },
    },
    loop = true, speed = 10,
})
```

[Autonomous vehicles & AI guide](../../01%20Documentation/vehicle-ai.md)

Registered in `LuaResourceRuntime.cs` (line 9006) as `LuaResourceRuntime.Bootstrap`.

## joinTraffic

```lua
Open77.vehicles.ai.joinTraffic(vehicleId, [options])
```

`NETWORK`: Uses the network backend.

Enrolls a controlled vehicle in REDengine's own traffic simulation: it joins the nearest traffic lane and follows traffic indefinitely.

Requires world.vehicles and controller ownership. Dispatches AIVehicleJoinTrafficCommand, the engine's own traffic enrollment (the command the Delamain cab receives when released to free roam): the engine chooses the lane, the path and the speed, so this task takes no speed, no behavior and no waypoints, and a request naming speed or behavior is refused with traffic_speed_unsupported or traffic_behavior_unsupported instead of accepting a value nothing can apply. Only timeoutMilliseconds (1000..3600000, default 300000) and pattern (default or stop; see setTrafficPattern) apply, and one task is submitted per vehicle: the engine crosses its own path boundaries without a new task, so a car can hold still in traffic legitimately. A traffic task is therefore never failed with no_route: it may stand still in a queue, at a light, behind a wreck or under the stop pattern for as long as the traffic does. It does not spawn ambient traffic or use the solo Delamain taxi system. No arrival event is expected; stop explicitly or handle failure/timeout. A timed native panic reaction on this task is the separate panic call.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |
| `options` | `table` | optional | — |

Returns: `state, or nil, reason`

```lua
-- REDengine picks the lane, the path and the speed
Open77.vehicles.ai.joinTraffic(carId, { timeoutMilliseconds = 3600000 })
```

[Autonomous vehicles & AI guide](../../01%20Documentation/vehicle-ai.md)

Registered in `LuaResourceRuntime.cs` (line 9012) as `LuaResourceRuntime.Bootstrap`.

## on

```lua
Open77.vehicles.ai.on(event, handler)
```

`NETWORK`: Uses the network backend.

Subscribes to this resource's vehicle AI lifecycle events.

Requires a function callback. Events: waypointReached, arrived, cancelled, failed, blocked, resumed and panic. panic fires for every outcome of a panic request after the call itself (active, ended, blocked, refused, cancelled -- including cancelled/replaced for the request a newer one supersedes); correlate it by state.panicRequest. The handler receives a decoded state table, not JSON text. Events are routed only to the controller's resource. blocked with reason=no_progress is a stall detector, not proof of an obstacle collision. No events are replayed; query state first.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `handler` | `function` | required | — |

Returns: `Event handler registration`

```lua
-- Charge the fare when the taxi arrives
Open77.vehicles.ai.on("arrived", function(state)
    local fare = fares[state.vehicleId]
    if fare then chargeFare(fare.playerId, fare.price) end
end)
```

[Autonomous vehicles & AI guide](../../01%20Documentation/vehicle-ai.md)

Registered in `LuaResourceRuntime.cs` (line 9041) as `LuaResourceRuntime.Bootstrap`.

## panic

```lua
Open77.vehicles.ai.panic(vehicleId, options)
```

`NETWORK`: Uses the network backend.

Requests a time-bounded native panic reaction on a running traffic enrollment.

Requires world.vehicles and ownership of a running or waiting joinTraffic task; another task returns nil, panic_unsupported_task, a finished one traffic_panic_inactive. options.threatPosition {x,y,z} is required (invalid_threatPosition); threatKind (player, npc or vehicle) and threatId are optional, and a named npc or vehicle must exist in the car's bucket (threat_unavailable). durationMilliseconds is 1000..30000, default 10000. Each accepted call is a new request with a new panicRequest token that replaces the previous one. The client simulating the car releases the kinematic join, waits until the engine holds no traffic slot and no vehicle command, sends the engine's own AIVehiclePanicCommand (allowSimplifiedMovement, ignoreTickets, useSpeedBasedLookupRange; tryDriveAwayFromPlayer only when the threat is that client's own player), and panicStatus turns from pending to active only once that command is observed executing. The duration starts there and survives a handoff or new native body, which re-confirm it without restarting it. Terminal outcomes: ended (duration_elapsed or the native command stopping on its own), blocked (join_unreleased, command_unreleased, not_executing, or application_timeout after 5 s unconfirmed), refused (native_rejected, native_failure, unsupported_vehicle, ...), cancelled (replaced, pattern_changed, or the task's own end reason, such as driver_dead when the driver's canonical health reaches zero or driver_unavailable when the driver was removed or left the car's bucket). The car then returns to its join through the bounded collision-recovery path; a recovery that cannot find a safe lane fails the task. After driver_dead nothing rejoins: the client releases the native command and the controller detaches. panicRemainingMs and the panic event (on) report progress. The threat position is not a native input: it is validated, published and used for correlation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |
| `options` | `table` | required | — |

Returns: `state, or nil, reason`

```lua
-- A grenade went off next to the taxi rank
local state = Open77.vehicles.ai.panic(taxiId, {
    threatPosition = { x = blast.x, y = blast.y, z = blast.z },
    threatKind = "player", threatId = throwerId, durationMilliseconds = 8000 })
if state then pending[taxiId] = state.panicRequest end
```

[Autonomous vehicles & AI guide](../../01%20Documentation/vehicle-ai.md)

Registered in `LuaResourceRuntime.cs` (line 9037) as `LuaResourceRuntime.Bootstrap`.

## removeDriver

```lua
Open77.vehicles.ai.removeDriver(vehicleId)
```

`NETWORK`: Uses the network backend.

Stops AI control and releases its driver seat.

Requires world.vehicles and ownership of this AI controller. Cancels an active task and unmounts (but does not delete) the optional NPC, restoring its previous AI mode. Returns the detached state, or nil, reason.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |

Returns: `state, or nil, reason`

```lua
-- The ride is over: the NPC lets go of the wheel
Open77.vehicles.ai.removeDriver(taxiId)
```

[Autonomous vehicles & AI guide](../../01%20Documentation/vehicle-ai.md)

Registered in `LuaResourceRuntime.cs` (line 9002) as `LuaResourceRuntime.Bootstrap`.

## setBehavior

```lua
Open77.vehicles.ai.setBehavior(vehicleId, behavior)
```

`NETWORK`: Uses the network backend.

Selects normal, cautious or aggressive AI behavior.

Requires world.vehicles and controller ownership. normal keeps native avoidance; cautious uses a 75% speed ceiling; aggressive enables native clearTrafficOnPath. These are bounded presets, not a promise of police AI or traffic-law compliance.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |
| `behavior` | `string` | required | — |

Returns: `state, or nil, reason`

```lua
-- The getaway driver stops caring about traffic
Open77.vehicles.ai.setBehavior(getawayId, "aggressive")
```

[Autonomous vehicles & AI guide](../../01%20Documentation/vehicle-ai.md)

Registered in `LuaResourceRuntime.cs` (line 9040) as `LuaResourceRuntime.Bootstrap`.

## setSpeed

```lua
Open77.vehicles.ai.setSpeed(vehicleId, speed)
```

`NETWORK`: Uses the network backend.

Updates the native driving speed limit.

Requires world.vehicles and controller ownership. speed is finite, 0.5..55 m/s; default 12. Changes the active command without restarting it each frame. Actual speed remains subject to road geometry, obstacles and the vehicle. Cautious behavior applies 75% of the configured ceiling.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |
| `speed` | `number` | required | — |

Returns: `state, or nil, reason`

```lua
-- Slow the bus down in the school zone
Open77.vehicles.ai.setSpeed(busId, 6)
```

[Autonomous vehicles & AI guide](../../01%20Documentation/vehicle-ai.md)

Registered in `LuaResourceRuntime.cs` (line 9039) as `LuaResourceRuntime.Bootstrap`.

## setTrafficPattern

```lua
Open77.vehicles.ai.setTrafficPattern(vehicleId, pattern)
```

`NETWORK`: Uses the network backend.

Changes the native crowd move pattern of a running traffic enrollment.

Requires world.vehicles and ownership of a running joinTraffic task; on any other task it returns nil, traffic_pattern_inactive. pattern is default or stop, the record's own crowd move patterns, and any other name -- including the crowd panic move type, which is the separate panic call -- is refused with invalid_traffic_pattern rather than forwarded. default resolves to the vehicle's declared default or normal spec; stop keeps its exact declared name, and the engine's boolean acceptance stays mandatory. The change rides the same task and the same lane enrollment -- the task id does not move -- and the new pattern is a fresh request the client simulating the car answers: state.patternStatus reports whether it did (pending, applied, failed, inactive) and state.patternReason why a failure happened. Applied is the engine's acceptance on the current executor, not a guaranteed speed, and a handoff of the car puts the same request back to pending; the executor also re-applies it to every new join command it sends (after a collision recovery or a panic) without re-reporting an unchanged answer. An accepted pattern also ends a pending or active panic (panicStatus cancelled, panicReason pattern_changed). stop the pattern holds the car under the native join; it is not the stop command, which cancels the task.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |
| `pattern` | `string` | required | — |

Returns: `state, or nil, reason`

```lua
-- Hold the bus at its stop, then hand it back to the engine
CreateThread(function()
    Open77.vehicles.ai.setTrafficPattern(busId, "stop")
    Wait(3000)
    Open77.vehicles.ai.setTrafficPattern(busId, "default")
end)
```

[Autonomous vehicles & AI guide](../../01%20Documentation/vehicle-ai.md)

Registered in `LuaResourceRuntime.cs` (line 9022) as `LuaResourceRuntime.Bootstrap`.

## state

```lua
Open77.vehicles.ai.state(vehicleId)
```

`NETWORK`: Uses the network backend.

Reads authoritative vehicle AI state.

Requires world.vehicles. Returns the controller state or nil when no driver is attached. Contains vehicleId, npcId, taskId, patternRequest, panicRequest and panicThreatId (strings), revision, mode, status, reason, execution, phase, speed, behavior, destination x/y/z, routeIndex/count, targetKind/id, distance, owner, epoch, pattern, patternStatus, patternReason, panicStatus (inactive until the first panic request), panicReason, panicThreatKind, panicThreatX/Y/Z, panicDurationMs and panicRemainingMs. execution is the current lease holder's own report for a running task (enrolling, executing, or recovering while the join is released for a collision or panic). phase is the server's operational summary: enrolling (no simulator, no executing command or no fresh accepted motion), moving, healthy_waiting (executing with fresh accepted motion at 0.5 m/s or less), recovering, or failed_retiring (a failed, cancelled -- stop() included -- or detached task); neither is evidence of lane-legal driving. A waiting_for_simulator status carries reason simulator_capacity when a nearby client was skipped at its capacity limit, otherwise simulator_unavailable. Six consecutive unsuccessful simulator grants produce terminal failed with reason simulator_recovery_failed: explicit refusals and silent lease expiry without accepted motion both count, and accepted motion from the current owner and epoch resets the streak. This server grant streak is separate from local native collision-recovery attempts. Motion is still subject to the ordinary vehicle authority lease.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |

Returns: `state, or nil, reason`

```lua
-- Where is the taxi in its job?
local state = Open77.vehicles.ai.state(taxiId)
if state then print(state.status, state.mode, state.speed) end
```

[Autonomous vehicles & AI guide](../../01%20Documentation/vehicle-ai.md)

Registered in `LuaResourceRuntime.cs` (line 9003) as `LuaResourceRuntime.Bootstrap`.

## stop

```lua
Open77.vehicles.ai.stop(vehicleId)
```

`NETWORK`: Uses the network backend.

Cancels the active task and stops the vehicle.

Requires world.vehicles and controller ownership. Idempotent. Keeps the AI controller and driver seat reserved for the next task. Emits cancelled once for an active task. Use removeDriver to release the seat.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |

Returns: `state, or nil, reason`

```lua
-- Red light: stop, then continue two seconds later
CreateThread(function()
    Open77.vehicles.ai.stop(busId)
    Wait(2000)
    Open77.vehicles.ai.joinTraffic(busId)
end)
```

[Autonomous vehicles & AI guide](../../01%20Documentation/vehicle-ai.md)

Registered in `LuaResourceRuntime.cs` (line 9038) as `LuaResourceRuntime.Bootstrap`.
