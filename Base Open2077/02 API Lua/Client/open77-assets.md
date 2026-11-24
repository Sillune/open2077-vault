# Open77.assets (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-assets.md) · [Page web](https://open2077.net/docs/api/client/open77-assets)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

2 functions.

## list

```lua
Open77.assets.list()
```

`SHARED`: Available without a live game instance.

Lists generic files declared by the current resource.

Returns the resource files declared by the current manifest. It is an allow-list view, not a directory listing, and does not grant access to undeclared files or arbitrary paths.

Returns: `array of relative paths`

```lua
-- Print every file this resource declared
for _, path in ipairs(Open77.assets.list()) do print(path) end
```

Registered in `ResourceHost.cpp` (line 24621) as `LuaAssetList`.

## texture

```lua
Open77.assets.texture(path)
```

`SHARED`: Available without a live game instance.

Validates a declared PNG texture for resource APIs.

The path must match the manifest's `files` allowlist. PNG textures are limited to 512 KiB and 512 by 512 pixels.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `path` | `string` | required | — |

Returns: `descriptor { type, asset, mime, width, height, bytes }, or nil`, `reason`

```lua
-- Validate the blip icon before using it
local icon, reason = Open77.assets.texture("icons/gang.png")
if icon then Open77.blips.setIcon(hqBlip, icon) else print("bad icon: " .. reason) end
```

Registered in `ResourceHost.cpp` (line 24620) as `LuaAssetTexture`.
