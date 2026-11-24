# Globals (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/globals.md) · [Page web](https://open2077.net/docs/api/server/globals)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

230 functions.

## AddEventHandler

```lua
AddEventHandler(event, handler)
```

`SHARED`: Available without a live game instance.

Listens for an event in this resource; the same function as Open77.events.on.

No permission. One handler list serves three sources: this VM's TriggerLocalEvent, the host-wide bus, and the platform's own lifecycle events (onResourceStart, onPlayerReady, playerDropped), so a resource listens for its own and the platform's events the same way. Every argument of a platform lifecycle event arrives as a **string** -- a player id, a vehicle id, a revision -- so convert with `tonumber` before handing one to a native that wants a number (`Open77.chat.send`) and compare with `tostring(id)` against ids read from natives. Handlers run as managed tasks and may Wait. The limit is 2,048 handlers per resource; beyond it registration raises `handler quota exceeded`. Returns the id RemoveEventHandler takes.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `handler` | `function` | required | — |

Returns: `handler id`

```lua
-- React to a player leaving: save their money before the session is gone
AddEventHandler("onPlayerDisconnected", function(playerId, reason)
    -- playerId is a string here, like every host-event argument
    local key = "money:" .. (Open77.players.identifier(playerId) or "")
    Open77.kvp.set(key, wallet[playerId] or 0)
    wallet[playerId] = nil
end)
```

Registered in `LuaResourceRuntime.cs` (line 6296) as `LuaResourceRuntime.Bootstrap`.

## AddStateBagChangeHandler

```lua
AddStateBagChangeHandler([key], [bagName], handler)
```

`SHARED`: Available without a live game instance.

Subscribes to bag changes, FiveM style.

Wraps `Open77.state.onChange` in the FiveM signature. `key` may be a key name, or nil or `"*"` for every key. `bagName` may be nil for every bag, `"global"`, `"player:<id>"`, `"<kind>:<id>"`, or `"entity:<id>"`, which matches any entity kind carrying that id. The handler receives `(bagName, key, value, reserved, replicated)`; `reserved` is always nil and `replicated` always true, so a ported signature lines up. It runs at a tick boundary rather than inside the write, so it may Wait. Only subscribed resources are touched: the filter is on the host side, so one resource's interest in a key does not marshal that key into every other resource.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string` | optional | — |
| `bagName` | `string` | optional | — |
| `handler` | `function` | required | — |

Returns: `subscription id`, `or nil, reason`

```lua
AddStateBagChangeHandler("cuffed", nil, function(bagName, key, value)
    print(("%s %s=%s"):format(bagName, key, tostring(value)))
end)
```

Registered in `LuaResourceRuntime.State.cs` (line 463) as `LuaResourceRuntime.Bootstrap`.

## AdoptElevator

```lua
AdoptElevator(engineEntity, x, y, z, bucket, initialFloor, flags, floorCount)
```

`NETWORK`: Uses the network backend.

Takes a native lift under server authority so every player sees the same cab.

The all-positional form of `Open77.elevators.adopt`. `engineEntity` is the lift's engine entity id, which only a client can find (`Open77.elevators.nearby` on the client side); the position is the cab's. `flags` is a bitmask of `1` powered, `2` locked, `4` interaction allowed, `8` doors closed. Answers the elevator id, or `nil, reason`. Requires `world.elevators`. Note the tail order: `flags` is the seventh argument and `floorCount` the eighth, which is why the named form is the safer one to write.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `engineEntity` | `value` | required | — |
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `z` | `number` | required | — |
| `bucket` | `integer` | required | — |
| `initialFloor` | `integer` | required | — |
| `flags` | `integer` | required | — |
| `floorCount` | `integer` | required | — |

```lua
-- Adopt the apartment lift a client just reported, powered and usable.
RegisterNetEvent("lift:found", function(engineEntity, x, y, z, floors)
    local lift, reason = AdoptElevator(
        engineEntity, x, y, z,   -- engineEntity, cab position
        0, 0,                    -- bucket, initialFloor
        1 | 4,                   -- flags: powered + interaction allowed
        floors)                  -- floorCount
    if not lift then return print("adopt failed: " .. tostring(reason)) end
    print(("lift %d adopted with %d floors"):format(lift, floors))
end)
```

Registered in `LuaResourceRuntime.cs` (line 1099) as `AdoptElevator`.

## AttachEffect

```lua
AttachEffect(targetId, kind, effect, slot, localAnchor, localSlot, ttlMs?, radius, hysteresis, localEvent?, soundEvent?, soundOnOwner?)
```

`NETWORK`: Uses the network backend.

`world.effects`;

`world.effects`; decimal effect ID or nil/reason. Prefer `Open77.effects.attach`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `targetId` | `value` | required | — |
| `kind` | `value` | required | — |
| `effect` | `value` | required | — |
| `slot` | `value` | required | — |
| `localAnchor` | `value` | required | — |
| `localSlot` | `value` | required | — |
| `ttlMs?` | `value` | required | — |
| `radius` | `value` | required | — |
| `hysteresis` | `value` | required | — |
| `localEvent?` | `value` | required | — |
| `soundEvent?` | `value` | required | — |
| `soundOnOwner?` | `value` | required | — |

```lua
-- A smoke plume pinned to a burning car's bonnet, gone in a minute.
local id, reason = AttachEffect(
    vehicleId, "vehicle", "smoke.column", "bonnet",   -- target, kind, effect, slot
    "root", "root",                                   -- localAnchor, localSlot
    60000,                                            -- ttlMs
    120.0, 30.0,                                      -- streaming radius, hysteresis
    nil, "amb_fire_medium", true)                     -- localEvent, soundEvent, soundOnOwner
if not id then return print("attach refused: " .. tostring(reason)) end
-- Open77.effects.attach(target, effect, options) is the same call with names.
```

Registered in `LuaResourceRuntime.cs` (line 1050) as `AttachEffect`.

## BanPlayer

```lua
BanPlayer(playerId, reason, seconds)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.players.ban`.

The low-level form of `Open77.players.ban` (bound to the same native). The two are the same function under two names; `Open77.players.ban` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `reason` | `string` | required | — |
| `seconds` | `integer` | required | — |

