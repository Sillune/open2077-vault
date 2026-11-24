# Open77.inspector (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-inspector.md) · [Page web](https://open2077.net/docs/api/client/open77-inspector)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

3 functions.

## enable

```lua
Open77.inspector.enable([enabled])
```

`GAME`: Requires a live game instance.

Turns the permanent aim ray on or off.

Turning it off also clears the outline: the outline belongs to the session that asked for it and must not outlive it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | optional | `true` |

Returns: `true`, `reason, when the first is nil`

```lua
-- Turn the permanent aim ray on while the admin tool is open
AddEventHandler("admin:opened", function() Open77.inspector.enable(true) end)
AddEventHandler("admin:closed", function() Open77.inspector.enable(false) end)
```

Registered in `ResourceHost.cpp` (line 25004) as `LuaInspectorEnable`.

## outline

```lua
Open77.inspector.outline([enabled])
```

`GAME`: Requires a live game instance.

Outlines the aimed object, independently of the ray.

Lets a caller read the target without painting the world. The outline uses `entRenderHighlightEvent`, the same event focus mode and scanning already use.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | optional | `true` |

Returns: `true`, `reason, when the first is nil`

```lua
-- Highlight what the player aims at, without the ray overlay
Open77.inspector.outline(true)
```

Registered in `ResourceHost.cpp` (line 25005) as `LuaInspectorOutline`.

## target

```lua
Open77.inspector.target()
```

`GAME`: Requires a live game instance.

What the aim ray is currently pointing at.

Reads a snapshot, never a probe. The ray has to be cast on the game thread, and the plugin refreshes it at 10 Hz, so calling this costs nothing and cannot land a raycast on the wrong thread.

Always returns a table when a game is present. `valid` separates "nothing in view" from "no game" — the latter returns `nil, reason`.

Returns: `table { valid, engineId, class, name, kind, position, distance }`, `reason, when the first is nil`

```lua
local t = Open77.inspector.target()
if t and t.valid then
    print(t.kind, t.name, t.distance)   --> door  Door  1.38
end
```

Registered in `ResourceHost.cpp` (line 25003) as `LuaInspectorTarget`.
