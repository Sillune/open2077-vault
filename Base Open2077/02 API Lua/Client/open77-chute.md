# Open77.chute (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-chute.md) · [Page web](https://open2077.net/docs/api/client/open77-chute)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

3 functions.

## arm

```lua
Open77.chute.arm(gravity, descentRate, [glideSpeed])
```

`GAME`: Requires a live game instance.

Arms the grav-chute with a gravity, descent rate and optional glide speed.

Requires `player.chute`. `player.chute` is its own permission rather than a member of `player.travel`, because the chute moves nobody: it rewrites the fall, and a gamemode that may teleport is not thereby a gamemode that may change gravity. Every bound is enforced in the native layer; the Lua side only refuses what Lua can get wrong first -- a missing argument, or one that is not a number.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `gravity` | `number` | required | — |
| `descentRate` | `number` | required | — |
| `glideSpeed` | `number` | optional | — |

Returns: `true, otherwise false`, `reason: permission_denied:player.chute, chute_unavailable_on_this_host`

```lua
-- Arm the grav-chute when the player jumps off the tower
AddEventHandler("tower:jumped", function()
    Open77.chute.arm(2.0, 3.0, 8.0) -- gravity, descent rate, glide speed
end)
```

Registered in `ResourceHost.cpp` (line 25442) as `LuaChuteArm`.

## disarm

```lua
Open77.chute.disarm()
```

`GAME`: Requires a live game instance.

Disarms the grav-chute.

Requires `player.chute`. Restores the gravity and descent the engine had before `arm`. Disarming when nothing is armed succeeds. Like `arm`, it is refused on a host with no chute backend (`chute_unavailable_on_this_host`).

Returns: `true, otherwise false`, `reason`

```lua
-- Landed: normal gravity again
AddEventHandler("tower:landed", function()
    Open77.chute.disarm()
end)
```

Registered in `ResourceHost.cpp` (line 25443) as `LuaChuteDisarm`.

## isArmed

```lua
Open77.chute.isArmed()
```

`GAME`: Requires a live game instance.

Whether this client last accepted an arm or a disarm.

Requires `player.chute`. It reports the last arm/disarm **this client accepted**, not a reading of the script side, which lags by up to one poll of the bridge loop. Treat it as bookkeeping, never as proof the player is falling slowly. Without the permission or a chute backend it answers false rather than an error.

Returns: `boolean`

```lua
-- Show the chute icon while armed
SetTick(function()
    chuteIcon.visible = Open77.chute.isArmed()
end)
```

Registered in `ResourceHost.cpp` (line 25444) as `LuaChuteIsArmed`.
