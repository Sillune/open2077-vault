# Index API Lua

**1934 fonctions Lua 5.4** dans **154 espaces de noms**, selon l’explorateur officiel capturé le 24 novembre 2026.

| Runtime | Fonctions | Vérifier en priorité |
| --- | ---: | --- |
| Serveur | 1109 | Autorité, identité, permissions, retours, persistance |
| Client | 825 | Présentation, entrées, WebUI, requêtes vers le serveur |

Les notes d’API contiennent les signatures d’origine, les descriptions, les retours et les exemples lorsqu’ils sont publiés. Le catalogue public inclut des entrées dérivées des tables du site en plus des cartes natives du Devkit.

- [Export Markdown officiel complet (fichier local)](Exports/API%20compl%C3%A8te.md.txt)
- [Explorateur officiel](https://open2077.net/docs/api)
- [[Base Open2077/01 Documentation/server-api|Guide API serveur]]
- [[Base Open2077/04 Devkit et données/Index Devkit et données|Données JSON, schémas, stubs et Devkit]]

## API reference by namespace

### Server runtime (1109 functions)

- [[Base Open2077/02 API Lua/Serveur/globals|Globals]] — 230 functions.
- [[Base Open2077/02 API Lua/Serveur/unnamed|Namespace sans nom]] — 1 function.
- [[Base Open2077/02 API Lua/Serveur/request|.request]] — 1 function.
- [[Base Open2077/02 API Lua/Serveur/citizen|Citizen]] — 8 functions.
- [[Base Open2077/02 API Lua/Serveur/db|db]] — 11 functions.
- [[Base Open2077/02 API Lua/Serveur/json|json]] — 2 functions.
- [[Base Open2077/02 API Lua/Serveur/resource-open77-doors|open77_doors]] — 18 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-abilities|Open77.abilities]] — 5 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-access|Open77.access]] — 24 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-acl|Open77.acl]] — 8 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-animations|Open77.animations]] — 11 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-appearance|Open77.appearance]] — 2 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-audio|Open77.audio]] — 10 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-callbacks|Open77.callbacks]] — 4 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-chat|Open77.chat]] — 7 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-clothing|Open77.clothing]] — 5 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-combat|Open77.combat]] — 11 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-consumables|Open77.consumables]] — 9 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-convars|Open77.convars]] — 4 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-cyberware|Open77.cyberware]] — 14 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-dash|Open77.dash]] — 6 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-data|Open77.data]] — 5 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-database|Open77.database]] — 10 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-effects|Open77.effects]] — 17 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-elevators|Open77.elevators]] — 10 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-environment|Open77.environment]] — 8 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-events|Open77.events]] — 7 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-eventverdict|Open77.EventVerdict]] — 3 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-exports|Open77.exports]] — 2 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-hacking|Open77.hacking]] — 12 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-helditems|Open77.heldItems]] — 4 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-http|Open77.http]] — 15 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-interest|Open77.interest]] — 3 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-io|Open77.io]] — 12 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-kvp|Open77.kvp]] — 11 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-locomotion|Open77.locomotion]] — 6 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-log|Open77.log]] — 4 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-loot|Open77.loot]] — 5 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-metrics|Open77.metrics]] — 3 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-motion|Open77.motion]] — 4 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-net|Open77.net]] — 9 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-notifications|Open77.notifications]] — 5 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-npcs|Open77.npcs]] — 45 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-npcs-tasks|Open77.npcs.tasks]] — 28 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-perspective|Open77.perspective]] — 2 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-playerinteractions|Open77.playerInteractions]] — 6 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-players|Open77.players]] — 88 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-preflight|Open77.preflight]] — 10 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-promise|Open77.Promise]] — 5 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-props|Open77.props]] — 15 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-ragdoll|Open77.ragdoll]] — 4 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-ready|Open77.ready]] — 5 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-reflex|Open77.reflex]] — 6 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-resource|Open77.resource]] — 8 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-routingbuckets|Open77.routingBuckets]] — 6 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-runtime|Open77.runtime]] — 6 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-sound|Open77.sound]] — 6 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-state|Open77.state]] — 9 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-stats|Open77.stats]] — 20 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-statuses|Open77.statuses]] — 3 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-time|Open77.time]] — 3 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-tunables|Open77.tunables]] — 6 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-vehicles|Open77.vehicles]] — 175 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-vehicles-ai|Open77.vehicles.ai]] — 15 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-voice|Open77.voice]] — 18 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-weapons|Open77.weapons]] — 19 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-world|Open77.world]] — 7 functions.
- [[Base Open2077/02 API Lua/Serveur/open77-zones|Open77.zones]] — 5 functions.
- [[Base Open2077/02 API Lua/Serveur/promise|promise]] — 1 function.
- [[Base Open2077/02 API Lua/Serveur/r|r]] — 16 functions.
- [[Base Open2077/02 API Lua/Serveur/redis|Redis]] — 8 functions.
- [[Base Open2077/02 API Lua/Serveur/redis-connect|Redis.connect]] — 1 function.
- [[Base Open2077/02 API Lua/Serveur/session|session]] — 11 functions.
- [[Base Open2077/02 API Lua/Serveur/sqlite|SQLite]] — 5 functions.
- [[Base Open2077/02 API Lua/Serveur/sqlite-open|SQLite.open]] — 1 function.

