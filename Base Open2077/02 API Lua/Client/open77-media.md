# Open77.media (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-media.md) · [Page web](https://open2077.net/docs/api/client/open77-media)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

5 functions.

## bind

```lua
Open77.media.bind(definition)
```

`GAME`: Requires a live game instance.

Creates a resource-owned world media screen.

Client-only; requires world.props and an available media backend. Binds a validated media definition to a prop or owned WebUI page. Returns the opaque screen ID as a string, or nil and a reason. The binding is local and owned by the calling resource; use the open77_media resource for its higher-level playback workflow.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `definition` | `table` | required | — |

Returns: `screen ID or nil`, `reason on failure`

Registered in `ResourceHost.cpp` (line 25517) as `LuaMediaBind`.

## clear

```lua
Open77.media.clear()
```

`GAME`: Requires a live game instance.

Releases all media-screen bindings owned by this resource.

Client-only; requires world.props. Requests resource-scoped cleanup of every local media screen and returns true. Does not remove another resource's screens. Safe when no screens exist; backend cleanup also participates in resource teardown.

Returns: `boolean success`, `reason on failure`

Registered in `ResourceHost.cpp` (line 25520) as `LuaMediaClear`.

## list

```lua
Open77.media.list()
```

`GAME`: Requires a live game instance.

Lists the calling resource's local media screens.

Client-only; requires world.props and an available media backend. Returns a snapshot array including screen IDs, definitions, drawn state, reason, distance, facing diagnostics and quad dimensions. This is local presentation state, not a server-global catalogue; other resource owners' screens are excluded.

Returns: `screen snapshot array, or false`, `reason on failure`

Registered in `ResourceHost.cpp` (line 25521) as `LuaMediaList`.

## unbind

```lua
Open77.media.unbind(id)
```

`GAME`: Requires a live game instance.

Removes an owned media-screen binding.

Client-only; requires world.props. Releases a screen by its opaque ID through the resource-scoped backend. Does not delete a shared prop or another resource's WebUI page. Returns false and a reason for invalid IDs, unavailable backends or refused ownership.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |

Returns: `boolean success`, `reason on failure`

Registered in `ResourceHost.cpp` (line 25519) as `LuaMediaUnbind`.

## update

```lua
Open77.media.update(id, patch)
```

`GAME`: Requires a live game instance.

Updates one of this resource's media screens.

Client-only; requires world.props. Reads the existing owned screen definition, applies and validates the supplied patch, then requests a backend update. An unknown or foreign ID fails with not_found; an invalid ID fails with invalid_screen_id. Other resources' bindings cannot be modified.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |
| `patch` | `table` | required | — |

Returns: `boolean success`, `reason on failure`

Registered in `ResourceHost.cpp` (line 25518) as `LuaMediaUpdate`.