```lua
-- BanPlayer is the low-level form of Open77.players.ban (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- /ban <id> <days> <reason>: a temporary ban published through the master
RegisterCommand("ban", function(source, args)
    local target, days = tonumber(args[1]), tonumber(args[2])
    if not target or not days then return Open77.chat.send(source, "Usage: /ban <id> <days> <reason>") end
    local reason = table.concat(args, " ", 3)
    local ok, err = Open77.players.ban(target, reason, days * 86400)
    if not ok then Open77.chat.send(source, "Ban failed: " .. err) end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 1027) as `BanPlayer`.

_This signature was inferred from the handler body._

## CancelEvent

```lua
CancelEvent()
```

`NETWORK`: Uses the network backend.

Server only. Vetoes the cancellable event this handler is running under.

Meaningful only inside a handler reached by TriggerCancellableEvent; anywhere else it answers false, not_in_cancellable_event rather than raising. The flag is shared by every resource in the dispatch, so the first cancel wins and the remaining handlers still run and can read it.

Returns: `true, or false, reason`

```lua
-- A chat filter: veto any message containing a banned word
AddEventHandler("chatMessage", function(source, name, text)
    if text:lower():find("badword", 1, true) then
        CancelEvent()
        Open77.chat.send(source, "That word is not allowed here.")
    end
end)
```

Registered in `LuaResourceRuntime.Events.cs` (line 175) as `LuaResourceRuntime.Bootstrap`.

## CancelNpcTask

```lua
CancelNpcTask(npcId, taskId, reason)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.npcs.tasks.cancel`.

The low-level form of `Open77.npcs.tasks.cancel` (bound to the same native). The two are the same function under two names; `Open77.npcs.tasks.cancel` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `npcId` | `integer` | required | — |
| `taskId` | `integer` | required | — |
| `reason` | `string` | required | — |

```lua
-- CancelNpcTask is the low-level form of Open77.npcs.tasks.cancel (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- The player paid: the guard stops attacking
local task = Open77.npcs.tasks.attack(guardId, playerId)
AddEventHandler("debt:paid", function()
    Open77.npcs.tasks.cancel(guardId, task)
end)
```

Registered in `LuaResourceRuntime.cs` (line 1085) as `CancelNpcTask`.

_This signature was inferred from the handler body._

## ClearNpcTasks

```lua
ClearNpcTasks(npcId, channel, reason)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.npcs.tasks.clear`.

The low-level form of `Open77.npcs.tasks.clear` (bound to the same native). The two are the same function under two names; `Open77.npcs.tasks.clear` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `npcId` | `integer` | required | — |
| `channel` | `integer` | required | — |
| `reason` | `string` | required | — |

```lua
-- ClearNpcTasks is the low-level form of Open77.npcs.tasks.clear (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Stop everything the NPC was doing
Open77.npcs.tasks.clear(npcId)
```

Registered in `LuaResourceRuntime.cs` (line 1086) as `ClearNpcTasks`.

_This signature was inferred from the handler body._

## ClearProps

```lua
ClearProps()
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.props.clear`.

The low-level form of `Open77.props.clear` (bound to the same native). The two are the same function under two names; `Open77.props.clear` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

```lua
-- ClearProps is the low-level form of Open77.props.clear (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- The event is over: remove every prop this resource placed
AddEventHandler("event:ended", function()
    print("removed", Open77.props.clear(), "props")
end)
```

Registered in `LuaResourceRuntime.cs` (line 1042) as `ClearProps`.

_This signature was inferred from the handler body._

## ClearTick

```lua
ClearTick(id)
```

`SHARED`: Available without a live game instance.

Cancels a tick created by SetTick.

Answers false for an id that is not a live tick. Cancellation goes through a flag rather than ClearTimeout because a tick pass pulls every due task out of the queue before resuming any of them, and a task cancelled by id while it is already in that batch would be put back. The flag is read at the top of the loop, so the body never runs again after this returns.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `boolean`

```lua
-- Count down on every tick until the round starts, then stop the tick
local tick
tick = SetTick(function()
    if GetGameTimer() >= roundStartAt then
        ClearTick(tick)
        startRound()
    end
end)
```

Registered in `LuaResourceRuntime.Compat.cs` (line 93) as `LuaResourceRuntime.Bootstrap`.

## ClearTimeout

```lua
ClearTimeout(id)
```

`SHARED`: Available without a live game instance.

Cancels a timer created by SetTimeout.

Drops the task before it runs. An id that has already fired, or was never issued, is ignored rather than raising.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

```lua
-- The bomb defusal: cancel the explosion timer when the bomb is defused
local fuse = SetTimeout(30000, function() detonate() end)
RegisterNetEvent("bomb:defused", function()
    ClearTimeout(fuse)
    Open77.chat.broadcast("Bomb defused!")
end)
```

Registered in `LuaResourceRuntime.cs` (line 6292) as `LuaResourceRuntime.Bootstrap`.

## CreateEffect

```lua
CreateEffect(name, x, y, z, qx, qy, qz, qw, bucket, [visible], radius, hysteresis, [ttl])
```

`NETWORK`: Uses the network backend.

Registers a looping particle effect at a world point and answers its id as a decimal string.

The all-positional form of `Open77.effects.create`. The orientation is a quaternion (`qx, qy, qz, qw`), not a yaw, and `visible` is the tenth argument. A registry entry is a **running** system that keeps playing until `Open77.effects.remove` or its `ttl` expires -- `PlayEffect` is the one-shot. Answers `nil, reason` on `invalid_position` or an unknown effect name. Requires `world.effects`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `z` | `number` | required | — |
| `qx` | `number` | required | — |
| `qy` | `number` | required | — |
| `qz` | `number` | required | — |
| `qw` | `number` | required | — |
| `bucket` | `integer` | required | — |
| `visible` | `boolean` | optional | — |
| `radius` | `number` | required | — |
| `hysteresis` | `number` | required | — |
| `ttl` | `integer` | optional | — |

```lua
-- A burning barrel that keeps burning until the resource removes it.
local fire, reason = CreateEffect(
    "fire.barrel", -1442.2, 127.4, 18.0,   -- name, x, y, z
    0.0, 0.0, 0.0, 1.0,                    -- quaternion: no rotation
    0, true,                               -- bucket, visible
    120.0, 30.0,                           -- streaming radius, hysteresis
    nil)                                   -- ttlMs: nil means it never expires on its own
if not fire then return print("effect refused: " .. tostring(reason)) end
SetTimeout(30000, function() Open77.effects.remove(fire) end)
```

Registered in `LuaResourceRuntime.cs` (line 1044) as `CreateEffect`.

## CreateLootDrop

```lua
CreateLootDrop(item, quantity, x, y, z, bucket, pickupRadius, label, model, ttlMs)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.loot.create`.

The low-level form of `Open77.loot.create` (wrapped by). The two are the same function under two names; `Open77.loot.create` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `item` | `string` | required | — |
| `quantity` | `integer` | required | — |
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `z` | `number` | required | — |
| `bucket` | `integer` | required | — |
| `pickupRadius` | `number` | required | — |
| `label` | `string` | required | — |
| `model` | `string` | required | — |
| `ttlMs` | `number` | required | — |

```lua
-- CreateLootDrop is the low-level form of Open77.loot.create (wrapped by);
-- the example shows the Open77.* spelling resources are told to prefer.
-- A dead player drops 200 eddies where they fell, gone after five minutes
AddEventHandler("onPlayerLifeStateChanged", function(playerId, revision, phase)
    if phase ~= "dead" then return end
    local pos = Open77.players.position(playerId)
    if pos then
        Open77.loot.create({ item = "Items.money", quantity = 200, position = pos, label = "Eddies", ttlMs = 300000 })
    end
end)
```

Registered in `LuaResourceRuntime.cs` (line 1052) as `CreateLootDrop`.

_This signature was inferred from the handler body._

## CreateNpc

```lua
CreateNpc(template, x, y, z, yaw, bucket, appearance, loadoutJson, aiMode, damagePolicy, health, maxHealth, streamingRadius, streamingHysteresis, [despawnWhenUnobserved], [persistent], behaviorJson, authorityPolicy)
```

`NETWORK`: Uses the network backend.

Spawns a server-owned NPC from a 2.31 `Character.*` record and answers its id.

The all-positional form of `Open77.npcs.create`. Seventeen arguments in a fixed order, where `aiMode` is `0` tasks / `1` frozen / `2` native / `3` observer and `damagePolicy` is `0` mortal / `1` immortal / `2` invulnerable; `loadoutJson` and `behaviorJson` are JSON **strings**, not tables. Answers the new id, or `nil, reason` when the record, the position or an argument is refused. Requires `world.npcs`. `Open77.npcs.create` takes the same values as a named table and fills in every default -- prefer it, and keep this one for ported code.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `template` | `string` | required | — |
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `z` | `number` | required | — |
| `yaw` | `number` | required | — |
| `bucket` | `integer` | required | — |
| `appearance` | `string` | required | — |
| `loadoutJson` | `string` | required | — |
| `aiMode` | `integer` | required | — |
| `damagePolicy` | `integer` | required | — |
| `health` | `number` | required | — |
| `maxHealth` | `number` | required | — |
| `streamingRadius` | `number` | required | — |
| `streamingHysteresis` | `number` | required | — |
| `despawnWhenUnobserved` | `boolean` | optional | — |
| `persistent` | `boolean` | optional | — |
| `behaviorJson` | `string` | required | — |
| `authorityPolicy` | `string` | required | — |

```lua
-- The raw positional spawn. Ten of the seventeen arguments are only the
-- defaults the table form would have filled in, which is why nobody writes
-- this by hand twice: Open77.npcs.create{ record = ..., position = ... }.
local npcId, reason = CreateNpc(
    "Character.Judy", -1442.2, 127.4, 18.0, 180.0, 0,  -- record, x, y, z, yaw, bucket
    nil, nil,                                          -- appearance, loadoutJson
    0, 2,                                              -- aiMode = tasks, damagePolicy = invulnerable
    100.0, 100.0, 180.0, 40.0,                         -- health, maxHealth, streaming radius/hysteresis
    false, true, nil)                                  -- despawnWhenUnobserved, persistent, behaviorJson
if not npcId then return print("spawn refused: " .. tostring(reason)) end
print(("shopkeeper %d is up"):format(npcId))
```

Registered in `LuaResourceRuntime.cs` (line 1089) as `CreateNpc`.

## CreateProp

```lua
CreateProp(model, x, y, z, yaw, bucket, [kind], [physics], [appearance], [collision], [visible], scaleX, scaleY, scaleZ, radius, hysteresis, [light], [ttl])
```

`NETWORK`: Uses the network backend.

Places a replicated prop from a catalogue model and answers its id as a decimal string.

The all-positional form of `Open77.props.create`. `model` is a catalogue alias (`Open77.props.catalog()` lists them), not a `.mesh` path; `kind` defaults to `"prop"` and `physics` to `"static"`. The id comes back as a **string**, because a prop id does not fit a Lua number. Answers `nil, reason` on `invalid_model`, `invalid_position` or a registry refusal. Requires `world.props`. Prefer `Open77.props.create`, which takes the same values as a table and needs only the four you care about.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `model` | `string` | required | — |
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `z` | `number` | required | — |
| `yaw` | `number` | required | — |
| `bucket` | `integer` | required | — |
| `kind` | `string` | optional | — |
| `physics` | `string` | optional | — |
| `appearance` | `string` | optional | — |
| `collision` | `boolean` | optional | — |
| `visible` | `boolean` | optional | — |
| `scaleX` | `number` | required | — |
| `scaleY` | `number` | required | — |
| `scaleZ` | `number` | required | — |
| `radius` | `number` | required | — |
| `hysteresis` | `number` | required | — |
| `light` | `string` | optional | — |
| `ttl` | `integer` | optional | — |

```lua
-- A crate at the drop point, gone after an hour. Eighteen positional
-- arguments for what Open77.props.create writes in four lines.
local crate, reason = CreateProp(
    "crate.small", -1442.2, 127.4, 18.0, 45.0, 0,   -- model, x, y, z, yaw, bucket
    nil, nil, nil,                                  -- kind, physics, appearance
    true, true,                                     -- collision, visible
    1.0, 1.0, 1.0,                                  -- scale
    120.0, 30.0,                                    -- streaming radius, hysteresis
    nil, 60 * 60 * 1000)                            -- light, ttlMs
if not crate then return print("no crate: " .. tostring(reason)) end
print("crate id " .. crate)   -- a string, not a number
```

Registered in `LuaResourceRuntime.cs` (line 1035) as `CreateProp`.

## CreateThread

```lua
CreateThread(body)
```

`SHARED`: Available without a live game instance.

Starts a coroutine this resource's scheduler manages.

Its body may Wait, which an ordinary function cannot. Returns the task id. A resource may hold 1,024 scheduled tasks at once and raises `task quota exceeded` beyond that; stopping the resource drops every task it owns. Command, event and callback handlers already run as managed tasks, so they may Wait without a CreateThread of their own.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `body` | `function` | required | — |

Returns: `task id`

```lua
CreateThread(function()
    while true do
        Wait(60000)
        print("still here")
    end
end)
```

Registered in `LuaResourceRuntime.cs` (line 6277) as `LuaResourceRuntime.Bootstrap`.

## CreateVehicle

```lua
CreateVehicle(record, x, y, z, yaw, bucket, appearance, health, flags, primary1, primary2, primary3, secondary1, secondary2, secondary3)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.vehicles.create`.

The low-level form of `Open77.vehicles.create` (wrapped by). The two are the same function under two names; `Open77.vehicles.create` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `record` | `string` | required | — |
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `z` | `number` | required | — |
| `yaw` | `number` | required | — |
| `bucket` | `integer` | required | — |
| `appearance` | `string` | required | — |
| `health` | `number` | required | — |
| `flags` | `integer` | required | — |
| `primary1` | `integer` | required | — |
| `primary2` | `integer` | required | — |
| `primary3` | `integer` | required | — |
| `secondary1` | `integer` | required | — |
| `secondary2` | `integer` | required | — |
| `secondary3` | `integer` | required | — |

```lua
-- CreateVehicle is the low-level form of Open77.vehicles.create (wrapped by);
-- the example shows the Open77.* spelling resources are told to prefer.
-- /car: spawn a Quadra a few metres in front of the caller and put them in the driver seat
RegisterCommand("car", function(source)
    local me = Open77.players.get(source)
    if not me or not me.position then return end
    local id, reason = Open77.vehicles.create({
        record = "Vehicle.v_sport1_quadra_turbo_r_player",
        position = { x = me.position.x + 4.0, y = me.position.y, z = me.position.z },
        yaw = me.heading,
        ttlMs = 30 * 60 * 1000, -- gone after 30 minutes if nobody removes it
    })
    if not id then return Open77.chat.send(source, "Spawn failed: " .. reason) end
    Open77.vehicles.warpPlayerIntoVehicle(source, id, Open77.vehicles.seats.driver)
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 1057) as `CreateVehicle`.

_This signature was inferred from the handler body._

## DamageNpc

```lua
DamageNpc(id, amount, source, cause)
```

`NETWORK`: Uses the network backend.

Applies damage to an owned NPC and answers whether it landed.

The all-positional form of `Open77.npcs.applyDamage`. `source` defaults to `"script"` and `cause` to `"generic"`; both are free labels carried to the death event, not an enumeration. Answers a plain `true`/`false` with no reason -- `false` covers a missing NPC, one this resource does not own, and a `world.npcs` permission the resource was not granted. An invulnerable NPC takes no damage, so read the health back if you need to know.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `amount` | `number` | required | — |
| `source` | `string` | required | — |
| `cause` | `string` | required | — |

```lua
-- Bleed a guard down over five seconds, then say what happened.
CreateThread(function()
    for _ = 1, 5 do
        if not DamageNpc(guardId, 20.0, "script", "poison") then
            return print("damage refused: no such NPC, or not ours")
        end
        Wait(1000)
    end
    local npc = Open77.npcs.get(guardId)
    print(("guard health: %.0f"):format(npc and npc.health or 0))
end)
```

Registered in `LuaResourceRuntime.cs` (line 1091) as `DamageNpc`.

## DamagePlayer

```lua
DamagePlayer(playerId, amount, attacker, kindName, weaponTdbId)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.players.damage`.

The low-level form of `Open77.players.damage` (wrapped by). The two are the same function under two names; `Open77.players.damage` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `amount` | `number` | required | — |
| `attacker` | `integer` | required | — |
| `kindName` | `string` | required | — |
| `weaponTdbId` | `integer` | required | — |

```lua
-- DamagePlayer is the low-level form of Open77.players.damage (wrapped by);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Fall damage reported by a zone trigger: 20 points, no attacker
RegisterNetEvent("parkour:badLanding", function()
    Open77.players.damage(source, 20, { cause = "fall" })
end)
```

Registered in `LuaResourceRuntime.cs` (line 1133) as `DamagePlayer`.

_This signature was inferred from the handler body._

## DeleteResourceKvp

```lua
DeleteResourceKvp(key)
```

`NETWORK`: Uses the network backend.

FiveM-style alias for `Open77.kvp.delete`.

See `SetResourceKvp`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string` | required | — |

Returns: `boolean`, `reason`

```lua
-- A FiveM port: forget a player's saved outfit
DeleteResourceKvp("outfit:" .. Open77.players.identifier(source))
```

Registered in `LuaResourceRuntime.Kvp.cs` (line 317) as `LuaResourceRuntime.Bootstrap`.

## DestroySound

```lua
DestroySound(id)
```

`NETWORK`: Uses the network backend.

DestroySound: server-authoritative network package audio.

Requires audio.network; network=false is rejected. Server owns the timeline, handles and bucket/optional-target audience. Clients cannot request broadcasts directly. getState reports intended playback, not decoder acknowledgement. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `true, or failure, reason`

```lua
-- Release the network handle once the concert resource is done with it
DestroySound(concertId)
```

Registered in `LuaResourceRuntime.cs` (line 9881) as `LuaAudioControl`.

## DoesEntityExist

```lua
DoesEntityExist(kind, id)
```

`NETWORK`: Uses the network backend.

Whether an id is live in the registry that owns that kind.

FiveM has one opaque handle space; Open77 has one registry per kind, so the kind has to be named: vehicle, npc, prop, loot, effect, elevator or player. The one-argument FiveM form cannot be answered and is refused with false, kind_required rather than guessed at. Each kind is answered by the registry that already owns it, under that registry's permissions, so a resource that may not read vehicles gets false -- the same answer an absent id gets. Call the namespace function directly when you need to tell denied from absent. Other reasons: id_required, invalid_entity_id, unknown_entity_kind.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `kind` | `string` | required | — |
| `id` | `integer` | required | — |

Returns: `boolean, or false, reason`

```lua
-- The tow job stored a vehicle id an hour ago; is the car still there?
if not DoesEntityExist("vehicle", job.vehicleId) then
    Open77.chat.send(job.playerId, "The car you were towing is gone.")
    return
end
```

Registered in `LuaResourceRuntime.Compat.cs` (line 260) as `LuaResourceRuntime.Bootstrap`.

## DropPlayer

```lua
DropPlayer(playerId, reason)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.players.disconnect`.

The low-level form of `Open77.players.disconnect` (bound to the same native). The two are the same function under two names; `Open77.players.disconnect` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `reason` | `string` | required | — |

```lua
-- DropPlayer is the low-level form of Open77.players.disconnect (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- /kick <id> <reason>
RegisterCommand("kick", function(source, args)
    local target = tonumber(args[1])
    if not target then return end
    local reason = table.concat(args, " ", 2)
    Open77.players.disconnect(target, reason ~= "" and reason or "Kicked by an admin.")
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 1026) as `DropPlayer`.

_This signature was inferred from the handler body._

## EnqueueNpcTask

```lua
EnqueueNpcTask(npcId, type, parametersJson, channel, priority, timeoutMilliseconds)
```

`NETWORK`: Uses the network backend.

Queues one task on an owned NPC's task channel and answers the task id.

The all-positional form of `Open77.npcs.tasks.enqueue`, and the single native behind every `Open77.npcs.tasks.*` helper. `parametersJson` is a JSON **string**, not a table -- encode it yourself. `channel` is `0` movement / `1` look / `2` action / `3` full body, and a queued task replaces whatever else that channel is doing at the same or lower `priority`. `timeoutMs` of `0` means no timeout. Answers the task id, or `nil, reason` (`npc_not_found`, a rejected task type). Requires `world.npcs`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `npcId` | `integer` | required | — |
| `type` | `string` | required | — |
| `parametersJson` | `string` | required | — |
| `channel` | `integer` | required | — |
| `priority` | `integer` | required | — |
| `timeoutMilliseconds` | `integer` | required | — |

```lua
-- Walk a guard to the door, on the movement channel, giving up after 20 s.
local taskId, reason = EnqueueNpcTask(
    guardId, "moveTo",                                     -- npc, task type
    '{"x":-1442.2,"y":127.4,"z":18.0,"speed":"walk"}',     -- parameters, as JSON TEXT
    0, 100, 20000)                                         -- channel = movement, priority, timeoutMs
if not taskId then return print("task refused: " .. tostring(reason)) end
-- Open77.npcs.tasks.moveTo(guardId, position, options) encodes all of that for you.
```

Registered in `LuaResourceRuntime.cs` (line 1084) as `EnqueueNpcTask`.

## Entity

```lua
Entity(entityId)
```

`SHARED`: Available without a live game instance.

Addresses one entity's state bag, FiveM style.

`Entity(id).state` is `Open77.state.entity(kind, id)` with the kind resolved by looking the id up in the vehicle, NPC and prop registries in turn. FiveM has one entity namespace and Open77 has three, so an id that exists in more than one registry is ambiguous: prefer `Open77.state.entity(kind, id)` in new code and keep this for ported scripts. The bag's audience is the entity's own viewer set, so it is visible exactly when the entity is, and it dies with it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entityId` | `integer` | required | — |

Returns: `a table with `state``

```lua
-- FiveM-style write of a vehicle's state bag: who holds the keys
Entity(vehicleId).state:set("keys", { "12", "13" })
```

Registered in `LuaResourceRuntime.State.cs` (line 449) as `LuaResourceRuntime.Bootstrap`.

## ExecuteCommand

```lua
ExecuteCommand(line)
```

`NETWORK`: Uses the network backend.

FiveM-style alias for `Open77.runtime.executeCommand`.

Requires `runtime.commands`, and reaches the resource lifecycle only with `resources.control`. That is the one deliberate difference from FiveM, where this call carries console authority undeclared.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `line` | `string` | required | — |

Returns: `true when queued, otherwise false`, `reason`

```lua
-- Restart the shop resource from an admin command (needs runtime.commands + resources.control)
RegisterCommand("reloadshop", function(source)
    ExecuteCommand("restart shop")
end, true)
```

Registered in `LuaResourceRuntime.RuntimeControl.cs` (line 367) as `LuaResourceRuntime.Bootstrap`.

## exports

```lua
exports(name, fn)
```

`SHARED`: Available without a live game instance.

Publish a server export, or index another resource's exports for a synchronous call.

Called, `exports(name, fn)` registers `fn` under `name` for this resource: `true` on success, or `nil, reason` with `invalid_export_name` (empty, longer than 64 characters or containing NUL), `export_function_required`, `export_registration_limit` (256 names per resource) or `resource_stopping`. Registering a name again replaces the previous function, which is what `server_export` entries in the manifest do at start-up through this same call. Indexed, `exports.other:name(...)` or `exports['other-resource']:name(...)` is the synchronous proxy: it runs that resource's export inline in its own VM and returns its values, and **raises** (`export other:name <reason>`) instead of returning `nil, reason`, because FiveM's does and ported resources rely on it. Neither `exports` nor a proxy accepts assignment; `exports.x = f` raises `Open77: exports is not assignable`. No permission on either side. The asynchronous, non-raising spelling is `Open77.exports.call`, which returns an `Open77.Promise`; see [Cross-resource server exports](https://open2077.net/docs/api/server/server-exports.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string (export name, 1-64 characters)` | required | — |
| `fn` | `function` | required | — |

Returns: `true`, `nil, reason when the registration is refused`

```lua
-- Publish a balance lookup, then call another resource's export inline.
exports("balance", function(playerId)
    return accounts[playerId] or 0
end)

RegisterCommand("rent", function(source)
    local ok, err = pcall(function()
        return exports.garage:chargeRent(source, 250)
    end)
    if not ok then print("garage refused:", err) end
end, false)
```

Registered in `LuaResourceRuntime.Exports.cs` (line 29) as `LuaResourceRuntime`.

## FadePlayerScreen

```lua
FadePlayerScreen(playerId, out, durationMs?, options?)
```

`NETWORK`: Uses the network backend.

Cover (`out` true) or uncover one player's screen with the native quest fade.

Cover (`out` true) or uncover one player's screen with the native quest fade. The client consumes the relay itself, so the target needs no client resource; `options.timeoutMs` (1000-60000, default 15000) is a ceiling the **client** enforces on real time, so a forgotten fade-in costs a pause, not a black screen. See [screen transitions](https://open2077.net/docs/api/server/screen-transitions.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |
| `out` | `value` | required | — |
| `durationMs?` | `value` | required | — |
| `options?` | `value` | required | — |

```lua
-- FadePlayerScreen is the low-level form of Open77.players.fade (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Bracket a teleport with a fade so the player never sees the streaming
CreateThread(function()
    Open77.players.fade(playerId, true, 400)
    Wait(400)
    Open77.players.teleport(playerId, { x = -1442.2, y = 127.4, z = 18.0 }, { fade = false })
    Open77.players.fade(playerId, false, 400)
end)
```

Registered in `LuaResourceRuntime.cs` (line 1130) as `FadePlayerScreen`.

## ForcePlayerOutOfVehicle

```lua
ForcePlayerOutOfVehicle(playerId, vehicleId)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.vehicles.forcePlayerOutOfVehicle`.

The low-level form of `Open77.vehicles.forcePlayerOutOfVehicle` (bound to the same native). The two are the same function under two names; `Open77.vehicles.forcePlayerOutOfVehicle` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `vehicleId` | `integer` | required | — |

```lua
-- ForcePlayerOutOfVehicle is the low-level form of Open77.vehicles.forcePlayerOutOfVehicle (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Police stop: pull the driver out of whatever they are driving
RegisterCommand("pullout", function(source, args)
    local target = tonumber(args[1])
    if not target then return end
    local ok, reason = Open77.vehicles.forcePlayerOutOfVehicle(target)
    if not ok then Open77.chat.send(source, "Could not eject: " .. reason) end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 1073) as `ForcePlayerOutOfVehicle`.

_This signature was inferred from the handler body._

## GetConvar

```lua
GetConvar(name, [default])
```

`NETWORK`: Uses the network backend.

Reads a convar as a string, or the caller's default.

A COMPATIBILITY SHIM for ported code, not the mechanism to reach for when writing something new -- see `Open77.tunables`, which is typed, range-checked, validated on every write, retunable from the Warden panel while the server runs, persisted across a restart, and scoped to the declaring resource. Resolution order: an in-memory `SetConvar` override, then the `convars` block of `server.jsonc`, then **this resource's own declared tunables by exact key**, then the default. That third step is how a port migrates to the better mechanism one key at a time. Never raises: an unset convar answers the default. Names are case-insensitive.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `default` | `any` | optional | — |

Returns: `string, or the default`

```lua
-- A FiveM port reads its config the way it always did
local webhook = GetConvar("myshop_webhook", "")
if webhook == "" then print("myshop_webhook is not set") end
```

Registered in `LuaResourceRuntime.Convars.cs` (line 116) as `LuaResourceRuntime.Bootstrap`.

## GetConvarBool

```lua
GetConvarBool(name, [default])
```

`NETWORK`: Uses the network backend.

Reads a convar as a boolean, or the caller's default.

`true`, `1`, `yes` and `on` are true; `false`, `0`, `no` and `off` are false; anything else is not a boolean and the default stands. The same vocabulary the console's tunable parser accepts, so an operator does not have to remember two.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `default` | `boolean` | optional | — |

Returns: `boolean, or the default`

```lua
-- Turn PvP on from the server config: "myserver_pvp": "true"
local pvp = GetConvarBool("myserver_pvp", false)
Open77.combat.setFriendlyFire(pvp)
```

Registered in `LuaResourceRuntime.Convars.cs` (line 131) as `LuaResourceRuntime.Bootstrap`.

## GetConvarInt

```lua
GetConvarInt(name, [default])
```

`NETWORK`: Uses the network backend.

Reads a convar as an integer, or the caller's default.

Never raises. A value that is not a whole number answers the default rather than failing -- a ported script reads a convar that may simply not be set, and a strict parse would turn a missing setting into a crash on the first frame.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `default` | `integer` | optional | — |

Returns: `integer, or the default`

```lua
-- The starting money of a new player, from the convars block
local startingMoney = GetConvarInt("myserver_starting_money", 500)
```

Registered in `LuaResourceRuntime.Convars.cs` (line 123) as `LuaResourceRuntime.Bootstrap`.

## GetCurrentResourceGeneration

```lua
GetCurrentResourceGeneration()
```

`NETWORK`: Uses the network backend.

Opaque generation identity of this server VM, changed on restart/reload.

Opaque generation identity of this server VM, changed on restart/reload.

```lua
-- Ignore a reply that belongs to the VM we had before this reload.
local generation = GetCurrentResourceGeneration()
AddEventHandler("bank:reply", function(payload)
    if payload.generation ~= generation then
        return print("stale reply from a previous generation, dropped")
    end
    print("fresh reply accepted")
end)
```

Registered in `LuaResourceRuntime.Exports.cs` (line 29) as `LuaResourceRuntime`.

## GetCurrentResourceName

```lua
GetCurrentResourceName()
```

`SHARED`: Available without a live game instance.

Return this manifest's resource name.

The `resource "..."` line of this resource's own `open77.lua`, as the host loaded it. A local read with no permission and no failure; `Open77.resource.name()` is the same function. Compare it with the argument of `onResourceStart` / `onResourceStop` to react to this resource's own lifecycle only.

Returns: `resource name`

```lua
AddEventHandler("onResourceStart", function(name)
    if name ~= GetCurrentResourceName() then return end
    print("up")
end)
```

Registered in `LuaResourceRuntime.cs` (line 956) as `GetCurrentResourceName`.

## GetEffect

```lua
GetEffect(id)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.effects.get`.

The low-level form of `Open77.effects.get` (bound to the same native). The two are the same function under two names; `Open77.effects.get` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

```lua
-- GetEffect is the low-level form of Open77.effects.get (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Is the campfire still burning?
local fire = Open77.effects.get(campfireId)
if not fire then relight() end
```

Registered in `LuaResourceRuntime.cs` (line 1047) as `GetEffect`.

_This signature was inferred from the handler body._

## GetEffects

```lua
GetEffects(bucket)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.effects.all`.

The low-level form of `Open77.effects.all` (bound to the same native). The two are the same function under two names; `Open77.effects.all` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | required | — |

```lua
-- GetEffects is the low-level form of Open77.effects.all (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Count the fires still burning in the arena instance
local burning = 0
for _, effect in ipairs(Open77.effects.all(ARENA_BUCKET)) do
    if effect.effect:sub(1, 5) == "fire." then burning = burning + 1 end
end
```

Registered in `LuaResourceRuntime.cs` (line 1048) as `GetEffects`.

_This signature was inferred from the handler body._

## GetElevator

```lua
GetElevator(id)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.elevators.get`.

The low-level form of `Open77.elevators.get` (bound to the same native). The two are the same function under two names; `Open77.elevators.get` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

```lua
-- GetElevator is the low-level form of Open77.elevators.get (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Only accept a new request when the lift is idle
local lift = Open77.elevators.get(elevatorId)
if lift and lift.phase == "idle" then Open77.elevators.goTo(elevatorId, 7) end
```

Registered in `LuaResourceRuntime.cs` (line 1106) as `GetElevator`.

_This signature was inferred from the handler body._

## GetElevators

```lua
GetElevators(bucket)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.elevators.all`.

The low-level form of `Open77.elevators.all` (bound to the same native). The two are the same function under two names; `Open77.elevators.all` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | required | — |

```lua
-- GetElevators is the low-level form of Open77.elevators.all (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Send every lift back to the ground floor at server start
for _, lift in ipairs(Open77.elevators.all()) do
    Open77.elevators.goTo(lift.id, 0)
end
```

Registered in `LuaResourceRuntime.cs` (line 1107) as `GetElevators`.

_This signature was inferred from the handler body._

## GetEntityRoutingBucket

```lua
GetEntityRoutingBucket(entityId)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.routingBuckets.getEntity`.

The low-level form of `Open77.routingBuckets.getEntity` (bound to the same native). The two are the same function under two names; `Open77.routingBuckets.getEntity` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entityId` | `integer` | required | — |

```lua
-- GetEntityRoutingBucket is the low-level form of Open77.routingBuckets.getEntity (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Which instance is this vehicle in?
print("vehicle bucket", Open77.routingBuckets.getEntity(vehicleEntityId))
```

Registered in `LuaResourceRuntime.cs` (line 979) as `GetEntityRoutingBucket`.

_This signature was inferred from the handler body._

## GetGameTimer

```lua
GetGameTimer()
```

`SHARED`: Available without a live game instance.

Process-monotonic milliseconds, the clock the server scheduler runs on.

The same unit and meaning as Open77.time.monotonic, and not wall clock: it stays correct for durations, epochs and latency even when the host clock is adjusted. Open77.time.unix and Open77.time.utc answer wall clock when that is what you need.

Returns: `integer milliseconds since the process started`

```lua
-- Measure how long a delivery took
local startedAt = GetGameTimer()
RegisterNetEvent("delivery:done", function()
    local seconds = (GetGameTimer() - startedAt) / 1000
    Open77.chat.send(source, ("Delivered in %.1f s"):format(seconds))
end)
```

Registered in `LuaResourceRuntime.cs` (line 6281) as `LuaResourceRuntime.Bootstrap`.

## GetHashKey

```lua
GetHashKey(text)
```

`SHARED`: Available without a live game instance.

TweakDBID of a record name -- not a Jenkins hash.

CRC-32/ISO-HDLC of the name in the low 32 bits with the name length in byte 4, which is what REDengine computes and the only hash that resolves to anything in this game. It is deliberately NOT the Jenkins one-at-a-time hash FiveM's GetHashKey returns, so a hash constant copied out of a FiveM resource names a GTA model and matches nothing here: port the string, not the number. Non-string input answers nil, invalid_argument. The client computes the identical value, and results are memoised up to 1,024 names per resource.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `text` | `string` | required | — |

Returns: `integer TweakDBID, or nil, invalid_argument`

```lua
-- Compare a record name with a TweakDBID a client sent (never a FiveM/Jenkins hash)
if GetHashKey("Vehicle.v_sport2_quadra_turbo_r") == recordHash then print("a Quadra") end
```

Registered in `LuaResourceRuntime.Compat.cs` (line 222) as `LuaResourceRuntime.Bootstrap`.

## GetInvokingResource

```lua
GetInvokingResource()
```

`NETWORK`: Uses the network backend.

Immediate caller resource name inside an exported coroutine;

Immediate caller resource name inside an exported coroutine; nil outside it.

```lua
-- Only the dispatch resource may push a city alert through this one.
exports("alert", function(playerId, text)
    local caller = GetInvokingResource()
    if caller ~= "open77_dispatch" then
        return false, "denied for " .. tostring(caller or "a direct call")
    end
    Open77.chat.send(playerId, text)
    return true
end)
```

Registered in `LuaResourceRuntime.Exports.cs` (line 29) as `LuaResourceRuntime`.

## GetInvokingResourceGeneration

```lua
GetInvokingResourceGeneration()
```

`NETWORK`: Uses the network backend.

Immediate caller's VM generation inside an exported coroutine;

Immediate caller's VM generation inside an exported coroutine; nil outside it.

```lua
-- Remember a caller's handshake, but only for the VM that made it: when that
-- resource restarts its generation changes and the entry stops matching.
local greeted = {}
exports("hello", function()
    local caller, generation = GetInvokingResource(), GetInvokingResourceGeneration()
    if not caller then return false, "not called through exports" end
    local key = caller .. ":" .. tostring(generation)
    if greeted[key] then return true, "already greeted" end
    greeted[key] = true
    print(("first call from %s, generation %s"):format(caller, tostring(generation)))
    return true
end)
```

Registered in `LuaResourceRuntime.Exports.cs` (line 29) as `LuaResourceRuntime`.

## GetLootDrop

```lua
GetLootDrop(id)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.loot.get`.

The low-level form of `Open77.loot.get` (bound to the same native). The two are the same function under two names; `Open77.loot.get` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

```lua
-- GetLootDrop is the low-level form of Open77.loot.get (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Is the drop still there before paying out?
local drop = Open77.loot.get(lootId)
if not drop then print("already picked up") end
```

Registered in `LuaResourceRuntime.cs` (line 1055) as `GetLootDrop`.

_This signature was inferred from the handler body._

## GetLootDrops

```lua
GetLootDrops(bucket)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.loot.all`.

The low-level form of `Open77.loot.all` (bound to the same native). The two are the same function under two names; `Open77.loot.all` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | required | — |

```lua
-- GetLootDrops is the low-level form of Open77.loot.all (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Count the drops still lying in the arena
print(#Open77.loot.all(ARENA_BUCKET), "drops on the ground")
```

Registered in `LuaResourceRuntime.cs` (line 1056) as `GetLootDrops`.

_This signature was inferred from the handler body._

## GetNpc

```lua
GetNpc(id)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.npcs.get`.

The low-level form of `Open77.npcs.get` (bound to the same native). The two are the same function under two names; `Open77.npcs.get` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

```lua
-- GetNpc is the low-level form of Open77.npcs.get (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Is the boss still alive?
local boss = Open77.npcs.get(bossId)
if boss and boss.health <= 0 then
    Open77.chat.broadcast("The boss is down!")
end
```

Registered in `LuaResourceRuntime.cs` (line 1081) as `GetNpc`.

_This signature was inferred from the handler body._

## GetNpcAttitude

```lua
GetNpcAttitude(id)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.npcs.getAttitude`.

The low-level form of `Open77.npcs.getAttitude` (bound to the same native). The two are the same function under two names; `Open77.npcs.getAttitude` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

```lua
-- GetNpcAttitude is the low-level form of Open77.npcs.getAttitude (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Debug: print how the guard regards everyone
local attitude = Open77.npcs.getAttitude(guardId)
if attitude then print("default attitude:", attitude.default) end
```

Registered in `LuaResourceRuntime.cs` (line 1094) as `GetNpcAttitude`.

_This signature was inferred from the handler body._

## GetNpcCapacity

```lua
GetNpcCapacity()
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.npcs.capacity`.

The low-level form of `Open77.npcs.capacity` (bound to the same native). The two are the same function under two names; `Open77.npcs.capacity` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

```lua
-- GetNpcCapacity is the low-level form of Open77.npcs.capacity (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Check both allocation limits before admitting another NPC
local capacity, reason = Open77.npcs.capacity()
if capacity and capacity.resourceUsed < capacity.resourceLimit and capacity.globalUsed < capacity.globalLimit then
    admitNpc()
end
```

Registered in `LuaResourceRuntime.cs` (line 1083) as `GetNpcCapacity`.

_This signature was inferred from the handler body._

## GetNpcRelationship

```lua
GetNpcRelationship(first, second)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.npcs.getRelationship`.

The low-level form of `Open77.npcs.getRelationship` (bound to the same native). The two are the same function under two names; `Open77.npcs.getRelationship` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `first` | `string` | required | — |
| `second` | `string` | required | — |

```lua
-- GetNpcRelationship is the low-level form of Open77.npcs.getRelationship (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Check before declaring: are the two gangs already at war?
if Open77.npcs.getRelationship("maelstrom", "valentinos") ~= "hostile" then
    Open77.npcs.setRelationship("maelstrom", "valentinos", "hostile")
end
```

Registered in `LuaResourceRuntime.cs` (line 1097) as `GetNpcRelationship`.

_This signature was inferred from the handler body._

## GetNpcs

```lua
GetNpcs(bucket, [includeForeign])
```

`NETWORK`: Uses the network backend.

Lists the NPCs in one routing bucket, this resource's own unless asked otherwise.

The all-positional form of `Open77.npcs.all`. `bucket` may be `nil` for every bucket. By default the list holds only NPCs this resource created; pass `includeForeign` true to see every resource's, which needs the `npcs.foreign` permission and answers `nil, "permission_denied:npcs.foreign"` without it. Every row carries `owned`, so a shared list can still be filtered. Without `world.npcs` the answer is an empty table, not an error.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | required | — |
| `includeForeign` | `boolean` | optional | — |

```lua
-- /npcs: how many NPCs we own here, and how many everybody owns.
RegisterCommand("npcs", function()
    local mine = GetNpcs(0, false)
    local all, reason = GetNpcs(0, true)
    if not all then return print("need npcs.foreign to see the rest: " .. tostring(reason)) end
    print(("bucket 0: %d ours, %d in total"):format(#mine, #all))
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 1082) as `GetNpcs`.

## GetNpcTarget

```lua
GetNpcTarget(id)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.npcs.target`.

The low-level form of `Open77.npcs.target` (bound to the same native). The two are the same function under two names; `Open77.npcs.target` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

```lua
-- GetNpcTarget is the low-level form of Open77.npcs.target (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Is the guard fighting a player right now?
local target = Open77.npcs.target(guardId)
if target and target.kind == "player" then
    Open77.chat.send(target.id, "The guard is after you!")
end
```

Registered in `LuaResourceRuntime.cs` (line 1098) as `GetNpcTarget`.

_This signature was inferred from the handler body._

## GetNpcTask

```lua
GetNpcTask(id, taskId)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.npcs.tasks.get`.

The low-level form of `Open77.npcs.tasks.get` (bound to the same native). The two are the same function under two names; `Open77.npcs.tasks.get` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `taskId` | `integer` | required | — |

```lua
-- GetNpcTask is the low-level form of Open77.npcs.tasks.get (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Wait until the walk is finished
CreateThread(function()
    local task = Open77.npcs.tasks.moveTo(npcId, door)
    while Open77.npcs.tasks.get(npcId, task) do Wait(250) end
    print("the NPC has arrived (or gave up)")
end)
```

Registered in `LuaResourceRuntime.cs` (line 1087) as `GetNpcTask`.

_This signature was inferred from the handler body._

## GetNpcTasks

```lua
GetNpcTasks(npcId)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.npcs.tasks.all`.

The low-level form of `Open77.npcs.tasks.all` (bound to the same native). The two are the same function under two names; `Open77.npcs.tasks.all` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `npcId` | `integer` | required | — |

```lua
-- GetNpcTasks is the low-level form of Open77.npcs.tasks.all (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Debug: what is the NPC doing?
for _, task in ipairs(Open77.npcs.tasks.all(npcId)) do
    print(task.id, task.type, task.status)
end
```

Registered in `LuaResourceRuntime.cs` (line 1088) as `GetNpcTasks`.

_This signature was inferred from the handler body._

## GetNpcTemplates

```lua
GetNpcTemplates()
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.npcs.templates`.

The low-level form of `Open77.npcs.templates` (bound to the same native). The two are the same function under two names; `Open77.npcs.templates` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

```lua
-- GetNpcTemplates is the low-level form of Open77.npcs.templates (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Print the legacy alias list once at startup
for _, template in ipairs(Open77.npcs.templates()) do
    print(template.name, template.record)
end
```

Registered in `LuaResourceRuntime.cs` (line 1092) as `GetNpcTemplates`.

_This signature was inferred from the handler body._

## GetNumResources

```lua
GetNumResources()
```

`NETWORK`: Uses the network backend.

How many resources this server has discovered.

Counts `Open77.resource.list()`, so it counts every discovered resource, running or not.

Returns: `integer`

```lua
-- Print the resource count on startup
print(("%d resources discovered"):format(GetNumResources()))
```

Registered in `LuaResourceRuntime.RuntimeControl.cs` (line 372) as `LuaResourceRuntime.Bootstrap`.

## GetPlayerDistance

```lua
GetPlayerDistance(reason, reason)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.players.distance`.

The low-level form of `Open77.players.distance` (bound to the same native). The two are the same function under two names; `Open77.players.distance` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `reason` | `value` | required | — |
| `reason` | `value` | required | — |

```lua
-- GetPlayerDistance is the low-level form of Open77.players.distance (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Handcuffs only work at arm's length
RegisterCommand("cuff", function(source, args)
    local target = tonumber(args[1])
    if not target then return end
    local metres = Open77.players.distance(source, target)
    if not metres or metres > 2.0 then
        return Open77.chat.send(source, "Get closer.")
    end
    Open77.players.setFrozen(target, true)
end, false)
```

Registered in `LuaResourceRuntime.PlayerReads.cs` (line 70) as `GetPlayerDistance`.

_This signature was inferred from the handler body._

## GetPlayerEndpoint

```lua
GetPlayerEndpoint(playerId)
```

`NETWORK`: Uses the network backend.

FiveM-style alias for `Open77.players.endpoint`.

Requires `players.identity.sensitive`, like the namespaced spelling. FiveM gates nothing here; Open77 does, and a port that silently logged addresses will start reporting `permission_denied:players.identity.sensitive` until its manifest says what it is doing.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `"ip:port", or nil`, `reason`

```lua
-- FiveM port: the same value under the FiveM name (needs players.identity.sensitive)
local endpoint = GetPlayerEndpoint(source)
```

Registered in `LuaResourceRuntime.PlayerConnection.cs` (line 166) as `LuaResourceRuntime.Bootstrap`.

## GetPlayerHealth

```lua
GetPlayerHealth(playerId)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.players.getHealth`.

The low-level form of `Open77.players.getHealth` (bound to the same native). The two are the same function under two names; `Open77.players.getHealth` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

```lua
-- GetPlayerHealth is the low-level form of Open77.players.getHealth (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- /hp <id>: an admin reads a player's health
RegisterCommand("hp", function(source, args)
    local hp = Open77.players.getHealth(tonumber(args[1]) or source)
    if hp then Open77.chat.send(source, ("Health %d / %d"):format(hp.health, hp.maxHealth)) end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 1131) as `GetPlayerHealth`.

_This signature was inferred from the handler body._

## GetPlayerHoloCallEyes

```lua
GetPlayerHoloCallEyes()
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.players.getHoloCallEyes`.

The low-level form of `Open77.players.getHoloCallEyes` (wrapped by). The two are the same function under two names; `Open77.players.getHoloCallEyes` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

```lua
-- GetPlayerHoloCallEyes is the low-level form of Open77.players.getHoloCallEyes (wrapped by);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Is the player already on a call before starting a new one?
local enabled = Open77.players.getHoloCallEyes(playerId)
if not enabled then Open77.players.setHoloCallEyes(playerId, true, { durationMs = 30000 }) end
```

Registered in `LuaResourceRuntime.cs` (line 962) as `GetHoloCallEyes`.

_This signature was inferred from the handler body._

## GetPlayerIdentifier

```lua
GetPlayerIdentifier(playerId)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.players.identifier`.

The low-level form of `Open77.players.identifier` (bound to the same native). The two are the same function under two names; `Open77.players.identifier` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

```lua
-- GetPlayerIdentifier is the low-level form of Open77.players.identifier (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Key the player's money on the durable identifier, never on the session id
AddEventHandler("onPlayerReady", function(playerId)
    playerId = tonumber(playerId)
    if not playerId or playerId < 1 then return end            -- never pass 0: it stops the resource
    local key = "money:" .. Open77.players.identifier(playerId)
    local balance = Open77.kvp.get(key, 500)
    Open77.chat.send(playerId, ("Welcome back. Balance: %d eddies"):format(balance))
end)
```

Registered in `LuaResourceRuntime.cs` (line 993) as `GetPlayerIdentifier`.

_This signature was inferred from the handler body._

## GetPlayerIdentifierByType

```lua
GetPlayerIdentifierByType(playerId, identifierType)
```

`NETWORK`: Uses the network backend.

One identifier of a connected player, as a bare value.

Returns a bare string without a type prefix. Type names are case-insensitive: `open77` and `userId` return the installation identity GUID, `name` the display name, and `fingerprint` the SHA-256 of the identity public key. Since `2.31.13+op77.101`, `license` returns the permanent Open77 account GUID without dashes, `steam` the linked SteamID in lowercase hexadecimal, and `gog` the linked GOG ID in decimal. These ownership fields come from the verified Master ticket; when unavailable, the typed read returns `nil, "identifier_not_linked"`. Unsupported types return `nil, "unknown_identifier_type"`; non-string types return `nil, "invalid_identifier_type"`. Keep identifiers as strings and use `license` for account-level persistence across linked devices. See [account and store identifiers](../../01%20Documentation/connection-control.md#steam-gog-and-permanent-account-identifiers).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `identifierType` | `string` | required | — |

Returns: `identifier, or nil, reason`

```lua
-- A FiveM-style bans table keyed on the account id
local account = GetPlayerIdentifierByType(source, "open77")
if bans[account] then Open77.players.kick(source, "You are banned.") end
```

Registered in `LuaResourceRuntime.Compat.cs` (line 327) as `LuaResourceRuntime.Bootstrap`.

## GetPlayerIdentifiers

```lua
GetPlayerIdentifiers(playerId)
```

`NETWORK`: Uses the network backend.

Every identifier of a connected player, in FiveM's array-of-strings shape.

Returns an array of `type:value` strings: `open77:<identity GUID>`, `name:<display name>` and `fingerprint:sha256:<hash>`. Since `2.31.13+op77.101`, the verified Master ticket also supplies `license:<account GUID without dashes>`, `steam:<lowercase hexadecimal SteamID>` and `gog:<decimal GOG ID>` when available. Missing identifiers are omitted; an unknown session returns `nil, "player_not_found"`. Iterate with `for _, id in ipairs(GetPlayerIdentifiers(playerId) or {}) do ... end`. One qualifying Steam or GOG account is sufficient for admission, so do not require both store IDs. No Discord identifier is exposed. For persistence, use the permanent Open77 `license` string. See [account and store identifiers](../../01%20Documentation/connection-control.md#steam-gog-and-permanent-account-identifiers).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `array of strings, or nil, player_not_found`

```lua
-- A FiveM whitelist loop runs unchanged
for _, id in ipairs(GetPlayerIdentifiers(source)) do
    if id:sub(1, 7) == "open77:" then print("account", id:sub(8)) end
end
```

Registered in `LuaResourceRuntime.Compat.cs` (line 342) as `LuaResourceRuntime.Bootstrap`.

## GetPlayerIdentity

```lua
GetPlayerIdentity(playerId)
```

`NETWORK`: Uses the network backend.

`{ userId, name, publicKey, fingerprint, joinedAt }` for an admitted player, or `nil`.

Alias of `Open77.players.identity`, with no permission required. Returns `{ userId, name, publicKey, fingerprint, joinedAt }` for an admitted player, or `nil`. Since `2.31.13+op77.101`, also includes `license`, `steam` and `gog` when supplied by the verified Master ticket. `license` is the permanent Open77 account GUID without dashes; `steam` is a lowercase hexadecimal SteamID; `gog` is a decimal GOG ID. Values are strings without type prefixes and unavailable fields are omitted. `userId` remains the installation identity GUID; use `license` for account-level persistence across linked devices. `joinedAt` is ISO 8601 UTC. See [account and store identifiers](../../01%20Documentation/connection-control.md#steam-gog-and-permanent-account-identifiers).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |

```lua
-- GetPlayerIdentity is the low-level form of Open77.players.identity (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Look up the account behind a session
RegisterCommand("whois", function(source, args)
    local identity = Open77.players.identity(tonumber(args[1]))
    if not identity then return Open77.chat.send(source, "No such player.") end
    Open77.chat.send(source, ("%s, account %s, joined %s"):format(identity.name, identity.userId, identity.joinedAt))
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 1029) as `GetPlayerIdentity`.

## GetPlayerLastMsg

```lua
GetPlayerLastMsg(playerId)
```

`NETWORK`: Uses the network backend.

FiveM spelling of the age of the newest packet the server holds for a player, in milliseconds.

The `ageMs` the rich player read carries -- the same measurement FiveM's native makes -- so a ported AFK or lag check keeps its thresholds. `nil, no_packet_yet` before the first snapshot, `nil, reason` for an unknown player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `milliseconds since the last packet, or nil`, `reason on failure`

```lua
-- Drop a client whose packets stopped 30 s ago, the way an AFK script does
CreateThread(function()
    while true do
        Wait(5000)
        for _, id in ipairs(GetPlayers()) do
            local age = GetPlayerLastMsg(id)
            if age and age > 30000 then DropPlayer(id, 'connection timed out') end
        end
    end
end)
```

Registered in `LuaResourceRuntime.Identity6.cs` (line 200) as `LuaResourceRuntime.Bootstrap`.

## GetPlayerLifeState

```lua
GetPlayerLifeState(playerId)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.players.getLifeState`.

The low-level form of `Open77.players.getLifeState` (bound to the same native). The two are the same function under two names; `Open77.players.getLifeState` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

```lua
-- GetPlayerLifeState is the low-level form of Open77.players.getLifeState (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Count only real deaths: the join spawn and an admin move are scripted kills too
AddEventHandler("onPlayerLifeStateChanged", function(playerId, revision, phase, reason)
    if phase ~= "dead" then return end
    local life = Open77.players.getLifeState(playerId)
    if not life or life.cause == "script" then return end   -- freeroam:join_spawn, open77_admin:tp, ...
    local killer = life.killer ~= 0 and life.killer or nil   -- 0, never nil, when there is no killer
    recordDeath(playerId, killer, life.cause, life.weapon)
end)
```

Registered in `LuaResourceRuntime.cs` (line 1108) as `GetPlayerLifeState`.

_This signature was inferred from the handler body._

## GetPlayerLocale

```lua
GetPlayerLocale(playerId)
```

`NETWORK`: Uses the network backend.

FiveM-style alias for `Open77.players.locale`.

Same table, same four refusals. FiveM's own `GetPlayerLocale` is not a real native (it reads a client-set convar); here the value is the engine's own language setting, reported by the client host.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `{ code, gameLanguage, voiceLanguage, subtitles, chromium, ageMs, revision }, or nil`, `reason`

```lua
-- Route a ported script's localisation table by the two-letter language
local locale = GetPlayerLocale(source)
local lang = locale and locale.code:sub(1, 2) or "en"
print(Locales[lang] and Locales[lang].welcome or Locales.en.welcome)
```

Registered in `LuaResourceRuntime.Locale6.cs` (line 98) as `LuaResourceRuntime.Bootstrap`.

## GetPlayerModel

```lua
GetPlayerModel(playerId)
```

`NETWORK`: Uses the network backend.

Read the authoritative model selection and owner's readiness.

Requires players.model.read and a connected positive integer player ID. Returns nil without an error for the original model, or a table with player, record, appearance, revision, ready, resetOnDeath and optional remainingMs. Ready is the subject's presentation acknowledgement, not proof that every observer has streamed the body. Cosmetic NPC presentation only: the real player retains identity, controls, health, equipment and saved appearance. Character.* records resolve against each client's installed game data, not a virtual allowlist. See player-models.md for ownership, events, cleanup and the known special-rig/vehicle limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `table? state`, `string? reason`

```lua
local state, reason = GetPlayerModel(playerId)
if state then print(state.record, state.revision, state.ready) end
```

[Player morphs and model lifecycle guide](../../01%20Documentation/player-models.md)

Registered in `LuaResourceRuntime.cs` (line 964) as `GetPlayerModel`.

## GetPlayerName

```lua
GetPlayerName(playerId)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.players.name`.

The low-level form of `Open77.players.name` (bound to the same native). The two are the same function under two names; `Open77.players.name` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

```lua
-- GetPlayerName is the low-level form of Open77.players.name (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- /whois: works from the console only with an explicit id (a console command has source == 0)
RegisterCommand("whois", function(source, args)
    local target = tonumber(args[1]) or tonumber(source)
    if not target or target < 1 then print("whois: needs a player id"); return end
    print(Open77.players.name(target) or "unknown session")
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 992) as `GetPlayerName`.

_This signature was inferred from the handler body._

## GetPlayerObservers

```lua
GetPlayerObservers(subjectId)
```

`NETWORK`: Uses the network backend.

Return the viewers currently observing a player.

Returns a table of active player IDs using the same directed relation as isInScope, including the owner. Unknown IDs return an empty table. Non-positive IDs return false, invalid_player. No visibility is granted. Querying in native code avoids a Lua call for every player in the server; revalidate delayed publications after yielding.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `subjectId` | `integer` | required | — |

Returns: `table player IDs`, `false, reason on invalid input`

```lua
for _, id in ipairs(Open77.players.observers(subjectId)) do
    print(id)
end
```

Registered in `LuaResourceRuntime.cs` (line 999) as `GetPlayerObservers`.

## GetPlayerPing

```lua
GetPlayerPing(playerId)
```

`NETWORK`: Uses the network backend.

FiveM-style alias for `Open77.players.ping`.

Same value, same three refusals.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `milliseconds, or nil`, `reason`

```lua
-- Kick players whose ping is too high for the race
if (GetPlayerPing(source) or 0) > 250 then
    Open77.chat.send(source, "Your ping is too high to race.")
end
```

Registered in `LuaResourceRuntime.PlayerConnection.cs` (line 165) as `LuaResourceRuntime.Bootstrap`.

## GetPlayerPosition

```lua
GetPlayerPosition(playerId)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.players.position`.

The low-level form of `Open77.players.position` (bound to the same native). The two are the same function under two names; `Open77.players.position` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

```lua
-- GetPlayerPosition is the low-level form of Open77.players.position (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- /where: print where a player is standing
RegisterCommand("where", function(source, args)
    local pos = Open77.players.position(tonumber(args[1]) or source)
    if not pos then return Open77.chat.send(source, "Unknown position.") end
    Open77.chat.send(source, ("%.1f, %.1f, %.1f (bucket %d)"):format(pos.x, pos.y, pos.z, pos.bucket))
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 994) as `GetPlayerPosition`.

_This signature was inferred from the handler body._

## GetPlayerRead

```lua
GetPlayerRead(playerId)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.players.get`.

The low-level form of `Open77.players.get` (bound to the same native). The two are the same function under two names; `Open77.players.get` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

```lua
-- GetPlayerRead is the low-level form of Open77.players.get (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
local read = Open77.players.get(playerId)
if read and read.position and (read.ageMs or 0) < 5000 then
  -- your freshness threshold, not ours
end
```

Registered in `LuaResourceRuntime.PlayerReads.cs` (line 68) as `GetPlayerRead`.

_This signature was inferred from the handler body._

## GetPlayerRoutingBucket

```lua
GetPlayerRoutingBucket(playerId)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.routingBuckets.getPlayer`.

The low-level form of `Open77.routingBuckets.getPlayer` (bound to the same native). The two are the same function under two names; `Open77.routingBuckets.getPlayer` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

```lua
-- GetPlayerRoutingBucket is the low-level form of Open77.routingBuckets.getPlayer (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Only players in the same bucket can trade
if Open77.routingBuckets.getPlayer(a) ~= Open77.routingBuckets.getPlayer(b) then
    return Open77.chat.send(a, "You are not in the same instance.")
end
```

Registered in `LuaResourceRuntime.cs` (line 977) as `GetPlayerRoutingBucket`.

_This signature was inferred from the handler body._

## GetPlayers

```lua
GetPlayers()
```

`NETWORK`: Uses the network backend.

Array of every authenticated player id, ascending.

Array of every authenticated player id, ascending. The host's roster, so a gamemode no longer needs its own copy built from join/leave events.

```lua
-- GetPlayers is the low-level form of Open77.players.all (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Announce a restart to everyone currently connected
RegisterCommand("announce", function(source, args)
    local text = table.concat(args, " ")
    for _, playerId in ipairs(Open77.players.all()) do
        Open77.notifications.send(playerId, { type = "warning", message = text })
    end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 995) as `GetPlayers`.

## GetPlayersInBucket

```lua
GetPlayersInBucket(bucket)
```

`NETWORK`: Uses the network backend.

Ids of the authenticated players whose routing bucket is `bucket`, ascending.

Ids of the authenticated players whose routing bucket is `bucket`, ascending. `false, invalid_bucket` for a negative or oversized bucket.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `value` | required | — |

```lua
-- GetPlayersInBucket is the low-level form of Open77.players.inBucket (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Start the race only when the lobby instance has at least two players
local racers = Open77.players.inBucket(RACE_BUCKET)
if #racers >= 2 then
    startRace(racers)
end
```

Registered in `LuaResourceRuntime.cs` (line 996) as `GetPlayersInBucket`.

## GetPlayersInScope

```lua
GetPlayersInScope(viewerId)
```

`NETWORK`: Uses the network backend.

Return the players currently in a viewer's replication scope.

Returns a table of active player IDs using the same directed relation as isInScope, including the owner. Unknown IDs return an empty table. Non-positive IDs return false, invalid_player. No visibility is granted. Querying in native code avoids a Lua call for every player in the server; revalidate delayed publications after yielding.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `viewerId` | `integer` | required | — |

Returns: `table player IDs`, `false, reason on invalid input`

```lua
for _, id in ipairs(Open77.players.inScope(viewerId)) do
    print(id)
end
```

Registered in `LuaResourceRuntime.cs` (line 998) as `GetPlayersInScope`.

## GetPlayersNearby

```lua
GetPlayersNearby(reason, radius, options)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.players.nearby`.

The low-level form of `Open77.players.nearby` (bound to the same native). The two are the same function under two names; `Open77.players.nearby` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `reason` | `value` | required | — |
| `radius` | `number` | required | — |
| `options` | `value` | required | — |

```lua
-- GetPlayersNearby is the low-level form of Open77.players.nearby (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
for _, witness in ipairs(Open77.players.nearby(robber, 40)) do
  Open77.notifications.send(witness.playerId, { text = ("%.0f m away"):format(witness.distance) })
end
```

Registered in `LuaResourceRuntime.PlayerReads.cs` (line 69) as `GetPlayersNearby`.

_This signature was inferred from the handler body._

## GetPlayerSpectateTarget

```lua
GetPlayerSpectateTarget(playerId)
```

`NETWORK`: Uses the network backend.

Who this player is watching, or `0`.

Who this player is watching, or `0`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |

```lua
-- GetPlayerSpectateTarget is the low-level form of Open77.players.spectating (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- A spectator cannot be damaged by what they are watching
local watching = Open77.players.spectating(playerId)
if watching ~= 0 then return end
```

Registered in `LuaResourceRuntime.cs` (line 1128) as `GetPlayerSpectateTarget`.

## GetPlayerSpectators

```lua
GetPlayerSpectators(playerId)
```

`NETWORK`: Uses the network backend.

Everyone watching this player, ascending.

Everyone watching this player, ascending.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |

```lua
-- GetPlayerSpectators is the low-level form of Open77.players.spectators (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Tell a streamer how many admins are watching them
local count = #Open77.players.spectators(playerId)
if count > 0 then
    Open77.chat.send(playerId, ("%d admin(s) are spectating you."):format(count))
end
```

Registered in `LuaResourceRuntime.cs` (line 1129) as `GetPlayerSpectators`.

## GetPlayersPositions

```lua
GetPlayersPositions()
```

`NETWORK`: Uses the network backend.

`{ [playerId] = { x, y, z, bucket } }` for every player with a snapshot younger than two seconds.

`{ [playerId] = { x, y, z, bucket } }` for every player with a snapshot younger than two seconds. Players without one are absent, not `nil` entries. One call replaces a loop of `position` over the roster.

```lua
-- GetPlayersPositions is the low-level form of Open77.players.positions (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Count who is inside the event zone, in one pass
local inside = 0
for playerId, pos in pairs(Open77.players.positions()) do
    if #(vector3(pos.x, pos.y, pos.z) - zoneCenter) < zoneRadius then
        inside = inside + 1
    end
end
```

Registered in `LuaResourceRuntime.cs` (line 1000) as `GetPlayersPositions`.

## GetPlayerStats

```lua
GetPlayerStats(playerId)
```

`NETWORK`: Uses the network backend.

Combined canonical health/stamina snapshot or `nil`.

Combined canonical health/stamina snapshot or `nil`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |

```lua
-- GetPlayerStats is the low-level form of Open77.stats.get (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- /stats: both pools in one read
RegisterCommand("stats", function(source)
    local stats = Open77.stats.get(source)
    if stats then
        Open77.chat.send(source, ("HP %d / %d, stamina %d / %d"):format(
            stats.health.value, stats.health.maximum, stats.stamina.value, stats.stamina.maximum))
    end
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 1132) as `GetPlayerStats`.

## GetPlayerTimeOnline

```lua
GetPlayerTimeOnline(playerId)
```

`NETWORK`: Uses the network backend.

FiveM spelling of the live session length, in milliseconds.

`Open77.players.sessionStats(playerId).sessionSeconds * 1000` -- milliseconds because that is the unit FiveM documents, so a ported playtime counter divides the way it always did. Ungated. `nil, reason` for an unknown player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `milliseconds online, or nil`, `reason on failure`

```lua
-- A ported hourly reward loop keeps its arithmetic
local rewarded = {}
CreateThread(function()
    while true do
        Wait(60000)
        for _, id in ipairs(GetPlayers()) do
            local ms = GetPlayerTimeOnline(id)
            if ms and ms >= 3600000 and not rewarded[id] then rewarded[id] = true; giveReward(id) end
        end
    end
end)
```

Registered in `LuaResourceRuntime.Identity6.cs` (line 195) as `LuaResourceRuntime.Bootstrap`.

## GetPlayerVehicleSeat

```lua
GetPlayerVehicleSeat(playerId)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.vehicles.getPlayerSeat`.

The low-level form of `Open77.vehicles.getPlayerSeat` (bound to the same native). The two are the same function under two names; `Open77.vehicles.getPlayerSeat` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

```lua
-- GetPlayerVehicleSeat is the low-level form of Open77.vehicles.getPlayerSeat (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Only the driver may pay for fuel
RegisterCommand("refuel", function(source)
    local seat = Open77.vehicles.getPlayerSeat(source)
    if not seat then return Open77.chat.send(source, "You are not in a vehicle.") end
    if seat.seat ~= "seat_front_left" then
        return Open77.chat.send(source, "Only the driver can refuel.")
    end
    Open77.vehicles.repair(seat.vehicleId, "mechanical")
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 1075) as `GetPlayerVehicleSeat`.

_This signature was inferred from the handler body._

## GetPlayerWeapons

```lua
GetPlayerWeapons(playerId)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.weapons.get`.

The low-level form of `Open77.weapons.get` (bound to the same native). The two are the same function under two names; `Open77.weapons.get` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

```lua
-- GetPlayerWeapons is the low-level form of Open77.weapons.get (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- The shop refuses a sale to a player who already carries a Lexington (cached view)
local weapons = Open77.weapons.get(source)
if weapons and weapons.activeRecord == "Items.Preset_Lexington_Default" then
    return Open77.chat.send(source, "You already have one.")
end
```

Registered in `LuaResourceRuntime.Weapons.cs` (line 40) as `GetPlayerWeapons`.

_This signature was inferred from the handler body._

## GetProp

```lua
GetProp(id)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.props.get`.

The low-level form of `Open77.props.get` (bound to the same native). The two are the same function under two names; `Open77.props.get` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

```lua
-- GetProp is the low-level form of Open77.props.get (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Is the crate still there before paying for its pickup?
local crate = Open77.props.get(crateId)
if not crate then return Open77.chat.send(source, "Somebody already took it.") end
```

Registered in `LuaResourceRuntime.cs` (line 1040) as `GetProp`.

_This signature was inferred from the handler body._

## GetProps

```lua
GetProps(bucket)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.props.all`.

The low-level form of `Open77.props.all` (bound to the same native). The two are the same function under two names; `Open77.props.all` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | required | — |

```lua
-- GetProps is the low-level form of Open77.props.all (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Count the props in the event instance
print(#Open77.props.all(EVENT_BUCKET), "props in the event")
```

Registered in `LuaResourceRuntime.cs` (line 1041) as `GetProps`.

_This signature was inferred from the handler body._

## GetRecordData

```lua
GetRecordData()
```

`NETWORK`: Uses the network backend.

Reads one row of the shipped 2.31 record catalogue: a vehicle, weapon, item or NPC.

The single native behind `Open77.data.vehicle`, `.weapon`, `.item` and `.npc`, called as `GetRecordData(kind, record)` where `kind` is one of those four names. `record` is a TweakDB **string**, never a Jenkins hash, and is refused over 256 characters. The answer is a table of the catalogue's columns plus `kind`, `source = "catalogue"` and `build = "2.31"`; an empty cell is absent rather than an empty string, so `row.displayName or row.record` works. Answers `nil, reason` -- `invalid_kind`, `invalid_record`, `record_unknown`. Display names are not localised here: send `localeKey` to a client and localise it there.

```lua
-- Refuse to sell a vehicle the 2.31 catalogue does not know.
RegisterNetEvent("shop:buy", function(record)
    local row, reason = GetRecordData("vehicle", record)
    if not row then
        return Open77.chat.send(source, "Unknown vehicle: " .. tostring(reason))
    end
    print(("%s, %d seats, from the %s catalogue"):format(
        row.displayName or row.record, row.seats or 0, row.source))
end)
```

Registered in `LuaResourceRuntime.Data.cs` (line 56) as `GetRecordData`.

## GetRegisteredCommands

```lua
GetRegisteredCommands()
```

`NETWORK`: Uses the network backend.

FiveM-style alias for `Open77.runtime.commands`.

Entries carry `source` in addition to FiveM's `name` and `resource`, because this list has two origins.

Returns: `array of tables, or nil`, `reason`

```lua
-- /commands: list every command with the resource that owns it
RegisterCommand("commands", function(source)
    for _, entry in ipairs(GetRegisteredCommands()) do
        Open77.chat.send(source, ("/%s (%s)"):format(entry.name, entry.resource))
    end
end, true)
```

Registered in `LuaResourceRuntime.RuntimeControl.cs` (line 368) as `LuaResourceRuntime.Bootstrap`.

## GetResourceByFindIndex

```lua
GetResourceByFindIndex(index)
```

`NETWORK`: Uses the network backend.

The name of one discovered resource, by ZERO-based index.

The one zero-based function in this API, because it exists only so that a `for i = 0, GetNumResources() - 1` loop copied out of a FiveM resource runs unchanged. Prefer `Open77.resource.list()`, which is an ordinary 1-based Lua array carrying state and version as well.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `index` | `integer` | required | — |

Returns: `string, or nil`

```lua
-- A FiveM loop over every resource runs unchanged (zero-based!)
for i = 0, GetNumResources() - 1 do
    print(GetResourceByFindIndex(i))
end
```

Registered in `LuaResourceRuntime.RuntimeControl.cs` (line 379) as `LuaResourceRuntime.Bootstrap`.

## GetResourceKvp

```lua
GetResourceKvp(key, [default])
```

`NETWORK`: Uses the network backend.

FiveM-style alias for `Open77.kvp.get`.

See `SetResourceKvp`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string` | required | — |
| `default` | `any` | optional | — |

Returns: `value, default or nil`, `reason`

```lua
-- A FiveM port reads the saved value back
local outfit = GetResourceKvp("outfit:" .. Open77.players.identifier(source))
```

Registered in `LuaResourceRuntime.Kvp.cs` (line 316) as `LuaResourceRuntime.Bootstrap`.

## GetResourceMetadata

```lua
GetResourceMetadata(name, [key])
```

`NETWORK`: Uses the network backend.

FiveM-style alias for `Open77.resource.metadata`.

FiveM's third `index` argument is not accepted: an Open77 manifest directive is a list, returned whole, rather than a repeated key addressed by position.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `key` | `string` | optional | — |

Returns: `table or field value, or nil`, `reason`

```lua
-- Read this resource's version from its manifest
print("running version", GetResourceMetadata(GetCurrentResourceName(), "version"))
```

Registered in `LuaResourceRuntime.RuntimeControl.cs` (line 369) as `LuaResourceRuntime.Bootstrap`.

## GetResourceState

```lua
GetResourceState(resourceName)
```

`NETWORK`: Uses the network backend.

Return the current server resource state: `discovered`, `starting`, `running`, `stopping`, `stopped`, `failed`, `blocked`, or `missing` for a name the server does not know.

Return the current server resource state: `discovered`, `starting`, `running`, `stopping`, `stopped`, `failed`, `blocked`, or `missing` for a name the server does not know. There is no `started`: the running state is `running`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `resourceName` | `value` | required | — |

```lua
-- GetResourceState is the low-level form of Open77.resource.state (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Wait for a dependency to be running before using its export
CreateThread(function()
    while Open77.resource.state("bank") ~= "running" do Wait(500) end
    print("bank is up")
end)
```

Registered in `LuaResourceRuntime.cs` (line 976) as `GetResourceState`.

## GetScreenEffectCatalog

```lua
GetScreenEffectCatalog()
```

`NETWORK`: Uses the network backend.

The screen-effect aliases this server accepts, ascending.

The screen-effect aliases this server accepts, ascending.

```lua
-- GetScreenEffectCatalog is the low-level form of Open77.effects.screenCatalog (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Validate an item's screen effect at startup
local known = {}
for _, alias in ipairs(Open77.effects.screenCatalog()) do known[alias] = true end
assert(known["drunk"], "this server does not support the drunk effect")
```

Registered in `LuaResourceRuntime.cs` (line 1127) as `GetScreenEffectCatalog`.

## GetSoundState

```lua
GetSoundState(id)
```

`NETWORK`: Uses the network backend.

GetSoundState: server-authoritative network package audio.

Requires audio.network; network=false is rejected. Server owns the timeline, handles and bucket/optional-target audience. Clients cannot request broadcasts directly. getState reports intended playback, not decoder acknowledgement. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `table, or nil, reason`

```lua
-- Where is the server timeline right now?
local state = GetSoundState(concertId)
if state then print(state.status, state.position) end
```

Registered in `LuaResourceRuntime.cs` (line 9882) as `LuaAudioState`.

## GetUnixTime

```lua
GetUnixTime()
```

`NETWORK`: Uses the network backend.

wall-clock seconds since 1970 UTC, fractional

wall-clock seconds since 1970 UTC, fractional

```lua
-- GetUnixTime is the low-level form of Open77.time.unix (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- A ban that expires in 24 hours
bans[userId] = { until_ = Open77.time.unix() + 86400 }
```

Registered in `LuaResourceRuntime.cs` (line 1032) as `GetUnixTime`.

## GetUtcTimestamp

```lua
GetUtcTimestamp()
```

`NETWORK`: Uses the network backend.

the same instant as an ISO 8601 string

the same instant as an ISO 8601 string

```lua
-- GetUtcTimestamp is the low-level form of Open77.time.utc (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Timestamp a log line
print(Open77.time.utc(), "player", source, "bought", itemId)
```

Registered in `LuaResourceRuntime.cs` (line 1033) as `GetUtcTimestamp`.

## GetVehicle

```lua
GetVehicle(id)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.vehicles.get`.

The low-level form of `Open77.vehicles.get` (bound to the same native). The two are the same function under two names; `Open77.vehicles.get` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

```lua
-- GetVehicle is the low-level form of Open77.vehicles.get (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Speed camera: check every car's speed once a second
CreateThread(function()
    while true do
        Wait(1000)
        for _, entry in ipairs(Open77.vehicles.all()) do
            local car = Open77.vehicles.get(entry.id)
            if car and car.speedKph > 120 and car.driverPlayerId then
                Open77.chat.send(car.driverPlayerId, ("Slow down! %.0f km/h"):format(car.speedKph))
            end
        end
    end
end)
```

Registered in `LuaResourceRuntime.cs` (line 1065) as `GetVehicle`.

_This signature was inferred from the handler body._

## GetVehicleLivery

```lua
GetVehicleLivery(vehicle)
```

`NETWORK`: Uses the network backend.

Reads the FiveM index of a car's livery.

Requires `world.vehicles`: it reads the car through `Open77.vehicles.getVisual` and `listPaintProfiles`. Answers the 0-based position of the car's current profile in `Open77.vehicles.listPaintProfiles`, or `-1` for generic paint, for a profile the list no longer holds (its pack stopped), and on a failure (then with the reason as a second value).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicle` | `vehicle id` | required | — |

Returns: `index, or -1`, `reason on failure`

```lua
local livery = GetVehicleLivery(vehicle)
if livery >= 0 then print('livery ' .. livery) end
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.Compat.cs` (line 386) as `LuaResourceRuntime.Bootstrap`.

## GetVehicleMod

```lua
GetVehicleMod(vehicle, modType)
```

`NETWORK`: Uses the network backend.

Reads the FiveM index of a car's choice for one mod type.

Requires `world.vehicles`: it reads the car through `Open77.vehicles.listParts` and `getVisual`. Answers the index of the current choice in the same index space `SetVehicleMod` uses, or `-1` for stock and on a failure (then with the reason as a second value).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicle` | `vehicle id` | required | — |
| `modType` | `integer` | required | — |

Returns: `index, or -1`, `reason on failure`

```lua
local spoiler = GetVehicleMod(vehicle, 0)
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.Compat.cs` (line 506) as `LuaResourceRuntime.Bootstrap`.

## GetVehicles

```lua
GetVehicles(bucket)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.vehicles.all`.

The low-level form of `Open77.vehicles.all` (bound to the same native). The two are the same function under two names; `Open77.vehicles.all` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | required | — |

```lua
-- GetVehicles is the low-level form of Open77.vehicles.all (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- /cars: how many server vehicles exist, and how many in the caller's bucket
RegisterCommand("cars", function(source)
    local everywhere = Open77.vehicles.all()
    local here = Open77.vehicles.all(Open77.players.get(source).bucket)
    Open77.chat.send(source, ("%d vehicles on the server, %d in your instance"):format(#everywhere, #here))
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 1067) as `GetVehicles`.

_This signature was inferred from the handler body._

## GetVehicleWindowTint

```lua
GetVehicleWindowTint(vehicle)
```

`NETWORK`: Uses the network backend.

Reads the window tint as a FiveM index.

Requires `world.vehicles`: it reads the car through `Open77.vehicles.getWindowTint`. Answers `3`, `2` or `5` for Open77 levels 1 to 3, and `-1` for factory glass (FiveM's "no tint set") or on a failure (then with the reason as a second value). A car set with `1` (PURE_BLACK) therefore reads back `5`, the index of the same level.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicle` | `vehicle id` | required | — |

Returns: `FiveM tint index, or -1`, `reason on failure`

```lua
if GetVehicleWindowTint(vehicle) == 5 then print('limo tint') end
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.Compat.cs` (line 447) as `LuaResourceRuntime.Bootstrap`.

## GlobalState

```lua
GlobalState()
```

`SHARED`: Available without a live game instance.

The server-wide state bag, as a table.

The FiveM spelling of `Open77.state.global`, and the same bag: `GlobalState.hour = 21` is `Open77.state.global:set("hour", 21)`. Reading a key that was never set answers nil. Writing needs the `state.write` permission and raises on refusal, because an assignment has no second return value to carry a reason; use `Open77.state.global:set` when you want to handle `false, reason` yourself. Every player in every routing bucket receives a global bag, so it is the one selector with no audience filter.

Returns: `the global bag, indexable by key`

```lua
-- Replicate the round number to every client (needs state.write)
GlobalState.round = 3
print(GlobalState.round)
```

Registered in `LuaResourceRuntime.State.cs` (line 442) as `LuaResourceRuntime.Bootstrap`.

## GoToElevator

```lua
GoToElevator(id, floor, travelMilliseconds, [force])
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.elevators.goTo`.

The low-level form of `Open77.elevators.goTo` (wrapped by). The two are the same function under two names; `Open77.elevators.goTo` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `floor` | `integer` | required | — |
| `travelMilliseconds` | `integer` | required | — |
| `force` | `boolean` | optional | — |

```lua
-- GoToElevator is the low-level form of Open77.elevators.goTo (wrapped by);
-- the example shows the Open77.* spelling resources are told to prefer.
-- A slow freight lift: 15 seconds to the top floor
Open77.elevators.goTo(elevatorId, 11, { travelMs = 15000 })
```

Registered in `LuaResourceRuntime.cs` (line 1100) as `GoToElevator`.

_This signature was inferred from the handler body._

## HealPlayer

```lua
HealPlayer(playerId, amount)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.players.heal`.

The low-level form of `Open77.players.heal` (bound to the same native). The two are the same function under two names; `Open77.players.heal` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `amount` | `number` | required | — |

```lua
-- HealPlayer is the low-level form of Open77.players.heal (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- A medkit pickup heals 50 points
RegisterNetEvent("shop:useMedkit", function()
    Open77.players.heal(source, 50)
end)
```

Registered in `LuaResourceRuntime.cs` (line 1134) as `HealPlayer`.

_This signature was inferred from the handler body._

## IsDuplicityVersion

```lua
IsDuplicityVersion()
```

`SHARED`: Available without a live game instance.

True in this VM; the client's answer is false.

The one call that lets a single shared_script branch between the two runtimes, and the only reason a shared file needs to know which side it is on. It takes no arguments and cannot be spoofed: the two runtimes install two different functions.

Returns: `true`

```lua
if IsDuplicityVersion() then
    RegisterCommand("give", handler, true)
else
    RegisterNetEvent("shop:given", render)
end
```

Registered in `LuaResourceRuntime.Compat.cs` (line 48) as `LuaResourceRuntime.Bootstrap`.

## IsPlayerAceAllowed

```lua
IsPlayerAceAllowed(playerId, permission)
```

`NETWORK`: Uses the network backend.

Whether a connected player holds a permission; the same function as Open77.acl.isAllowed.

Requires acl.read. The FiveM spelling, addressed by session player id, with the same answer and the same permission as the namespaced call. Use IsPrincipalAceAllowed to ask by account id instead.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `permission` | `string` | required | — |

Returns: `boolean, or false, reason`

```lua
-- A FiveM-style permission check in a command
RegisterCommand("weather", function(source, args)
    if not IsPlayerAceAllowed(source, "weather.control") then
        return Open77.chat.send(source, "You may not change the weather.")
    end
    Open77.environment.setWeather(args[1] or "sunny")
end, false)
```

Registered in `LuaResourceRuntime.Compat.cs` (line 305) as `LuaResourceRuntime.Bootstrap`.

## IsPlayerDead

```lua
IsPlayerDead(playerId)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.players.isDead`.

The low-level form of `Open77.players.isDead` (bound to the same native). The two are the same function under two names; `Open77.players.isDead` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

```lua
-- IsPlayerDead is the low-level form of Open77.players.isDead (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- A dead player cannot use the shop
RegisterCommand("shop", function(source)
    if Open77.players.isDead(source) then
        return Open77.chat.send(source, "You are in no state to shop.")
    end
    openShop(source)
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 1109) as `IsPlayerDead`.

_This signature was inferred from the handler body._

## IsPlayerFrozen

```lua
IsPlayerFrozen(playerId)
```

`NETWORK`: Uses the network backend.

Whether the canonical life state is currently frozen, by any resource.

Whether the canonical life state is currently frozen, by any resource.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |

```lua
-- IsPlayerFrozen is the low-level form of Open77.players.isFrozen (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Refuse a second /cuff on a player somebody already holds
if Open77.players.isFrozen(target) then
    return Open77.chat.send(source, "They are already restrained.")
end
```

Registered in `LuaResourceRuntime.cs` (line 1119) as `IsPlayerFrozen`.

## IsPlayerGhosted

```lua
IsPlayerGhosted(playerId)
```

`NETWORK`: Uses the network backend.

Whether the canonical life state is currently ghosted.

Whether the canonical life state is currently ghosted.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |

```lua
-- IsPlayerGhosted is the low-level form of Open77.players.isGhosted (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Spawn protection is still on: skip the damage report
if Open77.players.isGhosted(victimId) then return end
```

Registered in `LuaResourceRuntime.cs` (line 1117) as `IsPlayerGhosted`.

## IsPlayerInScope

```lua
IsPlayerInScope(viewerId, subjectId)
```

`NETWORK`: Uses the network backend.

Check whether a player is in another player's native replication scope.

Read-only directed visibility relation used by player roster and motion replication. Returns true for an active player viewing itself. Distinct players must be active, in the same routing bucket, and in the current area-of-interest graph. Unknown or disconnected players return false; invalid non-positive IDs return false, invalid_player. With deferred bootstrap, new peers remain outside scope until placement permits reconciliation. This does not grant interest or measure render visibility. Use it to filter presentation events and replays so they follow the native roster.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `viewerId` | `integer` | required | — |
| `subjectId` | `integer` | required | — |

Returns: `boolean visible`, `string reason on invalid input`

```lua
if Open77.players.isInScope(viewer, subject) then
    TriggerClientEvent("my:appearance", viewer, subject, appearance)
end
```

Registered in `LuaResourceRuntime.cs` (line 997) as `IsPlayerInScope`.

## IsPlayerModelReady

```lua
IsPlayerModelReady(playerId)
```

`NETWORK`: Uses the network backend.

Read the model owner's native presentation acknowledgement.

Requires players.model.read and a connected positive integer player ID. Returns false for the original character or a model still preparing; nil, reason is an error. Observe onPlayerModelReady(player, revision) for asynchronous readiness and match the requested revision. This does not certify observer rendering or every animation. Cosmetic NPC presentation only: the real player retains identity, controls, health, equipment and saved appearance. Character.* records resolve against each client's installed game data, not a virtual allowlist. See player-models.md for ownership, events, cleanup and the known special-rig/vehicle limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `boolean? ready`, `string? reason`

```lua
if IsPlayerModelReady(playerId) then print('Owner model ready') end
```

[Player morphs and model lifecycle guide](../../01%20Documentation/player-models.md)

Registered in `LuaResourceRuntime.cs` (line 7869) as `LuaResourceRuntime.Bootstrap`.

## IsPlayerVisible

```lua
IsPlayerVisible(playerId)
```

`NETWORK`: Uses the network backend.

Whether the canonical life state says this player is rendered.

Whether the canonical life state says this player is rendered. Answered from the server bit, never probed from a client.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |

```lua
-- IsPlayerVisible is the low-level form of Open77.players.isVisible (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- A hidden admin is skipped by the scoreboard
for _, playerId in ipairs(Open77.players.all()) do
    if Open77.players.isVisible(playerId) then
        table.insert(board, Open77.players.name(playerId))
    end
end
```

Registered in `LuaResourceRuntime.cs` (line 1121) as `IsPlayerVisible`.

## IsPrincipalAceAllowed

```lua
IsPrincipalAceAllowed(userId, permission)
```

`NETWORK`: Uses the network backend.

The same ACL check, addressed by account id instead of session id.

Requires acl.read. FiveM principals are durable; Open77 resolves permissions through the connected session, so a user who is not connected cannot be answered at all -- that is false, principal_not_connected rather than a bare false, which would read as "denied". The account id is matched against the connected players' identifiers case-insensitively.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `userId` | `string` | required | — |
| `permission` | `string` | required | — |

Returns: `boolean, or false, reason`

```lua
-- Check a permission by account id (the account must be connected)
local allowed, reason = IsPrincipalAceAllowed(userId, "shop.manage")
if not allowed then print("denied:", reason) end
```

Registered in `LuaResourceRuntime.Compat.cs` (line 312) as `LuaResourceRuntime.Bootstrap`.

## IsVehicleVisibleTo

```lua
IsVehicleVisibleTo(vehicleId, playerId)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.vehicles.isVisibleTo`.

The low-level form of `Open77.vehicles.isVisibleTo` (bound to the same native). The two are the same function under two names; `Open77.vehicles.isVisibleTo` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |
| `playerId` | `integer` | required | — |

```lua
-- IsVehicleVisibleTo is the low-level form of Open77.vehicles.isVisibleTo (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Keep an observed chassis out of an off-screen cleanup pass
local visible, reason = Open77.vehicles.isVisibleTo(vehicleId, playerId)
if reason then error(reason) end
if visible then return end
```

Registered in `LuaResourceRuntime.cs` (line 1066) as `IsVehicleVisibleTo`.

_This signature was inferred from the handler body._

## joaat

```lua
joaat(text)
```

`SHARED`: Available without a live game instance.

Alias of GetHashKey; returns a TweakDBID.

The same function value under the name ported code uses. Kept so that code reads naturally, but it is not the Jenkins hash the name promises -- see GetHashKey before copying any hash constant across.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `text` | `string` | required | — |

Returns: `integer TweakDBID, or nil, invalid_argument`

```lua
-- FiveM habit, same value as GetHashKey: a TweakDBID, not a Jenkins hash
local id = joaat("Vehicle.v_sport2_quadra_turbo_r")
```

Registered in `LuaResourceRuntime.Compat.cs` (line 241) as `LuaResourceRuntime.Bootstrap`.

## KillNpc

```lua
KillNpc(id, reason)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.npcs.kill`.

The low-level form of `Open77.npcs.kill` (bound to the same native). The two are the same function under two names; `Open77.npcs.kill` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `reason` | `string` | required | — |

```lua
-- KillNpc is the low-level form of Open77.npcs.kill (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- /killnpc <id>
RegisterCommand("killnpc", function(source, args)
    local npcId = tonumber(args[1])
    if npcId then Open77.npcs.kill(npcId) end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 1089) as `KillNpc`.

_This signature was inferred from the handler body._

## KillPlayer

```lua
KillPlayer(playerId, killer, causeName, weapon, [impulseX], [impulseY], [impulseZ])
```

`NETWORK`: Uses the network backend.

Kills a player where they stand and answers whether the life ledger accepted it.

The all-positional form of `Open77.players.kill`. `cause` must be one of `firearm`, `melee`, `explosion`, `fall`, `vehicle`, `environment`, `script`, `unknown` -- anything else is `invalid_argument`, so a label like `"admin"` does not work here. `killer` may be `nil` or `0` for no killer, `weapon` defaults to `resource:<your resource>`, and the optional impulse is capped at 1000 per axis. Answers `false, reason` (`player_not_found`, `invalid_argument`, `permission_denied:players.life.kill`). Requires `players.life.kill`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `killer` | `integer` | required | — |
| `causeName` | `string` | required | — |
| `weapon` | `string` | required | — |
| `impulseX` | `number` | optional | — |
| `impulseY` | `number` | optional | — |
| `impulseZ` | `number` | optional | — |

```lua
-- /slay <id>: an admin kills a player, blaming the script.
RegisterCommand("slay", function(source, args)
    local target = tonumber(args[1])
    if not target then return end
    local ok, reason = KillPlayer(target, source, "script", "admin command")
    if not ok then print(("slay refused: %s"):format(tostring(reason))) end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 1110) as `KillPlayer`.

## LoadResourceFile

```lua
LoadResourceFile(resourceName, path)
```

`SHARED`: Available without a live game instance.

Reads a file belonging to the calling resource.

The FiveM spelling of Open77.io.read, and no more powerful: on the server the path resolves under this resource's own data/ directory and requires the filesystem.read permission. Naming another resource answers nil, cross_resource_read_denied rather than quietly reading your own file -- cross-resource file access does not exist here, and pretending otherwise would make ported code look like it works. The client has the same global rooted at the resource directory instead, with no permission and a 1 MiB cap.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `resourceName` | `string` | required | — |
| `path` | `string` | required | — |

Returns: `file contents, or nil, reason`

```lua
-- Load a JSON price list shipped in the resource's data/ directory
local text = LoadResourceFile(GetCurrentResourceName(), "prices.json")
local prices = text and json.decode(text) or {}
```

Registered in `LuaResourceRuntime.Compat.cs` (line 197) as `LuaResourceRuntime.Bootstrap`.

## PauseElevator

```lua
PauseElevator(id)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.elevators.pause`.

The low-level form of `Open77.elevators.pause` (bound to the same native). The two are the same function under two names; `Open77.elevators.pause` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

```lua
-- PauseElevator is the low-level form of Open77.elevators.pause (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Power cut: every lift stops where it is
for _, lift in ipairs(Open77.elevators.all()) do Open77.elevators.pause(lift.id) end
```

Registered in `LuaResourceRuntime.cs` (line 1102) as `PauseElevator`.

_This signature was inferred from the handler body._

## PauseSound

```lua
PauseSound(id)
```

`NETWORK`: Uses the network backend.

PauseSound: server-authoritative network package audio.

Requires audio.network; network=false is rejected. Server owns the timeline, handles and bucket/optional-target audience. Clients cannot request broadcasts directly. getState reports intended playback, not decoder acknowledgement. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `true, or failure, reason`

```lua
-- Intermission: pause the concert stream for everyone
PauseSound(concertId)
```

Registered in `LuaResourceRuntime.cs` (line 9876) as `LuaAudioControl`.

## PerformHttpRequest

```lua
PerformHttpRequest(url, [callback], [method], [body], [headers])
```

`NETWORK`: Uses the network backend.

Performs an outbound HTTP request, off the tick thread.

Requires http.request, and the server must have enabled the bridge with a host allow-list. The request leaves the tick thread at once and the callback runs on this resource's own scheduler with (status, body, headers, err); status is 0 when the exchange did not complete and err then names the reason. method defaults to GET; body may be a string or a table, which is JSON-encoded for you, up to 256 KiB. A request refused before it leaves answers false, reason and still calls the callback with status 0. Namespaced as Open77.http.request.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `url` | `string` | required | — |
| `callback` | `function` | optional | — |
| `method` | `string` | optional | — |
| `body` | `string|table` | optional | — |
| `headers` | `table` | optional | — |

Returns: `true, or false, reason`

```lua
-- Post a Discord webhook when a player is banned
PerformHttpRequest("https://discord.com/api/webhooks/...", function(status, body)
    if status ~= 204 then print("webhook failed", status) end
end, "POST", { content = "Player banned: " .. name }, { ["Content-Type"] = "application/json" })
```

Registered in `LuaResourceRuntime.cs` (line 6543) as `LuaResourceRuntime.Bootstrap`.

## Play2DSound

```lua
Play2DSound(asset, [options])
```

`NETWORK`: Uses the network backend.

Play2DSound: server-authoritative network package audio.

Requires audio.network; network=false is rejected. Server owns the timeline, handles and bucket/optional-target audience. Clients cannot request broadcasts directly. getState reports intended playback, not decoder acknowledgement. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `asset` | `string` | required | — |
| `options` | `table` | optional | — |

Returns: `integer handle, or nil, reason`

```lua
-- A server-owned announcement every player in bucket 0 hears, in sync
local id = Play2DSound("audio/curfew.wav", { bucket = 0, volume = 0.5, duration = 12 })
```

Registered in `LuaResourceRuntime.cs` (line 9869) as `LuaAudioCreate`.

## Play3DSound

```lua
Play3DSound(asset, position, [options])
```

`NETWORK`: Uses the network backend.

Play3DSound: server-authoritative network package audio.

Requires audio.network; network=false is rejected. Server owns the timeline, handles and bucket/optional-target audience. Clients cannot request broadcasts directly. getState reports intended playback, not decoder acknowledgement. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `asset` | `string` | required | — |
| `position` | `table` | required | — |
| `options` | `table` | optional | — |

Returns: `integer handle, or nil, reason`

```lua
-- A looping street concert everyone in range hears at the same point of the track
local concertId = Play3DSound("audio/concert.mp3", { x = -1590.0, y = 390.0, z = 25.0 }, {
    loop = true, duration = 240, maxDistance = 60, bucket = 0,
})
```

Registered in `LuaResourceRuntime.cs` (line 9872) as `LuaAudioCreate`.

## PlayEffect

```lua
PlayEffect(name, x, y, z, qx, qy, qz, qw, bucket, range, [sound])
```

`NETWORK`: Uses the network backend.

Plays a one-shot particle effect at a world point for everyone in range.

The all-positional form of `Open77.effects.play`. Unlike `CreateEffect` nothing is registered: the burst plays once on every client within `range` metres of the point and is gone, so there is no id to remove. The orientation is a quaternion. An optional `sound` event plays with it. Answers `true`, or `nil, reason` on `invalid_position` or an unknown effect. Requires `world.effects`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `z` | `number` | required | — |
| `qx` | `number` | required | — |
| `qy` | `number` | required | — |
| `qz` | `number` | required | — |
| `qw` | `number` | required | — |
| `bucket` | `integer` | required | — |
| `range` | `number` | required | — |
| `sound` | `string` | optional | — |

```lua
-- Sparks and a clunk where a lockpick failed.
RegisterNetEvent("lockpick:failed", function()
    local me = Open77.players.position(source)
    if not me then return end
    local ok, reason = PlayEffect(
        "sparks.burst.small", me.x, me.y, me.z,   -- name, x, y, z
        0.0, 0.0, 0.0, 1.0,                       -- quaternion: no rotation
        0, 50.0,                                  -- bucket, range in metres
        nil)                                      -- optional sound event
    if not ok then print("sparks refused: " .. tostring(reason)) end
end)
```

Registered in `LuaResourceRuntime.cs` (line 1043) as `PlayEffect`.

## PlayEntityEffect

```lua
PlayEntityEffect(target, name, duration, [loop], [slot], [targetKind])
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.effects.playOn`.

The low-level form of `Open77.effects.playOn` (wrapped by). The two are the same function under two names; `Open77.effects.playOn` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `integer` | required | — |
| `name` | `string` | required | — |
| `duration` | `number` | required | — |
| `loop` | `boolean` | optional | — |
| `slot` | `string` | optional | — |
| `targetKind` | `string` | optional | — |

```lua
-- PlayEntityEffect is the low-level form of Open77.effects.playOn (wrapped by);
-- the example shows the Open77.* spelling resources are told to prefer.
local ok, err = Open77.effects.playOn({kind="player",id=tostring(playerId)}, nativeEvent, {duration=1})
if not ok then print(err) end
```

Registered in `LuaResourceRuntime.cs` (line 1049) as `PlayEntityEffect`.

_This signature was inferred from the handler body._

## PlayEntitySound

```lua
PlayEntitySound(target, soundEvent, [unique], [targetKind], [duration], [actionId], excluded)
```

`NETWORK`: Uses the network backend.

Plays a sound event on an entity, so it follows that entity instead of a fixed point.

The all-positional form of `Open77.effects.sound`. `targetKind` says what the id names and defaults to `"player"`; `unique` (third argument) stops the same `actionId` playing twice over itself; `duration` defaults to 5 seconds. The seventh argument is a table of player ids who will not hear it. Answers `true`, or `nil, reason` -- `invalid_sound_exclusions` when that table is malformed. Requires `world.effects`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `integer` | required | — |
| `soundEvent` | `string` | required | — |
| `unique` | `boolean` | optional | — |
| `targetKind` | `string` | optional | — |
| `duration` | `number` | optional | — |
| `actionId` | `string` | optional | — |
| `excluded` | `value` | required | — |

```lua
-- The ripperdoc's drill, heard by everyone near the patient except the
-- patient, who gets their own first-person cue from the client side.
local ok, reason = PlayEntitySound(
    patientId, "ono_v_effort_short",   -- target, sound event
    true, "player",                    -- unique, targetKind
    4.0, "ripper.drill",               -- duration seconds, actionId
    { patientId })                     -- players who do not hear it
if not ok then print("sound refused: " .. tostring(reason)) end
```

Registered in `LuaResourceRuntime.cs` (line 1051) as `PlayEntitySound`.

## Player

```lua
Player(playerId)
```

`SHARED`: Available without a live game instance.

Addresses one player's state bag, FiveM style.

`Player(id).state` is `Open77.state.player(id)`, and `Player(id).source` answers the id back, so a ported script that passes a player object around keeps working. An unknown id still returns a handle: the bag is created on first write and dies when the player disconnects. The bag reaches every player who can see that player, which is the roster of their routing bucket, so it is not a private channel.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `a table with `state` and `source``

```lua
-- FiveM-style write of a player's state bag
Player(source).state.cuffed = true
```

Registered in `LuaResourceRuntime.State.cs` (line 444) as `LuaResourceRuntime.Bootstrap`.

## playerConnecting

```lua
playerConnecting(name, setKickReason, deferrals)
```

`SERVER`: Available in server resources only.

The same gate as `onPlayerConnecting`, under the FiveM name and with the FiveM argument order, so a ported whitelist runs unchanged. It votes in the same tally; a resource may register either name or both. `deferrals` is the same table. `source` is **not** set: no player id exists yet, and the connection token is not one. `setKickReason(message)` records a message and refuses nobody — the same as in FiveM, where the refusal comes from `CancelEvent`. Open77 has `CancelEvent`, but it only means something inside a `TriggerCancellableEvent` dispatch, and the connection gate is not one: called here it answers `false, "not_in_cancellable_event"`. **Refuse with deferrals.done(message).** The runtime says this once per resource in the log the first time `setKickReason` is called.

Requires `players.gate`. Who gets in, decided in Lua. The guide, with a whitelist and a ban list to copy, is
Connection control.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `not specified` | required | — |
| `setKickReason` | `not specified` | required | — |
| `deferrals` | `not specified` | required | — |

```lua
-- A FiveM-style whitelist runs unchanged; refuse with deferrals.done(message)
AddEventHandler("playerConnecting", function(name, setKickReason, deferrals)
    deferrals.defer()
    deferrals.update("Checking the whitelist...")
    if not whitelisted[name] then
        return deferrals.done("You are not on the whitelist.")
    end
    deferrals.done()
end)
```

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-control) (line 759).

## playerDropped

```lua
playerDropped(reason)
```

`SERVER`: Available in server resources only.

FiveM's leave event, emitted for every disconnection beside `onPlayerDisconnected`. `source` is the player who left; `reason` is the same text `onPlayerDisconnected` receives. Both arrive in the same tick and either is a complete account of the leave — do not sequence work across the two.

Who gets in, decided in Lua. The guide, with a whitelist and a ban list to copy, is
Connection control.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `reason` | `not specified` | required | — |

```lua
-- Save the player's money when they leave; source is the player who left
AddEventHandler("playerDropped", function(reason)
    local key = "money:" .. (Open77.players.identifier(source) or "")
    Open77.kvp.set(key, wallet[source] or 0)
    wallet[source] = nil
end)
```

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-control) (line 761).

## playerJoining

```lua
playerJoining(oldId)
```

`SERVER`: Available in server resources only.

FiveM's join event, emitted when a player is assigned a session, and seen by a resource before `onPlayerConnected`. `source` is the new player id. `oldId` is always the empty string: FiveM puts the temporary connection id there, and an Open77 player is given one id and keeps it.

Who gets in, decided in Lua. The guide, with a whitelist and a ban list to copy, is
Connection control.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `oldId` | `not specified` | required | — |

```lua
-- FiveM's join event: source is the new player id, oldId is always empty here
AddEventHandler("playerJoining", function(oldId)
    print("player", source, "was assigned a session")
end)
```

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-control) (line 760).

## PlayerScreenEffect

```lua
PlayerScreenEffect(playerId, name, opts?)
```

`NETWORK`: Uses the network backend.

`boolean, reason?` — a full-screen post-process on ONE player.

`boolean, reason?` — a full-screen post-process on ONE player. Not replicated; permission `players.screenfx`. `name` false clears. See [Screen effects](https://open2077.net/docs/api/server/effects.md#screen-effects).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |
| `name` | `value` | required | — |
| `opts?` | `value` | required | — |

```lua
-- PlayerScreenEffect is the low-level form of Open77.effects.screen (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Drinking makes the screen wobble for 30 seconds
RegisterNetEvent("items:drink", function()
    Open77.effects.screen(source, "drunk", { strength = 0.5, duration = 30 })
end)
```

Registered in `LuaResourceRuntime.cs` (line 1126) as `PlayerScreenEffect`.

## PlaySound

```lua
PlaySound(id)
```

`NETWORK`: Uses the network backend.

PlaySound: server-authoritative network package audio.

Requires audio.network; network=false is rejected. Server owns the timeline, handles and bucket/optional-target audience. Clients cannot request broadcasts directly. getState reports intended playback, not decoder acknowledgement. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `true, or failure, reason`

```lua
-- Resume a network sound the server paused (its timeline is server-owned)
PlaySound(concertId)
```

Registered in `LuaResourceRuntime.cs` (line 9875) as `LuaAudioControl`.

## print

```lua
print(...)
```

`SHARED`: Available without a live game instance.

Write a resource-prefixed line to the server log.

The FiveM spelling of `Open77.log.info`: every argument is converted with the usual `tostring` rules, joined with a tab, and written to the host logger at `INF` under this resource's name -- there is no console of its own to write to. Control sequences (ANSI colours, cursor moves) are stripped before the line reaches the log, so a ported script's coloured output cannot corrupt the terminal or the log file. Returns nothing and never fails; no permission. `Open77.log.debug`, `warn` and `error` are the same call at the other levels.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `...` | `values` | required | — |

```lua
RegisterCommand("whoami", function(source)
    print("whoami from", source, GetPlayerName(source))
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 955) as `Print`.

## quat

```lua
quat(x, y, z, w)
```

`SHARED`: Available without a live game instance.

Builds a quaternion: a plain { x, y, z, w } table with three rotation methods.

Available in both runtimes. Deliberately minimal -- normalize (a zero quaternion becomes the identity, never NaN), mul (Hamilton product, this rotation then the other) and rotate (returns a vector3), plus unpack and table. No arithmetic operators and no #. It exists so the { x =, y =, z =, w = } orientation that Open77.effects and Open77.props already accept becomes computable; the result stays exactly that table. Identical on both runtimes by construction: one file, scripting/lua/open77_vector.lua, is embedded by the client build and by the server assembly, so the semantics cannot drift.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `z` | `number` | required | — |
| `w` | `number` | required | — |

Returns: `quat, or nil, reason`

```lua
-- Rotate a spawn offset by a 90-degree yaw
local yaw90 = quat(0, 0, 0.7071, 0.7071)
local offset = yaw90:rotate(vector3(2.0, 0.0, 0.0)) -- now points along +y
```

Registered in `open77_vector.lua` (line 498) as `LuaPrelude`.

## RegisterCommand

```lua
RegisterCommand(name, handler, [restricted])
```

`SHARED`: Available without a live game instance.

Registers a command usable from the server console and from chat.

No permission to register. The handler receives `(source, args, raw)`: `source` is a number -- the player id, or `0` for the server console and Warden; `args` is `table.pack` of the arguments after the command name, so `args.n` is their count and every one is a **string** (`"42"`, not `42`); `raw` is the command name followed by those arguments joined with single spaces, without the leading `/`. From chat the client splits the line shell-style (quotes group words, backslash escapes, at most 32 arguments) before sending it; from the console the line is split on spaces only, so quotes arrive literally. A chat `/name` and a console `name` reach the same handler, which runs on the next scheduler tick, not inline. Names are 1 to 64 characters of letters, digits, `_`, `.`, `:` or `-`, matched case-insensitively; registering one twice raises `duplicate command`. With `restricted = true` a player must hold the ACL entry `command.<name>` (lower-case) -- the console always may -- see Identity and ACL.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `handler` | `function` | required | — |
| `restricted` | `boolean` | optional | — |

Returns: `true`

```lua
-- /pay <playerId> <amount>: every argument is a string, convert what must be a number
RegisterCommand("pay", function(source, args, raw)
    local target, amount = tonumber(args[1]), tonumber(args[2])
    if source == 0 then return print("pay: run this from the game, not the console") end
    if not target or not amount then
        return Open77.chat.send(source, "Usage: /pay <playerId> <amount>")
    end
    transfer(source, target, amount)
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 6325) as `LuaResourceRuntime.Bootstrap`.

## RegisterNetEvent

```lua
RegisterNetEvent(event, [handler])
```

`NETWORK`: Uses the network backend.

Registers a handler for an authenticated client event; the same function as Open77.net.on.

Requires network.events. During the handler the global `source` is set from the authenticated connection and never from the client's payload -- that is the whole difference from AddEventHandler. Called with the name alone it only marks the name as reachable from a client and registers nothing. Such a handler also receives host-wide bus events, so one that acts on authority must validate `source` rather than assume it. Events carry at most 32 arguments in a 48 KiB envelope.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `handler` | `function` | optional | — |

Returns: `handler id, or true when only the name is declared`

```lua
-- The client asks to buy an item; `source` is the authenticated buyer
RegisterNetEvent("shop:buy", function(itemId)
    local price = prices[itemId]
    if not price then return end
    if takeMoney(source, price) then
        Open77.chat.send(source, "Purchased " .. itemId)
    end
end)
```

Registered in `LuaResourceRuntime.cs` (line 6313) as `LuaResourceRuntime.Bootstrap`.

## RemoveEffect

```lua
RemoveEffect(id, [reason])
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.effects.remove`.

The low-level form of `Open77.effects.remove` (bound to the same native). The two are the same function under two names; `Open77.effects.remove` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `reason` | `string` | optional | — |

```lua
-- RemoveEffect is the low-level form of Open77.effects.remove (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- The firefighters put the fire out
Open77.effects.remove(fireId)
```

Registered in `LuaResourceRuntime.cs` (line 1046) as `RemoveEffect`.

_This signature was inferred from the handler body._

## RemoveElevator

```lua
RemoveElevator(id)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.elevators.remove`.

The low-level form of `Open77.elevators.remove` (bound to the same native). The two are the same function under two names; `Open77.elevators.remove` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

```lua
-- RemoveElevator is the low-level form of Open77.elevators.remove (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Hand the lift back to the engine when the resource no longer needs it
Open77.elevators.remove(elevatorId)
```

Registered in `LuaResourceRuntime.cs` (line 1105) as `RemoveElevator`.

_This signature was inferred from the handler body._

## RemoveEventHandler

```lua
RemoveEventHandler(handlerId)
```

`SHARED`: Available without a live game instance.

Removes a handler by id; the same function as Open77.events.off.

No permission. Takes the id AddEventHandler, RegisterNetEvent or Open77.net.on returned. An unknown id is ignored rather than raising. Stopping the resource removes its handlers for you.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `handlerId` | `integer` | required | — |

```lua
-- Listen for the first player only, then stop listening
local handler
handler = AddEventHandler("onPlayerReady", function(playerId)
    Open77.chat.send(tonumber(playerId), "You are the first one in!")
    RemoveEventHandler(handler)
end)
```

Registered in `LuaResourceRuntime.cs` (line 6305) as `LuaResourceRuntime.Bootstrap`.

## RemoveLootDrop

```lua
RemoveLootDrop(id)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.loot.remove`.

The low-level form of `Open77.loot.remove` (bound to the same native). The two are the same function under two names; `Open77.loot.remove` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

```lua
-- RemoveLootDrop is the low-level form of Open77.loot.remove (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Round over: clear the leftover drops
for _, drop in ipairs(Open77.loot.all(ARENA_BUCKET)) do Open77.loot.remove(drop.id) end
```

Registered in `LuaResourceRuntime.cs` (line 1054) as `RemoveLootDrop`.

_This signature was inferred from the handler body._

## RemoveNpc

```lua
RemoveNpc(id)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.npcs.remove`.

The low-level form of `Open77.npcs.remove` (bound to the same native). The two are the same function under two names; `Open77.npcs.remove` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

```lua
-- RemoveNpc is the low-level form of Open77.npcs.remove (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Clean up every NPC this resource spawned when the event ends
AddEventHandler("event:ended", function()
    for _, npcId in ipairs(spawned) do Open77.npcs.remove(npcId) end
    spawned = {}
end)
```

Registered in `LuaResourceRuntime.cs` (line 1080) as `RemoveNpc`.

_This signature was inferred from the handler body._

## RemoveProp

```lua
RemoveProp(id, [reason])
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.props.remove`.

The low-level form of `Open77.props.remove` (bound to the same native). The two are the same function under two names; `Open77.props.remove` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `reason` | `string` | optional | — |

```lua
-- RemoveProp is the low-level form of Open77.props.remove (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- The player picked up the crate
RegisterNetEvent("crates:pickup", function(crateId)
    if Open77.props.remove(crateId) then giveReward(source) end
end)
```

Registered in `LuaResourceRuntime.cs` (line 1039) as `RemoveProp`.

_This signature was inferred from the handler body._

## RemoveStateBagChangeHandler

```lua
RemoveStateBagChangeHandler(subscriptionId)
```

`SHARED`: Available without a live game instance.

Cancels a subscription made by AddStateBagChangeHandler.

Takes the id that call returned; the same function as `Open77.state.offChange`. An id that was already cancelled, or never issued, is ignored rather than raising. Stopping the resource cancels its subscriptions for you, so this is for a resource that wants to stop listening while it keeps running.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `subscriptionId` | `integer` | required | — |

Returns: `true`, `or false, reason`

```lua
-- Stop watching the round state once the match is over
local sub = AddStateBagChangeHandler("round", "global", function(bag, key, value)
    print("round is now", value)
end)
AddEventHandler("arena:matchOver", function() RemoveStateBagChangeHandler(sub) end)
```

Registered in `LuaResourceRuntime.State.cs` (line 486) as `LuaResourceRuntime.Bootstrap`.

## RemoveVehicle

```lua
RemoveVehicle(id)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.vehicles.remove`.

The low-level form of `Open77.vehicles.remove` (bound to the same native). The two are the same function under two names; `Open77.vehicles.remove` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

```lua
-- RemoveVehicle is the low-level form of Open77.vehicles.remove (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- /dv: delete the vehicle the caller is sitting in
RegisterCommand("dv", function(source)
    local seat = Open77.vehicles.getPlayerSeat(source)
    if not seat then return Open77.chat.send(source, "You are not in a vehicle.") end
    Open77.vehicles.remove(seat.vehicleId)
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 1064) as `RemoveVehicle`.

_This signature was inferred from the handler body._

## RequestPlayerLifeResync

```lua
RequestPlayerLifeResync(playerId)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.players.requestLifeResync`.

The low-level form of `Open77.players.requestLifeResync` (bound to the same native). The two are the same function under two names; `Open77.players.requestLifeResync` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

```lua
-- RequestPlayerLifeResync is the low-level form of Open77.players.requestLifeResync (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- A player reports "I look dead but I am alive": push the canonical state again
RegisterCommand("resync", function(source)
    Open77.players.requestLifeResync(source)
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 1115) as `RequestPlayerLifeResync`.

_This signature was inferred from the handler body._

## ResetPlayerModel

```lua
ResetPlayerModel(playerId)
```

`NETWORK`: Uses the network backend.

Release this resource's model override and restore the original character.

Requires players.model.control and a connected positive integer player ID. Returns true when already original. Refuses model_owned_by_other_resource if another resource generation owns the override. Broadcasts restoration in the player's bucket and emits onPlayerModelChanged with a nil record and reason reset. Cosmetic NPC presentation only: the real player retains identity, controls, health, equipment and saved appearance. Character.* records resolve against each client's installed game data, not a virtual allowlist. See player-models.md for ownership, events, cleanup and the known special-rig/vehicle limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `boolean`, `string? reason`

```lua
local ok, reason = ResetPlayerModel(playerId)
if not ok then print(reason) end
```

[Player morphs and model lifecycle guide](../../01%20Documentation/player-models.md)

Registered in `LuaResourceRuntime.cs` (line 965) as `ResetPlayerModel`.

## ResetVehiclePaint

```lua
ResetVehiclePaint(id)
```

`NETWORK`: Uses the network backend.

Drops a custom paint job and returns the vehicle to its record's own colours.

The all-positional form of `Open77.vehicles.resetPaint`, and the undo of `SetVehiclePaint`. Answers a plain `true`/`false` with no reason: `false` covers an unknown vehicle and a missing `world.vehicles` permission alike. A vehicle that was never repainted answers `true` and changes nothing.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

```lua
-- /unpaint <id>: hand a vehicle back its factory colours.
RegisterCommand("unpaint", function(source, args)
    local id = tonumber(args[1])
    if id and not ResetVehiclePaint(id) then
        Open77.chat.send(source, "No such vehicle, or not ours.")
    end
end, true)
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 1060) as `ResetVehiclePaint`.

## RespawnPlayer

```lua
RespawnPlayer(playerId, x, y, z, yaw, bucket, health, grace)
```

`NETWORK`: Uses the network backend.

Brings a dead player back at a chosen pose, bucket and health.

The all-positional form of `Open77.players.respawn`. All eight arguments are required: position, `yaw`, `bucket`, `health` and a spawn-protection `graceMs` which must be between 0 and 60000. A successful respawn also moves the player's routing bucket, so this is the one call to make rather than respawning and then re-bucketing. Answers `false, reason` (`invalid_argument`, `player_not_found`, a life-ledger reason). Requires `players.life.respawn`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `z` | `number` | required | — |
| `yaw` | `number` | required | — |
| `bucket` | `integer` | required | — |
| `health` | `number` | required | — |
| `grace` | `integer` | required | — |

```lua
-- Respawn anyone who dies at the clinic, with five seconds of protection.
AddEventHandler("open77:playerDied", function(playerId)
    local ok, reason = RespawnPlayer(
        playerId, -1442.2, 127.4, 18.0, 180.0,   -- player, x, y, z, yaw
        0, 100.0, 5000)                          -- bucket, health, graceMs
    if not ok then print(("respawn refused: %s"):format(tostring(reason))) end
end)
```

Registered in `LuaResourceRuntime.cs` (line 1112) as `RespawnPlayer`.

## RestorePlayerHealth

```lua
RestorePlayerHealth(playerId)
```

`NETWORK`: Uses the network backend.

Fill health.

Fill health.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |

```lua
-- RestorePlayerHealth is the low-level form of Open77.stats.restoreHealth (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Round start: full health for everyone in the arena
for _, playerId in ipairs(Open77.players.inBucket(ARENA_BUCKET)) do
    Open77.stats.restoreHealth(playerId)
end
```

Registered in `LuaResourceRuntime.cs` (line 1137) as `RestorePlayerHealth`.

## RestorePlayerStamina

```lua
RestorePlayerStamina(playerId)
```

`NETWORK`: Uses the network backend.

Fill stamina.

Fill stamina.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |

```lua
-- RestorePlayerStamina is the low-level form of Open77.stats.restoreStamina (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- An energy drink
RegisterNetEvent("items:energyDrink", function()
    Open77.stats.restoreStamina(source)
end)
```

Registered in `LuaResourceRuntime.cs` (line 1145) as `RestorePlayerStamina`.

## ResumeElevator

```lua
ResumeElevator(id)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.elevators.resume`.

The low-level form of `Open77.elevators.resume` (bound to the same native). The two are the same function under two names; `Open77.elevators.resume` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

```lua
-- ResumeElevator is the low-level form of Open77.elevators.resume (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Power is back
for _, lift in ipairs(Open77.elevators.all()) do Open77.elevators.resume(lift.id) end
```

Registered in `LuaResourceRuntime.cs` (line 1103) as `ResumeElevator`.

_This signature was inferred from the handler body._

## ReviveNpc

```lua
ReviveNpc(id, health)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.npcs.revive`.

The low-level form of `Open77.npcs.revive` (wrapped by). The two are the same function under two names; `Open77.npcs.revive` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `health` | `number` | required | — |

```lua
-- ReviveNpc is the low-level form of Open77.npcs.revive (wrapped by);
-- the example shows the Open77.* spelling resources are told to prefer.
-- The medic revives the fallen bodyguard at half health
Open77.npcs.revive(bodyguardId, 50)
```

Registered in `LuaResourceRuntime.cs` (line 1090) as `ReviveNpc`.

_This signature was inferred from the handler body._

## RevivePlayer

```lua
RevivePlayer(playerId, health, grace)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.players.revive`.

The low-level form of `Open77.players.revive` (wrapped by). The two are the same function under two names; `Open77.players.revive` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `health` | `number` | required | — |
| `grace` | `integer` | required | — |

```lua
-- RevivePlayer is the low-level form of Open77.players.revive (wrapped by);
-- the example shows the Open77.* spelling resources are told to prefer.
-- A medic revives a downed player nearby
RegisterCommand("revive", function(source)
    local patient = Open77.players.closest(source, { radius = 3 })
    if not patient or not Open77.players.isDead(patient.playerId) then
        return Open77.chat.send(source, "Nobody to revive here.")
    end
    Open77.players.revive(patient.playerId, { health = 0.3, graceMs = 5000 })
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 1111) as `RevivePlayer`.

_This signature was inferred from the handler body._

## SaveResourceFile

```lua
SaveResourceFile(resourceName, path, data)
```

`SHARED`: Available without a live game instance.

Writes a file belonging to the calling resource. Server only.

Open77.io.write under the FiveM name, rooted at this resource's data/ directory and gated by filesystem.write. Naming another resource answers false, cross_resource_write_denied. There is no client counterpart: a client resource's directory is the signed download and is not writable.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `resourceName` | `string` | required | — |
| `path` | `string` | required | — |
| `data` | `string` | required | — |

Returns: `true, or false, reason`

```lua
-- Persist the high scores as JSON in data/
SaveResourceFile(GetCurrentResourceName(), "scores.json", json.encode(scores))
```

Registered in `LuaResourceRuntime.Compat.cs` (line 205) as `LuaResourceRuntime.Bootstrap`.

## SeekSound

```lua
SeekSound(id, seconds)
```

`NETWORK`: Uses the network backend.

SeekSound: server-authoritative network package audio.

Requires audio.network; network=false is rejected. Server owns the timeline, handles and bucket/optional-target audience. Clients cannot request broadcasts directly. getState reports intended playback, not decoder acknowledgement. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `seconds` | `number` | required | — |

Returns: `true, or failure, reason`

```lua
-- Jump the shared timeline to the second track
SeekSound(concertId, 180)
```

Registered in `LuaResourceRuntime.cs` (line 9878) as `LuaAudioControl`.

## SetCombatDamageMultiplier

```lua
SetCombatDamageMultiplier(multiplier)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.combat.setDamageMultiplier`.

The low-level form of `Open77.combat.setDamageMultiplier` (bound to the same native). The two are the same function under two names; `Open77.combat.setDamageMultiplier` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `multiplier` | `number` | required | — |

```lua
-- SetCombatDamageMultiplier is the low-level form of Open77.combat.setDamageMultiplier (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- /dmg <factor>: an admin halves or doubles all damage for the round
RegisterCommand("dmg", function(source, args)
    local factor = tonumber(args[1])
    if factor then Open77.combat.setDamageMultiplier(factor) end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 1150) as `SetCombatDamageMultiplier`.

_This signature was inferred from the handler body._

## SetCombatFriendlyFire

```lua
SetCombatFriendlyFire([enabled])
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.combat.setFriendlyFire`.

The low-level form of `Open77.combat.setFriendlyFire` (bound to the same native). The two are the same function under two names; `Open77.combat.setFriendlyFire` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | optional | — |

```lua
-- SetCombatFriendlyFire is the low-level form of Open77.combat.setFriendlyFire (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- PvP is on everywhere on this server
Open77.combat.setFriendlyFire(true)
```

Registered in `LuaResourceRuntime.cs` (line 1148) as `SetCombatFriendlyFire`.

_This signature was inferred from the handler body._

## SetCombatHeadshotMultiplier

```lua
SetCombatHeadshotMultiplier(multiplier)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.combat.setHeadshotMultiplier`.

The low-level form of `Open77.combat.setHeadshotMultiplier` (bound to the same native). The two are the same function under two names; `Open77.combat.setHeadshotMultiplier` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `multiplier` | `number` | required | — |

```lua
-- SetCombatHeadshotMultiplier is the low-level form of Open77.combat.setHeadshotMultiplier (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- A "one in the head" round: headshots do five times the damage
Open77.combat.setHeadshotMultiplier(5)
```

Registered in `LuaResourceRuntime.cs` (line 1151) as `SetCombatHeadshotMultiplier`.

_This signature was inferred from the handler body._

## SetCombatKindMultiplier

```lua
SetCombatKindMultiplier(kindName, multiplier)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.combat.setKindDamageMultiplier`.

The low-level form of `Open77.combat.setKindDamageMultiplier` (bound to the same native). The two are the same function under two names; `Open77.combat.setKindDamageMultiplier` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `kindName` | `string` | required | — |
| `multiplier` | `number` | required | — |

```lua
-- SetCombatKindMultiplier is the low-level form of Open77.combat.setKindDamageMultiplier (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Fists-only round: guns and explosions do nothing, melee does double
Open77.combat.setKindDamageMultiplier("ranged", 0)
Open77.combat.setKindDamageMultiplier("explosion", 0)
Open77.combat.setKindDamageMultiplier("melee", 2)
```

Registered in `LuaResourceRuntime.cs` (line 1153) as `SetCombatKindMultiplier`.

_This signature was inferred from the handler body._

## SetCombatTeam

```lua
SetCombatTeam(playerId, team)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.combat.setTeam`.

The low-level form of `Open77.combat.setTeam` (bound to the same native). The two are the same function under two names; `Open77.combat.setTeam` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `team` | `integer` | required | — |

```lua
-- SetCombatTeam is the low-level form of Open77.combat.setTeam (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Split the arena players into two teams; team-mates cannot hurt each other
for i, playerId in ipairs(Open77.players.inBucket(ARENA_BUCKET)) do
    Open77.combat.setTeam(playerId, (i % 2) + 1)
end
```

Registered in `LuaResourceRuntime.cs` (line 1149) as `SetCombatTeam`.

_This signature was inferred from the handler body._

## SetCombatWeaponMultiplier

```lua
SetCombatWeaponMultiplier(weaponTdbId, multiplier)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.combat.setWeaponDamageMultiplier`.

The low-level form of `Open77.combat.setWeaponDamageMultiplier` (bound to the same native). The two are the same function under two names; `Open77.combat.setWeaponDamageMultiplier` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `weaponTdbId` | `integer` | required | — |
| `multiplier` | `number` | required | — |

```lua
-- SetCombatWeaponMultiplier is the low-level form of Open77.combat.setWeaponDamageMultiplier (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- The Lexington is a training pistol here: a quarter of its damage
Open77.combat.setWeaponDamageMultiplier("Items.Preset_Lexington_Default", 0.25)
```

Registered in `LuaResourceRuntime.cs` (line 1152) as `SetCombatWeaponMultiplier`.

_This signature was inferred from the handler body._

## SetConvar

```lua
SetConvar(name, value)
```

`NETWORK`: Uses the network backend.

Records a server-wide, in-memory convar override.

Server-wide, not per resource -- `SetConvar` here is visible to `GetConvar` in another resource, which is what the FiveM code being ported assumes and is exactly why a convar is a poor place for anything two resources might both want to own. **In memory only**: the value is gone on the next restart and the `convars` block of `server.jsonc` comes back. Making it durable would mean writing to the operator's tracked configuration file from Lua. Booleans and numbers are coerced to their string form. 512 overrides, 4096-byte values, names `[A-Za-z0-9_.:-]` up to 128 bytes.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `value` | `string|number|boolean` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Toggle a server-wide flag other resources read with GetConvar
RegisterCommand("maintenance", function(source, args)
    SetConvar("myserver_maintenance", args[1] == "on")
end, true)
```

Registered in `LuaResourceRuntime.Convars.cs` (line 140) as `LuaResourceRuntime.Bootstrap`.

## SetElevatorFlags

```lua
SetElevatorFlags(id, flags)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.elevators.setFlags`.

The low-level form of `Open77.elevators.setFlags` (bound to the same native). The two are the same function under two names; `Open77.elevators.setFlags` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `flags` | `integer` | required | — |

```lua
-- SetElevatorFlags is the low-level form of Open77.elevators.setFlags (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Lock the lift for the heist: keep it powered but not usable
local f = Open77.elevators.flags
Open77.elevators.setFlags(elevatorId, f.powered | f.locked)
```

Registered in `LuaResourceRuntime.cs` (line 1104) as `SetElevatorFlags`.

_This signature was inferred from the handler body._

## SetEntityInvincible

```lua
SetEntityInvincible(kind, id, enabled)
```

`NETWORK`: Uses the network backend.

FiveM's invincibility toggle under its own name: `Open77.players.setGodMode` for a player, refused by name for every other kind.

Requires `players.stats.apply` (or the legacy `players.damage.apply`), exactly as `setGodMode` does -- it is that function under the FiveM name, and a permission refusal comes back before any id is looked at. FiveM's native takes one opaque entity handle; here the kind is named, as `DoesEntityExist` does, and only `"player"` has a canonical damage immunity to switch: the combat ledger's god mode, read back as `Open77.players.get(id).godMode` or `getStats(id).godMode`. `"vehicle"`, `"npc"`, `"prop"`, `"loot"`, `"effect"` and `"elevator"` answer `false, unsupported_entity_kind` rather than accepting and ignoring the call: a vehicle's damage is its own scopes (`setEngineHealth`, the body zones) and an NPC's its `damagePolicy`, and a ported script that expects an indestructible car should learn that here rather than on the first hit. The one-argument FiveM form is `false, kind_required`; a name that is no kind at all is `unknown_entity_kind`; a missing id is `id_required`; a non-boolean `enabled` is `invalid_argument`, and so is a player the ledger does not hold yet (not spawned). This is not the fall-damage toggle (`setFallDamage`) and not the grav-chute's blanket shield.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `kind` | `string` | required | — |
| `id` | `integer` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `boolean, or false, reason`

```lua
-- A ported safe zone: the FiveM line runs unchanged once the kind is named.
AddEventHandler("safezone:entered", function(playerId)
    local ok, reason = SetEntityInvincible("player", playerId, true)
    if not ok then print("could not shield " .. playerId .. ": " .. reason) end
end)
AddEventHandler("safezone:left", function(playerId)
    SetEntityInvincible("player", playerId, false)
end)
```

Registered in `LuaResourceRuntime.Compat.cs` (line 291) as `LuaResourceRuntime.Bootstrap`.

## SetEntityRoutingBucket

```lua
SetEntityRoutingBucket(entityId, bucket)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.routingBuckets.setEntity`.

The low-level form of `Open77.routingBuckets.setEntity` (bound to the same native). The two are the same function under two names; `Open77.routingBuckets.setEntity` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entityId` | `integer` | required | — |
| `bucket` | `integer` | required | — |

```lua
-- SetEntityRoutingBucket is the low-level form of Open77.routingBuckets.setEntity (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Move the entity into the arena instance
Open77.routingBuckets.setEntity(entityId, ARENA_BUCKET)
```

Registered in `LuaResourceRuntime.cs` (line 980) as `SetEntityRoutingBucket`.

_This signature was inferred from the handler body._

## SetHttpHandler

```lua
SetHttpHandler(handler)
```

`NETWORK`: Uses the network backend.

FiveM spelling of a whole-resource inbound HTTP handler.

`Open77.http.listen('/', handler)`: everything under `/<resource>/` reaches the handler with the same `(req, res)` shapes; `SetHttpHandler(nil)` is `unlisten('/')`. Requires `http.serve` and the server's `httpHandlers` listener; see `Open77.http.listen` for the request table, the response helpers, the 16 KiB body cap and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `handler` | `function(req, res), or nil to remove` | required | — |

Returns: `public route string (or true when removing), or nil`, `reason on failure`

```lua
-- A ported FiveM webhook resource keeps its shape
SetHttpHandler(function(req, res)
    if req.path == '/status' then
        return res.send(200, json.encode({ players = #GetPlayers() }), { ['Content-Type'] = 'application/json' })
    end
    res.send(404, 'not found')
end)
```

Registered in `LuaResourceRuntime.Http.cs` (line 298) as `LuaResourceRuntime.Bootstrap`.

## SetNpcAttitude

```lua
SetNpcAttitude(npcId, [attitude], scope, [targetId], [group])
```

`NETWORK`: Uses the network backend.

Sets how one owned NPC feels about everyone, one player, one NPC, or a group.

The all-positional form of `Open77.npcs.setAttitude`. `scope` picks which of the two trailing arguments is read: `0` everyone (neither), `1` a player id and `2` an NPC id (`targetId`), `3` a group name (`group`). `attitude` is a token (`"friendly"`, `"neutral"`, `"hostile"`) and `nil` clears the row back to the default. Answers `false, reason` -- `npc_attitude_invalid`, `npc_attitude_target_invalid`, `npc_not_found`. Requires `world.npcs`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `npcId` | `integer` | required | — |
| `attitude` | `string` | optional | — |
| `scope` | `integer` | required | — |
| `targetId` | `integer` | optional | — |
| `group` | `string` | optional | — |

```lua
-- A bodyguard hostile to everyone by default, but never to its client.
local ok, reason = SetNpcAttitude(guardId, "hostile", 0)      -- scope 0: everyone
if not ok then return print("attitude refused: " .. tostring(reason)) end
SetNpcAttitude(guardId, "friendly", 1, clientPlayerId)        -- scope 1: one player
SetNpcAttitude(guardId, "friendly", 3, nil, "valentinos")     -- scope 3: a group
```

Registered in `LuaResourceRuntime.cs` (line 1093) as `SetNpcAttitude`.

## SetNpcBucket

```lua
SetNpcBucket(id, bucket)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.npcs.setBucket`.

The low-level form of `Open77.npcs.setBucket` (bound to the same native). The two are the same function under two names; `Open77.npcs.setBucket` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `bucket` | `integer` | required | — |

```lua
-- SetNpcBucket is the low-level form of Open77.npcs.setBucket (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Move the arena boss into the arena instance
Open77.npcs.setBucket(bossId, ARENA_BUCKET)
```

Registered in `LuaResourceRuntime.cs` (line 1079) as `SetNpcBucket`.

_This signature was inferred from the handler body._

## SetNpcGroup

```lua
SetNpcGroup(id, group)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.npcs.setGroup`.

The low-level form of `Open77.npcs.setGroup` (bound to the same native). The two are the same function under two names; `Open77.npcs.setGroup` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `group` | `string` | required | — |

```lua
-- SetNpcGroup is the low-level form of Open77.npcs.setGroup (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Two spawned gang members become allies through a shared group
Open77.npcs.setGroup(firstId, "maelstrom")
Open77.npcs.setGroup(secondId, "maelstrom")
```

Registered in `LuaResourceRuntime.cs` (line 1095) as `SetNpcGroup`.

_This signature was inferred from the handler body._

## SetNpcRelationship

```lua
SetNpcRelationship(first, second, attitude)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.npcs.setRelationship`.

The low-level form of `Open77.npcs.setRelationship` (bound to the same native). The two are the same function under two names; `Open77.npcs.setRelationship` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `first` | `string` | required | — |
| `second` | `string` | required | — |
| `attitude` | `string` | required | — |

```lua
-- SetNpcRelationship is the low-level form of Open77.npcs.setRelationship (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Gang war: every Maelstrom NPC this resource owns hates every Valentino
Open77.npcs.setRelationship("maelstrom", "valentinos", "hostile")
```

Registered in `LuaResourceRuntime.cs` (line 1096) as `SetNpcRelationship`.

_This signature was inferred from the handler body._

## SetNpcTransform

```lua
SetNpcTransform(id, x, y, z, yaw)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.npcs.setTransform`.

The low-level form of `Open77.npcs.setTransform` (wrapped by). The two are the same function under two names; `Open77.npcs.setTransform` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `z` | `number` | required | — |
| `yaw` | `number` | required | — |

```lua
-- SetNpcTransform is the low-level form of Open77.npcs.setTransform (wrapped by);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Teleport the shopkeeper back behind the counter
Open77.npcs.setTransform(shopkeeperId, { position = counter, yaw = 180.0 })
```

Registered in `LuaResourceRuntime.cs` (line 1078) as `SetNpcTransform`.

_This signature was inferred from the handler body._

## SetPlayerArmor

```lua
SetPlayerArmor(playerId, armor)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.players.setArmor`.

The low-level form of `Open77.players.setArmor` (bound to the same native). The two are the same function under two names; `Open77.players.setArmor` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `armor` | `number` | required | — |

```lua
-- SetPlayerArmor is the low-level form of Open77.players.setArmor (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Buying a vest at the shop
RegisterNetEvent("shop:buyArmor", function()
    Open77.players.setArmor(source, 100)
end)
```

Registered in `LuaResourceRuntime.cs` (line 1138) as `SetPlayerArmor`.

_This signature was inferred from the handler body._

## SetPlayerDownedDamageable

```lua
SetPlayerDownedDamageable(playerId, enabled)
```

`NETWORK`: Uses the network backend.

`players.stats.apply`;

`players.stats.apply`; boolean/reason. Also exposed as `Open77.stats.setDownedDamageable`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |
| `enabled` | `value` | required | — |

```lua
-- SetPlayerDownedDamageable is the low-level form of Open77.players.setDownedDamageable (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Battle-royale rule: a downed player can still be finished
AddEventHandler("onPlayerReady", function(playerId)
    Open77.players.setDownedDamageable(playerId, true)
end)
```

Registered in `LuaResourceRuntime.cs` (line 1140) as `SetPlayerDownedDamageable`.

## SetPlayerFrozen

```lua
SetPlayerFrozen(playerId, frozen)
```

`NETWORK`: Uses the network backend.

Hold the player's body where it stands, or release this resource's hold.

Hold the player's body where it stands, or release this resource's hold. `boolean, reason?`

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |
| `frozen` | `value` | required | — |

```lua
-- SetPlayerFrozen is the low-level form of Open77.players.setFrozen (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- /freeze <id> toggles a hold on a player
RegisterCommand("freeze", function(source, args)
    local target = tonumber(args[1])
    if not target then return end
    local frozen = Open77.players.isFrozen(target)
    Open77.players.setFrozen(target, not frozen)
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 1118) as `SetPlayerFrozen`.

## SetPlayerGhosted

```lua
SetPlayerGhosted(playerId, ghosted, options?)
```

`NETWORK`: Uses the network backend.

Toggle the server-owned non-solid spawn state and its optional automatic clear policy.

Toggle the server-owned non-solid spawn state and its optional automatic clear policy.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |
| `ghosted` | `value` | required | — |
| `options?` | `value` | required | — |

```lua
-- SetPlayerGhosted is the low-level form of Open77.players.setGhosted (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Spawn protection: non-solid for 10 seconds or until they walk 15 m away
AddEventHandler("onPlayerReady", function(playerId)
    Open77.players.setGhosted(playerId, true, { clearMs = 10000, clearDistance = 15.0 })
end)
```

Registered in `LuaResourceRuntime.cs` (line 1116) as `SetPlayerGhosted`.

## SetPlayerGodMode

```lua
SetPlayerGodMode(playerId, [enabled])
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.players.setGodMode`.

The low-level form of `Open77.players.setGodMode` (bound to the same native). The two are the same function under two names; `Open77.players.setGodMode` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `enabled` | `boolean` | optional | — |

```lua
-- SetPlayerGodMode is the low-level form of Open77.players.setGodMode (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- /god toggles invulnerability for an admin
RegisterCommand("god", function(source)
    godMode[source] = not godMode[source]
    Open77.players.setGodMode(source, godMode[source])
    Open77.chat.send(source, godMode[source] and "God mode on." or "God mode off.")
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 1139) as `SetPlayerGodMode`.

_This signature was inferred from the handler body._

## SetPlayerHealth

```lua
SetPlayerHealth(playerId, value)
```

`NETWORK`: Uses the network backend.

Explicit health setter;

Explicit health setter; zero routes through life authority.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |
| `value` | `value` | required | — |

```lua
-- SetPlayerHealth is the low-level form of Open77.stats.setHealth (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- /sethp <id> <value>
RegisterCommand("sethp", function(source, args)
    local target, value = tonumber(args[1]), tonumber(args[2])
    if target and value then Open77.stats.setHealth(target, value) end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 1135) as `SetPlayerHealth`.

## SetPlayerIntoVehicle

```lua
SetPlayerIntoVehicle(playerId, vehicleId, seat, [moveBucketValue], [exitLockedValue], [animatedValue])
```

`NETWORK`: Uses the network backend.

Teleports a player straight into a seat, moving their bucket to the vehicle's.

The all-positional form of `Open77.vehicles.setPlayerIntoVehicle`. Seats use FiveM's numbering: `-1` driver, `0` front passenger, `1` and `2` the rear. `moveBucket` defaults to true -- pass false and a player in another bucket is refused `wrong_bucket` instead of being moved. `animated` false is the instant snap. A dead or respawning player is refused `player_unavailable`. Other refusals are `vehicle_not_found`, `invalid_seat` and a seat already taken. Requires `world.vehicles`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `vehicleId` | `integer` | required | — |
| `seat` | `value` | required | — |
| `moveBucketValue` | `boolean` | optional | — |
| `exitLockedValue` | `boolean` | optional | — |
| `animatedValue` | `boolean` | optional | — |

```lua
-- /taxi: drop the caller into the driver's seat of the nearest cab.
RegisterCommand("taxi", function(source)
    local ok, reason = SetPlayerIntoVehicle(
        source, cabId, -1,      -- player, vehicle, seat -1 = driver
        true, false, false)     -- moveBucket, exitLocked, animated
    if not ok then Open77.chat.send(source, "No ride: " .. tostring(reason)) end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 1072) as `SetPlayerIntoVehicle`.

## SetPlayerMaxHealth

```lua
SetPlayerMaxHealth(playerId, maximum)
```

`NETWORK`: Uses the network backend.

Explicit health maximum setter.

Explicit health maximum setter.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |
| `maximum` | `value` | required | — |

```lua
-- SetPlayerMaxHealth is the low-level form of Open77.stats.setHealthMax (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- A perk: 150 maximum health
Open77.stats.setHealthMax(playerId, 150)
```

Registered in `LuaResourceRuntime.cs` (line 1136) as `SetPlayerMaxHealth`.

## SetPlayerMaxStamina

```lua
SetPlayerMaxStamina(playerId, maximum)
```

`NETWORK`: Uses the network backend.

Explicit stamina maximum setter.

Explicit stamina maximum setter.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |
| `maximum` | `value` | required | — |

```lua
-- SetPlayerMaxStamina is the low-level form of Open77.stats.setStaminaMax (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- A "runner" class gets a bigger stamina pool
Open77.stats.setStaminaMax(playerId, 150)
```

Registered in `LuaResourceRuntime.cs` (line 1144) as `SetPlayerMaxStamina`.

## SetPlayerModel

```lua
SetPlayerModel(playerId, record, [options])
```

`NETWORK`: Uses the network backend.

Select a server-owned NPC body for a connected player.

Requires players.model.control. The player must be connected, alive and gameplay-ready. This resource generation exclusively owns the override; another owner fails with model_owned_by_other_resource. Options: appearance (ASCII CName up to 128 characters, default empty), durationMs (integer 0–86400000, default 0), resetOnDeath (boolean, default false); unknown fields are rejected. True, revision means accepted, not visible. Observe onPlayerModelReady/onPlayerModelFailed; a missing record fails asynchronously and restores the original body. Identical options extend the lease without respawn. Stopping the owner, timeout or disconnect releases state. Cosmetic NPC presentation only: the real player retains identity, controls, health, equipment and saved appearance. Character.* records resolve against each client's installed game data, not a virtual allowlist. See player-models.md for ownership, events, cleanup and the known special-rig/vehicle limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `record` | `string` | required | — |
| `options` | `table` | optional | — |

Returns: `boolean accepted`, `integer revision or string reason`

```lua
local ok, revision = SetPlayerModel(playerId, 'Character.Rogue', { durationMs = 60000, resetOnDeath = true })
```

[Player morphs and model lifecycle guide](../../01%20Documentation/player-models.md)

Registered in `LuaResourceRuntime.cs` (line 7856) as `LuaResourceRuntime.Bootstrap`.

## SetPlayerRegen

```lua
SetPlayerRegen(playerId, rate)
```

`NETWORK`: Uses the network backend.

Set health regeneration rate.

Set health regeneration rate.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |
| `rate` | `value` | required | — |

```lua
-- SetPlayerRegen is the low-level form of Open77.stats.setHealthRegenRate (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Hardcore server: health never comes back on its own
AddEventHandler("onPlayerReady", function(playerId)
    Open77.stats.setHealthRegenRate(playerId, 0)
end)
```

Registered in `LuaResourceRuntime.cs` (line 1141) as `SetPlayerRegen`.

## SetPlayerRegenEnabled

```lua
SetPlayerRegenEnabled(playerId, enabled)
```

`NETWORK`: Uses the network backend.

Toggle health regeneration.

Toggle health regeneration.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |
| `enabled` | `value` | required | — |

```lua
-- SetPlayerRegenEnabled is the low-level form of Open77.stats.setHealthRegenEnabled (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Bleeding: no regeneration until bandaged
Open77.stats.setHealthRegenEnabled(playerId, false)
RegisterNetEvent("items:bandage", function()
    Open77.stats.setHealthRegenEnabled(source, true)
end)
```

Registered in `LuaResourceRuntime.cs` (line 1142) as `SetPlayerRegenEnabled`.

## SetPlayerRoutingBucket

```lua
SetPlayerRoutingBucket(playerId, bucket)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.routingBuckets.setPlayer`.

The low-level form of `Open77.routingBuckets.setPlayer` (bound to the same native). The two are the same function under two names; `Open77.routingBuckets.setPlayer` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `bucket` | `integer` | required | — |

```lua
-- SetPlayerRoutingBucket is the low-level form of Open77.routingBuckets.setPlayer (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Each apartment is its own instance: bucket 1000 + house id
RegisterNetEvent("house:enter", function(houseId)
    Open77.routingBuckets.setPlayer(source, 1000 + houseId)
end)
```

Registered in `LuaResourceRuntime.cs` (line 978) as `SetPlayerRoutingBucket`.

_This signature was inferred from the handler body._

## SetPlayerStamina

```lua
SetPlayerStamina(playerId, value)
```

`NETWORK`: Uses the network backend.

Explicit stamina setter.

Explicit stamina setter.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |
| `value` | `value` | required | — |

```lua
-- SetPlayerStamina is the low-level form of Open77.stats.setStamina (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- The player just ran a marathon: stamina to 10
Open77.stats.setStamina(playerId, 10)
```

Registered in `LuaResourceRuntime.cs` (line 1143) as `SetPlayerStamina`.

## SetPlayerStaminaRegen

```lua
SetPlayerStaminaRegen(playerId, rate)
```

`NETWORK`: Uses the network backend.

Set stamina regeneration rate.

Set stamina regeneration rate.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |
| `rate` | `value` | required | — |

```lua
-- SetPlayerStaminaRegen is the low-level form of Open77.stats.setStaminaRegenRate (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- A heavy backpack halves stamina regeneration
Open77.stats.setStaminaRegenRate(playerId, 5)
```

Registered in `LuaResourceRuntime.cs` (line 1146) as `SetPlayerStaminaRegen`.

## SetPlayerStaminaRegenEnabled

```lua
SetPlayerStaminaRegenEnabled(playerId, enabled)
```

`NETWORK`: Uses the network backend.

Toggle stamina regeneration.

Toggle stamina regeneration.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |
| `enabled` | `value` | required | — |

```lua
-- SetPlayerStaminaRegenEnabled is the low-level form of Open77.stats.setStaminaRegenEnabled (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Exhaustion: no stamina regeneration for 20 seconds
Open77.stats.setStaminaRegenEnabled(playerId, false)
SetTimeout(20000, function() Open77.stats.setStaminaRegenEnabled(playerId, true) end)
```

Registered in `LuaResourceRuntime.cs` (line 1147) as `SetPlayerStaminaRegenEnabled`.

## SetPlayerVehicleExitLocked

```lua
SetPlayerVehicleExitLocked(playerId, [locked], vehicleId)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.vehicles.setPlayerExitLocked`.

The low-level form of `Open77.vehicles.setPlayerExitLocked` (bound to the same native). The two are the same function under two names; `Open77.vehicles.setPlayerExitLocked` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `locked` | `boolean` | optional | — |
| `vehicleId` | `integer` | required | — |

```lua
-- SetPlayerVehicleExitLocked is the low-level form of Open77.vehicles.setPlayerExitLocked (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Arrest: the suspect cannot leave the cruiser until released
Open77.vehicles.warpPlayerIntoVehicle(suspectId, cruiserId, Open77.vehicles.seats.rearLeft)
Open77.vehicles.setPlayerExitLocked(suspectId, true, cruiserId)
```

Registered in `LuaResourceRuntime.cs` (line 1074) as `SetPlayerVehicleExitLocked`.

_This signature was inferred from the handler body._

## SetPlayerVisible

```lua
SetPlayerVisible(playerId, visible)
```

`NETWORK`: Uses the network backend.

Hide a player's body from every other client, or show it again.

Hide a player's body from every other client, or show it again. Presentation only; see [Player visibility](#player-visibility) below.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |
| `visible` | `value` | required | — |

```lua
-- SetPlayerVisible is the low-level form of Open77.players.setVisible (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- /vanish toggles an invisible admin
RegisterCommand("vanish", function(source)
    hidden[source] = not hidden[source]
    Open77.players.setVisible(source, not hidden[source])
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 1120) as `SetPlayerVisible`.

## SetPropBucket

```lua
SetPropBucket(id, bucket)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.props.setBucket`.

The low-level form of `Open77.props.setBucket` (bound to the same native). The two are the same function under two names; `Open77.props.setBucket` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `bucket` | `integer` | required | — |

```lua
-- SetPropBucket is the low-level form of Open77.props.setBucket (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Move the stage props into the concert instance
for _, propId in ipairs(stageProps) do Open77.props.setBucket(propId, CONCERT_BUCKET) end
```

Registered in `LuaResourceRuntime.cs` (line 1038) as `SetPropBucket`.

_This signature was inferred from the handler body._

## SetPropTransform

```lua
SetPropTransform(id, x, y, z, yaw, [scaleX], [scaleY], [scaleZ])
```

`NETWORK`: Uses the network backend.

Moves and rotates a prop, optionally rescaling it, and answers whether it took.

The all-positional form of `Open77.props.setTransform`. The three scale arguments are optional and keep their current value when `nil`. An **attached** prop is refused `prop_attached` rather than moved, because the follow tick would overwrite the move within 100 ms -- detach it first. Other refusals are `not_found`, `invalid_position` and `permission_denied:world.props`. Requires `world.props`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `z` | `number` | required | — |
| `yaw` | `number` | required | — |
| `scaleX` | `number` | optional | — |
| `scaleY` | `number` | optional | — |
| `scaleZ` | `number` | optional | — |

```lua
-- Slide a barrier aside when the gate opens, then put it back.
local ok, reason = SetPropTransform(barrierId, -1442.2, 131.0, 18.0, 90.0)
if not ok then print("could not move the barrier: " .. tostring(reason)) end
SetTimeout(10000, function()
    SetPropTransform(barrierId, -1442.2, 127.4, 18.0, 90.0)
end)
```

Registered in `LuaResourceRuntime.cs` (line 1037) as `SetPropTransform`.

## SetResourceKvp

```lua
SetResourceKvp(key, value)
```

`NETWORK`: Uses the network backend.

FiveM-style alias for `Open77.kvp.set`.

The same three bare globals the client publishes, over the server's own store. FiveM's typed variants (`SetResourceKvpInt`, `GetResourceKvpFloat`, ...) are deliberately absent: they are one line of Lua each, and spelling the two Open77 runtimes identically is worth more than matching FiveM on a side that never had the API at all.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string` | required | — |
| `value` | `any` | required | — |

Returns: `boolean`, `reason`

```lua
-- A FiveM port saves a player's outfit id
SetResourceKvp("outfit:" .. Open77.players.identifier(source), "corpo_suit")
```

Registered in `LuaResourceRuntime.Kvp.cs` (line 315) as `LuaResourceRuntime.Bootstrap`.

## SetRoutingBucketEntityLockdownMode

```lua
SetRoutingBucketEntityLockdownMode(bucket, mode)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.routingBuckets.setLockdownMode`.

The low-level form of `Open77.routingBuckets.setLockdownMode` (bound to the same native). The two are the same function under two names; `Open77.routingBuckets.setLockdownMode` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | required | — |
| `mode` | `string` | required | — |

```lua
-- SetRoutingBucketEntityLockdownMode is the low-level form of Open77.routingBuckets.setLockdownMode (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Nothing client-created may exist in the competitive arena
Open77.routingBuckets.setLockdownMode(ARENA_BUCKET, "strict")
```

Registered in `LuaResourceRuntime.cs` (line 981) as `SetRoutingBucketLockdownMode`.

_This signature was inferred from the handler body._

## SetRoutingBucketPopulationEnabled

```lua
SetRoutingBucketPopulationEnabled(bucket, [enabled])
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.routingBuckets.setPopulationEnabled`.

The low-level form of `Open77.routingBuckets.setPopulationEnabled` (bound to the same native). The two are the same function under two names; `Open77.routingBuckets.setPopulationEnabled` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | required | — |
| `enabled` | `boolean` | optional | — |

```lua
-- SetRoutingBucketPopulationEnabled is the low-level form of Open77.routingBuckets.setPopulationEnabled (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- No traffic or crowd in the arena instance
Open77.routingBuckets.setPopulationEnabled(ARENA_BUCKET, false)
```

Registered in `LuaResourceRuntime.cs` (line 982) as `SetRoutingBucketPopulationEnabled`.

_This signature was inferred from the handler body._

## SetSoundPosition

```lua
SetSoundPosition(id, position)
```

`NETWORK`: Uses the network backend.

SetSoundPosition: server-authoritative network package audio.

Requires audio.network; network=false is rejected. Server owns the timeline, handles and bucket/optional-target audience. Clients cannot request broadcasts directly. getState reports intended playback, not decoder acknowledgement. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `position` | `table` | required | — |

Returns: `true, or failure, reason`

```lua
-- The parade float moved: move the shared sound with it
SetSoundPosition(paradeId, Open77.vehicles.getPosition(floatId))
```

Registered in `LuaResourceRuntime.cs` (line 9880) as `LuaAudioControl`.

## SetSoundVolume

```lua
SetSoundVolume(id, volume)
```

`NETWORK`: Uses the network backend.

SetSoundVolume: server-authoritative network package audio.

Requires audio.network; network=false is rejected. Server owns the timeline, handles and bucket/optional-target audience. Clients cannot request broadcasts directly. getState reports intended playback, not decoder acknowledgement. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `volume` | `number` | required | — |

Returns: `true, or failure, reason`

```lua
-- Turn the concert down for everyone
SetSoundVolume(concertId, 0.6)
```

Registered in `LuaResourceRuntime.cs` (line 9879) as `LuaAudioControl`.

## SetTick

```lua
SetTick(body)
```

`SHARED`: Available without a live game instance.

Runs a function once per server tick.

A scheduler task looping with an implicit Wait(0), so it is bounded by the same budget and counted against the same 1,024-task quota as a CreateThread loop. The difference is the pcall: an error inside the function is logged and the tick survives it, up to five *consecutive* failures, after which the tick cancels itself rather than filling the log for the rest of the session. One successful pass resets the count. Answers nil, invalid_tick_function for a non-function and nil, task_limit_or_stopping when no task slot is available.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `body` | `function` | required | — |

Returns: `tick id, or nil, reason`

```lua
-- Check the zone every tick; the tick survives an occasional error
SetTick(function()
    for _, playerId in ipairs(Open77.players.inBucket(0)) do
        checkZone(playerId)
    end
end)
```

Registered in `LuaResourceRuntime.Compat.cs` (line 66) as `LuaResourceRuntime.Bootstrap`.

## SetTimeout

```lua
SetTimeout(milliseconds, body)
```

`SHARED`: Available without a live game instance.

Runs a function once, later.

Schedules a managed task due after the delay, so the body may Wait. Counts against the same 1,024-task budget as CreateThread. Returns the timer id ClearTimeout takes.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `milliseconds` | `integer` | required | — |
| `body` | `function` | required | — |

Returns: `timer id`

```lua
-- Respawn the player 8 seconds after they died
AddEventHandler("onPlayerLifeStateChanged", function(playerId, revision, phase)
    if phase == "dead" then
        SetTimeout(8000, function()
            Open77.players.respawn(playerId, { position = hospital })
        end)
    end
end)
```

Registered in `LuaResourceRuntime.cs` (line 6287) as `LuaResourceRuntime.Bootstrap`.

## SetVehicleLivery

```lua
SetVehicleLivery(vehicle, livery)
```

`NETWORK`: Uses the network backend.

Applies a livery by its FiveM index.

Requires `world.vehicles`; a call that changes the look also requires `world.vehicles.appearance`. The FiveM spelling over `Open77.vehicles` (see the native for the details), with its permissions and its `false, reason` refusals. The vehicle handle is the canonical vehicle id. The index is the 0-based position in `Open77.vehicles.listPaintProfiles` for that car (the build's own templates first, then those running resources registered), and `-1` goes back to generic paint. An index past the list is `invalid_profile`; a non-integer is `invalid_argument`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicle` | `vehicle id` | required | — |
| `livery` | `integer` | required | — |

Returns: `true, or false`, `reason`

```lua
SetVehicleLivery(vehicle, 0) -- the first livery this model lists
SetVehicleLivery(vehicle, -1) -- generic paint again
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.Compat.cs` (line 374) as `LuaResourceRuntime.Bootstrap`.

## SetVehicleLocked

```lua
SetVehicleLocked(id, [locked])
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.vehicles.setLocked`.

The low-level form of `Open77.vehicles.setLocked` (bound to the same native). The two are the same function under two names; `Open77.vehicles.setLocked` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `locked` | `boolean` | optional | — |

```lua
-- SetVehicleLocked is the low-level form of Open77.vehicles.setLocked (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Personal vehicles spawn locked; the owner unlocks with /unlock
local id = Open77.vehicles.create({ record = record, position = spot })
Open77.vehicles.setLocked(id, true)
Open77.vehicles.setLockedForPlayer(id, ownerId, false)
```

Registered in `LuaResourceRuntime.cs` (line 1061) as `SetVehicleLocked`.

_This signature was inferred from the handler body._

## SetVehicleMod

```lua
SetVehicleMod(vehicle, modType, modIndex, [customTires])
```

`NETWORK`: Uses the network backend.

Chooses a kit part or a rim by FiveM mod type and index.

Requires `world.vehicles`; a call that changes the look also requires `world.vehicles.appearance`. The FiveM spelling over `Open77.vehicles` (see the native for the details), with its permissions and its `false, reason` refusals. The vehicle handle is the canonical vehicle id. Mod types 23 and 24 (front and back wheels) are the rims; any other type is the registered parts whose catalogue entry declares that `modType`. The index space of a type on a car is every (part, variant) pair that fits it, ordered by part id then variant id, so an index is stable while the catalogue is. One choice per mod type, like a FiveM slot: choosing a part takes the other parts of that type off. `-1` is stock. An index past the list is `invalid_mod_index`; `customTires` has no counterpart and is ignored.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicle` | `vehicle id` | required | — |
| `modType` | `integer` | required | — |
| `modIndex` | `integer` | required | — |
| `customTires` | `boolean` | optional | — |

Returns: `true, or false`, `reason`

```lua
SetVehicleMod(vehicle, 0, 1)  -- the second spoiler option registered with modType = 0
SetVehicleMod(vehicle, 23, -1) -- factory rims
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.Compat.cs` (line 479) as `LuaResourceRuntime.Bootstrap`.

## SetVehicleNeonLightEnabled

```lua
SetVehicleNeonLightEnabled(vehicle, index, toggle)
```

`NETWORK`: Uses the network backend.

Switches a car's neon on or off.

Requires `world.vehicles`; a call that changes the look also requires `world.vehicles.appearance`. The FiveM spelling over `Open77.vehicles` (see the native for the details), with its permissions and its `false, reason` refusals. The vehicle handle is the canonical vehicle id. Cyberpunk's neon is one light set laid out per chassis, not four strips, so any `index` switches all of it. Switching on a car that has no neon yet lights it in FiveM's default colour, magenta; switching off a car without one is accepted and does nothing. `toggle` must be a boolean (`invalid_argument`).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicle` | `vehicle id` | required | — |
| `index` | `integer` | required | — |
| `toggle` | `boolean` | required | — |

Returns: `true, or false`, `reason`

```lua
for index = 0, 3 do SetVehicleNeonLightEnabled(vehicle, index, true) end
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.Compat.cs` (line 401) as `LuaResourceRuntime.Bootstrap`.

## SetVehicleNeonLightsColour

```lua
SetVehicleNeonLightsColour(vehicle, r, g, b)
```

`NETWORK`: Uses the network backend.

Sets the colour of a car's neon.

Requires `world.vehicles`; a call that changes the look also requires `world.vehicles.appearance`. The FiveM spelling over `Open77.vehicles` (see the native for the details), with its permissions and its `false, reason` refusals. The vehicle handle is the canonical vehicle id. Keeps the on/off state, so a colour set before the lights are switched on is the colour they come on in. Each channel is 0-255.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicle` | `vehicle id` | required | — |
| `r` | `integer` | required | — |
| `g` | `integer` | required | — |
| `b` | `integer` | required | — |

Returns: `true, or false`, `reason`

```lua
SetVehicleNeonLightsColour(vehicle, 0, 229, 255)
SetVehicleNeonLightEnabled(vehicle, 0, true)
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.Compat.cs` (line 414) as `LuaResourceRuntime.Bootstrap`.

## SetVehiclePaint

```lua
SetVehiclePaint(id, primaryR, primaryG, primaryB, secondaryR, secondaryG, secondaryB)
```

`NETWORK`: Uses the network backend.

Repaints a vehicle: primary and secondary colour as six 0-255 channels.

The all-positional form of `Open77.vehicles.setPaint`. The six trailing arguments are the primary R, G, B then the secondary R, G, B, each clamped to 0-255. Answers a plain `true`/`false` with no reason: `false` covers an unknown vehicle and a missing `world.vehicles` permission alike. `Open77.vehicles.setPaint(id, { primary = ..., secondary = ... })` takes colours as tables and can change only one of the two.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `primaryR` | `integer` | required | — |
| `primaryG` | `integer` | required | — |
| `primaryB` | `integer` | required | — |
| `secondaryR` | `integer` | required | — |
| `secondaryG` | `integer` | required | — |
| `secondaryB` | `integer` | required | — |

```lua
-- Paint the gang's cars: deep red body, black trim.
for _, vehicle in ipairs(Open77.vehicles.all(0) or {}) do
    if vehicle.owned then
        SetVehiclePaint(vehicle.id, 160, 16, 24, 20, 20, 20)
    end
end
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 1059) as `SetVehiclePaint`.

## SetVehicleTransform

```lua
SetVehicleTransform(id, x, y, z, yaw)
```

`NETWORK`: Uses the network backend.

Teleports a vehicle to a position and heading.

The all-positional form of `Open77.vehicles.setTransform`. Five arguments, no options: the vehicle keeps its occupants, its damage and its paint. Answers a plain `true`/`false` with no reason -- `false` covers an unknown vehicle and a missing `world.vehicles` permission alike. This moves the vehicle, not the players' buckets: use `Open77.routingBuckets.setEntity` if it should also change bucket.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `z` | `number` | required | — |
| `yaw` | `number` | required | — |

```lua
-- /recall <id>: bring a stranded vehicle back to the garage door.
RegisterCommand("recall", function(source, args)
    local id = tonumber(args[1])
    if not id then return end
    if not SetVehicleTransform(id, -1442.2, 127.4, 18.0, 180.0) then
        Open77.chat.send(source, "No such vehicle, or not ours.")
    end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 1063) as `SetVehicleTransform`.

## SetVehicleWheelType

```lua
SetVehicleWheelType(vehicle, wheelType)
```

`NETWORK`: Uses the network backend.

Accepted for ported tuning menus; changes nothing.

Requires `world.vehicles`: it reads the car to check it exists. There are no wheel families in this game: every rim is a catalogue part, chosen with `SetVehicleMod` mod type 23 or 24 (or `Open77.vehicles.setWheels`). The call answers true for a car that exists, so a ported menu does not fail on its first line, and `false, reason` otherwise.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicle` | `vehicle id` | required | — |
| `wheelType` | `integer` | required | — |

Returns: `true, or false`, `reason`

```lua
SetVehicleWheelType(vehicle, 7) -- accepted, no effect
SetVehicleMod(vehicle, 23, 2)  -- the rim itself
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.Compat.cs` (line 425) as `LuaResourceRuntime.Bootstrap`.

## SetVehicleWindowTint

```lua
SetVehicleWindowTint(vehicle, tint)
```

`NETWORK`: Uses the network backend.

Sets the window tint by its FiveM index.

Requires `world.vehicles`; a call that changes the look also requires `world.vehicles.appearance`. The FiveM spelling over `Open77.vehicles` (see the native for the details), with its permissions and its `false, reason` refusals. The vehicle handle is the canonical vehicle id. FiveM's index maps onto the three Open77 levels: `-1`, `0` (NONE) and `4` (STOCK) are factory glass, `3` (LIGHTSMOKE) is level 1, `2` (DARKSMOKE) level 2, `1` (PURE_BLACK) and `5` (LIMO) level 3. `6` (GREEN) has no counterpart and is refused as `unsupported_property:windowTint` rather than drawn black; anything else is `invalid_tint`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicle` | `vehicle id` | required | — |
| `tint` | `integer` | required | — |

Returns: `true, or false`, `reason`

```lua
SetVehicleWindowTint(vehicle, 1) -- PURE_BLACK: the darkest level
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.Compat.cs` (line 436) as `LuaResourceRuntime.Bootstrap`.

## SpectatePlayer

```lua
SpectatePlayer(playerId, targetId|false, options?)
```

`NETWORK`: Uses the network backend.

Put one player behind another player's shoulder: ghost, hide, and a follow camera on the target, as one transaction that fails closed.

Put one player behind another player's shoulder: ghost, hide, and a follow camera on the target, as one transaction that fails closed. `targetId` false ends it and gives the body back, as do death, disconnect, reconnect, a bucket change and this resource stopping. `options`: `blendMs`, `distance`, `height`. See [Spectating](https://open2077.net/docs/api/server/cameras.md#spectating).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `value` | required | — |
| `targetId|false` | `value` | required | — |
| `options?` | `value` | required | — |

```lua
-- SpectatePlayer is the low-level form of Open77.players.spectate (documented alias);
-- the example shows the Open77.* spelling resources are told to prefer.
-- /spectate <id>, and /spectate alone to stop
RegisterCommand("spectate", function(source, args)
    local target = tonumber(args[1])
    local ok, reason = Open77.players.spectate(source, target, { distance = 5.0 })
    if not ok then Open77.chat.send(source, "Cannot spectate: " .. reason) end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 1124) as `SpectatePlayer`.

## StartResource

```lua
StartResource(name)
```

`NETWORK`: Uses the network backend.

FiveM-style alias for `Open77.resource.start`.

Requires `resources.control`. Queued, not immediate.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |

Returns: `true when queued, otherwise false`, `reason`

```lua
-- Start the event resource when an admin opens the event
RegisterCommand("event-open", function(source)
    local ok, reason = StartResource("summer_event")
    if not ok then Open77.chat.send(source, reason) end
end, true)
```

Registered in `LuaResourceRuntime.RuntimeControl.cs` (line 370) as `LuaResourceRuntime.Bootstrap`.

## StopResource

```lua
StopResource(name)
```

`NETWORK`: Uses the network backend.

FiveM-style alias for `Open77.resource.stop`.

Requires `resources.control`. Queued, not immediate.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |

Returns: `true when queued, otherwise false`, `reason`

```lua
-- Stop the event resource when the event closes
RegisterCommand("event-close", function(source)
    StopResource("summer_event")
end, true)
```

Registered in `LuaResourceRuntime.RuntimeControl.cs` (line 371) as `LuaResourceRuntime.Bootstrap`.

## StopSound

```lua
StopSound(id)
```

`NETWORK`: Uses the network backend.

StopSound: server-authoritative network package audio.

Requires audio.network; network=false is rejected. Server owns the timeline, handles and bucket/optional-target audience. Clients cannot request broadcasts directly. getState reports intended playback, not decoder acknowledgement. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `true, or failure, reason`

```lua
-- The concert is over: stop it for everyone in the bucket
StopSound(concertId)
```

Registered in `LuaResourceRuntime.cs` (line 9877) as `LuaAudioControl`.

## TeleportElevator

```lua
TeleportElevator(id, floor)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.elevators.teleport`.

The low-level form of `Open77.elevators.teleport` (bound to the same native). The two are the same function under two names; `Open77.elevators.teleport` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `floor` | `integer` | required | — |

```lua
-- TeleportElevator is the low-level form of Open77.elevators.teleport (bound to the same native);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Admin recovery for a lift stuck between floors
RegisterCommand("liftfix", function(source, args)
    Open77.elevators.teleport(tonumber(args[1]), 0)
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 1101) as `TeleportElevator`.

_This signature was inferred from the handler body._

## TriggerCancellableEvent

```lua
TriggerCancellableEvent(event, [...])
```

`NETWORK`: Uses the network backend.

Server only. Publishes a host-wide event handlers may veto, and returns a verdict handle.

Only events published this way can be cancelled; an ordinary TriggerEvent ignores CancelEvent(). Handlers run inline during the host drain, in resource-name order then registration order, so Wait() is not available inside one. Every handler still runs after a veto (FiveM semantics) and a later one reads the flag with WasEventCanceled(). The handle settles one tick later: verdict:await() answers true when a handler cancelled, verdict:canceled() answers nil plus status while pending, verdict:status() answers pending, settled or unknown. An unread verdict is forgotten 30 s after it settles. Same caps and reserved list as TriggerEvent.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `...` | `any` | optional | — |

Returns: `verdict handle, or nil, reason`

```lua
-- Let other resources veto a purchase before it happens
RegisterNetEvent("shop:buy", function(itemId)
    local buyer = source
    local verdict = TriggerCancellableEvent("shop:beforeBuy", buyer, itemId)
    if verdict and verdict:await() then return end -- somebody said no
    sell(buyer, itemId)
end)
```

Registered in `LuaResourceRuntime.Events.cs` (line 237) as `LuaResourceRuntime.Bootstrap`.

## TriggerClientEvent

```lua
TriggerClientEvent(event, playerId, [...])
```

`NETWORK`: Uses the network backend.

Sends an event to one player's client, or to every connected client with `-1`.

Requires `network.events`. `playerId` is an authenticated player id (an integer of 1 or more) or `-1` to broadcast; the arguments after it are JSON-marshalled and handed to the client's `RegisterNetEvent` handler of that name. Answers `true`, or `false, reason` for `permission_denied:network.events`, `network_unavailable` and a reserved name (`reserved_*_event`: the `open77:dash:`, `open77:abilities:`, `open77:reflex:`, `open77:cyberware:`, latent, screen and travel prefixes). Three shapes are not refusals but host exceptions that stop the resource with `runtime_error`: a target of `0` (the `source` of a command typed at the dedicated console) or below `-1`, a name empty or over 128 characters or more than 32 arguments (`Invalid client event envelope.`), and a payload over 48 KiB (`Network event payload exceeds 48 KiB.`). Guard `source` before replying to it. `Open77.net.emitClient` is the namespaced spelling.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `playerId` | `integer >= 1, or -1 for every client` | required | — |
| `...` | `JSON-serialisable values, at most 32` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- Tell one player their balance, and everybody that the heist started
RegisterCommand("balance", function(source)
    if tonumber(source) < 1 then print("balance: run in game, not at the console"); return end
    TriggerClientEvent("bank:balance", source, getBalance(source))
end, false)
TriggerClientEvent("heist:started", -1, { vault = "arasaka" })
```

Registered in `LuaResourceRuntime.cs` (line 991) as `TriggerClientEvent`.

## TriggerEvent

```lua
TriggerEvent(event, [...])
```

`SHARED`: Available without a live game instance.

Publishes an event host-wide: every running resource that handles the name receives it.

No permission, to publish or to receive. Delivery is queued and never re-entrant: publishing appends to the host's queue, the host drains it at the next tick boundary and pushes the event into each VM in resource-name order, so one resource's error cannot abort the fan-out. Arguments cross by value through the export marshaller -- 32 values, 4,096 nodes, depth 16, 48 KiB -- and functions, userdata, cyclic tables and non-finite numbers are refused. `source` is not propagated, so a publisher that wants receivers to know which player caused the event passes the id as an argument. Platform event names are refused with reserved_event. The client function of this name is local to its own VM; this one is the bus. Failure tokens: invalid_event_name, reserved_event, resource_stopping, resource_preparing, event_argument_limit, event_payload_not_serializable, event_queue_limit.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `...` | `any` | optional | — |

Returns: `true, or false, reason`

```lua
-- Announce a robbery to every resource on the server; source is passed by hand
TriggerEvent("bank:robbed", source, amount)
```

Registered in `LuaResourceRuntime.Events.cs` (line 186) as `LuaResourceRuntime.Bootstrap`.

## TriggerLatentClientEvent

```lua
TriggerLatentClientEvent(name, target, bytesPerSecond, [...])
```

`NETWORK`: Uses the network backend.

Sends a client event too large for one net event, paced at a byte rate (FiveM spelling).

The same call as `TriggerClientEvent` with FiveM's bytes-per-second argument in third place, for a payload the 48 KiB event envelope cannot carry: up to 4 MiB, cut into 40 KiB frames on a reserved name, paced at the requested rate (clamped to 1 KiB/s .. 2.5 MiB/s, and never more than 64 frames per second for all of a resource's streams together), reassembled by the client host and delivered to `RegisterNetEvent` handlers under the ORIGINAL name -- the receiving resource cannot tell it was latent. `target` is one player id or `-1` for everybody connected. Requires `network.events`. Answers the stream id, or `nil, reason`: `permission_denied:network.events`, `invalid_event_name`, `reserved_*_event`, `invalid_player_id`, `player_not_found`, `invalid_rate`, `event_argument_limit`, `latent_stream_limit` (16 in flight), `latent_payload_not_serializable`, `latent_payload_too_large` (over 4 MiB, nothing sent). Read progress with `Open77.net.latentStatus(id)`; stop it with `cancelLatent(id)`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `target` | `integer player id, or -1 for everybody` | required | — |
| `bytesPerSecond` | `number` | required | — |
| `...` | `event arguments, the rules of TriggerClientEvent` | optional | — |

Returns: `stream id string, or nil`, `reason on failure`

```lua
-- Push the whole item catalogue to a joining player without stalling their session
local id, reason = TriggerLatentClientEvent('shop:catalogue', playerId, 256 * 1024, catalogue, revision)
if not id then print('catalogue not sent: ' .. tostring(reason)) end
-- client: RegisterNetEvent('shop:catalogue', function(catalogue, revision) ... end) -- unchanged
```

Registered in `LuaResourceRuntime.Latent.cs` (line 394) as `LuaResourceRuntime.Bootstrap`.

## TriggerLocalEvent

```lua
TriggerLocalEvent(event, [...])
```

`NETWORK`: Uses the network backend.

Server only. Fires an event in this resource's VM and nowhere else.

The behaviour server TriggerEvent had before the host-wide bus: no marshalling, no reserved-name filter, no host round trip. Use it for a resource's own internal signalling, and wherever a fixture must simulate a host lifecycle event inside its own VM.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `...` | `any` | optional | — |

Returns: `true`

```lua
-- Internal signalling that never leaves this resource
TriggerLocalEvent("shop:restocked", "ammo", 200)
AddEventHandler("shop:restocked", function(item, count)
    print(item, "restocked to", count)
end)
```

Registered in `LuaResourceRuntime.cs` (line 6364) as `LuaResourceRuntime.Bootstrap`.

## TriggerVehicleHorn

```lua
TriggerVehicleHorn(id, duration)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.vehicles.triggerHorn`.

The low-level form of `Open77.vehicles.triggerHorn` (wrapped by). The two are the same function under two names; `Open77.vehicles.triggerHorn` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `duration` | `integer` | required | — |

```lua
-- TriggerVehicleHorn is the low-level form of Open77.vehicles.triggerHorn (wrapped by);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Honk when the player's rental is unlocked remotely
Open77.vehicles.unlock(vehicleId)
Open77.vehicles.triggerHorn(vehicleId, 200)
```

Registered in `LuaResourceRuntime.cs` (line 1062) as `TriggerVehicleHorn`.

_This signature was inferred from the handler body._

## UpdateEffect

```lua
UpdateEffect(id, [x], [y], [z], [qx], [qy], [qz], [qw], [bucket], [visible], [radius], [hysteresis], [ttl])
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.effects.update`.

The low-level form of `Open77.effects.update` (wrapped by). The two are the same function under two names; `Open77.effects.update` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `x` | `number` | optional | — |
| `y` | `number` | optional | — |
| `z` | `number` | optional | — |
| `qx` | `number` | optional | — |
| `qy` | `number` | optional | — |
| `qz` | `number` | optional | — |
| `qw` | `number` | optional | — |
| `bucket` | `integer` | optional | — |
| `visible` | `boolean` | optional | — |
| `radius` | `number` | optional | — |
| `hysteresis` | `number` | optional | — |
| `ttl` | `integer` | optional | — |

```lua
-- UpdateEffect is the low-level form of Open77.effects.update (wrapped by);
-- the example shows the Open77.* spelling resources are told to prefer.
-- The fire moves with the burning car
Open77.effects.update(fireId, { position = Open77.vehicles.getPosition(carId) })
```

Registered in `LuaResourceRuntime.cs` (line 1045) as `UpdateEffect`.

_This signature was inferred from the handler body._

## UpdateLootDrop

```lua
UpdateLootDrop(id, quantity, x, y, z, bucket, pickupRadius, label, model)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.loot.update`.

The low-level form of `Open77.loot.update` (wrapped by). The two are the same function under two names; `Open77.loot.update` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `quantity` | `integer` | required | — |
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `z` | `number` | required | — |
| `bucket` | `integer` | required | — |
| `pickupRadius` | `number` | required | — |
| `label` | `string` | required | — |
| `model` | `string` | required | — |

```lua
-- UpdateLootDrop is the low-level form of Open77.loot.update (wrapped by);
-- the example shows the Open77.* spelling resources are told to prefer.
-- The wind blows the crate two metres; quantity and label stay
Open77.loot.update(lootId, { position = { x = -1444.2, y = 127.4, z = 18.0 } })
```

Registered in `LuaResourceRuntime.cs` (line 1053) as `UpdateLootDrop`.

_This signature was inferred from the handler body._

## UpdateNpc

```lua
UpdateNpc(id, appearance, loadoutJson, aiMode, damagePolicy, health, maxHealth, ragdoll, behaviorJson)
```

`NETWORK`: Uses the network backend.

Low-level alias of `Open77.npcs.update`.

The low-level form of `Open77.npcs.update` (wrapped by). The two are the same function under two names; `Open77.npcs.update` is the spelling resources are told to prefer, and its card carries the arguments, the return values and the failure reasons.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `appearance` | `string` | required | — |
| `loadoutJson` | `string` | required | — |
| `aiMode` | `integer` | required | — |
| `damagePolicy` | `integer` | required | — |
| `health` | `number` | required | — |
| `maxHealth` | `number` | required | — |
| `ragdoll` | `integer` | required | — |
| `behaviorJson` | `string` | required | — |

```lua
-- UpdateNpc is the low-level form of Open77.npcs.update (wrapped by);
-- the example shows the Open77.* spelling resources are told to prefer.
-- Change several things in one replicated patch
Open77.npcs.update(npcId, {
    health = 100,
    damagePolicy = "mortal",
    behavior = { combatEnabled = true },
})
```

Registered in `LuaResourceRuntime.cs` (line 1077) as `UpdateNpc`.

_This signature was inferred from the handler body._

## UpdateProp

```lua
UpdateProp(id, [appearance], [physics], [collision], [visible], [bucket], [light], [radius], [hysteresis], [ttl])
```

`NETWORK`: Uses the network backend.

Patches a prop in place: appearance, physics, visibility, bucket, light or lifetime.

The all-positional form of `Open77.props.update`. Every argument after the id is optional and `nil` means leave it alone, so this is a patch and not a replacement. Answers `false, reason` -- `not_found` for an id this resource does not own, `invalid_argument` for a malformed value. Requires `world.props`. The transform is not here: that is `SetPropTransform`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `appearance` | `string` | optional | — |
| `physics` | `string` | optional | — |
| `collision` | `boolean` | optional | — |
| `visible` | `boolean` | optional | — |
| `bucket` | `integer` | optional | — |
| `light` | `string` | optional | — |
| `radius` | `number` | optional | — |
| `hysteresis` | `number` | optional | — |
| `ttl` | `integer` | optional | — |

```lua
-- Blackout: hide the neon sign, then bring it back lit twenty seconds later.
local ok, reason = UpdateProp(signId, nil, nil, nil, false)
if not ok then return print("sign refused the patch: " .. tostring(reason)) end
SetTimeout(20000, function()
    UpdateProp(signId, nil, nil, nil, true, nil, "neon.pink")
end)
```

Registered in `LuaResourceRuntime.cs` (line 1036) as `UpdateProp`.

## UpdateVehicleState

```lua
UpdateVehicleState(id, health, flags, primaryR, primaryG, primaryB, secondaryR, secondaryG, secondaryB, doors, windows, tires, bodyDamage, brokenGlass, brokenLights, detachedParts)
```

`NETWORK`: Uses the network backend.

Writes a vehicle's whole replicated state at once: health, flags, paint, damage.

The all-positional form of `Open77.vehicles.update`, and unlike that one it is a **replacement, not a patch**: all sixteen arguments are written, so read the vehicle first and pass back whatever you are not changing. `bodyDamage` must be a table of exactly 30 numbers between 0 and 1 -- anything else raises a Lua error instead of answering `false`, which is the one refusal here that is not a return value. `doors`, `windows` and `tires` are bitmasks. Answers a plain `true`/`false`. Requires `world.vehicles`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `health` | `number` | required | — |
| `flags` | `integer` | required | — |
| `primaryR` | `integer` | required | — |
| `primaryG` | `integer` | required | — |
| `primaryB` | `integer` | required | — |
| `secondaryR` | `integer` | required | — |
| `secondaryG` | `integer` | required | — |
| `secondaryB` | `integer` | required | — |
| `doors` | `integer` | required | — |
| `windows` | `integer` | required | — |
| `tires` | `integer` | required | — |
| `bodyDamage` | `table` | required | — |
| `brokenGlass` | `integer` | required | — |
| `brokenLights` | `integer` | required | — |
| `detachedParts` | `integer` | required | — |

```lua
-- Repair a wreck: full health, clean body, nothing broken. Read first,
-- because every one of the sixteen arguments is written, not patched.
local v = GetVehicle(vehicleId)
if not v then return print("no such vehicle") end
local clean = {}
for i = 1, 30 do clean[i] = 0.0 end
local ok = UpdateVehicleState(vehicleId, 1000.0, v.flags,
    v.primaryR, v.primaryG, v.primaryB, v.secondaryR, v.secondaryG, v.secondaryB,
    0, 0, 0,        -- doors, windows, tires: no damage bits
    clean, 0, 0, 0) -- bodyDamage (30 values), glass, lights, detached parts
print(ok and "repaired" or "refused: unknown vehicle, or not ours")
```

Registered in `LuaResourceRuntime.cs` (line 1058) as `UpdateVehicleState`.

## vec

```lua
vec(...)
```

`SHARED`: Available without a live game instance.

Builds a vector of the size the arguments imply.

Available in both runtimes. With two, three or four numbers it builds a vector2, vector3 or vector4. With a single table it picks the size from the axes actually present: w means vector4, else z means vector3, else y means vector2 -- which makes vec(position) the short way to wrap a position returned by any existing API. A quaternion is refused, because vec makes vectors. Identical on both runtimes by construction: one file, scripting/lua/open77_vector.lua, is embedded by the client build and by the server assembly, so the semantics cannot drift.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `...` | `number|table` | required | — |

Returns: `vector2, vector3 or vector4, or nil, reason`

```lua
-- Wrap a position any API returned, then measure a distance with #
local me = vec(Open77.players.position(source))
if #(me - vec(-1442.2, 127.4, 18.0)) < 10.0 then print("at the garage") end
```

Registered in `open77_vector.lua` (line 499) as `LuaPrelude`.

## vector2

```lua
vector2(x, y)
```

`SHARED`: Available without a live game instance.

Builds a 2D vector: a plain { x, y } table with vector behaviour.

Available in both runtimes. Same operators, swizzles and methods as vector3, minus cross, which answers nil, "cross_requires_vector3". Takes two finite numbers, or one vector or { x =, y = } table to copy. Identical on both runtimes by construction: one file, scripting/lua/open77_vector.lua, is embedded by the client build and by the server assembly, so the semantics cannot drift.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `x` | `number` | required | — |
| `y` | `number` | required | — |

Returns: `vector2, or nil, reason`

```lua
-- Distance on the map, ignoring height
local a = vector2(-1442.2, 127.4)
local b = vector2(-1448.2, 96.1)
print(("%.1f m apart on the map"):format(#(a - b)))
```

Registered in `open77_vector.lua` (line 495) as `LuaPrelude`.

## vector3

```lua
vector3(x, y, z)
```

`SHARED`: Available without a live game instance.

Builds a 3D vector: a plain { x, y, z } table with vector behaviour.

Available in both runtimes, from the shared prelude scripting/lua/open77_vector.lua. Takes three finite numbers, or one vector or { x =, y =, z = } table to copy. The result IS a table with real x, y and z fields, so every API that accepts a position table accepts it unchanged and json.encode emits {"x":..,"y":..,"z":..}. Supports + - * / (by a vector, a position table or a number), unary minus, # for magnitude, == (exact, same kind only), tostring/.. as vector3(1.0, 2.0, 3.0), the swizzles .xy/.xz/.yz/.xyz/.zyx, and the methods length, lengthSquared, distance, distanceSquared, normalize (zero stays zero), dot, cross, lerp (unclamped), unpack and table. Fields are read-only except the components themselves, which Lua 5.4 cannot intercept. Operators raise; this constructor returns nil, reason. See the Vectors and quaternions guide. Identical on both runtimes by construction: one file, scripting/lua/open77_vector.lua, is embedded by the client build and by the server assembly, so the semantics cannot drift.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `z` | `number` | required | — |

Returns: `vector3, or nil, reason`

```lua
-- Is the player within 10 m of the garage?
local garage = vector3(-1442.2, 127.4, 18.0)
local me = vector3(Open77.players.position(source))
if #(me - garage) < 10.0 then print("at the garage") end
```

Registered in `open77_vector.lua` (line 496) as `LuaPrelude`.

## vector4

```lua
vector4(x, y, z, w)
```

`SHARED`: Available without a live game instance.

Builds a 4D vector: a plain { x, y, z, w } table with vector behaviour.

Available in both runtimes. Same operators and methods as vector3, minus cross. Its swizzles reach w, so v.xyz narrows a vector4 to a vector3. Takes four finite numbers, or one vector or { x =, y =, z =, w = } table to copy. Identical on both runtimes by construction: one file, scripting/lua/open77_vector.lua, is embedded by the client build and by the server assembly, so the semantics cannot drift.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `z` | `number` | required | — |
| `w` | `number` | required | — |

Returns: `vector4, or nil, reason`

```lua
-- Carry a spawn point and its heading in one value
local spawn = vector4(-1442.2, 127.4, 18.0, 90.0)
Open77.players.respawn(playerId, { position = spawn.xyz, heading = spawn.w })
```

Registered in `open77_vector.lua` (line 497) as `LuaPrelude`.

## VehicleDrivingCommand

```lua
VehicleDrivingCommand(operation, vehicleId, options?)
```

`NETWORK`: Uses the network backend.

`world.vehicles`;

`world.vehicles`; driving-state table or nil/reason (state may be nil when absent). Prefer [vehicle AI methods](https://open2077.net/docs/api/server/vehicle-ai.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `operation` | `value` | required | — |
| `vehicleId` | `value` | required | — |
| `options?` | `value` | required | — |

```lua
-- Send a taxi to the club, then read the AI driving state back.
local state, reason = VehicleDrivingCommand("driveTo", cabId, {
    position = { x = -1442.2, y = 127.4, z = 18.0 },
    speed = 12.0,
})
if not state then return print("driving refused: " .. tostring(reason)) end
-- The "state" operation reads without commanding anything; it answers nil
-- (with no reason) when the vehicle simply has no driving state yet.
print("driving state: " .. tostring(VehicleDrivingCommand("state", cabId)))
```

Registered in `LuaResourceRuntime.cs` (line 1068) as `VehicleDriving`.

## Wait

```lua
Wait(milliseconds)
```

`SHARED`: Available without a live game instance.

Suspends the current managed coroutine.

Usable inside a CreateThread, a command handler, an event handler, a callback handler or an export body -- anywhere the scheduler owns the coroutine. Accepts 0 to 86,400,000 ms and raises `invalid wait` outside that range. Cancellable event handlers are the one exception: they run inline during the host drain, where a yield would outlive the verdict, so Wait is not available there.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `milliseconds` | `integer` | required | — |

```lua
-- A countdown in chat, one line per second
CreateThread(function()
    for n = 5, 1, -1 do
        Open77.chat.broadcast(("Starting in %d..."):format(n))
        Wait(1000)
    end
    Open77.chat.broadcast("Go!")
end)
```

Registered in `LuaResourceRuntime.cs` (line 6282) as `LuaResourceRuntime.Bootstrap`.

## WasEventCanceled

```lua
WasEventCanceled()
```

`NETWORK`: Uses the network backend.

Server only. Whether the cancellable event being dispatched has already been vetoed.

True only inside a cancellable dispatch some handler has cancelled, in this resource or an earlier one. False everywhere else, including inside an ordinary event handler and at top level.

Returns: `boolean`

```lua
-- Log which messages another resource's filter already rejected
AddEventHandler("chatMessage", function(source, name, text)
    if WasEventCanceled() then
        print(("filtered message from %s: %s"):format(name, text))
    end
end)
```

Registered in `LuaResourceRuntime.Events.cs` (line 180) as `LuaResourceRuntime.Bootstrap`.
