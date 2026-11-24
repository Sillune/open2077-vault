# Open77.convars (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-convars.md) · [Page web](https://open2077.net/docs/api/server/open77-convars)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

4 functions.

## get

```lua
Open77.convars.get(name, [default])
```

`NETWORK`: Uses the network backend.

Namespaced spelling of `GetConvar`.

The same function value.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `default` | `any` | optional | — |

Returns: `string, or the default`

```lua
-- Namespaced spelling of GetConvar
local webhook = Open77.convars.get("myshop_webhook", "")
```

[Convars](../../01%20Documentation/server-api.md#convars)

Registered in `LuaResourceRuntime.Convars.cs` (line 148) as `LuaResourceRuntime.Bootstrap`.

## getBool

```lua
Open77.convars.getBool(name, [default])
```

`NETWORK`: Uses the network backend.

Namespaced spelling of `GetConvarBool`.

The same function value.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `default` | `boolean` | optional | — |

Returns: `boolean, or the default`

```lua
-- Namespaced spelling of GetConvarBool
Open77.combat.setFriendlyFire(Open77.convars.getBool("myserver_pvp", false))
```

[Convars](../../01%20Documentation/server-api.md#convars)

Registered in `LuaResourceRuntime.Convars.cs` (line 150) as `LuaResourceRuntime.Bootstrap`.

## getInt

```lua
Open77.convars.getInt(name, [default])
```

`NETWORK`: Uses the network backend.

Namespaced spelling of `GetConvarInt`.

The same function value.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `default` | `integer` | optional | — |

Returns: `integer, or the default`

```lua
-- Namespaced spelling of GetConvarInt
local startingMoney = Open77.convars.getInt("myserver_starting_money", 500)
```

[Convars](../../01%20Documentation/server-api.md#convars)

Registered in `LuaResourceRuntime.Convars.cs` (line 149) as `LuaResourceRuntime.Bootstrap`.

## set

```lua
Open77.convars.set(name, value)
```

`NETWORK`: Uses the network backend.

Namespaced spelling of `SetConvar`.

The same function value.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `value` | `string|number|boolean` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Namespaced spelling of SetConvar
Open77.convars.set("myserver_maintenance", true)
```

[Convars](../../01%20Documentation/server-api.md#convars)

Registered in `LuaResourceRuntime.Convars.cs` (line 151) as `LuaResourceRuntime.Bootstrap`.
