# Globals (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/globals.md) · [Page web](https://open2077.net/docs/api/client/globals)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

81 functions.

## AddEventHandler

```lua
AddEventHandler(event, handler)
```

`SHARED`: Available without a live game instance.

Listens for a local event.

Registers a callback on the current resource's local event bus. The handler is owned by the current resource generation and is discarded automatically when that generation stops or reloads; it never receives network traffic unless the event was separately registered as a network event.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `handler` | `function` | required | — |

Returns: `handler id`

```lua
-- Run setup when this resource starts on the client
AddEventHandler("onClientResourceStart", function(name)
    if name ~= GetCurrentResourceName() then return end
    Open77.hud.notify("Welcome to the city")
end)
```

Registered in `ResourceHost.cpp` (line 24468) as `LuaAddEventHandler`.

## AddStateBagChangeHandler

```lua
AddStateBagChangeHandler(key, bagName, handler)
```

`CLIENT`: Client-only surface.

FiveM compatibility: watches a state-bag key with FiveM's argument list.

No permission. `bagName` is nil for every bag, `global`, `player:<id>`, `<kind>:<id>`, or `entity:<id>` -- the last matches any entity kind with that id. `key` may be nil for every key. The handler is called as `fn(bagName, key, value, reserved, replicated)`, where `reserved` is always nil and `replicated` always true, so a ported signature lines up. It shares the subscription index with `Open77.state.onChange`, so only this resource is touched.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string|nil` | required | — |
| `bagName` | `string|nil` | required | — |
| `handler` | `function` | required | — |

Returns: `subscription id, or nil`, `reason: invalid_state_selector, invalid_handler_or_watcher_limit`

```lua
-- A FiveM-style watcher: react when the server changes the weather key
AddStateBagChangeHandler("weather", "global", function(bagName, key, value)
    Open77.hud.notify("Weather: " .. tostring(value))
end)
```

Registered in `ResourceHost.cpp` (line 24480) as `LuaAddStateBagChangeHandler`.

## AllowAim

```lua
AllowAim(enabled)
```

`GAME`: Requires a live game instance.

Allow or block a local-player action.

Requires players.controls and a strict boolean. Freeze true acquires a block; Allow false acquires a block. The inverse releases only this resource VM's block, never another owner or vanilla restriction. Automatically released on resource stop, death and session teardown. Does not freeze vehicles, moving platforms or server teleports. See player-utilities.md for each native restriction and limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
-- No aiming inside the safe zone; released again when the player leaves it
AddEventHandler("safezone:entered", function() AllowAim(false) end)
AddEventHandler("safezone:left", function() AllowAim(true) end)
```

Registered in `ResourceHost.cpp` (line 12381) as `LuaPlayerControl`.

## AllowCrouch

```lua
AllowCrouch(enabled)
```

`GAME`: Requires a live game instance.

Allow or block a local-player action.

Requires players.controls and a strict boolean. Freeze true acquires a block; Allow false acquires a block. The inverse releases only this resource VM's block, never another owner or vanilla restriction. Automatically released on resource stop, death and session teardown. Does not freeze vehicles, moving platforms or server teleports. See player-utilities.md for each native restriction and limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
-- A cutscene where the player must stay standing
AllowCrouch(false)
SetTimeout(10000, function() AllowCrouch(true) end)
```

Registered in `ResourceHost.cpp` (line 12383) as `LuaPlayerControl`.

## AllowDodge

```lua
AllowDodge(enabled)
```

`GAME`: Requires a live game instance.

Allow or block a local-player action.

Requires players.controls and a strict boolean. Freeze true acquires a block; Allow false acquires a block. The inverse releases only this resource VM's block, never another owner or vanilla restriction. Automatically released on resource stop, death and session teardown. Does not freeze vehicles, moving platforms or server teleports. See player-utilities.md for each native restriction and limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
-- The fighting tournament forbids dodging in the "no dodge" round
AllowDodge(false)
```

Registered in `ResourceHost.cpp` (line 12384) as `LuaPlayerControl`.

## AllowEncumbrance

```lua
AllowEncumbrance(allow)
```

`GAME`: Requires a live game instance.

Enables or suppresses native overweight penalties.

