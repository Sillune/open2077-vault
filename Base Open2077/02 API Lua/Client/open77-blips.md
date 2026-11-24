# Open77.blips (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-blips.md) · [Page web](https://open2077.net/docs/api/client/open77-blips)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

24 functions.

## attachToEntity

```lua
Open77.blips.attachToEntity(id, entity, [slot], [offset])
```

`GAME`: Requires a live game instance.

Makes a blip follow an Open77 entity.

Changes an owned blip from a fixed world position to tracking a streamed Open77 entity. The entity id is ephemeral and the call requires `ui.vanilla.map`; remove or reattach the blip when its gameplay target changes.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |
| `entity` | `entity` | required | — |
| `slot` | `string` | optional | `poi_mappin` |
| `offset` | `table { x, y, z }` | optional | — |

Returns: `boolean`, `reason`

```lua
-- The delivery car's blip follows the car
local blip = Open77.blips.create({ position = car.position, sprite = "vehicle", label = "Delivery car" })
Open77.blips.attachToEntity(blip, car.entity)
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 25462) as `LuaBlipAttachToEntity`.

## clear

```lua
Open77.blips.clear()
```

`GAME`: Requires a live game instance.

Removes every blip owned by the current resource.

Removes every vanilla-map blip owned by the calling resource generation. Other resources' blips are left untouched, and generation teardown performs the same cleanup automatically.

Returns: `true`

```lua
-- The heist is over: drop every blip this resource placed
AddEventHandler("heist:ended", function()
    Open77.blips.clear()
end)
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 25483) as `LuaBlipClear`.

## clearWaypoint

```lua
Open77.blips.clearWaypoint()
```

`GAME`: Requires a live game instance.

Clears the player's map waypoint, whoever set it.

Untracks the waypoint, including one the player placed themselves, which is what `DeleteWaypoint` means, and destroys the pin only when this resource owns it; another resource's pin stays registered. Clearing when there was no waypoint is a success that reports `false` as its second value, so a caller reconciling state can tell the two apart without an error. Requires `ui.vanilla.map`.

Returns: `true`, `whether a waypoint was actually cleared`, `reason on failure: permission_denied:ui.vanilla.map, waypoint_unavailable_on_this_host, wrong_thread, world_unavailable`

```lua
-- The taxi fare is cancelled: remove the route
local ok, wasSet = Open77.blips.clearWaypoint()
if ok and wasSet then exports.open77_chat:addMessage("Route cancelled.") end
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 25481) as `LuaBlipClearWaypoint`.

## create

```lua
Open77.blips.create(options)
```

`GAME`: Requires a live game instance.

Creates a resource-owned vanilla map pin.

Requires `ui.vanilla.map`. Provide exactly one of `position` or `entity`. Set `routable = true` on a positional blip to use Cyberpunk's trusted custom-waypoint definition; its sprite becomes `CustomPositionVariant` and `track(id)` can then start native GPS routing. The returned 64-bit generation handle is a decimal string and is cleaned up automatically when the resource stops. Experimental SVG icons and per-blip RGB/RGBA colors require the matching native client adapter. Declare SVG assets in the manifest; use icon = { asset, size } and color = "#RRGGBB" or "#RRGGBBAA". PNG blip icons are not supported by this adapter.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table { position|entity, sprite?, title?, description?, label?, icon?, color?, range?, active?, visibleThroughWalls?, routable?, slot?, offset? }` | required | — |

Returns: `decimal-string blip id, or nil`, `reason`

```lua
local id = assert(Open77.blips.create({
  position = { x = 10, y = 20, z = 30 },
  routable = true, label = "Street race"
}))
assert(Open77.blips.track(id))
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 23005) as `LuaBlipCreate`.

## get

```lua
Open77.blips.get(id)
```

`GAME`: Requires a live game instance.

Reads one blip owned by the current resource.

Returns a read-only snapshot of one blip owned by the calling resource. The handle must still belong to the current generation; a stale or foreign id is rejected.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |

Returns: `snapshot table, or nil`, `reason`

```lua
-- Is the blip still active?
local blip = Open77.blips.get(blipId)
if blip and not blip.active then Open77.blips.setActive(blipId, true) end
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 25484) as `LuaBlipGet`.

## list

