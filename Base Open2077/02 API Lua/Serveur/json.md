# json (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/json.md) · [Page web](https://open2077.net/docs/api/server/json)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

2 functions.

## decode

```lua
json.decode(text)
```

`SHARED`: Available without a live game instance.

Decodes JSON into safe Lua values.

Produces only strings, numbers, booleans and tables of those, within the same depth and node bounds as the encoder. Malformed input answers nil rather than raising, so a caller can tell a bad payload from an empty one.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `text` | `string` | required | — |

Returns: `value, or nil`

```lua
-- Parse the props column of a garage row
local props = json.decode(row.props)
if not props then print("corrupt garage row", row.id) end
```

Registered in `LuaResourceRuntime.cs` (line 6260) as `LuaResourceRuntime.Bootstrap`.

## encode

```lua
json.encode(value)
```

`SHARED`: Available without a live game instance.

Serialises a bounded Lua value to JSON.

The same encoder the event and export marshallers use, so a value this accepts is a value that crosses a resource boundary. Functions, userdata, cyclic tables and non-finite numbers are refused rather than guessed at. `json` is a plain global on the server; the client spells it Open77.json.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `value` | `any` | required | — |

Returns: `string, or nil`

```lua
-- Store a table as JSON in the kvp store
Open77.kvp.set("garage:" .. identifier, json.encode(props))
```

Registered in `LuaResourceRuntime.cs` (line 6260) as `LuaResourceRuntime.Bootstrap`.
