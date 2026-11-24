# Index Devkit et données

Le Devkit récupéré correspond à la révision [`1d2ebadf16b2`](https://github.com/Open2077/open77-devkit/tree/1d2ebadf16b2630be00a0065e79296a10f2e54c7). Son manifeste indique le build **`2.50.0+op77.208`**, un instantané de **1814** cartes et **9** catalogues. L’explorateur du site rassemble **1934** entrées. Compare les contrats au build installé avant de les utiliser.

| Fichier local | Contenu |
| --- | --- |
| [cards.json](Index%20officiel/index/cards.json) | Cartes natives et métadonnées |
| [events.json](Index%20officiel/index/events.json) | Événements et contrats |
| [permissions.json](Index%20officiel/index/permissions.json) | Permissions par runtime |
| [guides.json](Index%20officiel/index/guides.json) | Recherche textuelle par guide et section |
| [releases.json](Index%20officiel/index/releases.json) | Historique des builds indexés |
| [manifest.json](Index%20officiel/index/manifest.json) | Provenance de l’index et comptages |
| [manifest.schema.json](Index%20officiel/index/schemas/manifest.schema.json) | Directives de `open77.lua` |
| [server-config.schema.json](Index%20officiel/index/schemas/server-config.schema.json) | Clés de `server.jsonc` |
| [Serveur](Index%20officiel/index/open77-server.d.lua) · [Client](Index%20officiel/index/open77-client.d.lua) | Stubs Lua séparés pour l’autocomplétion |
| [fivem-map.json](Index%20officiel/index/fivem-map.json) | Correspondances FiveM documentées |
| [api.json](Exports%20du%20site/api.json) · [meta.json](Exports%20du%20site/meta.json) | Export du site, 1814 cartes natives brutes et navigation |
| [Captures et SHA-256](Exports%20du%20site/Inventaire%20de%20capture.json) | URL exactes des fichiers téléchargés, tailles et empreintes |
| [Index pour agents](Exports%20du%20site/llms.txt) | Liens officiels vers les guides, l’API et le Devblog |

## Catalogues de jeu

- [animations](Index%20officiel/index/catalogues/animations.json)
- [animsets](Index%20officiel/index/catalogues/animsets.json)
- [items](Index%20officiel/index/catalogues/items.json)
- [npc-templates](Index%20officiel/index/catalogues/npc-templates.json)
- [props](Index%20officiel/index/catalogues/props.json)
- [sfx](Index%20officiel/index/catalogues/sfx.json)
- [vehicles](Index%20officiel/index/catalogues/vehicles.json)
- [vfx](Index%20officiel/index/catalogues/vfx.json)
- [weapons](Index%20officiel/index/catalogues/weapons.json)

Les identifiants des catalogues servent à chercher les références exactes. Vérifie la prise en charge en jeu sur le build cible. [[Base Open2077/01 Documentation/data-reference|Game data reference]]

## Utiliser le MCP

Lis [[Base Open2077/01 Documentation/agents|le guide officiel pour les agents]] pour la connexion au [MCP hébergé](https://mcp.open2077.net/mcp) ou l’installation locale. Le MCP hébergé sert à interroger la documentation. Le Devkit local peut détecter ton serveur et vérifier tes ressources. [Instructions et outils du Devkit](https://github.com/Open2077/open77-devkit)
