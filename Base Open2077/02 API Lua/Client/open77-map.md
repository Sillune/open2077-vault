# Open77.map (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-map.md) · [Page web](https://open2077.net/docs/api/client/open77-map)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

21 functions.

## addTab

```lua
Open77.map.addTab(options)
```

`GAME`: Requires a live game instance.

Add a resource WebUI tab beside the native Map tab.

Client-only, map.control; after resource preparation. Options: id, label, exactly one of url (declared local web file) or page (owned, hidden WebUI), optional integer order. Returns qualified resource:id and the page. Max five custom tabs. Shell owns visibility/focus/bounds; pages keep state between visits. See native-map.md for events, ready handshake, errors and cleanup.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table` | required | — |

Returns: `id, page; or nil, reason`

```lua
AddEventHandler('onClientResourceStart', function()
    local id, page = Open77.map.addTab({id='jobs', label='Jobs', url='web/jobs.html', order=10})
    if not id then print(page) end
end)
```

[Native map & waypoints guide](../../01%20Documentation/native-map.md)

Registered in `ResourceHost.cpp` (line 25357) as `LuaMapAddTab`.

## cancelPick

```lua
Open77.map.cancelPick()
```

`GAME`: Requires a live game instance.

Cancels this resource's outstanding native map request.

Requires map.control. Returns true or false, reason. The requester receives open77:map:pointPicked with status='cancelled' when a picker was active. Resource shutdown also releases its UI ownership without delivering events to its replacement instance.

Returns: `boolean, or false, reason`

```lua
-- ESC cancels the pending "choose a destination" picker
RegisterKeyMapping("taxi_cancel", "Cancel destination", "BACKSPACE", function()
    Open77.map.cancelPick()
end)
```

[Native map & waypoints guide](../../01%20Documentation/native-map.md)

Registered in `ResourceHost.cpp` (line 25368) as `LuaMapCancelPick`.

## close

```lua
Open77.map.close()
```

`GAME`: Requires a live game instance.

Closes a map session opened by this resource.

Requires map.control. Returns true on accepted closure, otherwise false, reason. Cannot close a user-opened map or another resource's map. Closure is asynchronous. A pending picker is cancelled.

Returns: `boolean, or false, reason`

```lua
-- Close the map this resource opened once the destination is chosen
AddEventHandler("open77:map:pointPicked", function(result)
    if result.status == "selected" then Open77.map.close() end
end)
```

[Native map & waypoints guide](../../01%20Documentation/native-map.md)

Registered in `ResourceHost.cpp` (line 25366) as `LuaMapClose`.

## focus

```lua
Open77.map.focus(position)
```

`GAME`: Requires a live game instance.

Focus an owned map on a world position without moving the player.

Requires map.control and ownership acquired through open or pickPoint. Pass finite x/y/z within +/-16000. Returns an asynchronous request ID; commandResult reports application. Rejects closed, busy, unready or foreign-owned maps. See native-map.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `position` | `table` | required | — |

Returns: `requestId, or nil, reason`

```lua
-- After opening this resource's map and observing ready=true:
local request, reason = Open77.map.focus({x=600, y=-2200, z=180})
```

[Native map & waypoints guide](../../01%20Documentation/native-map.md)

Registered in `ResourceHost.cpp` (line 25361) as `LuaMapFocus`.

## getScreen

```lua
Open77.map.getScreen()
```

`GAME`: Requires a live game instance.

Read the native map shell, appearance and registered tabs.

Client-only, map.read. Returns title, accentColor, activeTab, open, revision and tabs. Available while closed. Map is built in; resource tabs are namespaced. See native-map.md for WebUI lifecycle and ownership.

Returns: `screen, or nil, reason`

```lua
local screen = Open77.map.getScreen()
if screen then print(screen.title, screen.activeTab) end
```

[Native map & waypoints guide](../../01%20Documentation/native-map.md)

Registered in `ResourceHost.cpp` (line 25353) as `LuaMapGetScreen`.

## getSelectedMarker

```lua
Open77.map.getSelectedMarker()
```

`GAME`: Requires a live game instance.

Reads the marker highlighted in the vanilla map.

Requires map.read. Returns a marker table or nil. kind distinguishes waypoint, quest, poi, resource and unknown. Selection is not the same as the player's waypoint; highlighting a marker does not change navigation.

Returns: `value, or nil, reason`

```lua
-- Read what the player highlighted on the map
local marker = Open77.map.getSelectedMarker()
if marker then print(marker.kind, marker.position.x, marker.position.y) end
```

[Native map & waypoints guide](../../01%20Documentation/native-map.md)

Registered in `ResourceHost.cpp` (line 25349) as `LuaMapGetSelectedMarker`.

## getView

```lua
Open77.map.getView()
```

`GAME`: Requires a live game instance.

Read the native map camera and readiness snapshot.

Requires map.read. Returns ready, zoomLevel, zoomLevels, zoom and cameraMode, or nil, reason. The native scalar zoom is not a world distance. Wait for ready before controlling an owned map. See native-map.md.

Returns: `view, or nil, reason`

```lua
local view, reason = Open77.map.getView()
if view and view.ready then print(view.cameraMode, view.zoomLevel) end
```

[Native map & waypoints guide](../../01%20Documentation/native-map.md)

Registered in `ResourceHost.cpp` (line 25352) as `LuaMapGetView`.

## getWaypoint

```lua
Open77.map.getWaypoint()
```

`GAME`: Requires a live game instance.

Reads the player's manual vanilla waypoint.

Returns nil (without an error) when no player waypoint is tracked. Quest, POI and Open77 resource markers are excluded. Requires map.read. The result has a string id, kind='waypoint', source='player', position={x,y,z} and revision. Errors return nil, reason; map_unavailable means the world snapshot is unavailable or stale.

Returns: `value, or nil, reason`

```lua
-- Send the player's manual waypoint to the server as the taxi destination
local waypoint = Open77.map.getWaypoint()
if waypoint then TriggerServerEvent("taxi:order", waypoint.position) end
```

[Native map & waypoints guide](../../01%20Documentation/native-map.md)

Registered in `ResourceHost.cpp` (line 25348) as `LuaMapGetWaypoint`.

## isOpen

```lua
Open77.map.isOpen()
```

`GAME`: Requires a live game instance.

Checks whether the vanilla world map is open.

Requires map.read. Returns a boolean, or nil, reason when map state is unavailable. Uses native map controller lifecycle callbacks.

Returns: `value, or nil, reason`

```lua
-- Pause the HUD timer while the map is open
SetTick(function()
    if not Open77.map.isOpen() then timer = timer + 1 end
end)
```

[Native map & waypoints guide](../../01%20Documentation/native-map.md)

Registered in `ResourceHost.cpp` (line 25350) as `LuaMapIsOpen`.

## open

```lua
Open77.map.open([options])
```

`GAME`: Requires a live game instance.

Opens the standalone native map, optionally focused on a world location.

Requires map.control. options.position={x,y,z} optionally focuses the map. Returns requestId, or nil, reason. Acceptance is asynchronous; observe open77:map:opened or open77:map:requestFailed. Respects game UI restrictions and exclusive ownership; map_busy means another map session exists.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table` | optional | — |