### Client runtime (825 functions)

- [[Base Open2077/02 API Lua/Client/globals|Globals]] — 81 functions.
- [[Base Open2077/02 API Lua/Client/citizen|Citizen]] — 8 functions.
- [[Base Open2077/02 API Lua/Client/resource-open77-doors|open77_doors]] — 3 functions.
- [[Base Open2077/02 API Lua/Client/open77-abilities|Open77.abilities]] — 8 functions.
- [[Base Open2077/02 API Lua/Client/open77-anchors|Open77.anchors]] — 6 functions.
- [[Base Open2077/02 API Lua/Client/open77-animations|Open77.animations]] — 16 functions.
- [[Base Open2077/02 API Lua/Client/open77-appearance|Open77.appearance]] — 8 functions.
- [[Base Open2077/02 API Lua/Client/open77-assets|Open77.assets]] — 2 functions.
- [[Base Open2077/02 API Lua/Client/open77-audio|Open77.audio]] — 10 functions.
- [[Base Open2077/02 API Lua/Client/open77-blips|Open77.blips]] — 24 functions.
- [[Base Open2077/02 API Lua/Client/open77-callbacks|Open77.callbacks]] — 4 functions.
- [[Base Open2077/02 API Lua/Client/open77-camera|Open77.camera]] — 29 functions.
- [[Base Open2077/02 API Lua/Client/open77-character|Open77.character]] — 31 functions.
- [[Base Open2077/02 API Lua/Client/open77-chute|Open77.chute]] — 3 functions.
- [[Base Open2077/02 API Lua/Client/open77-clipboard|Open77.clipboard]] — 1 function.
- [[Base Open2077/02 API Lua/Client/open77-consumables|Open77.consumables]] — 9 functions.
- [[Base Open2077/02 API Lua/Client/open77-cyberware|Open77.cyberware]] — 20 functions.
- [[Base Open2077/02 API Lua/Client/open77-dash|Open77.dash]] — 1 function.
- [[Base Open2077/02 API Lua/Client/open77-data|Open77.data]] — 5 functions.
- [[Base Open2077/02 API Lua/Client/open77-debug|Open77.debug]] — 3 functions.
- [[Base Open2077/02 API Lua/Client/open77-debugdraw|Open77.debugDraw]] — 2 functions.
- [[Base Open2077/02 API Lua/Client/open77-doors|Open77.doors]] — 25 functions.
- [[Base Open2077/02 API Lua/Client/open77-elevators|Open77.elevators]] — 4 functions.
- [[Base Open2077/02 API Lua/Client/open77-environment|Open77.environment]] — 6 functions.
- [[Base Open2077/02 API Lua/Client/open77-equipment|Open77.equipment]] — 11 functions.
- [[Base Open2077/02 API Lua/Client/open77-events|Open77.events]] — 3 functions.
- [[Base Open2077/02 API Lua/Client/open77-exports|Open77.exports]] — 2 functions.
- [[Base Open2077/02 API Lua/Client/open77-gizmos|Open77.gizmos]] — 14 functions.
- [[Base Open2077/02 API Lua/Client/open77-hacking|Open77.hacking]] — 6 functions.
- [[Base Open2077/02 API Lua/Client/open77-hud|Open77.hud]] — 11 functions.
- [[Base Open2077/02 API Lua/Client/open77-input|Open77.input]] — 18 functions.
- [[Base Open2077/02 API Lua/Client/open77-inspector|Open77.inspector]] — 3 functions.
- [[Base Open2077/02 API Lua/Client/open77-json|Open77.json]] — 2 functions.
- [[Base Open2077/02 API Lua/Client/open77-kvp|Open77.kvp]] — 11 functions.
- [[Base Open2077/02 API Lua/Client/open77-locomotion|Open77.locomotion]] — 1 function.
- [[Base Open2077/02 API Lua/Client/open77-log|Open77.log]] — 4 functions.
- [[Base Open2077/02 API Lua/Client/open77-loot|Open77.loot]] — 6 functions.
- [[Base Open2077/02 API Lua/Client/open77-map|Open77.map]] — 21 functions.
- [[Base Open2077/02 API Lua/Client/open77-markers|Open77.markers]] — 7 functions.
- [[Base Open2077/02 API Lua/Client/open77-media|Open77.media]] — 5 functions.
- [[Base Open2077/02 API Lua/Client/open77-motion|Open77.motion]] — 7 functions.
- [[Base Open2077/02 API Lua/Client/open77-movement|Open77.movement]] — 5 functions.
- [[Base Open2077/02 API Lua/Client/open77-nameplates|Open77.nameplates]] — 10 functions.
- [[Base Open2077/02 API Lua/Client/open77-net|Open77.net]] — 6 functions.
- [[Base Open2077/02 API Lua/Client/open77-network|Open77.network]] — 7 functions.
- [[Base Open2077/02 API Lua/Client/open77-npcs|Open77.npcs]] — 6 functions.
- [[Base Open2077/02 API Lua/Client/open77-pausemenu|Open77.pauseMenu]] — 4 functions.
- [[Base Open2077/02 API Lua/Client/open77-perspective|Open77.perspective]] — 7 functions.
- [[Base Open2077/02 API Lua/Client/open77-photomode|Open77.photoMode]] — 4 functions.
- [[Base Open2077/02 API Lua/Client/open77-playerinteractions|Open77.playerInteractions]] — 9 functions.
- [[Base Open2077/02 API Lua/Client/open77-players|Open77.players]] — 52 functions.
- [[Base Open2077/02 API Lua/Client/open77-prediction|Open77.prediction]] — 2 functions.
- [[Base Open2077/02 API Lua/Client/open77-prevention|Open77.prevention]] — 4 functions.
- [[Base Open2077/02 API Lua/Client/open77-promise|Open77.Promise]] — 5 functions.
- [[Base Open2077/02 API Lua/Client/open77-props|Open77.props]] — 14 functions.
- [[Base Open2077/02 API Lua/Client/open77-puppets|Open77.puppets]] — 6 functions.
- [[Base Open2077/02 API Lua/Client/open77-reflex|Open77.reflex]] — 3 functions.
- [[Base Open2077/02 API Lua/Client/open77-remotecamera|Open77.remoteCamera]] — 15 functions.
- [[Base Open2077/02 API Lua/Client/open77-resource|Open77.resource]] — 7 functions.
- [[Base Open2077/02 API Lua/Client/open77-rtti|Open77.rtti]] — 5 functions.
- [[Base Open2077/02 API Lua/Client/open77-runtime|Open77.runtime]] — 6 functions.
- [[Base Open2077/02 API Lua/Client/open77-screen|Open77.screen]] — 14 functions.
- [[Base Open2077/02 API Lua/Client/open77-session|Open77.session]] — 18 functions.
- [[Base Open2077/02 API Lua/Client/open77-settings|Open77.settings]] — 3 functions.
- [[Base Open2077/02 API Lua/Client/open77-sfx|Open77.sfx]] — 6 functions.
- [[Base Open2077/02 API Lua/Client/open77-state|Open77.state]] — 7 functions.
- [[Base Open2077/02 API Lua/Client/open77-stats|Open77.stats]] — 5 functions.
- [[Base Open2077/02 API Lua/Client/open77-time|Open77.time]] — 1 function.
- [[Base Open2077/02 API Lua/Client/open77-travel|Open77.travel]] — 11 functions.
- [[Base Open2077/02 API Lua/Client/open77-vehicles|Open77.vehicles]] — 51 functions.
- [[Base Open2077/02 API Lua/Client/open77-vfx|Open77.vfx]] — 12 functions.
- [[Base Open2077/02 API Lua/Client/open77-voice|Open77.voice]] — 18 functions.
- [[Base Open2077/02 API Lua/Client/open77-wardrobe|Open77.wardrobe]] — 3 functions.
- [[Base Open2077/02 API Lua/Client/open77-weapons|Open77.weapons]] — 22 functions.
- [[Base Open2077/02 API Lua/Client/open77-webui|Open77.webui]] — 5 functions.
- [[Base Open2077/02 API Lua/Client/open77-world|Open77.world]] — 16 functions.
- [[Base Open2077/02 API Lua/Client/open77-zones|Open77.zones]] — 3 functions.
- [[Base Open2077/02 API Lua/Client/promise|promise]] — 1 function.
- [[Base Open2077/02 API Lua/Client/webui-page|WebUI.Page]] — 11 functions.