```lua
Open77.blips.list()
```

`GAME`: Requires a live game instance.

Lists blips owned by the current resource.

Lists snapshots of all vanilla-map blips owned by the calling resource generation. The returned tables are copies and changing them does not mutate the map.

Returns: `array of snapshot tables`

```lua
-- Print every blip this resource owns
for _, blip in ipairs(Open77.blips.list()) do
    print(blip.id, blip.label)
end
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 25485) as `LuaBlipList`.

## remove

```lua
Open77.blips.remove(id)
```

`GAME`: Requires a live game instance.

Removes one owned blip.

Deletes one blip owned by the calling resource and invalidates its handle. Removal is resource-scoped, so one package cannot erase another package's map state.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |

Returns: `boolean`, `reason`

```lua
-- The package was delivered: remove its map pin
RegisterNetEvent("delivery:done", function()
    Open77.blips.remove(deliveryBlip)
end)
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 25482) as `LuaBlipRemove`.

## setActive

```lua
Open77.blips.setActive(id, active)
```

`GAME`: Requires a live game instance.

Activates or deactivates a vanilla mappin.

Updates the active/highlighted state of an owned vanilla-map blip. This changes presentation only and does not create a route or change server state.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |
| `active` | `boolean` | required | — |

Returns: `boolean`, `reason`

```lua
-- Highlight the shop blip only while it is open
RegisterNetEvent("shop:hours", function(open)
    Open77.blips.setActive(shopBlip, open)
end)
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 25469) as `LuaBlipSetActive`.

## setColor

```lua
Open77.blips.setColor(id, color)
```

`GAME`: Requires a live game instance.

Sets an owned blip color or restores its source colors.

Experimental per-blip styling; requires the matching native client adapter. Accepts exactly #RRGGBB, #RRGGBBAA or false to clear the override. Works with native sprites and custom SVGs without changing other blips sharing the sprite. For SVGs it replaces RGB and multiplies each painted shape's source alpha. Clearing the icon does not clear this override; visual-only updates preserve native tracking. On Cyberpunk 2077 2.5, use Stable 156 or later for corrected map, minimap, legend and world presentation. Existing options remain compatible.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |
| `color` | `string|false` | required | — |

Returns: `boolean`, `reason`

```lua
assert(Open77.blips.setColor(jobBlip, "#FFCC0080"))
assert(Open77.blips.setColor(jobBlip, false))
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 24507) as `LuaBlipSetColor`.

## setDescription

```lua
Open77.blips.setDescription(id, description)
```

`GAME`: Requires a live game instance.

Sets the custom fullscreen-map description of an owned blip.

Requires `ui.vanilla.map`. The description is displayed by Open77's fullscreen-map tooltip for the selected native mappin.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |
| `description` | `string` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- A tooltip with the shop's opening hours on the fullscreen map
Open77.blips.setDescription(shopBlip, "Open 20:00 - 04:00. Cash only.")
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 25466) as `LuaBlipSetDescription`.

## setIcon

```lua
Open77.blips.setIcon(id, icon)
```

`GAME`: Requires a live game instance.

Sets a declared SVG icon or restores the native sprite.

Experimental native SVG adapter; requires a matching OPEN//77 client. On Cyberpunk 2077 2.5, use Stable 156 or later for the corrected adapter. Accepts a declared resource-relative .svg path, { asset, size }, or false to restore the native sprite. Size is 16–128 native Ink units (default 48). Color overrides remain when the icon is cleared. Visual-only edits preserve tracking; malformed or unsupported SVG leaves the existing icon unchanged. URLs and PNG/WebP are rejected.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |
| `icon` | `string|table|false` | required | — |

Returns: `boolean`, `reason`

```lua
local ok, reason = Open77.blips.setIcon(hqBlip, { asset = "icons/gang.svg", size = 48 })
if not ok then print(reason) end
Open77.blips.setIcon(hqBlip, false)
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 23013) as `LuaBlipSetIcon`.

## setLabel

```lua
Open77.blips.setLabel(id, label)
```

`GAME`: Requires a live game instance.

Changes the fullscreen-map tooltip title.

Open77 keeps the label in private script data, independent from the vanilla variant's generic title.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |
| `label` | `string` | required | — |

Returns: `boolean`, `reason`

```lua
-- Rename the blip as the job progresses
Open77.blips.setLabel(jobBlip, "Pick up the package")
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 25465) as `LuaBlipSetLabel`.

