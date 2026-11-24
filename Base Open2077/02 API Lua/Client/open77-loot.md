# Open77.loot (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-loot.md) · [Page web](https://open2077.net/docs/api/client/open77-loot)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

6 functions.

## acceptPickup

```lua
Open77.loot.acceptPickup(id, item, quantity)
```

`GAME`: Requires a live game instance.

Applique localement un pickup déjà accepté par le serveur.

API interne appelée uniquement après `open77:loot:pickupResult`. Elle projette l'objet dans l'inventaire REDengine local; l'inventaire persistant reste une responsabilité serveur.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `item` | `string` | required | — |
| `quantity` | `integer` | required | — |

Returns: `true en cas de succès`, `raison du refus`

```lua
-- Reserved for open77_loot: apply a pickup the server already accepted
-- (a gameplay resource listens to open77:loot:result instead)
Open77.loot.acceptPickup(dropId, "Items.money", 200)
```

Registered in `ResourceHost.cpp` (line 25059) as `LuaLootAccept`.

## clear

```lua
Open77.loot.clear()
```

`GAME`: Requires a live game instance.

Supprime toutes les projections locales et leurs prompts.

Clears the client-side native loot projection when the caller has `world.loot`. This is intended for the authoritative loot replication package during snapshot replacement; ordinary gameplay resources should request server-owned changes instead.

```lua
-- The replication package wipes the projection before applying a fresh snapshot
Open77.loot.clear()
```

Registered in `ResourceHost.cpp` (line 25057) as `LuaLootClear`.

## remove

```lua
Open77.loot.remove(id)
```

`GAME`: Requires a live game instance.

Retire une projection locale de loot.

Removes one replicated native loot entity from this client's projection and requires `world.loot`. It does not by itself delete the server record, so gameplay code should normally use the authoritative loot package API.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `true en cas de succès`, `raison du refus`

```lua
-- Reserved for open77_loot: drop the local projection of a removed server drop
Open77.loot.remove(dropId)
```

Registered in `ResourceHost.cpp` (line 25056) as `LuaLootRemove`.

## requestPickup

```lua
Open77.loot.requestPickup(id)
```

`NETWORK`: Uses the network backend.

Demande au serveur de ramasser un drop.

Le serveur contrôle l'existence, le routing bucket et la distance calculée depuis son dernier snapshot du joueur.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `true si la requête réseau est partie`, `raison du refus local`

```lua
-- Press E on the nearest drop to ask the server for it; the answer arrives on open77:loot:result
RegisterKeyMapping("loot_pickup", "Pick up", "E", function()
    if nearestDropId then Open77.loot.requestPickup(nearestDropId) end
end)
AddEventHandler("open77:loot:result", function(id, accepted, reason)
    if not accepted then Open77.hud.notify("Pickup refused: " .. tostring(reason)) end
end)
```

Registered in `ResourceHost.cpp` (line 25058) as `LuaLootRequest`.

## setAuthorityEnabled

```lua
Open77.loot.setAuthorityEnabled(enabled)
```

`GAME`: Requires a live game instance.

Active ou désactive la projection de loot autoritaire.

Réservé à `open77_loot`. Quand il est actif, les choix, conteneurs et drops vanilla sont neutralisés et les objets physiques deviennent de simples représentations du registre serveur.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true en cas de succès`, `raison du refus`

```lua
-- Reserved for open77_loot: turn vanilla loot off and project the server registry
Open77.loot.setAuthorityEnabled(true)
```

Registered in `ResourceHost.cpp` (line 25054) as `LuaLootAuthority`.

## upsert

```lua
Open77.loot.upsert(drop)
```

`GAME`: Requires a live game instance.

Crée ou met à jour la projection locale d'un drop serveur.

API interne de réplication. Demande la permission `world.loot`; un script gameplay ne doit pas inventer un drop côté client.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `drop` | `table { id, item, quantity, position, radius, label, model, revision }` | required | — |

Returns: `true en cas de succès`, `raison du refus`

```lua
-- Reserved for open77_loot: project a server drop locally
Open77.loot.upsert({ id = 42, item = "Items.money", quantity = 200, position = { x = -1442.2, y = 127.4, z = 18.0 },
    radius = 2, label = "Eddies", revision = 1 })
```

Registered in `ResourceHost.cpp` (line 25055) as `LuaLootUpsert`.
