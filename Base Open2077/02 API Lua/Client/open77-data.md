# Open77.data (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-data.md) · [Page web](https://open2077.net/docs/api/client/open77-data)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

5 functions.

## item

```lua
Open77.data.item(record)
```

`GAME`: Requires a live game instance.

Looks up an `Items.*` clothing or item record in the game's live TweakDB.

No permission required; this is static game data. **The client ships no catalogue**: it is inside Cyberpunk 2077, so it reads the real database and gets the display name in the player's own language for zero shipped bytes. `source` is `tweakdb` here and `catalogue` on the server, which answers the same call from a reduced shipped table. `recordClass` -- the record's runtime RTTI class -- is always present once the lookup succeeds and is the honest local answer to FiveM's `GetEntityModel`; every other field is best effort, and an **absent field means the engine did not expose it**, not that the record lacks it. A `record` is a TweakDB string, never a Jenkins hash. Fails with `record_unknown`, `record_wrong_kind`, `invalid_record` or `tweakdb_unavailable`. See [Game data catalogues](https://open2077.net/docs/api/client/data-catalogues.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `record` | `string` | required | — |

Returns: `{ record, kind, recordClass, tweakDbId, source, ... }, or nil`, `reason`

```lua
-- Show the localized name of a clothing item in the shop
local info = Open77.data.item("Items.Cap_01_basic_01")
if info then print(info.displayName or info.record) end
```

[Game data catalogues guide](../../01%20Documentation/data-catalogues.md)

Registered in `ResourceHost.cpp` (line 24934) as `LuaDataRecord`.

## localize

```lua
Open77.data.localize(key)
```

`GAME`: Requires a live game instance.

Resolves a localisation key to the text the player would actually see.

No permission required. Accepts `LocKey#12345`, a bare numeric key -- so a `localeKey` read off a server catalogue row can be passed straight in -- or a secondary key string. The FiveM `GetLabelText` equivalent. Fails with `localization_unavailable` when the engine declines the key; the server refuses this call outright, because the locale tables only exist inside the game. See [Game data catalogues](https://open2077.net/docs/api/client/data-catalogues.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `string|integer` | required | — |

Returns: `text, or nil`, `reason`

```lua
-- Turn a locale key from a server catalogue row into the player's language
print(Open77.data.localize("LocKey#12345"))
```

[Game data catalogues guide](../../01%20Documentation/data-catalogues.md)

Registered in `ResourceHost.cpp` (line 24937) as `LuaDataLocalize`.

## npc

```lua
Open77.data.npc(template)
```

`GAME`: Requires a live game instance.

Looks up a `Character.*` NPC record in the game's live TweakDB.

No permission required; this is static game data. **The client ships no catalogue**: it is inside Cyberpunk 2077, so it reads the real database and gets the display name in the player's own language for zero shipped bytes. `source` is `tweakdb` here and `catalogue` on the server, which answers the same call from a reduced shipped table. `recordClass` -- the record's runtime RTTI class -- is always present once the lookup succeeds and is the honest local answer to FiveM's `GetEntityModel`; every other field is best effort, and an **absent field means the engine did not expose it**, not that the record lacks it. A `record` is a TweakDB string, never a Jenkins hash. Fails with `record_unknown`, `record_wrong_kind`, `invalid_record` or `tweakdb_unavailable`. See [Game data catalogues](https://open2077.net/docs/api/client/data-catalogues.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `template` | `string` | required | — |

Returns: `{ record, kind, recordClass, tweakDbId, source, ... }, or nil`, `reason`

```lua
-- Check that a record exists before asking the server to spawn it
if Open77.data.npc("Character.Judy") then TriggerServerEvent("npcs:spawn", "Character.Judy") end
```

[Game data catalogues guide](../../01%20Documentation/data-catalogues.md)

Registered in `ResourceHost.cpp` (line 24934) as `LuaDataRecord`.

## vehicle

```lua
Open77.data.vehicle(record)
```

`GAME`: Requires a live game instance.

Looks up a `Vehicle.*` record in the game's live TweakDB.

No permission required; this is static game data. **The client ships no catalogue**: it is inside Cyberpunk 2077, so it reads the real database and gets the display name in the player's own language for zero shipped bytes. `source` is `tweakdb` here and `catalogue` on the server, which answers the same call from a reduced shipped table. `recordClass` -- the record's runtime RTTI class -- is always present once the lookup succeeds and is the honest local answer to FiveM's `GetEntityModel`; every other field is best effort, and an **absent field means the engine did not expose it**, not that the record lacks it. A `record` is a TweakDB string, never a Jenkins hash. Fails with `record_unknown`, `record_wrong_kind`, `invalid_record` or `tweakdb_unavailable`. See [Game data catalogues](https://open2077.net/docs/api/client/data-catalogues.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `record` | `string` | required | — |

Returns: `{ record, kind, recordClass, tweakDbId, source, ... }, or nil`, `reason`

```lua
-- The dealership lists a car with its real display name and class
local car = Open77.data.vehicle("Vehicle.v_sport2_quadra_turbo_r")
if car then print(car.displayName, car.classLabel, car.seats) end
```

[Game data catalogues guide](../../01%20Documentation/data-catalogues.md)

Registered in `ResourceHost.cpp` (line 24934) as `LuaDataRecord`.

## weapon

```lua
Open77.data.weapon(record)
```

`GAME`: Requires a live game instance.

Looks up an `Items.*` weapon record in the game's live TweakDB.

No permission required; this is static game data. **The client ships no catalogue**: it is inside Cyberpunk 2077, so it reads the real database and gets the display name in the player's own language for zero shipped bytes. `source` is `tweakdb` here and `catalogue` on the server, which answers the same call from a reduced shipped table. `recordClass` -- the record's runtime RTTI class -- is always present once the lookup succeeds and is the honest local answer to FiveM's `GetEntityModel`; every other field is best effort, and an **absent field means the engine did not expose it**, not that the record lacks it. A `record` is a TweakDB string, never a Jenkins hash. Fails with `record_unknown`, `record_wrong_kind`, `invalid_record` or `tweakdb_unavailable`. See [Game data catalogues](https://open2077.net/docs/api/client/data-catalogues.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `record` | `string` | required | — |

Returns: `{ record, kind, recordClass, tweakDbId, source, ... }, or nil`, `reason`

```lua
-- Show a weapon's name in the shop UI
local gun = Open77.data.weapon("Items.Preset_Lexington_Default")
if gun then print(gun.displayName or gun.record) end
```

[Game data catalogues guide](../../01%20Documentation/data-catalogues.md)

Registered in `ResourceHost.cpp` (line 24934) as `LuaDataRecord`.