## setPosition

```lua
Open77.blips.setPosition(id, position)
```

`GAME`: Requires a live game instance.

Moves a blip to a world position.

Also detaches an entity-following blip.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |
| `position` | `table { x, y, z }` | required | — |

Returns: `boolean`, `reason`

```lua
-- Move the "next checkpoint" blip as the race progresses
RegisterNetEvent("race:checkpoint", function(position)
    Open77.blips.setPosition(checkpointBlip, position)
end)
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 25461) as `LuaBlipSetPosition`.

## setRange

```lua
Open77.blips.setRange(id, metres)
```

`GAME`: Requires a live game instance.

Switches a blip off beyond a distance and back on when the player returns.

Requires `ui.vanilla.map`. `metres` is 0..4000; `false` or `0` clears the gate. This is Open77's own distance test driving the vanilla `SetMappinActive`, not a mappin property: `gamemappinsMappinData` has no range field. It is therefore NOT FiveM's `SetBlipAsShortRange`, which hides a blip from the minimap only -- 2.31 offers no per-surface control, so out of range means off on the HUD, the minimap and the world map alike. It composes with `active` rather than overriding it. An entity-following blip is measured from the body it follows, and keeps its last state while that body is not streamed in.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |
| `metres` | `number or false` | required | — |

Returns: `boolean`, `reason`

```lua
-- The vendor blip only shows within 300 m
Open77.blips.setRange(vendorBlip, 300)
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 25471) as `LuaBlipSetRange`.

## setSprite

```lua
Open77.blips.setSprite(id, sprite)
```

`GAME`: Requires a live game instance.

Changes the vanilla mappin variant.

Accepts an exact 2.31 variant name, a stable alias, or an integer from 0 through 146. See `wiki/blips.md` for the complete list.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |
| `sprite` | `string|integer` | required | — |

Returns: `boolean`, `reason`

```lua
-- Turn the marker red once the zone becomes dangerous
Open77.blips.setSprite(zoneBlip, "danger")
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 25463) as `LuaBlipSetSprite`.

## setTitle

```lua
Open77.blips.setTitle(id, title)
```

`GAME`: Requires a live game instance.

Sets the custom fullscreen-map title of an owned blip.

Requires `ui.vanilla.map`. This is the semantic title alias of `setLabel`; the HUD does not permanently render it beside the icon.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |
| `title` | `string` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Title shown when the blip is selected on the fullscreen map
Open77.blips.setTitle(garageBlip, "Your garage")
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 25464) as `LuaBlipSetTitle`.

## setTrackingAlternative

```lua
Open77.blips.setTrackingAlternative(id, target)
```

`GAME`: Requires a live game instance.

Sets another owned blip as the routing alternative.

Passing nil clears it. This does not force the world-map controller to start tracking.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |
| `target` | `decimal string|nil` | required | — |

Returns: `boolean`, `reason`

```lua
-- Route to the back entrance if the front is tracked
Open77.blips.setTrackingAlternative(frontDoorBlip, backDoorBlip)
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 25472) as `LuaBlipSetTrackingAlternative`.

## setVisibleThroughWalls

```lua
Open77.blips.setVisibleThroughWalls(id, visible)
```

`GAME`: Requires a live game instance.

Changes vanilla through-wall visibility.

Controls whether an owned world marker remains visible when geometry occludes it. The setting affects presentation on this client only and requires `ui.vanilla.map`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |
| `visible` | `boolean` | required | — |

Returns: `boolean`, `reason`

