# Open77.anchors (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-anchors.md) · [Page web](https://open2077.net/docs/api/client/open77-anchors)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

6 functions.

## clear

```lua
Open77.anchors.clear()
```

`GAME`: Requires a live game instance.

Removes every world anchor owned by the calling resource.

Other resources are unaffected. Anchors are also cleared automatically on stop, reload, world exit, and when their page is destroyed.

Returns: `true on success, otherwise false`, `reason`

```lua
-- Leaving the mission removes every prompt this resource pinned
AddEventHandler("mission:ended", function()
    Open77.anchors.clear()
end)
```

Registered in `ResourceHost.cpp` (line 25528) as `LuaAnchorClear`.

## create

```lua
Open77.anchors.create(options)
```

`GAME`: Requires a live game instance.

Anchors a world point, projected by the plugin on every game frame.

No permission of its own: the capability is the one `Open77.camera.project` already grants ungated, with the sampling moved off the Lua tick.

Options require exactly one of `position` (a fixed world point) or `entity` (followed live, with an optional world-axis `offset`) -- passing both, or neither, is refused with `position_or_entity_required`. They also accept `tag` (`[A-Za-z0-9_.:-]`, 32 bytes), `maxDistance` (default 100.0), `minDistance` (default 0.0) and `visible` (default true).

`render` selects the delivery and is fixed for the anchor's life: `page` (the default) publishes coordinates to a WebUI page, while `card`, `ring` and `dot` are drawn natively by the plugin in the frame that presents them. `page` is REQUIRED only for `render = "page"`; a native style has nowhere to publish and needs no surface at all, though a page passed alongside one is still ownership-checked. A page this resource does not own is refused with `stale_or_unowned_page`.

Native styles take a `presentation` table: `label`, `sublabel` (192 bytes each), `key` (`card` only, 8 bytes), `color`, `accent`, `background` (`#RRGGBB` or `#RRGGBBAA`; anything else is refused as `invalid_color` / `invalid_accent` / `invalid_background`), `scale`, `thickness`, `radius` and `groundOffset` (`ring`), `progress` (`card`; negative draws no bar) and `showDistance`. The colour defaults are the OPEN//77 design tokens -- `color` `#F2F6F8`, `accent` `#22D8E2`, `background` `#0A1220F0` -- and they matter more than defaults usually do, because `presentation` is patched whole: see `Open77.anchors.update`.

The batched `open77:anchors` event a page receives is the COMPLETE set for that surface; an id that is absent is hidden, out of its distance band, behind the camera, or its entity is not streamed. Treat absence as gone rather than holding the last coordinate. Limits are 32 anchors per resource and 128 in total.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table` | required | — |

Returns: `anchor handle string, or nil`, `reason`

```lua
-- A native "press E" card pinned above the delivery crate, visible within 30 m
local anchor, reason = Open77.anchors.create({
    render = "card",
    position = { x = -1442.2, y = 127.4, z = 18.8 },
    maxDistance = 30,
    presentation = { label = "Deliver package", key = "E", accent = "#00D8FF" },
})
if not anchor then print("anchor refused: " .. reason) end
```

Registered in `ResourceHost.cpp` (line 25525) as `LuaAnchorCreate`.

## frame

```lua
Open77.anchors.frame()
```

`GAME`: Requires a live game instance.

Open77.anchors.frame() -> { { id, projected, onScreen, x, y, depth,

distance, resolved = {x, y, z} }, ... }

The per-frame half of `list()` and nothing else: what the plugin
projected on its last frame, for a loop that reads it every cycle. The
full row -- tag, render, entity, offset, surface, the distance band,
the declared position -- is the resource's own configuration, which it
wrote and does not need read back thirty times a second; `list()` still
carries all of it. Two tables and one string per anchor rather than
six and four, which at the 32-anchor quota is the difference between a
frame read that costs a cycle a few dozen microseconds and one that
costs it a hundred.

Returns: `value`

Registered in `ResourceHost.cpp` (line 25530) as `LuaAnchorFrame`.

_This signature was inferred from the handler body._

## list

```lua
Open77.anchors.list()
```

`GAME`: Requires a live game instance.

Lists the calling resource's world anchors with their last projection.

Each snapshot carries its requested definition plus `render` (echoing its style), `resolved` (where the anchor actually was on the last frame -- for an entity anchor the live transform plus the offset, not what was registered) and `screen` (`x`, `y` normalised to [0..1] with a top-left origin, and `depth` in metres in front of the camera), `distance`, `onScreen`, and `projected`. An invisible anchor is still projected and still listed, so this stays the way to decide which of a resource's world points deserve one of its 32 slots.

Returns: `array of { id, tag, kind, render, entity, offset, surface, maxDistance, minDistance, visible, projected, onScreen, distance, position, resolved, screen }, or nil`, `reason`

```lua
-- Which of our anchors is on screen and closest? That one gets the key
local best
for _, anchor in ipairs(Open77.anchors.list() or {}) do
    if anchor.onScreen and (not best or anchor.distance < best.distance) then best = anchor end
end
```

Registered in `ResourceHost.cpp` (line 25529) as `LuaAnchorList`.

## remove

```lua
Open77.anchors.remove(id)
```

`GAME`: Requires a live game instance.

Removes one resource-owned world anchor.

Removing an unknown handle succeeds, so teardown is idempotent.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- The crate was picked up: drop its prompt
RegisterNetEvent("delivery:done", function()
    Open77.anchors.remove(crateAnchor)
end)
```

Registered in `ResourceHost.cpp` (line 25527) as `LuaAnchorRemove`.

## update

```lua
Open77.anchors.update(id, patch)
```

`GAME`: Requires a live game instance.

Patches a resource-owned world anchor.

Top-level fields are patched individually: only what you supply changes. **`presentation` is the exception and is patched WHOLE.** `update(handle, { presentation = { label = "Locked" } })` does not keep the colours you set at creation -- it resets `color`, `accent`, `background` and every other presentation field to its default. Always resend the complete presentation table, or keep it in a local and patch that. A prompt changes its label, its key and its hold progress on the same frame or not at all, which is why the table has one dirty flag rather than eleven.

Neither `page` nor the anchor's kind is patchable; patching a field the kind does not own -- `position` on an entity anchor, `entity` or `offset` on a position anchor -- is refused with `anchor_kind_mismatch` rather than ignored. `render` is fixed at creation and a patch carrying it is refused with `render_is_not_patchable`. Re-pointing an entity anchor at a new `entity` is supported, which is the respawn case. Ownership prevents mutation of another resource's anchor.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |
| `patch` | `table` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Change the prompt text; presentation is replaced whole, so resend every field
Open77.anchors.update(crateAnchor, {
    presentation = { label = "Locked", key = "E", accent = "#FF3355" },
})
```

Registered in `ResourceHost.cpp` (line 25526) as `LuaAnchorUpdate`.