Returns: `requestId, or nil, reason`

```lua
-- Open the map focused on the meeting point
Open77.map.open({ position = { x = -1442.2, y = 127.4, z = 18.0 } })
```

[Native map & waypoints guide](../../01%20Documentation/native-map.md)

Registered in `ResourceHost.cpp` (line 25365) as `LuaMapOpen`.

## pickPoint

```lua
Open77.map.pickPoint([options])
```

`GAME`: Requires a live game instance.

Lets the player select a native map point without travel.

Requires map.control. Optional position={x,y,z} and timeoutMilliseconds (1000..300000, default 60000). Returns requestId or nil, reason. open77:map:pointPicked is delivered only to the requesting resource, with requestId, status, position and marker. Use the vanilla waypoint gesture to select. Fast travel, taxi confirmation and Open77 admin map teleport are suppressed during the picker.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table` | optional | — |

Returns: `requestId, or nil, reason`

```lua
-- Let the player pick a taxi destination on the map, one minute to decide
local requestId = Open77.map.pickPoint({ timeoutMilliseconds = 60000 })
AddEventHandler("open77:map:pointPicked", function(result)
    if result.requestId ~= requestId then return end
    if result.status == "selected" then TriggerServerEvent("taxi:order", result.position) end
end)
```

[Native map & waypoints guide](../../01%20Documentation/native-map.md)

Registered in `ResourceHost.cpp` (line 25367) as `LuaMapPickPoint`.

## recenter

```lua
Open77.map.recenter()
```

`GAME`: Requires a live game instance.

Recenter an owned native map on the local player.

Requires map.control and an open, ready map owned by this resource. Moves only the map camera. Returns a request ID or nil, reason; observe open77:map:commandResult. User-opened maps cannot be hijacked.

Returns: `requestId, or nil, reason`

```lua
local request, reason = Open77.map.recenter()
if not request then print(reason) end
```

[Native map & waypoints guide](../../01%20Documentation/native-map.md)

Registered in `ResourceHost.cpp` (line 25362) as `LuaMapRecenter`.

## removeTab

```lua
Open77.map.removeTab(id)
```

`GAME`: Requires a live game instance.

Unregister this resource's map shell tab.

Client-only, map.control. Local or qualified owned ID. Cannot remove Map or a foreign tab. Active removal returns to Map and releases focus. The page remains hidden; explicitly destroy it when no longer needed. Resource-stop cleanup is automatic.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |

Returns: `true, or nil, reason`

```lua
local ok, reason = Open77.map.removeTab('jobs')
```

[Native map & waypoints guide](../../01%20Documentation/native-map.md)

Registered in `ResourceHost.cpp` (line 25358) as `LuaMapRemoveTab`.

## resetAppearance

```lua
Open77.map.resetAppearance()
```

`GAME`: Requires a live game instance.

Release this resource's map title and accent overrides.

Client-only, map.control; after resource preparation. Restores preceding resource claims or defaults, without clearing another resource's settings or unregistering any tabs.

Returns: `true, or nil, reason`

```lua
local ok, reason = Open77.map.resetAppearance()
```

[Native map & waypoints guide](../../01%20Documentation/native-map.md)

Registered in `ResourceHost.cpp` (line 25356) as `LuaMapResetAppearance`.

## selectTab

```lua
Open77.map.selectTab(id)
```

`GAME`: Requires a live game instance.

Select an owned WebUI tab or the native Map on an open shell.

Client-only, map.control. Accepts own local/qualified ID or map. Requires an open shell, rejects point-picker sessions and foreign tabs. Returns queue acceptance; applied next tick. Observe tabEntered/tabLeft (owner), tabChanged (map.read), or JS tabState. Does not implicitly open the map.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |

Returns: `true, or nil, reason`

```lua
local ok, reason = Open77.map.selectTab('map')
```

[Native map & waypoints guide](../../01%20Documentation/native-map.md)

Registered in `ResourceHost.cpp` (line 25359) as `LuaMapSelectTab`.

## setAccentColor

```lua
Open77.map.setAccentColor(color)
```

`GAME`: Requires a live game instance.

Set the native map shell's resource-owned accent color.

Client-only, map.control; after resource preparation. Exact #RRGGBB string. Updates the selected native tab and supplies the color to resource WebUIs through screenChanged/tabState events. Does not recolor the native roads or markers. See native-map.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `color` | `string` | required | — |

Returns: `true, or nil, reason`

```lua
AddEventHandler('onClientResourceStart', function()
    Open77.map.setAccentColor('#FFB347')
end)
```

[Native map & waypoints guide](../../01%20Documentation/native-map.md)

Registered in `ResourceHost.cpp` (line 25355) as `LuaMapSetAccent`.

## setCameraMode

```lua
Open77.map.setCameraMode(mode)
```

`GAME`: Requires a live game instance.

Select the native top-down or perspective map camera.

Requires map.control and ownership of an open, ready map. mode must be top_down or perspective. Returns a request ID or nil, reason. Never changes the gameplay camera or teleports the player. See native-map.md for asynchronous result events and cleanup.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `mode` | `string` | required | — |

Returns: `requestId, or nil, reason`

```lua
local request, reason = Open77.map.setCameraMode("perspective")
```

[Native map & waypoints guide](../../01%20Documentation/native-map.md)

Registered in `ResourceHost.cpp` (line 25364) as `LuaMapSetCameraMode`.

## setTabLabel

```lua
Open77.map.setTabLabel(id, label)
```

`GAME`: Requires a live game instance.

Rename this resource's native map shell tab.

Client-only, map.control. Local or qualified owned ID; label is 1-40 UTF-8 bytes, without controls or pipe. Updates live without recreating the browser or losing page state. Map and other resources' tabs are immutable to the caller.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |
| `label` | `string` | required | — |

Returns: `true, or nil, reason`

```lua
local ok, reason = Open77.map.setTabLabel('jobs', 'Contracts')
```

[Native map & waypoints guide](../../01%20Documentation/native-map.md)

Registered in `ResourceHost.cpp` (line 25360) as `LuaMapSetTabLabel`.

## setTitle

```lua
Open77.map.setTitle(text)
```

`GAME`: Requires a live game instance.

Customize the upper-left native map shell title.

Client-only, map.control; call after resource preparation. 1-120 UTF-8 bytes, no control characters or pipe. Resource-owned override: latest setter wins, resetAppearance or resource stop restores the preceding claim. See native-map.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `text` | `string` | required | — |

Returns: `true, or nil, reason`

```lua
AddEventHandler('onClientResourceStart', function()
    Open77.map.setTitle('NIGHT CITY ROLEPLAY')
end)
```

[Native map & waypoints guide](../../01%20Documentation/native-map.md)

Registered in `ResourceHost.cpp` (line 25354) as `LuaMapSetTitle`.

## setZoomLevel

```lua
Open77.map.setZoomLevel(level)
```

`GAME`: Requires a live game instance.

Select a native discrete zoom level on an owned map.

Requires map.control and ownership of an open, ready map. level is an integer from 0 to getView().zoomLevels-1. Returns request ID or nil, reason. The ID acknowledges queueing, not camera-transition completion; observe commandResult and viewChanged.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `level` | `integer` | required | — |

Returns: `requestId, or nil, reason`

```lua
local view = Open77.map.getView()
if view and view.ready and view.zoomLevels > 2 then Open77.map.setZoomLevel(2) end
```

[Native map & waypoints guide](../../01%20Documentation/native-map.md)

Registered in `ResourceHost.cpp` (line 25363) as `LuaMapSetZoomLevel`.

## state

```lua
Open77.map.state()
```

`GAME`: Requires a live game instance.

Reads a coherent native map snapshot.

Requires map.read. Returns open, picking, waypoint, selectedMarker, revision and sampleAgeMs, or nil, reason. Read this on resource startup before subscribing to change events; no historical placement event is synthesized.

Returns: `value, or nil, reason`

```lua
-- Read the whole map state once at startup, before subscribing to events
local map = Open77.map.state()
if map and map.waypoint then print("waypoint already set at", map.waypoint.position.x) end
```

[Native map & waypoints guide](../../01%20Documentation/native-map.md)

Registered in `ResourceHost.cpp` (line 25351) as `LuaMapState`.