```lua
-- Team-mates stay visible through buildings
Open77.blips.setVisibleThroughWalls(teamBlip, true)
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 25470) as `LuaBlipSetVisibleThroughWalls`.

## setWaypoint

```lua
Open77.blips.setWaypoint(position)
```

`GAME`: Requires a live game instance.

Places the player's map waypoint, replacing whatever was tracked.

Registers a routable custom-position mappin and makes it the tracked destination, so the vanilla GPS calculates a road route to it exactly as it does for a waypoint the player set by hand. The pin is owned by the calling resource and is destroyed when that resource stops, so a crashed taxi script leaves no permanent arrow on the map. Calling it again moves the same pin rather than registering a second one. Requires `ui.vanilla.map`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `position` | `table { x, y, z }` | required | — |

Returns: `true`, `reason on failure: permission_denied:ui.vanilla.map, invalid_position, waypoint_unavailable_on_this_host, too_many_blips, wrong_thread, world_unavailable`

```lua
-- The taxi order sets the customer's GPS to the pickup point
RegisterNetEvent("taxi:pickup", function(position)
    Open77.blips.setWaypoint(position)
end)
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 25480) as `LuaBlipSetWaypoint`.

## sprites

```lua
Open77.blips.sprites()
```

`GAME`: Requires a live game instance.

Lists every usable mappin variant in the current build.

Returns 147 entries for Cyberpunk 2077 2.31. This metadata query does not require map mutation permission.

Returns: `array of { name, value }`

```lua
-- Print every usable sprite name once
for _, sprite in ipairs(Open77.blips.sprites()) do
    print(sprite.value, sprite.name)
end
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 25486) as `LuaBlipSprites`.

## track

```lua
Open77.blips.track(id)
```

`GAME`: Requires a live game instance.

Selects this blip as the vanilla manually tracked destination.

Follows the same internal `MappinSystem` operation as selecting a pin on the fullscreen map. This changes the tracked id; native road calculation additionally requires the blip to have been created with `routable = true`. Returns `changed = false` when this blip is already tracked.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |

Returns: `true`, `whether tracking changed`, `reason on failure`

```lua
-- Start GPS routing to the race start
local blip = Open77.blips.create({ position = raceStart, routable = true, label = "Street race" })
Open77.blips.track(blip)
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 25473) as `LuaBlipTrack`.

## untrack

```lua
Open77.blips.untrack(id)
```

`GAME`: Requires a live game instance.

Safely clears manual tracking for this blip.

The native action is invoked only if this owned blip is currently tracked, so a resource cannot untrack a quest or another system's pin.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |

Returns: `true`, `whether it was tracked`

```lua
-- Stop routing once the player arrives
RegisterNetEvent("race:arrived", function()
    Open77.blips.untrack(raceBlip)
end)
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 25474) as `LuaBlipUntrack`.

## update

```lua
Open77.blips.update(id, patch)
```

`GAME`: Requires a live game instance.

Updates several properties of an owned blip.

Updates mutable fields on an owned blip. Visual-only icon/color changes preserve its native identity and tracking selection. Structural changes register a replacement before removing the previous mappin. position switches to positional mode; entity switches to attached mode. routable cannot be changed. Invalid SVG or color replacements leave the existing visual unchanged. On Cyberpunk 2077 2.5, use Stable 156 or later for corrected map, minimap, legend and world presentation. Existing options remain compatible.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |
| `patch` | `table` | required | — |

Returns: `boolean`, `reason`

```lua
-- Several changes in one operation
Open77.blips.update(jobBlip, { label = "Drop-off", sprite = "objective", active = true })
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 23006) as `LuaBlipUpdate`.

## waypoint

```lua
Open77.blips.waypoint()
```

`GAME`: Requires a live game instance.

Where the player's map waypoint is, or nil when there is none.

Reads the vanilla waypoint, the manually tracked mappin, whoever set it: this resource, another resource, or the player right-clicking the fullscreen map. `nil` with no reason means there is simply no waypoint; `nil` WITH a reason means the question could not be asked. A GPS script has to tell those apart, because cancelling a fare is correct for only the first. Requires `ui.vanilla.map`.

Returns: `table { x, y, z }, or nil when no waypoint is set`, `reason (only on failure): permission_denied:ui.vanilla.map, waypoint_unavailable_on_this_host, native_unavailable, wrong_thread, world_unavailable`

```lua
-- The taxi drives to wherever the customer put their map waypoint
local to = Open77.blips.waypoint()
if to then
    TriggerServerEvent("taxi:order", to)
else
    Open77.hud.notify("Set a waypoint on the map first")
end
```

[Custom SVG blips & colors](../../01%20Documentation/custom-blip-icons.md)

Registered in `ResourceHost.cpp` (line 25479) as `LuaBlipWaypoint`.