Requires `players.controls` and a strict boolean. `false` acquires resource-owned suppression of vanilla overweight warnings and movement penalties, removing an existing encumbered effect. `true` releases only this resource's claim; `resetControls`, resource stop and player/session teardown also release it. Inventory contents, measured weight and carry capacity are unchanged; server inventory rules remain separate. Update `open77_equipment` to 0.2.1 for suppression by default, or set its `Open77EquipmentConfig.nativeEncumbrance = true` to release that package's claim. See [native weight penalties](../../01%20Documentation/player-utilities.md#native-weight-penalties).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `allow` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
assert(Open77.players.allowEncumbrance(false))
-- Release this resource's suppression when appropriate:
assert(Open77.players.allowEncumbrance(true))
```

Registered in `ResourceHost.cpp` (line 13415) as `LuaPlayerControl`.

## AllowInteraction

```lua
AllowInteraction(enabled)
```

`GAME`: Requires a live game instance.

Allow or block a local-player action.

Requires players.controls and a strict boolean. Freeze true acquires a block; Allow false acquires a block. The inverse releases only this resource VM's block, never another owner or vanilla restriction. Automatically released on resource stop, death and session teardown. Does not freeze vehicles, moving platforms or server teleports. See player-utilities.md for each native restriction and limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
-- Freeze the world's prompts while a dialogue is on screen
AllowInteraction(false)
AddEventHandler("dialogue:closed", function() AllowInteraction(true) end)
```

Registered in `ResourceHost.cpp` (line 12385) as `LuaPlayerControl`.

## AllowJump

```lua
AllowJump(enabled)
```

`GAME`: Requires a live game instance.

Allow or block a local-player action.

Requires players.controls and a strict boolean. Freeze true acquires a block; Allow false acquires a block. The inverse releases only this resource VM's block, never another owner or vanilla restriction. Automatically released on resource stop, death and session teardown. Does not freeze vehicles, moving platforms or server teleports. See player-utilities.md for each native restriction and limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
-- No bunny-hopping across the racing track
AddEventHandler("race:started", function() AllowJump(false) end)
AddEventHandler("race:finished", function() AllowJump(true) end)
```

Registered in `ResourceHost.cpp` (line 12381) as `LuaPlayerControl`.

## AllowRunning

```lua
AllowRunning(enabled)
```

`GAME`: Requires a live game instance.

Allow or block a local-player action.

Requires players.controls and a strict boolean. Freeze true acquires a block; Allow false acquires a block. The inverse releases only this resource VM's block, never another owner or vanilla restriction. Automatically released on resource stop, death and session teardown. Does not freeze vehicles, moving platforms or server teleports. See player-utilities.md for each native restriction and limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
-- A tired player can only walk for 20 seconds
AllowRunning(false)
SetTimeout(20000, function() AllowRunning(true) end)
```

Registered in `ResourceHost.cpp` (line 12382) as `LuaPlayerControl`.

## AllowShoot

```lua
AllowShoot(enabled)
```

`GAME`: Requires a live game instance.

Allow or block a local-player action.

Requires players.controls and a strict boolean. Freeze true acquires a block; Allow false acquires a block. The inverse releases only this resource VM's block, never another owner or vanilla restriction. Automatically released on resource stop, death and session teardown. Does not freeze vehicles, moving platforms or server teleports. See player-utilities.md for each native restriction and limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
-- Weapons stay out but cannot fire in the lobby
AllowShoot(false)
```

Registered in `ResourceHost.cpp` (line 12382) as `LuaPlayerControl`.

## AllowWeapons

```lua
AllowWeapons(enabled)
```

`GAME`: Requires a live game instance.

Allow or block a local-player action.

Requires players.controls and a strict boolean. Freeze true acquires a block; Allow false acquires a block. The inverse releases only this resource VM's block, never another owner or vanilla restriction. Automatically released on resource stop, death and session teardown. Does not freeze vehicles, moving platforms or server teleports. See player-utilities.md for each native restriction and limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
-- The hospital is a no-weapon zone
AddEventHandler("hospital:entered", function() AllowWeapons(false) end)
AddEventHandler("hospital:left", function() AllowWeapons(true) end)
```

Registered in `ResourceHost.cpp` (line 12385) as `LuaPlayerControl`.

## ClearTick

```lua
ClearTick(id)
```

`SHARED`: Available without a live game instance.

Cancels a tick created by `SetTick`.

Takes effect on the next pass: the tick body never runs again after this returns. Tick ids and timer ids come from one per-resource counter and never collide, so this is the cancellation `ClearTimeout` performs; both names exist because FiveM has both.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `boolean`

```lua
-- Draw a marker every frame until the player reaches the point
local tick
tick = SetTick(function()
    if #(vector3(Open77.character.position()) - target) < 2.0 then
        ClearTick(tick)
        Open77.hud.notify("You made it")
    end
end)
```

Registered in `ResourceHost.cpp` (line 24531) as `LuaClearTick`.

## ClearTimeout

```lua
ClearTimeout(id)
```

`SHARED`: Available without a live game instance.

Cancels a timer created by `SetTimeout`.

Cancels a pending timeout owned by the current resource generation. It returns `false` when the id is unknown or the callback has already begun; cancelling cannot interrupt a callback that is currently running.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `boolean`

```lua
-- The player pressed the key again before the delay ran out: cancel the pending action
local pending = SetTimeout(3000, function() openDoor() end)
RegisterKeyMapping("door_cancel", "Cancel door", "X", function()
    ClearTimeout(pending)
end)
```

Registered in `ResourceHost.cpp` (line 24467) as `LuaClearTimeout`.

## CreateThread

```lua
CreateThread(body)
```

`SHARED`: Available without a live game instance.

Starts a coroutine managed by the host.

Its body may call `Wait`, which an ordinary function cannot.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `body` | `function` | required | — |

```lua
-- Show a reminder every 30 seconds until the delivery is done
CreateThread(function()
    while not delivered do
        Open77.hud.notify("Deliver the package to Kabuki")
        Wait(30000)
    end
end)
```

Registered in `ResourceHost.cpp` (line 24464) as `LuaCreateThread`.

## DeleteResourceKvp

```lua
DeleteResourceKvp(key)
```

`SHARED`: Available without a live game instance.

FiveM-style alias for `Open77.kvp.delete`.

FiveM-familiar alias for `Open77.kvp.delete`. It atomically removes only the named key owned by the calling resource and reports whether that key existed.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string` | required | — |

Returns: `boolean`, `reason`

```lua
-- Forget the saved HUD position
DeleteResourceKvp("hud_position")
```

Registered in `ResourceHost.cpp` (line 24509) as `LuaKvpDelete`.

## DestroySound

```lua
DestroySound(id)
```

`GAME`: Requires a live game instance.

DestroySound: local package audio.

Requires audio.play; network=true is rejected. MP3/PCM WAV decode asynchronously off the game thread. getState reports actual decoder state and cached PCM reuse. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `true, or failure, reason`

```lua
-- Free the engine sound when the player gets out of the car
DestroySound(engineLoop)
engineLoop = nil
```

Registered in `ResourceHost.cpp` (line 12183) as `LuaAudioControl`.

## DoesEntityExist

```lua
DoesEntityExist(kind, id)
```

`GAME`: Requires a live game instance.

Whether an id is live in the registry that owns that kind.

FiveM has one opaque handle space; Open77 has one registry per kind, so the kind is required: vehicle, npc, elevator, prop, effect, player (loot is server-only). The one-argument FiveM form cannot be answered and returns false, kind_required. Each kind answers under its own registry's permissions, so a resource that may not read that kind gets false -- call the namespace function directly to tell denied from absent. Other reasons: id_required, invalid_entity_id, unknown_entity_kind, kind_unavailable_on_client.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `kind` | `string` | required | — |
| `id` | `integer` | required | — |

Returns: `boolean`, `reason for the refusal`

```lua
-- The blip points at a vehicle; drop the blip if the car is gone
if not DoesEntityExist("vehicle", trackedVehicleId) then
    Open77.blips.remove(blipId)
end
```

Registered in `ResourceHost.cpp` (line 24536) as `LuaDoesEntityExist`.

## Entity

```lua
Entity(id)
```

`CLIENT`: Client-only surface.

FiveM compatibility: a table carrying one entity's state bag.

No permission. FiveM has one entity namespace and Open77 has three registries, so the id is probed against the mirror as vehicle, then npc, then prop, and falls back to vehicle. `Open77.state.entity(kind, id)` is always unambiguous and is what a new resource should use.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer|string` | required | — |

Returns: `table { state }`

```lua
-- FiveM-style read of a vehicle's state bag
local owner = Entity(vehicleId).state.owner
if owner == Open77.players.localId() then print("this is my car") end
```

Registered in `ResourceHost.cpp` (line 24479) as `LuaCompatEntity`.

## ExecuteCommand

```lua
ExecuteCommand(line)
```

`CLIENT`: Client-only surface.

FiveM-style alias for `Open77.runtime.executeCommand`.

Resolves against the client registry only and reports whether it matched; it never forwards to the server by itself. The chat composer uses that answer to decide whether to spend a net event.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `line` | `string` | required | — |

Returns: `boolean`, `reason`

```lua
-- Run a local command from a key binding
RegisterKeyMapping("toggle_hud", "Toggle HUD", "H", function()
    ExecuteCommand("hud toggle")
end)
```

Registered in `ResourceHost.cpp` (line 24516) as `LuaExecuteCommand`.

## exports

```lua
exports(name, body)
```

`SHARED`: Available without a live game instance.

Declares a function callable by other resources, and proxies calls to theirs.

Publishes a named function from the current resource so another resource can call it through `Open77.exports.call`. The registration is tied to the current generation, so stale functions cannot survive a stop or hot reload.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `body` | `function` | required | — |

Returns: `boolean`

```lua
-- Publish a function other client resources can call
exports("isInSafeZone", function()
    return inSafeZone
end)
```

Registered in `ResourceHost.cpp` (line 24502) as `LuaExportsCall`.

## FreezePosition

```lua
FreezePosition(enabled)
```

`GAME`: Requires a live game instance.

Request or release a native local-player freeze.

Requires players.controls and a strict boolean. Freeze true acquires a block; Allow false acquires a block. The inverse releases only this resource VM's block, never another owner or vanilla restriction. Automatically released on resource stop, death and session teardown. Does not freeze vehicles, moving platforms or server teleports. See player-utilities.md for each native restriction and limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
-- Hold the player still while the shop menu is open
FreezePosition(true)
AddEventHandler("shop:closed", function() FreezePosition(false) end)
```

Registered in `ResourceHost.cpp` (line 12380) as `LuaPlayerControl`.

## FreezeRotation

```lua
FreezeRotation(enabled)
```

`GAME`: Requires a live game instance.

Request or release a native local-player freeze.

Requires players.controls and a strict boolean. Freeze true acquires a block; Allow false acquires a block. The inverse releases only this resource VM's block, never another owner or vanilla restriction. Automatically released on resource stop, death and session teardown. Does not freeze vehicles, moving platforms or server teleports. See player-utilities.md for each native restriction and limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
-- Lock the camera direction for a fixed-angle photo
FreezeRotation(true)
SetTimeout(5000, function() FreezeRotation(false) end)
```

Registered in `ResourceHost.cpp` (line 12380) as `LuaPlayerControl`.

## GetCurrentResourceName

```lua
GetCurrentResourceName()
```

`SHARED`: Available without a live game instance.

Name of the current resource.

Returns the immutable manifest name of the resource whose Lua VM is executing. Use it for diagnostics and namespacing, not as a mutable display label.

Returns: `string`

```lua
-- Only react to this resource's own start
AddEventHandler("onClientResourceStart", function(name)
    if name == GetCurrentResourceName() then print("started") end
end)
```

Registered in `ResourceHost.cpp` (line 24503) as `LuaResourceName`.

## GetGameTimer

```lua
GetGameTimer()
```

`SHARED`: Available without a live game instance.

Monotonic milliseconds since an arbitrary epoch.

The FiveM name for the clock behind `Open77.time.monotonic()`, which reports seconds. The epoch is arbitrary and is not wall clock: only the difference between two readings means anything, and the client and the server share no origin. The server has the same global. See the FiveM compatibility guide.

Returns: `integer milliseconds`

```lua
-- A cooldown: ignore the key for 5 seconds after use
local lastUse = 0
RegisterKeyMapping("flashlight", "Flashlight", "L", function()
    if GetGameTimer() - lastUse < 5000 then return end
    lastUse = GetGameTimer()
    toggleFlashlight()
end)
```

Registered in `ResourceHost.cpp` (line 24529) as `LuaGetGameTimer`.

## GetHashKey

```lua
GetHashKey(text)
```

`SHARED`: Available without a live game instance.

TweakDBID of a record name -- not a GTA hash.

CRC-32/ISO-HDLC of the name in the low 32 bits, the name length in byte 4, which is what REDengine computes. This is deliberately NOT the Jenkins one-at-a-time hash FiveM returns: Open77 records are TweakDB strings, so a hash constant copied out of a FiveM resource names a GTA model and matches nothing here. The server computes the identical value.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `text` | `string` | required | — |

Returns: `integer TweakDBID`

```lua
-- Compare a record name with the hash a server event carried
if GetHashKey("Vehicle.v_sport2_quadra_turbo_r") == recordHash then
    print("that is a Quadra")
end
```

Registered in `ResourceHost.cpp` (line 24534) as `LuaGetHashKey`.

## GetInvokingResource

```lua
GetInvokingResource()
```

`SHARED`: Available without a live game instance.

Real caller of the currently executing export.

Returns nil outside an export callback. Use this value for service ownership instead of accepting a forgeable owner argument.

Returns: `resource name, or nil`

```lua
-- A service export trusts the caller's real name, never an argument
exports("registerHud", function(definition)
    local owner = GetInvokingResource()
    huds[owner] = definition
    return true
end)
```

Registered in `ResourceHost.cpp` (line 24504) as `LuaInvokingResource`.

## GetInvokingResourceGeneration

```lua
GetInvokingResourceGeneration()
```

`SHARED`: Available without a live game instance.

Generation of the resource invoking the current export.

Returns nil outside an export callback. A changed generation lets a service retire handles left by an older VM.

Returns: `integer, or nil`

```lua
-- Retire handles a reloaded caller left behind
exports("open", function()
    local owner, generation = GetInvokingResource(), GetInvokingResourceGeneration()
    if generations[owner] ~= generation then
        closeAll(owner)
        generations[owner] = generation
    end
    return newHandle(owner)
end)
```

Registered in `ResourceHost.cpp` (line 24505) as `LuaInvokingResourceGeneration`.

## GetPlayerModel

```lua
GetPlayerModel([playerId])
```

`GAME`: Requires a live game instance.

Read this client's NPC presentation state for a player.

Requires players.model.read. Omit playerId or pass nil for the local player; an explicit ID must be a positive integer. Returns nil without an error for the original character, or a table with player, record, appearance, revision, entity, ready, status and optional failure. The entity handle is local and disposable, never a network player ID. Status is preparing, active, parked, not_streamed or failed. Cosmetic NPC presentation only: the real player retains identity, controls, health, equipment and saved appearance. Character.* records resolve against each client's installed game data, not a virtual allowlist. See player-models.md for ownership, events, cleanup and the known special-rig/vehicle limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `table? state`, `string? reason`

```lua
local model, reason = Open77.players.getModel()
if model then print(model.record, model.ready, model.status) end
```

[Player morphs and model lifecycle guide](../../01%20Documentation/player-models.md)

Registered in `ResourceHost.cpp` (line 25140) as `LuaPlayerModel`.

## GetRegisteredCommands

```lua
GetRegisteredCommands()
```

`CLIENT`: Client-only surface.

FiveM-style alias for `Open77.runtime.commands`.

Client-side. Each entry carries `name`, `resource`, `restricted` and, when the registrar declared them, `help` and `parameters` -- two fields more than FiveM's, because the composer needs them to build a completion.

Returns: `array of tables`

```lua
-- Build a help list from every client command
for _, entry in ipairs(GetRegisteredCommands()) do
    print(("/%s - %s"):format(entry.name, entry.help or ""))
end
```

Registered in `ResourceHost.cpp` (line 24517) as `LuaGetRegisteredCommands`.

## GetResourceKvp

```lua
GetResourceKvp(key, [default])
```

`SHARED`: Available without a live game instance.

FiveM-style alias for `Open77.kvp.get`.

FiveM-familiar alias for `Open77.kvp.get`. It returns the original stored Lua type or nil when absent and stays inside the active connection-address and calling-resource namespace.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string` | required | — |
| `default` | `any` | optional | — |

Returns: `value or nil`, `reason`

```lua
-- Restore the HUD position the player chose last time
local position = GetResourceKvp("hud_position", "top_left")
```

Registered in `ResourceHost.cpp` (line 24508) as `LuaKvpGet`.

## GetResourceState

```lua
GetResourceState(resource)
```

`SHARED`: Available without a live game instance.

State of a resource on this client: `discovered`, `starting`, `running`, `stopping`, `stopped`, `failed`, `blocked`, or `missing` for a name this host does not know. There is no `started`: a resource that is up reads `running`, so `== "started"` is never true.

Reads the lifecycle state currently known by the client resource host: `discovered`, `starting`, `running`, `stopping`, `stopped`, `failed`, `blocked`, or `missing` for a name this host does not know. There is no `started`: a resource that is up reads `running`, so `== "started"` is never true. A snapshot only: it does not start, stop, or wait for the target resource. `Open77.resource.state` is the same function.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `resource` | `string` | required | — |

Returns: `one of the eight state names`

```lua
-- Only use the notifications package if it is actually running (the state is "running", never "started")
if GetResourceState("open77_notifications") == "running" then
    Open77.exports.call("open77_notifications", "show", { message = "Ready" })
end
```

Registered in `ResourceHost.cpp` (line 24506) as `LuaGetResourceState`.

## GetSoundState

```lua
GetSoundState(id)
```

`GAME`: Requires a live game instance.

GetSoundState: local package audio.

Requires audio.play; network=true is rejected. MP3/PCM WAV decode asynchronously off the game thread. getState reports actual decoder state and cached PCM reuse. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `table, or nil, reason`

```lua
-- Restart the siren loop if it has stopped playing
local state = GetSoundState(sirenId)
if state and state.status == "stopped" then PlaySound(sirenId) end
```

Registered in `ResourceHost.cpp` (line 12287) as `LuaAudioState`.

## GlobalState

```lua
GlobalState()
```

`NETWORK`: Uses the network backend.

The server-wide state bag, read-only on the client.

The FiveM spelling of `Open77.state.global`, mirroring what the server replicated. Reading a key the server never set, or that this client has not received yet, answers nil — check `Open77.state.ready()` before treating a nil as an absence. Writing is refused: the client never writes a bag, because a bag is shared state and a client that could write one could write another player's job or another vehicle's keys. Ask with `Open77.state.request`, which the server arbitrates and republishes.

Returns: `the global bag, indexable by key`

```lua
-- Read the round number the server replicated
CreateThread(function()
    while not Open77.state.ready() do Wait(100) end
    Open77.hud.notify("Round " .. tostring(GlobalState.round))
end)
```

Registered in `ResourceHost.cpp` (line 24477) as `PushStateBag`.

## IsDuplicityVersion

```lua
IsDuplicityVersion()
```

`SHARED`: Available without a live game instance.

False in the client VM, true in the server VM.

How one shared script branches between the two sides.

Returns: `boolean`

```lua
-- One shared script, two sides
if IsDuplicityVersion() then
    RegisterCommand("ping", function(source) print("pong from", source) end, false)
else
    RegisterNetEvent("app:pong", function() Open77.hud.notify("pong") end)
end
```

Registered in `ResourceHost.cpp` (line 24532) as `LuaIsDuplicityVersion`.

## IsPlayerAiming

```lua
IsPlayerAiming([playerId])
```

`GAME`: Requires a live game instance.

Graph-produced aiming state, including toggle aim.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- A stealth mission: warn when the player raises their weapon near the guards
SetTick(function()
    if IsPlayerAiming() and nearGuards then
        Open77.hud.notify("Lower your weapon", { replace = true })
    end
end)
```

Registered in `ResourceHost.cpp` (line 12366) as `LuaPlayerPredicate`.

## IsPlayerCrouching

```lua
IsPlayerCrouching([playerId])
```

`GAME`: Requires a live game instance.

Crouched locomotion state.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- Sneaking bonus: quieter footsteps while crouched
if IsPlayerCrouching() then noiseLevel = 0.3 end
```

Registered in `ResourceHost.cpp` (line 12368) as `LuaPlayerPredicate`.

## IsPlayerDiving

```lua
IsPlayerDiving([playerId])
```

`GAME`: Requires a live game instance.

Native underwater swimming state.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- Warn about the oxygen the moment the player goes under
SetTick(function()
    if IsPlayerDiving() then Open77.hud.notify("Watch your oxygen", { replace = true }) end
end)
```

Registered in `ResourceHost.cpp` (line 12365) as `LuaPlayerPredicate`.

## IsPlayerDriver

```lua
IsPlayerDriver([playerId])
```

`GAME`: Requires a live game instance.

Mounted in the driver seat.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- The radio key only works for the driver
RegisterKeyMapping("car_radio", "Car radio", "R", function()
    if IsPlayerDriver() then toggleRadio() end
end)
```

Registered in `ResourceHost.cpp` (line 12371) as `LuaPlayerPredicate`.

## IsPlayerFalling

```lua
IsPlayerFalling([playerId])
```

`GAME`: Requires a live game instance.

Falling phase.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- Deploy the parachute automatically on a long fall
SetTick(function()
    if IsPlayerFalling() and not chuteOpen then openChute() end
end)
```

Registered in `ResourceHost.cpp` (line 12370) as `LuaPlayerPredicate`.

## IsPlayerGrounded

```lua
IsPlayerGrounded([playerId])
```

`GAME`: Requires a live game instance.

Locomotion ground contact.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- The dash key only works from the ground
RegisterKeyMapping("dash", "Dash", "SHIFT", function()
    if IsPlayerGrounded() then dash() end
end)
```

Registered in `ResourceHost.cpp` (line 12367) as `LuaPlayerPredicate`.

## IsPlayerInVehicle

```lua
IsPlayerInVehicle([playerId])
```

`GAME`: Requires a live game instance.

Native vehicle mounting state.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- Show the speedometer only while in a vehicle
SetTick(function()
    speedometer.visible = IsPlayerInVehicle() == true
end)
```

Registered in `ResourceHost.cpp` (line 12371) as `LuaPlayerPredicate`.

## IsPlayerJumping

```lua
IsPlayerJumping([playerId])
```

`GAME`: Requires a live game instance.

Jump or double-jump phase.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- Count jumps for a parkour score
local lastJumping = false
SetTick(function()
    local jumping = IsPlayerJumping()
    if jumping and not lastJumping then jumps = jumps + 1 end
    lastJumping = jumping == true
end)
```

Registered in `ResourceHost.cpp` (line 12370) as `LuaPlayerPredicate`.

## IsPlayerModelReady

```lua
IsPlayerModelReady([playerId])
```

`GAME`: Requires a live game instance.

Check whether the model is ready on this client.

Requires players.model.read. Omitted/nil playerId selects the local player; explicit IDs must be positive integers. False means no active ready model, including the original character; nil, reason indicates failure. This is local projection readiness, not an acknowledgement from every observer. Cosmetic NPC presentation only: the real player retains identity, controls, health, equipment and saved appearance. Character.* records resolve against each client's installed game data, not a virtual allowlist. See player-models.md for ownership, events, cleanup and the known special-rig/vehicle limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean? ready`, `string? reason`

```lua
local ready, reason = Open77.players.isModelReady()
if ready then print('Local presentation ready') end
```

[Player morphs and model lifecycle guide](../../01%20Documentation/player-models.md)

Registered in `ResourceHost.cpp` (line 25142) as `LuaPlayerModel`.

## IsPlayerModelValid

```lua
IsPlayerModelValid(record)
```

`GAME`: Requires a live game instance.

Validate a Character record in the client's installed game data.

Requires players.model.read. Does not spawn or animate the model. True confirms an existing Character record, not compatibility of its rig, locomotion, seats or authored appearances. Invalid inputs and absent records return false, reason; permission failures return nil, reason. Cosmetic NPC presentation only: the real player retains identity, controls, health, equipment and saved appearance. Character.* records resolve against each client's installed game data, not a virtual allowlist. See player-models.md for ownership, events, cleanup and the known special-rig/vehicle limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `record` | `string` | required | — |

Returns: `boolean? valid`, `string? reason`

```lua
local ok, reason = Open77.players.isModelValid('Character.Rogue')
if not ok then print(reason) end
```

[Player morphs and model lifecycle guide](../../01%20Documentation/player-models.md)

Registered in `ResourceHost.cpp` (line 25144) as `LuaPlayerModelValid`.

## IsPlayerPassenger

```lua
IsPlayerPassenger([playerId])
```

`GAME`: Requires a live game instance.

Mounted in a non-driver seat.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- A passenger may use the phone while the car moves; the driver may not
if IsPlayerPassenger() then openPhone() end
```

Registered in `ResourceHost.cpp` (line 12372) as `LuaPlayerPredicate`.

## IsPlayerReloading

```lua
IsPlayerReloading([playerId])
```

`GAME`: Requires a live game instance.

Reloading state.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- Hide the crosshair while reloading
SetTick(function()
    Open77.hud.setVisible("crosshair", IsPlayerReloading() ~= true)
end)
```

Registered in `ResourceHost.cpp` (line 12367) as `LuaPlayerPredicate`.

## IsPlayerShooting

```lua
IsPlayerShooting([playerId])
```

`GAME`: Requires a live game instance.

Graph trigger state, not confirmation of a projectile or damage.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- Muzzle flash light effect while the trigger is held
SetTick(function()
    if IsPlayerShooting() then flashLight() end
end)
```

Registered in `ResourceHost.cpp` (line 12366) as `LuaPlayerPredicate`.

## IsPlayerSliding

```lua
IsPlayerSliding([playerId])
```

`GAME`: Requires a live game instance.

Sliding locomotion state.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- Sliding through the finish line counts double
if IsPlayerSliding() then score = score + 2 end
```

Registered in `ResourceHost.cpp` (line 12368) as `LuaPlayerPredicate`.

## IsPlayerSprinting

```lua
IsPlayerSprinting([playerId])
```

`GAME`: Requires a live game instance.

Sprint state rather than a velocity threshold.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- Stamina drink wears off faster while sprinting
SetTick(function()
    if IsPlayerSprinting() then buffTimer = buffTimer - 2 end
end)
```

Registered in `ResourceHost.cpp` (line 12369) as `LuaPlayerPredicate`.

## IsPlayerSwimming

```lua
IsPlayerSwimming([playerId])
```

`GAME`: Requires a live game instance.

Native surface-swimming or diving state.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- Drop the held item when the player enters the water
SetTick(function()
    if IsPlayerSwimming() and holdingBox then dropBox() end
end)
```

Registered in `ResourceHost.cpp` (line 12365) as `LuaPlayerPredicate`.

## IsPlayerVaulting

```lua
IsPlayerVaulting([playerId])
```

`GAME`: Requires a live game instance.

Vault traversal state.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- Play a grunt sound on a vault
local wasVaulting = false
SetTick(function()
    local vaulting = IsPlayerVaulting()
    if vaulting and not wasVaulting then Play2DSound("audio/grunt.wav") end
    wasVaulting = vaulting == true
end)
```

Registered in `ResourceHost.cpp` (line 12369) as `LuaPlayerPredicate`.

## joaat

```lua
joaat(text)
```

`SHARED`: Available without a live game instance.

Alias of `GetHashKey`; returns a TweakDBID.

Kept so ported code reads naturally, but it is not the Jenkins hash the name suggests -- see GetHashKey.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `text` | `string` | required | — |

Returns: `integer TweakDBID`

```lua
-- FiveM habit, same value as GetHashKey: a TweakDBID, not a Jenkins hash
local id = joaat("Items.Preset_Lexington_Default")
```

Registered in `ResourceHost.cpp` (line 24535) as `LuaGetHashKey`.

## LoadResourceFile

```lua
LoadResourceFile(resourceName, path)
```

`SHARED`: Available without a live game instance.

Reads a file belonging to the calling resource.

The FiveM spelling of `Open77.resource.readFile`, and no more powerful: the path resolves under this resource's own root, keeps the 1 MiB cap, and naming another resource returns nil, cross_resource_read_denied rather than quietly reading your own file. A traversal returns nil, invalid_resource_path. The server has the same global rooted at the resource's data/ directory, plus SaveResourceFile.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `resourceName` | `string` | required | — |
| `path` | `string` | required | — |

Returns: `file contents, or nil plus a reason`

```lua
-- Load a JSON config shipped with the resource
local text = LoadResourceFile(GetCurrentResourceName(), "config.json")
local config = text and json.decode(text) or {}
```

Registered in `ResourceHost.cpp` (line 24533) as `LuaLoadResourceFile`.

## PauseSound

```lua
PauseSound(id)
```

`GAME`: Requires a live game instance.

PauseSound: local package audio.

Requires audio.play; network=true is rejected. MP3/PCM WAV decode asynchronously off the game thread. getState reports actual decoder state and cached PCM reuse. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `true, or failure, reason`

```lua
-- Pause the radio while the phone call is on
PauseSound(radioId)
AddEventHandler("phone:hungUp", function() PlaySound(radioId) end)
```

Registered in `ResourceHost.cpp` (line 12205) as `LuaAudioControl`.

## Play2DSound

```lua
Play2DSound(asset, [options])
```

`GAME`: Requires a live game instance.

Play2DSound: local package audio.

Requires audio.play; network=true is rejected. MP3/PCM WAV decode asynchronously off the game thread. getState reports actual decoder state and cached PCM reuse. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `asset` | `string` | required | — |
| `options` | `table` | optional | — |

Returns: `integer handle, or nil, reason`

```lua
-- A UI click on every menu selection
AddEventHandler("menu:selected", function()
    Play2DSound("audio/click.wav", { volume = 0.5 })
end)
```

Registered in `ResourceHost.cpp` (line 12278) as `LuaAudioCreate`.

## Play3DSound

```lua
Play3DSound(asset, position, [options])
```

`GAME`: Requires a live game instance.

Play3DSound: local package audio.

Requires audio.play; network=true is rejected. MP3/PCM WAV decode asynchronously off the game thread. getState reports actual decoder state and cached PCM reuse. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `asset` | `string` | required | — |
| `position` | `table` | required | — |
| `options` | `table` | optional | — |

Returns: `integer handle, or nil, reason`

```lua
-- A ringing phone at the booth position, audible within 15 m
local ring = Play3DSound("audio/ring.mp3", { x = -1442.2, y = 127.4, z = 18.0 }, { maxDistance = 15, loop = true })
```

Registered in `ResourceHost.cpp` (line 12278) as `LuaAudioCreate`.

## Player

```lua
Player(id)
```

`CLIENT`: Client-only surface.

FiveM compatibility: a table carrying one player's state bag and `source`.

No permission. `Player(id).state` is the same bag as `Open77.state.player(id)`, read-only on the client. `source` is the id that was passed in. Present so ported code runs unchanged; new code can use either.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer|string` | required | — |

Returns: `table { state, source }`

```lua
-- FiveM-style read of a player's state bag
local job = Player(playerId).state.job
print("their job is", job)
```

Registered in `ResourceHost.cpp` (line 24478) as `LuaCompatPlayer`.

## PlaySound

```lua
PlaySound(id)
```

`GAME`: Requires a live game instance.

PlaySound: local package audio.

Requires audio.play; network=true is rejected. MP3/PCM WAV decode asynchronously off the game thread. getState reports actual decoder state and cached PCM reuse. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `true, or failure, reason`

```lua
-- Play a sound handle that was created paused or stopped
local id = Play2DSound("audio/alarm.wav", { paused = true })
AddEventHandler("alarm:triggered", function() PlaySound(id) end)
```

Registered in `ResourceHost.cpp` (line 12205) as `LuaAudioControl`.

## print

```lua
print(…)
```

`SHARED`: Available without a live game instance.

Writes to the Open77 log, prefixed with the resource name.

Writes a resource-prefixed line to the Open77 log. Values are converted with bounded formatting; this does not send a chat message or write into a WebUI page.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `…` | `any` | required | — |

```lua
-- Goes to the Open77 log, prefixed with the resource name
local x, y, z = Open77.character.position()
print(("player at %.1f, %.1f, %.1f"):format(x, y, z))
```

Registered in `ResourceHost.cpp` (line 24462) as `LuaPrint`.

## quat

```lua
quat(x, y, z, w)
```

`SHARED`: Available without a live game instance.

Builds a quaternion: a plain { x, y, z, w } table with three rotation methods.

Available in both runtimes. Deliberately minimal -- normalize (a zero quaternion becomes the identity, never NaN), mul (Hamilton product, this rotation then the other) and rotate (returns a vector3), plus unpack and table. No arithmetic operators and no #. It exists so the { x =, y =, z =, w = } orientation that Open77.effects and Open77.props already accept becomes computable; the result stays exactly that table.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `z` | `number` | required | — |
| `w` | `number` | required | — |

Returns: `quat, or nil, reason`

```lua
-- Rotate a forward offset by the player's current orientation
local rot = quat(0, 0, 0.7071, 0.7071) -- 90 degrees around z
local ahead = rot:rotate(vector3(0.0, 2.0, 0.0))
```

Registered in `open77_vector.lua` (line 498) as `LuaPrelude`.

## RegisterCommand

```lua
RegisterCommand(name, handler, [restricted], [suggestion])
```

`CLIENT`: Client-only surface.

FiveM-style alias for `Open77.runtime.registerCommand`.

Client-side. The server runtime has its own `RegisterCommand` with the same handler shape but real ACL enforcement behind `restricted`; read both before porting a command that is meant to be admin-only.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `handler` | `function` | required | — |
| `restricted` | `boolean` | optional | — |
| `suggestion` | `table` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- /hudpos <top_left|bottom_right>: a local client command
RegisterCommand("hudpos", function(source, args)
    if not args[1] then return end
    SetResourceKvp("hud_position", args[1])
    Open77.hud.notify("HUD position saved")
end, false)
```

Registered in `ResourceHost.cpp` (line 24515) as `LuaRegisterCommand`.

## RegisterKeyMapping

```lua
RegisterKeyMapping(id, name, key, [onPressed], [onReleased])
```

`GAME`: Requires a live game instance.

Declares a named, rebindable action bound to a key.

Requires `input.actions`. Accepts `{ id, name, key (or defaultKey), hold?, consume?, onPressed, onReleased? }` or `(id, name, key, onPressed [, onReleased])`. Returns `true, effectiveKey` or `false, reason`. The callback receives the mapping ID; hold mode adds releases. Defaults to observing input; Stable 156 adds `consume = true` to suppress the bound native input while invoking Lua. Keyboard, MOUSE1..5, MOUSEWHEELUP/DOWN and gamepad buttons/triggers are supported; wheel holds emit press/release pulses and triggers use a 0.5 threshold. Gamepad behavior remains experimental. Pause and voice capture support keyboard and mouse buttons; use Lua/rebind for wheel/gamepad controls. Mappings pause while a WebUI owns input. Re-registering preserves saved rebinds; unregister/stop/rebind/focus changes release consumption. See [key mappings](../../01%20Documentation/keybindings.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |
| `name` | `string` | required | — |
| `key` | `string` | required | — |
| `onPressed` | `function` | optional | — |
| `onReleased` | `function` | optional | — |

Returns: `boolean: true on installation, false on refusal`, `string: effective key on success, reason on refusal`

```lua
-- A rebindable "open inventory" key, defaulting to I
RegisterKeyMapping("inventory_open", "Open inventory", "I", function()
    openInventory()
end)
```

Registered in `ResourceHost.cpp` (line 24512) as `LuaRegisterKeyMapping`.

## RegisterNetEvent

```lua
RegisterNetEvent(event, [handler])
```

`NETWORK`: Uses the network backend.

Allows an event to arrive from the server.

Allows an authenticated server event with this name to enter the current resource, and optionally attaches a handler in the same call. Registration is generation-scoped, requires `network.events`, and does not turn arbitrary client events into server events.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `handler` | `function` | optional | — |

Returns: `handler id`

```lua
-- Receive a server notification and show it natively
RegisterNetEvent("bank:deposit", function(amount)
    Open77.hud.notify(("Deposited %d eddies"):format(amount))
end)
```

Registered in `ResourceHost.cpp` (line 24471) as `LuaRegisterNetEvent`.

## RemoveEventHandler

```lua
RemoveEventHandler(event, handler)
```

`SHARED`: Available without a live game instance.

Removes a handler.

Removes the exact callback previously registered for the named local event. It returns `false` when that event/callback pair is not owned by the current resource generation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `handler` | `function` | required | — |

Returns: `boolean`

```lua
-- Stop listening after the tutorial step is done
local function onStep(step)
    if step == "done" then RemoveEventHandler("tutorial:step", onStep) end
end
AddEventHandler("tutorial:step", onStep)
```

Registered in `ResourceHost.cpp` (line 24469) as `LuaRemoveEventHandler`.

## RemoveStateBagChangeHandler

```lua
RemoveStateBagChangeHandler(subscription)
```

`CLIENT`: Client-only surface.

FiveM compatibility: stops a state-bag change subscription.

No permission. The same function as `Open77.state.offChange`; takes the id either registration returned.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `subscription` | `integer` | required | — |

Returns: `true, otherwise false`

```lua
-- Watch the vehicle's owner key only while the player sits in it
local sub = AddStateBagChangeHandler("owner", "vehicle:" .. vehicleId, function(_, _, value)
    print("owner is now", value)
end)
AddEventHandler("vehicle:left", function() RemoveStateBagChangeHandler(sub) end)
```

Registered in `ResourceHost.cpp` (line 24481) as `LuaStateOffChange`.

## require

```lua
require(module)
```

`SHARED`: Available without a live game instance.

Loads a local Lua module or a published dependency library into the caller's client VM.

Client-only Lua module loader. Loads a local module, or a published module from a declared, running dependency using `@provider` / `@provider/module`. The dependency form requires the development client update introduced with PolyZone. Dots map to folders; omit `.lua`. Code executes synchronously in the caller's VM, permissions and budget. Results are cached by resolved file per caller generation; no returned value becomes `true`. Lookup/loading failures return `nil, reason`; circular imports are rejected. Reload consumers to adopt changed library code. Unlike exports, no live Lua object crosses a VM boundary. No HTTP, native `package` loader or unrestricted filesystem access is exposed; the dedicated-server sandbox has no `require`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `module` | `string` | required | — |

Returns: `module value (true when no value is returned), or nil on loading failure`, `reason on failure`

```lua
local PZ, reason = require('@polyzone')
assert(PZ, reason)
local zone = PZ.CircleZone:Create({x=100, y=200, z=20}, 15, {name='meeting_point'})
```

Registered in `ResourceHost.cpp` (line 24463) as `LuaRequire`.

## Screen2DPointTo3DSpace

```lua
Screen2DPointTo3DSpace(x, y, [maxDistance])
```

`GAME`: Requires a live game instance.

Unproject a normalized screen point into a world-space ray.

Requires world.query. x/y use normalized top-left viewport coordinates in [0,1]. maxDistance defaults to 100 metres and must be finite in [0.1,1000]. Returns origin, unit direction, endpoint position and maxDistance; this is not a collision result. Uses the active camera and checks projection consistency. Returns nil/reason if the camera or projection is unavailable. Identical global alias: Screen2DPointTo3DSpace. See screen-picking.md and context-menu.md. Requires the new screen-picking client, not the earlier .62 release.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `maxDistance` | `number` | optional | — |

Returns: `ray table, or nil, reason`

```lua
-- What is under the middle of the screen, up to 25 m away?
local ray, reason = Screen2DPointTo3DSpace(0.5, 0.5, 25)
if ray then
    local hit = Open77.world.raycast(ray.origin, ray.position, { entities = true })
    if hit and hit.hit and hit.entity then print("looking at", hit.entity.className) end
end
```

Registered in `ResourceHost.cpp` (line 24882) as `LuaCameraScreenRay`.

## SeekSound

```lua
SeekSound(id, seconds)
```

`GAME`: Requires a live game instance.

SeekSound: local package audio.

Requires audio.play; network=true is rejected. MP3/PCM WAV decode asynchronously off the game thread. getState reports actual decoder state and cached PCM reuse. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `seconds` | `number` | required | — |

Returns: `true, or failure, reason`

```lua
-- Skip the intro of the track
SeekSound(musicId, 12.5)
```

Registered in `ResourceHost.cpp` (line 12206) as `LuaAudioControl`.

## SetResourceKvp

```lua
SetResourceKvp(key, value)
```

`SHARED`: Available without a live game instance.

FiveM-style alias for `Open77.kvp.set`.

FiveM-familiar alias for `Open77.kvp.set`. It stores one typed value in the calling resource's persistent namespace for the active connection address; it cannot select another server or resource.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string` | required | — |
| `value` | `any` | required | — |

Returns: `boolean`, `reason`

```lua
-- Remember the player's chosen HUD position on this machine
SetResourceKvp("hud_position", "bottom_right")
```

Registered in `ResourceHost.cpp` (line 24507) as `LuaKvpSet`.

## SetSoundPosition

```lua
SetSoundPosition(id, position)
```

`GAME`: Requires a live game instance.

SetSoundPosition: local package audio.

Requires audio.play; network=true is rejected. MP3/PCM WAV decode asynchronously off the game thread. getState reports actual decoder state and cached PCM reuse. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `position` | `table` | required | — |

Returns: `true, or failure, reason`

```lua
-- Keep the boombox sound following the carried prop
SetTick(function()
    SetSoundPosition(boomboxSound, vector3(Open77.character.position()))
end)
```

Registered in `ResourceHost.cpp` (line 12281) as `LuaAudioControl`.

## SetSoundVolume

```lua
SetSoundVolume(id, volume)
```

`GAME`: Requires a live game instance.

SetSoundVolume: local package audio.

Requires audio.play; network=true is rejected. MP3/PCM WAV decode asynchronously off the game thread. getState reports actual decoder state and cached PCM reuse. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `volume` | `number` | required | — |

Returns: `true, or failure, reason`

```lua
-- Fade the music out over two seconds
CreateThread(function()
    for v = 10, 0, -1 do
        SetSoundVolume(musicId, v / 10)
        Wait(200)
    end
    StopSound(musicId)
end)
```

Registered in `ResourceHost.cpp` (line 12281) as `LuaAudioControl`.

## SetTick

```lua
SetTick(body)
```

`SHARED`: Available without a live game instance.

Runs a function once per host tick.

A scheduler task looping with an implicit `Wait(0)`: same per-frame budget, same task cap, stopped with the resource. An error inside the function is logged and the tick survives it; after five consecutive failures the tick logs `tick cancelled after 5 consecutive failures` and cancels itself. One successful pass resets the count.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `body` | `function` | required | — |

Returns: `tick id, or nil plus invalid_tick_function / task_limit_or_stopping`

```lua
-- Show the prompt every frame while the player stands at the delivery point
local point = vector3(-1442.2, 127.4, 18.0)
local tick = SetTick(function()
    if #(vector3(Open77.character.position()) - point) < 2.0 then
        Open77.hud.notify("Press E to deliver", { replace = true })
    end
end)
AddEventHandler("delivery:done", function() ClearTick(tick) end)
```

Registered in `ResourceHost.cpp` (line 24530) as `LuaSetTick`.

## SetTimeout

```lua
SetTimeout(milliseconds, body)
```

`SHARED`: Available without a live game instance.

Runs a function once, later.

Schedules a Lua callback after at least the requested number of milliseconds without blocking the game thread. The timer belongs to the current resource generation and is cancelled automatically on stop or hot reload.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `milliseconds` | `integer` | required | — |
| `body` | `function` | required | — |

Returns: `timer id`

```lua
-- Hide the hint after 4 seconds
Open77.hud.notify("Press E to open the trunk")
SetTimeout(4000, function() Open77.hud.clearNotify() end)
```

Registered in `ResourceHost.cpp` (line 24466) as `LuaSetTimeout`.

## StopSound

```lua
StopSound(id)
```

`GAME`: Requires a live game instance.

StopSound: local package audio.

Requires audio.play; network=true is rejected. MP3/PCM WAV decode asynchronously off the game thread. getState reports actual decoder state and cached PCM reuse. Assets must be declared in the resource manifest files; absolute paths, URLs and traversal are rejected. Handles are scoped to the creating resource VM; stop rewinds, pause preserves position, seek uses seconds, destroy releases permanently. Volume 0..1, range <=1000m. Network loops require actual duration. See package-audio.md for options, lifecycle, quotas and errors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `true, or failure, reason`

```lua
-- Stop the alarm when the player leaves the building
AddEventHandler("building:left", function() StopSound(alarmId) end)
```

Registered in `ResourceHost.cpp` (line 12280) as `LuaAudioControl`.

## TriggerEvent

```lua
TriggerEvent(event, [...])
```

`SHARED`: Available without a live game instance.

Publishes an event to every handler in this resource's VM.

Dispatches an event only inside the client resource runtime. Arguments must fit the runtime's bounded script-value representation; use `TriggerServerEvent` for traffic that must cross the network.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `...` | `any` | optional | — |

Returns: `true, or false, reason`

```lua
-- Tell the rest of this resource that the menu closed
TriggerEvent("shop:closed")
AddEventHandler("shop:closed", function() FreezePosition(false) end)
```

Registered in `ResourceHost.cpp` (line 25380) as `LuaTriggerEvent`.

## TriggerServerEvent

```lua
TriggerServerEvent(event, [...])
```

`NETWORK`: Uses the network backend.

Sends an event to the server.

Serializes and sends an event to the connected server with the caller's authenticated player identity. It requires `network.events`; tables must be serializable and payload limits are enforced before anything is queued.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `...` | `any (up to 32 values: strings, numbers, booleans, tables)` | optional | — |

Returns: `boolean`

```lua
-- Ask the server to buy an item; the server decides
RegisterKeyMapping("buy_ammo", "Buy ammo", "B", function()
    TriggerServerEvent("shop:buy", "ammo_box")
end)
```

Registered in `ResourceHost.cpp` (line 24472) as `LuaTriggerServerEvent`.

## vec

```lua
vec(...)
```

`SHARED`: Available without a live game instance.

Builds a vector of the size the arguments imply.

Available in both runtimes. With two, three or four numbers it builds a vector2, vector3 or vector4. With a single table it picks the size from the axes actually present: w means vector4, else z means vector3, else y means vector2 -- which makes vec(position) the short way to wrap a position returned by any existing API. A quaternion is refused, because vec makes vectors.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `...` | `number|table` | required | — |

Returns: `vector2, vector3 or vector4, or nil, reason`

```lua
-- Wrap any position table and measure distance with #
local shop = vec(-1442.2, 127.4, 18.0)
if #(vec(Open77.character.state().position) - shop) < 3.0 then
    Open77.hud.notify("Press E to shop")
end
```

Registered in `open77_vector.lua` (line 499) as `LuaPrelude`.

## vector2

```lua
vector2(x, y)
```

`SHARED`: Available without a live game instance.

Builds a 2D vector: a plain { x, y } table with vector behaviour.

Available in both runtimes. Same operators, swizzles and methods as vector3, minus cross, which answers nil, "cross_requires_vector3". Takes two finite numbers, or one vector or { x =, y = } table to copy.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `x` | `number` | required | — |
| `y` | `number` | required | — |

Returns: `vector2, or nil, reason`

```lua
-- Screen-space distance between two points
local a, b = vector2(0.5, 0.5), vector2(0.7, 0.4)
print(#(a - b))
```

Registered in `open77_vector.lua` (line 495) as `LuaPrelude`.

## vector3

```lua
vector3(x, y, z)
```

`SHARED`: Available without a live game instance.

Builds a 3D vector: a plain { x, y, z } table with vector behaviour.

Available in both runtimes, from the shared prelude scripting/lua/open77_vector.lua. Takes three finite numbers, or one vector or { x =, y =, z = } table to copy. The result IS a table with real x, y and z fields, so every API that accepts a position table accepts it unchanged and json.encode emits {"x":..,"y":..,"z":..}. Supports + - * / (by a vector, a position table or a number), unary minus, # for magnitude, == (exact, same kind only), tostring/.. as vector3(1.0, 2.0, 3.0), the swizzles .xy/.xz/.yz/.xyz/.zyx, and the methods length, lengthSquared, distance, distanceSquared, normalize (zero stays zero), dot, cross, lerp (unclamped), unpack and table. Fields are read-only except the components themselves, which Lua 5.4 cannot intercept. Operators raise; this constructor returns nil, reason. See the Vectors and quaternions guide.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `z` | `number` | required | — |

Returns: `vector3, or nil, reason`

```lua
-- Is the player within 3 m of the shop counter?
local counter = vector3(-1442.2, 127.4, 18.0)
if #(vector3(Open77.character.position()) - counter) < 3.0 then
    Open77.hud.notify("Press E to shop", { replace = true })
end
```

Registered in `open77_vector.lua` (line 496) as `LuaPrelude`.

## vector4

```lua
vector4(x, y, z, w)
```

`SHARED`: Available without a live game instance.

Builds a 4D vector: a plain { x, y, z, w } table with vector behaviour.

Available in both runtimes. Same operators and methods as vector3, minus cross. Its swizzles reach w, so v.xyz narrows a vector4 to a vector3. Takes four finite numbers, or one vector or { x =, y =, z =, w = } table to copy.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `z` | `number` | required | — |
| `w` | `number` | required | — |

Returns: `vector4, or nil, reason`

```lua
-- A spawn point with its heading as w
local spawn = vector4(-1442.2, 127.4, 18.0, 90.0)
Open77.travel.teleport(spawn.x, spawn.y, spawn.z, spawn.w)
```

Registered in `open77_vector.lua` (line 497) as `LuaPrelude`.

## Wait

```lua
Wait(milliseconds)
```

`SHARED`: Available without a live game instance.

Suspends the current coroutine.

Only usable inside a `CreateThread`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `milliseconds` | `integer` | required | — |

```lua
-- A three-second countdown on the HUD
CreateThread(function()
    for n = 3, 1, -1 do
        Open77.hud.notify(("Round starts in %d"):format(n), { replace = true })
        Wait(1000)
    end
end)
```

Registered in `ResourceHost.cpp` (line 24465) as `LuaWait`.
