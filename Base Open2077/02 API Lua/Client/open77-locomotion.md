# Open77.locomotion (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-locomotion.md) · [Page web](https://open2077.net/docs/api/client/open77-locomotion)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

1 function.

## register

```lua
Open77.locomotion.register(path)
```

`GAME`: Requires a live game instance.

Register a custom locomotion package.

Requires locomotion.register. Reads a version 1 package metadata file relative to this resource. The shared registration script runs on both runtimes. The client validates the graph and mounted native assets; registration does not assign a style. Stable 147 supports 1,024 registered packages across resources/reload generations; existing standalone packages stay compatible. See [Custom locomotion](https://open2077.net/docs/api/client/custom-locomotion.md#registration).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `path` | `string` | required | — |

Returns: `style ID string, or nil`, `reason on failure`

```lua
local style, reason = Open77.locomotion.register('package.json')
```

[Custom locomotion & collections guide](../../01%20Documentation/custom-locomotion.md)

Registered in `scripting/src/LocomotionBindings.inl` (line 160) as `LuaLocomotionRegister`.
