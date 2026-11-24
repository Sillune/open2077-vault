# Open77.json (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-json.md) · [Page web](https://open2077.net/docs/api/client/open77-json)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

2 functions.

## decode

```lua
Open77.json.decode(text)
```

`SHARED`: Available without a live game instance.

Parses a JSON string.

Parses JSON into Open77's bounded Lua value model. Invalid JSON returns `nil, "invalid_json"`; decoded objects and arrays contain only supported scalar/table values.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `text` | `string` | required | — |

Returns: `value, or nil`, `reason`

```lua
-- Parse a config file shipped with the resource
local text = Open77.resource.readFile("config.json")
local config, reason = Open77.json.decode(text or "")
if not config then print("bad config: " .. tostring(reason)) end
```

Registered in `ResourceHost.cpp` (line 25569) as `LuaJsonDecode`.

## encode

```lua
Open77.json.encode(value)
```

`SHARED`: Available without a live game instance.

Serialises a Lua value to JSON.

Serializes a supported Lua value to JSON using the same bounded representation used by events, exports and WebUI. Functions, userdata, cycles and unsupported table shapes return `nil, reason`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `value` | `any` | required | — |

Returns: `string, or nil`, `reason`

```lua
-- Save a table as a string in the kvp store
Open77.kvp.set("hud_layout", Open77.json.encode({ x = 0.1, y = 0.9 }))
```

Registered in `ResourceHost.cpp` (line 25568) as `LuaJsonEncode`.
