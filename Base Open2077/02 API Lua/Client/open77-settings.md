# Open77.settings (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-settings.md) · [Page web](https://open2077.net/docs/api/client/open77-settings)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

3 functions.

## request

```lua
Open77.settings.request(group)
```

`GAME`: Requires a live game instance.

Asks the script bridge to list the variables of a group.

Asynchronous: request, then read with `values` a beat later.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `group` | `string` | required | — |

Returns: `boolean`, `reason`

```lua
-- Ask for the graphics group, then read it a beat later
CreateThread(function()
    Open77.settings.request("graphics")
    Wait(250)
    for _, v in ipairs(Open77.settings.values()) do print(v.name, v.value) end
end)
```

Registered in `ResourceHost.cpp` (line 24757) as `LuaSettingsRequest`.

## set

```lua
Open77.settings.set(group, name, value)
```

`GAME`: Requires a live game instance.

Writes a settings variable.

Submits one local Open77 setting change through the settings backend. The key, value type and published bounds are validated; use `values()` to read the resulting projection.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `group` | `string` | required | — |
| `name` | `string` | required | — |
| `value` | `integer` | required | — |

Returns: `boolean`, `reason`

```lua
-- Write one setting from a custom settings page
Open77.settings.set("audio", "master_volume", 80)
```

Registered in `ResourceHost.cpp` (line 24759) as `LuaSettingsSet`.

## values

```lua
Open77.settings.values()
```

`GAME`: Requires a live game instance.

The variables as the script last reported them.

Returns a snapshot of the settings currently published by the client backend. The returned table is a copy and modifying it does not apply changes.

Returns: `table of { name, value, minimum, maximum, step }`

```lua
-- Fill a settings page from the last reported values
for _, v in ipairs(Open77.settings.values()) do
    print(("%s = %d (%d..%d)"):format(v.name, v.value, v.minimum, v.maximum))
end
```

Registered in `ResourceHost.cpp` (line 24758) as `LuaSettingsValues`.
