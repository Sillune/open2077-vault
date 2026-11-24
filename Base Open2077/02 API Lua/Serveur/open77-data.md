# Open77.data (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-data.md) · [Page web](https://open2077.net/docs/api/server/open77-data)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

5 functions.

## item

```lua
Open77.data.item(record)
```

`NETWORK`: Uses the network backend.

Looks up an `Items.*` clothing or item record in the shipped 2.31 catalogue.

No permission required; this is static game data. A `record` is a **TweakDB string**, never a Jenkins hash -- the same choice the `Citizen` alias layer's `GetHashKey` already made, and a resource ported from FiveM must expect it. The server answers from a reduced catalogue embedded in the host and parsed lazily on first use; a **client** answers the same call from the live TweakDB, in the player's own language. Read `source` to tell them apart: `catalogue` here, `tweakdb` there. A field that could only be derived says so -- `seatsSource` of `class` on a vehicle means the seat count came from the chassis class in the record name, because no repository source holds a real one. An unknown record fails with `record_unknown`. See [Game data catalogues](https://open2077.net/docs/api/server/data-catalogues.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `record` | `string` | required | — |

Returns: `{ record, kind, source, build, ... }, or nil`, `reason`

```lua
-- Validate a shop entry against the shipped catalogue at startup
for _, entry in ipairs(shopItems) do
    assert(Open77.data.item(entry.record), "unknown item " .. entry.record)
end
```

[Game data catalogues guide](../../01%20Documentation/data-catalogues.md)

Registered in `LuaResourceRuntime.cs` (line 8691) as `LuaResourceRuntime.Bootstrap`.

## localize

```lua
Open77.data.localize(key)
```

`NETWORK`: Uses the network backend.

Refused on the server: localisation only exists inside the game.

Always answers `nil, "localization_client_only"`. The locale tables live in the game's own resources, which a dedicated server does not have, and handing back the key would let it pass for a translation. Send the row's `localeKey` to a client and call `Open77.data.localize` there. See [Game data catalogues](https://open2077.net/docs/api/server/data-catalogues.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string|integer` | required | — |

Returns: `nil`, `reason`

```lua
-- Always refused on the server: send the key to the client instead
local text, reason = Open77.data.localize("LocKey#12345") -- nil, "localization_client_only"
TriggerClientEvent("ui:localize", playerId, "LocKey#12345")
```

[Game data catalogues guide](../../01%20Documentation/data-catalogues.md)

Registered in `LuaResourceRuntime.cs` (line 8697) as `LuaResourceRuntime.Bootstrap`.

## npc

```lua
Open77.data.npc(template)
```

`NETWORK`: Uses the network backend.

Looks up an NPC entity template path in the shipped 2.31 catalogue.

No permission required; this is static game data. A `record` is a **TweakDB string**, never a Jenkins hash -- the same choice the `Citizen` alias layer's `GetHashKey` already made, and a resource ported from FiveM must expect it. The server answers from a reduced catalogue embedded in the host and parsed lazily on first use; a **client** answers the same call from the live TweakDB, in the player's own language. Read `source` to tell them apart: `catalogue` here, `tweakdb` there. A field that could only be derived says so -- `seatsSource` of `class` on a vehicle means the seat count came from the chassis class in the record name, because no repository source holds a real one. An unknown record fails with `record_unknown`. See [Game data catalogues](https://open2077.net/docs/api/server/data-catalogues.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `template` | `string` | required | — |

Returns: `{ record, kind, source, build, ... }, or nil`, `reason`

```lua
-- Check a spawn request against the catalogue before creating the NPC
local info = Open77.data.npc(record)
if not info then return Open77.chat.send(source, "Unknown character record.") end
```

[Game data catalogues guide](../../01%20Documentation/data-catalogues.md)

Registered in `LuaResourceRuntime.cs` (line 8692) as `LuaResourceRuntime.Bootstrap`.

## vehicle

```lua
Open77.data.vehicle(record)
```

`NETWORK`: Uses the network backend.

Looks up a `Vehicle.*` record in the shipped 2.31 catalogue.

No permission required; this is static game data. A `record` is a **TweakDB string**, never a Jenkins hash -- the same choice the `Citizen` alias layer's `GetHashKey` already made, and a resource ported from FiveM must expect it. The server answers from a reduced catalogue embedded in the host and parsed lazily on first use; a **client** answers the same call from the live TweakDB, in the player's own language. Read `source` to tell them apart: `catalogue` here, `tweakdb` there. A field that could only be derived says so -- `seatsSource` of `class` on a vehicle means the seat count came from the chassis class in the record name, because no repository source holds a real one. An unknown record fails with `record_unknown`. See [Game data catalogues](https://open2077.net/docs/api/server/data-catalogues.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `record` | `string` | required | — |

Returns: `{ record, kind, source, build, ... }, or nil`, `reason`

```lua
-- The dealership only sells records the catalogue knows, and prices by class
local car = Open77.data.vehicle("Vehicle.v_sport2_quadra_turbo_r")
if car then print(car.classLabel, car.seats) end
```

[Game data catalogues guide](../../01%20Documentation/data-catalogues.md)

Registered in `LuaResourceRuntime.cs` (line 8689) as `LuaResourceRuntime.Bootstrap`.

## weapon

```lua
Open77.data.weapon(record)
```

`NETWORK`: Uses the network backend.

Looks up a weapon record or hexadecimal TweakDBID in the shipped 2.31 catalogue.

No permission required; this is static game data. On the server, `record` accepts either an `Items.*` TweakDB record name or a hexadecimal TweakDBID **string**, such as `"0x0000001e1601aa89"` (hex digits and the `0x` prefix are case-insensitive); it is not a Jenkins hash or a Lua numeric ID. The embedded weapons catalogue resolves either form to the same record and class metadata. The result has `kind = "weapon"`, `source = "catalogue"` and `build = "2.31"`. This is authoritative catalogue classification, not proof of weapon ownership or a hit, native RPG damage, or a damage price. Unknown records or IDs and malformed hex return `nil, "record_unknown"`; non-string, empty or over-256-character input returns `nil, "invalid_record"`. The client API is unchanged: it looks up a record name in live TweakDB, in the player's language, with `source = "tweakdb"`. See [Game data catalogues](https://open2077.net/docs/api/server/data-catalogues.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `record` | `string (Items.* record name or hexadecimal TweakDBID)` | required | — |

Returns: `{ record, displayName, class, category, itemType, equipArea, quality, localeKey, tweakDbId, canonical, kind, source, build }, or nil`, `reason`

```lua
-- Refuse an unknown weapon record before giving it to the player
if not Open77.data.weapon(record) then return Open77.chat.send(source, "No such weapon.") end
```

[Game data catalogues guide](../../01%20Documentation/data-catalogues.md)

Registered in `LuaResourceRuntime.cs` (line 8690) as `LuaResourceRuntime.Bootstrap`.
