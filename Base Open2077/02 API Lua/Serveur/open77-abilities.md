# Open77.abilities (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-abilities.md) · [Page web](https://open2077.net/docs/api/server/open77-abilities)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

5 functions.

## cancel

```lua
Open77.abilities.cancel(playerId)
```

`NETWORK`: Uses the network backend.

Cancels this resource's in-flight ability action, keeping the entitlement.

Requires `players.abilities.manage`. Use it to interrupt a slam in progress without taking the ability away; `revoke` does both.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `{ ok = true }, or nil`, `reason`

```lua
-- Entering a safe zone interrupts a slam in progress but keeps the ability
RegisterNetEvent("safezone:entered", function()
    Open77.abilities.cancel(source)
end)
```

[Ground Slam guide](../../01%20Documentation/ground-slam.md)

Registered in `LuaResourceRuntime.cs` (line 8035) as `LuaResourceRuntime.Bootstrap`.

## current

```lua
Open77.abilities.current(playerId)
```

`NETWORK`: Uses the network backend.

The player's current ability projection snapshot.

Requires `players.abilities.read`. Reports what the server believes the client is projecting, which is admission state rather than proof of a rendered animation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `projection snapshot, or nil`, `reason`

```lua
-- Is the player mid-slam?
local snapshot = Open77.abilities.current(playerId)
if snapshot and snapshot.active then print("slam phase", snapshot.phase) end
```

[Ground Slam guide](../../01%20Documentation/ground-slam.md)

Registered in `LuaResourceRuntime.cs` (line 8036) as `LuaResourceRuntime.Bootstrap`.

## define

```lua
Open77.abilities.define(definition)
```

`NETWORK`: Uses the network backend.

Registers a server-owned ability definition for this resource.

Requires `players.abilities.define`. The definition names the ability and its native parameters; grants later refer to it by id. Definitions belong to the declaring resource, so a reload republishes them. See the [ground slam guide](https://open2077.net/docs/api/server/ground-slam.md) for the definition fields and the projection contract.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `definition` | `table` | required | — |

Returns: `{ ok = true }, or nil`, `reason`

```lua
-- A cosmetic, harmless ground slam bound to L
assert(Open77.abilities.define({
    id = "myserver.quake", version = 1, profile = "ground_slam",
    config = { inputKey = "l", allowGround = true, allowAir = true, cosmetic = true, nonlethal = true,
               damage = 0, staminaCost = 20, cooldownMs = 5000, radius = 4 },
}))
```

[Ground Slam guide](../../01%20Documentation/ground-slam.md)

Registered in `LuaResourceRuntime.cs` (line 8032) as `LuaResourceRuntime.Bootstrap`.

## grant

```lua
Open77.abilities.grant(playerId, definition)
```

`NETWORK`: Uses the network backend.

Grants a defined ability to one player.

Requires `players.abilities.manage`. Acceptance is entitlement, not rendering: native projection on the target client can still be pending when this returns. Only the granting resource may revoke or cancel it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `definition` | `string (definition id)` | required | — |

Returns: `{ ok = true }, or nil`, `reason`

```lua
-- Give the ability to whoever buys the implant
RegisterNetEvent("shop:buySlam", function()
    local result, reason = Open77.abilities.grant(source, "myserver.quake")
    if not result then Open77.chat.send(source, "Not granted: " .. reason) end
end)
```

[Ground Slam guide](../../01%20Documentation/ground-slam.md)

Registered in `LuaResourceRuntime.cs` (line 8033) as `LuaResourceRuntime.Bootstrap`.

## revoke

```lua
Open77.abilities.revoke(playerId)
```

`NETWORK`: Uses the network backend.

Removes this resource's ability grant from a player and cancels its action.

Requires `players.abilities.manage`. Scoped to the calling resource: another resource's grant on the same player is untouched. Resource stop revokes automatically.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `{ ok = true }, or nil`, `reason`

```lua
-- Take the ability away when the player is arrested
Open77.abilities.revoke(playerId)
```

[Ground Slam guide](../../01%20Documentation/ground-slam.md)

Registered in `LuaResourceRuntime.cs` (line 8034) as `LuaResourceRuntime.Bootstrap`.
