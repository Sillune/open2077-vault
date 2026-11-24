# Open2077 — Sources pour une IA

Ce fichier accompagne un projet de serveur OPEN//77, la plateforme multijoueur communautaire de Cyberpunk 2077. Il suffit à une IA qui peut lire le Web. Les URL ci-dessous mènent à la documentation, aux signatures Lua et au code officiel.

## Consigne à donner à l’IA

> Aide-moi à écrire des ressources Lua 5.4 pour mon serveur OPEN//77. Identifie le build exact sur mon serveur à partir de mes fichiers ou de mes logs, ou demande-le si nécessaire. Consulte les sources officielles ci-dessous. Pour chaque native utilisée, identifie son runtime, sa signature, ses permissions, ses types de retour et sa disponibilité pour ce build. Vérifie les noms d’événements et leurs paramètres dans le guide et le code officiel compatibles. Donne le chemin de chaque fichier, les déclarations nécessaires dans `open77.lua`, la configuration minimale, ainsi que les étapes de validation et de test. Cite les URL des contrats et exemples utilisés. Indique quelles fonctions sont confirmées et ce qui reste à tester en jeu.

## Où chercher la documentation et l’API

| Besoin | Source officielle |
| --- | --- |
| Trouver une page ou un espace de noms | [Index pour agents](https://open2077.net/llms.txt) |
| Récupérer toute la documentation publique en un fichier | [Export intégral](https://open2077.net/llms-full.txt). Si le lecteur limite la taille des réponses, ouvre les pages unitaires listées dans l’index. |
| Consulter les guides dans un navigateur | [Documentation](https://open2077.net/docs) |
| Charger une page en Markdown | [Exemple avec les ressources serveur](https://open2077.net/docs/server-resources.md). La plupart des pages sous `/docs` ont leur équivalent `.md`. |
| Trouver une native par nom ou runtime | [Explorateur Lua](https://open2077.net/docs/api) · [Export API Markdown](https://open2077.net/docs/api.md) |
| Vérifier une API serveur | [Guide API serveur](https://open2077.net/docs/server-api.md) · [Globals serveur](https://open2077.net/docs/api/server/globals.md) |
| Vérifier une API client | [Globals client](https://open2077.net/docs/api/client/globals.md) · [Exemple de namespace client](https://open2077.net/docs/api/client/open77-vehicles.md) |
| Obtenir les données des cartes natives | [Dépôt du site](https://github.com/Open2077/open77-app) · [`api.json`](https://raw.githubusercontent.com/Open2077/open77-app/main/content/api/api.json). Utilise aussi l’explorateur, qui réunit les entrées issues des tables Markdown du site. |
| Vérifier un événement, une permission ou un schéma | [Index du Devkit](https://github.com/Open2077/open77-devkit/tree/main/index) · [Documentation Devkit](https://open2077.net/docs/agents.md) |
| Rechercher un identifiant de jeu | [Game data reference](https://open2077.net/docs/data-reference.md) · [Game data catalogues](https://open2077.net/docs/data-catalogues.md) · [Données du Devkit](https://github.com/Open2077/open77-devkit/tree/main/index/catalogues) |
| Trouver un exemple réellement utilisé | [Ressources officielles](https://github.com/Open2077/open77-resources) · [Exemples](https://github.com/Open2077/open77-resources/tree/main/examples) · [Modes de jeu](https://github.com/Open2077/open77-resources/tree/main/gamemodes) · [Pack RP](https://github.com/Open2077/open77-resources/tree/main/rp) |
| Vérifier les versions et correctifs | [Téléchargements serveur et release notes](https://open2077.net/host) · [Devblog](https://open2077.net/devblog) · [Liste des builds indexés par le Devkit](https://github.com/Open2077/open77-devkit/blob/main/index/releases.json) |

## Devkit MCP officiel

Pour un client compatible MCP, ajoute le service HTTP [mcp.open2077.net/mcp](https://mcp.open2077.net/mcp). Le paramètre `build` permet de sélectionner un build pris en charge. Dans une URL, encode le caractère `+` d’un numéro de build en `%2B`.

L’installation locale officielle se lance avec `npx -y @open2077/mcp init` depuis le dossier serveur. Elle sert à détecter le build et à connecter les clients pris en charge. Installe-la uniquement si tu souhaites configurer ton client IA. [Guide officiel pour agents](https://open2077.net/docs/agents.md) · [Dépôt du Devkit](https://github.com/Open2077/open77-devkit)

| Outils MCP | Usage |
| --- | --- |
| `open77_build`, `open77_changes` | Confirmer quel build est interrogé et consulter les changements connus. |
| `open77_search`, `open77_api`, `open77_namespace`, `open77_guide` | Trouver un contrat, confirmer le runtime, les permissions et les retours, puis lire un guide. |
| `open77_events`, `open77_permissions`, `open77_data` | Vérifier les événements et leurs paramètres, les permissions et les identifiants de jeu. |
| `open77_manifest_schema`, `open77_server_config_schema` | Confirmer les directives du manifeste et les clés de configuration serveur. |
| `open77_workspace`, `open77_validate` | Avec le Devkit local, détecter le workspace et contrôler une ressource. Le verdict du serveur est disponible si le binaire compatible est détecté. |

## Points à vérifier avant d’écrire du code

1. Localise `server.jsonc`, `resources/` et le manifeste `open77.lua` de chaque ressource impliquée. Vérifie `resources.load`, l’autodémarrage, les dépendances et l’ordre de chargement. Utilise la branche `main` des exemples en priorité, puis retrouve une révision compatible si ton serveur utilise un build différent. [Ressources serveur](https://open2077.net/docs/server-resources.md) · [Dépôt des ressources](https://github.com/Open2077/open77-resources)
2. Mets les décisions de gameplay et les mutations persistantes côté serveur. Le client et le WebUI affichent l’état et envoient des demandes. Valide l’identité de session, les arguments, les droits et la fréquence des requêtes sur le serveur. [Modèle de session](https://open2077.net/docs.md) · [Identité](https://open2077.net/docs/identity.md) · [ACL](https://open2077.net/docs/server-acl.md) · [Callbacks réseau](https://open2077.net/docs/callbacks.md)
3. Vérifie chaque contrat par runtime. Pour un nom partagé, cherche les deux cartes. Si le Devkit et le site donnent des résultats différents, compare l’identifiant du build, le code de la ressource et, si possible, le résultat sur le binaire installé. Les alias issus de FiveM doivent être confirmés dans la documentation OPEN//77. [Explorateur API](https://open2077.net/docs/api) · [Index des contrats Devkit](https://github.com/Open2077/open77-devkit/tree/main/index)
4. Fais tourner la ressource sur une instance de test. Vérifie les logs, le démarrage, le rechargement, la reconnexion et les scénarios avec plusieurs joueurs quand la ressource modifie un état répliqué. [Hébergement](https://open2077.net/docs/host-a-server.md) · [Arguments et logs](https://open2077.net/docs/server-startup.md) · [Tests en jeu](https://open2077.net/docs/agent-testing.md)

## Parcours par objectif

| Objectif | Guides |
| --- | --- |
| Installer et administrer | [Serveur dédié](https://open2077.net/docs/host-a-server.md) · [Ressources et chargement](https://open2077.net/docs/server-resources.md) |
| Concevoir un gamemode | [Écrire un gamemode](https://open2077.net/docs/writing-a-gamemode.md) · [Kernel](https://open2077.net/docs/gamemode-kernel.md) · [Readiness](https://open2077.net/docs/readiness-gate.md) |
| Relier le client et le serveur | [Callbacks](https://open2077.net/docs/callbacks.md) · [State bags](https://open2077.net/docs/state-bags.md) · [Open77.events côté serveur](https://open2077.net/docs/api/server/open77-events.md) · [Côté client](https://open2077.net/docs/api/client/open77-events.md) |
| Persister des données | [SQLite](https://open2077.net/docs/sqlite.md) · [Redis](https://open2077.net/docs/redis.md) · [API base de données](https://open2077.net/docs/api/server/open77-database.md) |
| Étudier un serveur RP | [Pack RP officiel](https://github.com/Open2077/open77-resources/tree/main/rp) · [Construire un métier](https://open2077.net/docs/build-a-job.md) |


## Index détaillé des guides


### Prise en main et hébergement

#### Start here

- [OPEN//77 documentation](https://open2077.net/docs.md) — How OPEN//77 packages gameplay as resources: the session model, the authoritative server, a hello-world manifest, and the conventions shared by every Lua API.
- [Open Alpha: play, host and build](https://open2077.net/docs/alpha-access.md) — Play, host a server and build with Lua in Open Alpha. Public downloads, free accounts and server licenses without an access application or approval.
- [How OPEN//77 works](https://open2077.net/docs/platform.md) — What OPEN//77 is and is not, how players connect, why it is built on dedicated servers rather than peer-to-peer sessions, the project roadmap, and answers to the questions people ask most.
- [The OPEN//77 launcher](https://open2077.net/docs/launcher.md) — The player's guide to the OPEN//77 launcher: installing it, the SmartScreen warning on an unsigned executable and what it actually means, signing in, how it verifies your game build, the mod menu and everything it refuses to do silently, joining a world, direct-connect links, how it updates itself, and where its log is.
- [The Workshop: install and share resources](https://open2077.net/docs/community-hub.md) — The free OPEN//77 Workshop: paste a creation link into Warden to install, update or roll back a resource; publish your own as a ZIP or GitHub release with screenshots and a hover clip; what moderators check, versions, licenses, discussion and the upload rejection reasons.

#### Hosting & administration

- [Host your own server](https://open2077.net/docs/host-a-server.md) — A step-by-step guide to running your own OPEN//77 dedicated server: get the Windows or Linux build, mint a license key, walk the first-run setup, edit server.jsonc, launch it, and watch it appear automatically in the launcher.
- [Server startup arguments and logging](https://open2077.net/docs/server-startup.md) — Configure server log levels and verbose output, override JSONC fields with startup arguments or environment variables, provision public VPS addresses and validate configuration before boot.
- [Server licensing](https://open2077.net/docs/server-licensing.md) — How a server joins the OPEN//77 platform: create an account, mint a license key in the keymaster, put the key in your server config, and how the master authorises the server on boot.
- [Configure a SQL database](https://open2077.net/docs/database.md) — Connect an OPEN//77 server to MySQL or MariaDB: install SQL, create a dedicated account, configure credentials, verify readiness from Lua, diagnose failures and back up persistent data.
- [Your server in the launcher](https://open2077.net/docs/launcher-for-server-owners.md) — How the OPEN//77 launcher reaches your world: how a server appears in the browser at all, what a catalogue row actually carries, the open77:// deep-link contract for your own Join buttons, what a connecting player must have, and how a world declares the mods every player must load.
- [Connection preflight: the launcher's server check](https://open2077.net/docs/connection-preflight.md) — Check and greet players in the OPEN//77 launcher before the game starts: the server check flow, the preflight block of server.jsonc, the default open77_preflight resource (welcome, bans, whitelist applications, rules, maintenance), Warden requests, console commands, compatibility and troubleshooting.
- [Branding your server](https://open2077.net/docs/server-branding.md) — Give your server an icon and a banner from the Warden panel, with no SSH and no restart: what the two images are, the real size and shape limits (including the combined budget that decides whether your server is listed at all), and what to check when an image does not appear.
- [Launcher colors & text styles](https://open2077.net/docs/launcher-text-formatting.md) — Style your server name and description with color codes, bold, italic and variable-width text. Try the live palette, preview your name and copy ready-to-use examples.

#### Server administration

- [Warden](https://open2077.net/docs/warden.md) — Warden is the OPEN//77 in-server admin panel: a browser control room for live console, Lua resources, players, configuration and admins. Review operations that require a restart.
- [RCON: authenticated TCP administration](https://open2077.net/docs/rcon.md) — Configure authenticated TCP administration, connect dashboard backends, implement the Open77 RCON protocol, stream logs and transfer files with Warden permissions, TLS and resumable sessions.
- [Prometheus metrics](https://open2077.net/docs/metrics.md) — Monitor CPU, RAM, ticks, network traffic, players and gameplay. Configure secure Prometheus collection, build Grafana graphs and export custom Lua metrics for stress tests.
- [Manage Workshop resources with Warden](https://open2077.net/docs/community-hub-warden.md) — Install a Workshop creation from its pasted link, review the plan, update, uninstall and roll back; the Workshop chip in the Resources tab, revoked releases, preload restart staging, local configuration and retained backups.
- [Players tab in Warden](https://open2077.net/docs/warden-players.md) — The live roster in Warden — ping, health, life state, match — and the per-row admin actions: warn, kick, ban, grant rights, heal, freeze and move, plus why spectate cannot be adapted from a browser.

### Développement de ressources

#### Lua & resources

- [Server resources](https://open2077.net/docs/server-resources.md) — Resource manifests, the client and server runtime split, signing and download, hot reload, and the security model around resources the server hands to connecting players.
- [The Lua resource runtime](https://open2077.net/docs/resource-runtime.md) — What a resource actually gets: the complete manifest reference, the scheduler and lifecycle events, local and network events, modules and cross-resource exports, the core Open77 namespaces, WebUI surfaces, and every sandbox quota.
- [Lua modules and require](https://open2077.net/docs/lua-modules.md) — Local modules, published dependency libraries and `require`: the per-resource cache, the permissions a module runs under, what reload does to it, and how a module differs from an export.
- [Vectors and quaternions](https://open2077.net/docs/vectors.md) — `vector2`, `vector3`, `vector4`, `vec` and `quat` in both Lua runtimes: arithmetic, swizzles, `#` for magnitude, distance helpers — and the guarantee that plain tables keep working everywhere.
- [Cross-resource server exports](https://open2077.net/docs/server-exports.md) — Publish and call asynchronous exports across isolated server resources: complete Lua examples, caller identity, permissions, safe values, timeouts and reload behavior.
- [Built-in resource exports](https://open2077.net/docs/resource-exports.md) — Every client export published by the official OPEN//77 resource packages, with signatures per package and how to call them safely across resources.
- [FiveM compatibility](https://open2077.net/docs/fivem-compatibility.md) — The `Citizen` table, `SetTick`, generic promises and the shared-script helpers that let a FiveM script run unchanged, the three places OPEN//77 deliberately answers differently, and what is still missing.

#### Data & services

- [SQLite databases from Lua](https://open2077.net/docs/sqlite.md) — Open, configure and close SQLite files from server Lua with no server configuration: the sqlite.access permission, open options, query/insert/execute/batch/transaction/backup, parameters, concurrency, lifecycle, errors and limits.
- [Redis from Lua](https://open2077.net/docs/redis.md) — Connect to Redis from server Lua with no server configuration: the redis.access permission, URLs and options, every command, pipelines, transactions with WATCH, server-side scripts, scans, blocking queues, publish/subscribe, reconnection, errors and limits.
- [HTTP requests from Lua](https://open2077.net/docs/http-client.md) — Call web APIs, webhooks and file hosts from server Lua: every method, JSON, form and multipart bodies, binary data, authentication, redirects, timeouts, cancellation, uploads, downloads, the operator allow-list, errors and limits.

#### Networking & access

- [Player identity](https://open2077.net/docs/identity.md) — Durable player identifiers, session-scoped ids, display names, the rename flow, and what a game server can actually verify about a connecting player.
- [Connection control](https://open2077.net/docs/connection-control.md) — Connect events, admission deferrals, whitelists, bans and disconnect reasons.
- [Preflight in Lua](https://open2077.net/docs/preflight-lua.md) — Own the launcher's server check from a Lua resource: onPreflight and its events, the session object, fail-open rules, Open77.access for bans and whitelist requests, player.preflight at the gate, and recipes for applications, Discord, rules, maintenance and queues.
- [Commands & permissions](https://open2077.net/docs/server-acl.md) — Exporting a public identity, whitelisting a player, and declaring restricted server commands behind the access control list.
- [Network callbacks](https://open2077.net/docs/callbacks.md) — Ask the other side a question and wait for the answer: `Open77.callbacks` in both directions, promise results, per-resource namespacing, timeouts, limits and why a callback is a request rather than a grant.
- [Replicated state bags](https://open2077.net/docs/state-bags.md) — Key/value state attached to the server, a player or an entity that replicates itself: selectors, change handlers, who is entitled to see what, the caps, and why a client can never write a bag.
- [The join-time readiness gate](https://open2077.net/docs/readiness-gate.md) — Open77.ready: the host-owned barrier that stops one resource teleporting, spawning or killing a player another resource is not finished with. Holds, timeouts, the session number, and the rule every gamemode must follow.
- [Client persistent KVP](https://open2077.net/docs/client-kvp.md) — Endpoint- and resource-isolated local key/value persistence: CRUD, search, atomic operations, quotas, and why it must never hold secrets.
- [Prediction](https://open2077.net/docs/prediction.md) — Instant feedback for the acting player: what the client, open77_cyberware and open77_prediction each provide, the seven prediction families, the server verdict and smooth rollback, operator switches and latency ceiling, gamemode restrictions, telemetry, and an index of the related NPC, door, weapon and combat APIs.

#### Gamemodes & configuration

- [Writing a gamemode](https://open2077.net/docs/writing-a-gamemode.md) — Build an OPEN//77 gamemode with local state-machine files or shared server services, move players safely, isolate rounds in routing buckets, and validate authority and lifecycle.
- [The gamemode kernel](https://open2077.net/docs/gamemode-kernel.md) — Modular gamemode architecture: shared server services through asynchronous exports, isolated ownership, generated state-machine patterns and gameplay conventions.
- [Operator tunables](https://open2077.net/docs/tunables.md) — Open77.tunables: declaring the numbers a server owner may retune live from the admin panel, the file-scope local that silently defeats it, and choosing between live, promote and per-match capture.
- [Deathmatch](https://open2077.net/docs/deathmatch.md) — Free-for-all and arena modes, loadouts, bots, scoring and configuration.
- [Cordon: battle royale](https://open2077.net/docs/cordon.md) — Squads, the closing cordon, loot, revives and configuration.
- [Chimera boss encounter (PvE)](https://open2077.net/docs/chimera.md) — Run the Phantom Liberty Chimera as a server-controlled multiplayer boss: install, exports, phases and repair, weak spots, difficulty scaling, targeting, the staff lab menu and a raid built around it.

### Systèmes de jeu

#### Players & movement

- [Players around you](https://open2077.net/docs/client-players.md) — Enumerate the other players a client can already see — ids, bodies, distances and the nearest one — with no permission at all, and why a player id does not outlive a body.
- [Player checks & controls](https://open2077.net/docs/player-utilities.md) — Read player state and apply resource-owned control restrictions, including native encumbrance suppression and equipment package configuration.
- [Local avatar visibility](https://open2077.net/docs/local-puppet-visibility.md) — Hide the local player body and held weapon in first or third person, with resource-owned requests and a Prop Hunt example.
- [Health and stamina](https://open2077.net/docs/player-stats.md) — Read player statistics and control health, stamina and regeneration.
- [Freezing a player](https://open2077.net/docs/player-freeze.md) — Hold a player's body still: the authoritative server freeze on the engine's own movement restriction, the presentation-local client hold, what a freeze stops and what it deliberately does not.
- [Active ragdoll](https://open2077.net/docs/active-ragdoll.md) — Knock players down with a physically simulated body that still reacts, seen by every player: server leases and profiles, the client Open77.ragdoll API, the reaction library, the camera that follows the body, and the limits.
- [Travel and settling](https://open2077.net/docs/travel.md) — Move the local player's body and know when it has really arrived: the two teleports and why there are two, what “settled” actually tests, raw settle tickets, noclip and the map pick.

#### Characters & appearance

- [NPC spawning](https://open2077.net/docs/npcs.md) — Spawn Character records directly; manage streaming, authority, tasks, health and replicated behavior.
- [Character previews](https://open2077.net/docs/character-previews.md) — Create local selection mannequins and networked NPCs with saved player faces, skin and clothing.
- [NPC AI & voice control](https://open2077.net/docs/npc-behavior.md) — Control AI, combat, perception and voice lines per NPC with authoritative server Lua APIs.
- [World population: pedestrians and traffic](https://open2077.net/docs/population.md) — How the seated pedestrians, the walking crowd and ambient traffic are built, every setting that tunes them (convars, tunables, server NPC limits), runtime commands, and ready-made profiles from off to high population.
- [NPC catalogue](https://open2077.net/docs/npc-catalogue.md) — Search 6,582 Character records, copy spawn IDs and inspect appearances and compatibility warnings.
- [Player morphs](https://open2077.net/docs/player-models.md) — Server-owned NPC impersonation, readiness, replication, restoration and model compatibility.
- [Player model catalogue](https://open2077.net/docs/player-model-catalogue.md) — Search all 6,582 Character records for player morphs, with downloadable IDs and the admin menu workflow.
- [Custom clothing](https://open2077.net/docs/custom-clothing.md) — Register custom clothing on client and server, package native assets, browse and equip records, and preserve outfits through saves and resource reloads.
- [Layered outfits](https://open2077.net/docs/layered-outfits.md) — Wear up to 54 Equipment-EX compatible clothing layers over the base slots, saved with the character and replicated in multiplayer: the fitting room LAYERS tab, server exports and convars, Open77.layers, events and Equipment-EX mods.

#### Player body & creator

- [Changing body type at run time](https://open2077.net/docs/runtime-body.md) — Switch the local player between a male and a female body without reloading a save: stats, inventory, worn clothing and weapons follow, other players see it in place.
- [Building a character creator](https://open2077.net/docs/character-creator.md) — Read the game's character-creator catalogue from Lua, turn choices into a body and back, apply looks live, and save them on the server with open77_appearance.
- [Character creation in the world](https://open2077.net/docs/runtime-character-creation.md) — Spawn new players in the world instead of the vanilla creator and let them build a persistent character with open77_creator, the NativeUI creator freeroam uses.

#### Animations & roleplay

- [Synchronized animations](https://open2077.net/docs/rp-animations.md) — How synchronized role-play animations work: install the service, build your first Lua action, use the /anim menu, manage sequences and cancellation, and understand client/server authority and temporary third person.
- [Custom locomotion](https://open2077.net/docs/custom-locomotion.md) — Register animation packages, assign movement styles, play and cancel one-shot clips, and tune smooth transitions and stride timing.
- [Animation catalogue](https://open2077.net/docs/rp-animation-catalogue.md) — Browse 15 RP profiles and 73 exact clip variants, including giving, examination and wounded poses, with placement, rig bindings and experimental playback coverage.
- [Player interactions](https://open2077.net/docs/player-interactions.md) — Coordinate giving, healing, carrying and escorting, plus experimental persistent player attachments. Covers server ownership, consent, lifecycle, revision checks and limits.
- [Synchronized attachments](https://open2077.net/docs/attachments.md) — Attach props to player slots or vehicles with local position and rotation. Learn server authority, client rendering, bone discovery, revision-safe handoffs and lifecycle cleanup. Requires matching attachment-capable client/server builds and assets.
- [Animated actions: a pose, a prop in the hands, a real duration](https://open2077.net/docs/animated-actions.md) — Making a server-side job action look like something: a catalogue pose through Open77.animations, an object attached to a hand or the chest through Open77.props.attach, and a UI-kit progress bar for the time it takes. Workspot versus layer profiles, a carried crate with measured numbers and a live tuning command, a stationary repair with its cancel path, hand-sized prop aliases and the refusals to expect.
- [Roleplay utilities](https://open2077.net/docs/rp-kit.md) — `open77_rp_basics`: cuff, escort and search composed from primitives that already shipped, how a hold always ends, and why `carry` refuses instead of faking a player-to-player attachment this engine does not have.
- [Blue holocall eyes](https://open2077.net/docs/holocall-eyes.md) — Enable synchronized blue holocall eye glow with server-owned Lua leases, automatic cleanup, routing-bucket snapshots and local/F7 presentation.

#### Weapons & consumables

- [Weapons](https://open2077.net/docs/weapons-api.md) — Weapon templates, slots, holstering and server assignments.
- [Dual wielding](https://open2077.net/docs/dual-wielding.md) — Equip paired weapons with client and server Lua APIs, manage six loadout slots, read per-hand ammunition and reload state, and build a two-weapon HUD.
- [Grenades & healing](https://open2077.net/docs/consumables-api.md) — Configure grenade, MaxDoc inhaler and Bounce Back injector charges, equipment and regeneration with client and server Lua APIs.
- [Weapon customization](https://open2077.net/docs/weapon-customization.md) — Tune weapon statistics and impact blasts, launch nearby vehicles and characters, and test with the English weapon workshop and native Lua APIs.

#### Cyberware & abilities

- [Loot](https://open2077.net/docs/loot.md) — Authoritative ground drops: creating and projecting item drops, validating pickups, the onLootPickup hook, and where native UI ends and server authority begins.
- [Cyberware overview](https://open2077.net/docs/cyberware.md) — Build multiplayer cyberware with server-owned implants, configurable grades, persistent installation, native presentation and permission-controlled Lua APIs.
- [Cyberware styles and colors](https://open2077.net/docs/cyberware-customization.md) — Discover native cyberarm styles and palettes, apply two RGB colors, save and restore local customization, and understand ownership and multiplayer limits.
- [Gorilla Arms](https://open2077.net/docs/gorilla-arms.md) — Add native Gorilla Arms to your server: normal and charged punches, synchronized knockback, configurable damage and cosmetics, plus Lua installation examples and optional doctor and arena resources.
- [Dash / Air Dash](https://open2077.net/docs/dash.md) — Native ground and air dash as a server-owned session grant: definitions, charges, stamina and cooldown, one Air Dash per airtime, correlated `onDashChanged` / `onDashRejected` events, and the optional parkour example.
- [Ground Slam](https://open2077.net/docs/ground-slam.md) — A native ground or mid-air slam as a session grant: define, grant and revoke, the key the player presses, the damage and radius options, impact events and the limits.
- [Overdrive: a Sandevistan-style boost for a shared world](https://open2077.net/docs/reflex-overdrive.md) — Grant a real-time reflex overdrive: the owner moves, swings and reloads faster while nobody else is slowed and no clock is touched. Definitions, tiers, presets, the observer presentation, the rebindable key and the release paths.
- [Hacking and counterplay](https://open2077.net/docs/hacking.md) — Short Circuit, Overheat, Cyberware Malfunction, Cripple Movement, Reboot Optics and Weapon Glitch as kinds of one server-owned upload; Self-ICE, active purge, statuses, protection scopes and the `onHackingTransition` ledger.
- [Ability activation leases](https://open2077.net/docs/ability-activation-leases.md) — Why an ability activates the moment the key goes down: the server hands each player an expiring budget of activations, the client spends from it, and the server checks afterwards. Nothing to configure; what changes is when the waiting happens.

#### Vehicles & driving

- [Vehicles](https://open2077.net/docs/vehicles.md) — Network vehicle identity, streaming, authority leases, seats and proxy replication, doors and glass, damage profiles, and the full asymmetric server and client Lua API.
- [Sirens & key binding](https://open2077.net/docs/vehicle-sirens.md) — Install synchronized driver siren controls, configure the default key and integrate a client menu.
- [Changing seats](https://open2077.net/docs/vehicle-seat-switching.md) — Animate front and rear seat changes, control automatic driver takeover, cancel requests and handle Lua client/server lifecycle events.
- [Autonomous vehicles & AI](https://open2077.net/docs/vehicle-ai.md) — Server-owned autonomous driving, optional visible NPC drivers, reserved driver seats, routes, follow and chase, speed, behavior, lifecycle events and simulator handover.
- [Armed vehicles & Lua API](https://open2077.net/docs/vehicle-weapons.md) — Mounted weapons in multiplayer: driver authority, shots and damage, supported models, protocol 1.24, current limitations and twelve read-only client Lua APIs for weapon selection, ammo, heat and turret aim.
- [Passenger drive-by](https://open2077.net/docs/drive-by.md) — Enable or restrict passenger window combat, read its phases and seat, and combine client/server Lua policies with automatic window and cabin protection.
- [Armed vehicle catalogue](https://open2077.net/docs/armed-vehicles.md) — Exact spawn IDs for 172 vehicle records declaring weapon mounts, with live-tested Basilisk, Outlaw and Shion variants, mount profiles, spawn examples and the machine-readable catalogue.

#### Vehicle customization

- [Vehicle customization](https://open2077.net/docs/vehicle-customization.md) — Tune networked cars from the server: liveries, neon, lamp colours, rims and body parts, plate, tint, stages, engine sounds and stickers. Categories, permissions and how the look reaches every player. Ships with the next Open77 release.
- [Vehicle customization natives](https://open2077.net/docs/vehicle-customization-natives.md) — Every vehicle customization native by category: arguments, results, refusal reasons, permissions and limits, server and client.
- [Paint & lights hue](https://open2077.net/docs/vehicle-paint.md) — Server-controlled RGB paint, replication and change events.
- [Vehicle customization content packs](https://open2077.net/docs/vehicle-customization-packs.md) — What a rim, livery, sticker, stage, sound, plate or tint pack holds, how it reaches players, the vehicle kit pipeline, fit tables and the 12-decal cap.
- [Vehicle customization model coverage](https://open2077.net/docs/vehicle-customization-models.md) — Why coverage is data, one fit per record, what happens on a model a pack does not cover, and how a server covers more models.
- [Vehicle customization: persistence and multiplayer](https://open2077.net/docs/vehicle-customization-persistence.md) — The per-car visual value, late joiners and dormant choices, keeping a tuned car across a restart, events and the measured limits.
- [Vehicle customization: FiveM equivalents](https://open2077.net/docs/vehicle-customization-fivem.md) — SetVehicleMod, SetVehicleLivery, neon, window tint and plate: the FiveM aliases, their index mappings and the natives with another shape.
- [Vehicle customization worked examples](https://open2077.net/docs/vehicle-customization-examples.md) — A tuning shop, neon with refusals, a stage and sound bench, a garage that keeps tuned cars, a client menu, and Light RP's Auto Studio.

#### World & props

- [Time and weather](https://open2077.net/docs/weather.md) — Synchronised session time and weather: the network model, shared configuration, ACL-gated commands, random weather events, and the server and client APIs.
- [Time scale](https://open2077.net/docs/world-time.md) — Slow motion for a scripted beat: Open77.world.setTimeScale on the client as a claim the dilation guard stands down for, per routing bucket from the server, how to measure it against the world clock, every release path, and why it is presentation and not a Sandevistan.
- [Networked doors](https://open2077.net/docs/doors.md) — Server-owned automatic and interactive doors: dynamic discovery, native animations, elevator landing safety, routing buckets, access rules and client/server Lua exports.
- [Elevators](https://open2077.net/docs/elevators.md) — Server-authoritative native lifts: adopting a LiftDevice, bucket and chunk streaming, movement and pause control, late-join catch-up, and the reference resource commands.
- [World props](https://open2077.net/docs/props.md) — Server-owned world props and lights: creation, streaming and cleanup.
- [Entity gizmos](https://open2077.net/docs/gizmos.md) — Build native 3D placement editors with Open77.gizmos: Lua functions, target capabilities, snapping, undo/redo, lifecycle events, safe restoration and server-authorized edits. Development-build feature with documented validation limits.
- [Props and effects](https://open2077.net/docs/props-and-effects.md) — Placing server-owned objects, lights and replicated effects from Lua: the create definition, the curated model catalogue, the bucket and ground-height traps that make a prop invisible, lights and what respawns them, carrying, quotas and removal reasons, with the admin commands and two worked scenes.
- [Visual and audio effects](https://open2077.net/docs/effects.md) — Resource-owned world and entity visual effects plus spatialised sound effects, with inspection, cleanup and the curated alias catalogues.
- [Sky advertisements](https://open2077.net/docs/sky-advertising.md) — Create and preload custom sky hologram advertisements.

#### Map, markers & GPS

- [Native map & custom tabs](https://open2077.net/docs/native-map.md) — Customize the native map title, accent and resource WebUI tabs with Lua and JavaScript lifecycle events. Control map views, read player waypoints and pick destinations without fast travel.
- [Pause menu city map](https://open2077.net/docs/map-providers.md) — Make the pause menu's CITY MAP open your own map: register as a map provider, answer show requests within 1.5 seconds, priorities, the fallback to the game's map and client requirements.
- [3D city map, minimap and GPS](https://open2077.net/docs/map3d.md) — Install and configure the 3D city map that replaces the game's map and minimap: theme, blips for client and server, waypoint and GPS route events, map jamming, the pause menu entry and performance settings.
- [Blips and map pins](https://open2077.net/docs/blips.md) — Native map pins, custom SVG icons and per-blip colors, entity attachment, GPS tracking, distance gates and the complete sprite catalogue.
- [Custom SVG blips](https://open2077.net/docs/custom-blip-icons.md) — Create resource-owned SVG map icons, change colors and transparency from Lua, preserve native tracking, and handle format limits and errors.
- [3D world markers](https://open2077.net/docs/markers.md) — Eight native mesh shapes, RGBA colors, dimensions and rotation. Lua creation, updates, loading diagnostics, resource ownership and cleanup.

#### World drawing & zones

- [Drawing in the world](https://open2077.net/docs/world-drawing.md) — Every way a Lua resource puts something at a point in Night City: native ground markers, per-frame world anchors and the native card, ring and dot overlays, the marker-plus-prompt POI facade, proximity zones and nameplates — with lifecycle and cleanup for each.
- [World-anchored POIs](https://open2077.net/docs/worldui.md) — The open77_worldui facade: a ground marker and its interaction prompt created and destroyed as one owned handle, the definition reference, and the invisible-ring trap it exists to avoid.
- [World queries](https://open2077.net/docs/world-queries.md) — Query native rays, collision groups, Burning status, nearby entities, ground height and entity transforms.
- [Proximity zones](https://open2077.net/docs/zones.md) — The open77_zones service: enter and exit events with hysteresis, the definition reference, the polling model and quotas, and why every containment rule must still be re-derived on the server.
- [PolyZone](https://open2077.net/docs/polyzone.md) — Polygons, boxes, circles and spheres, entity zones and groups: containment queries, entry and exit callbacks, and the in-game zone editor with native debug drawing.

#### Targeting & interactions

- [Screen picking](https://open2077.net/docs/screen-picking.md) — `Open77.camera.screenRay`, `screenRaycast` and `Open77.input.cursor`: resolve the entity under a screen point through the native physics owner, with the identity kinds a client can and cannot promise.
- [Context menu](https://open2077.net/docs/context-menu.md) — The bundled ALT context menu: opt-in targeting of players, vehicles, doors, time and weather with resource-registered actions, the weapon-wheel claim it holds, and the exports a gamemode composes it from.
- [Contextual interactions](https://open2077.net/docs/interactions.md) — Custom world and NPC prompts: attaching interactions to players or NPCs, multiple actions and hold keys, the definition reference, and keeping server authority intact.
- [Vanilla device prompts](https://open2077.net/docs/device-interactions.md) — Take the vanilla prompt off a vending machine, ATM, terminal or door panel and put your own shop there: naming a device, ownership, the `open77:deviceUsed` event and every refusal reason.
- [Quickhacks](https://open2077.net/docs/quickhack.md) — The game's scanner and quickhack interface for your server: players, NPCs and vehicles as targets, quickhacks, decks and RAM, server-checked uploads, report cards, bundled Ping examples, hooks, events and the game's sounds.
- [Quickhack examples and ideas](https://open2077.net/docs/quickhack-examples.md) — Start from the bundled Ping quickhacks (player, NPC, vehicle, device), then complete resources to copy: a PvP netrunner class, a heist door breach, mission NPCs, a police ID scanner, safe-zone rules and a RAM economy.

#### Jobs & world interaction

- [Overview & setup](https://open2077.net/docs/world-interaction.md) — Choose and enable job resources, customize the salvage and street-food examples, and understand which world interactions are supported.
- [Build your first Lua job](https://open2077.net/docs/build-a-job.md) — Create a repair activity in Lua: place a work spot, choose an animation, require a part, pay the worker and handle cancellation.
- [Actions & carrying](https://open2077.net/docs/actions.md) — Lua reference for timed actions, work spots, eligibility, hooks and carrying. Connect your job rules to prompts, animations and progress bars.
- [Items, payments & storage](https://open2077.net/docs/transactions.md) — Connect inventories and wallets, save recipes and sales as one operation, handle payment retries and create persistent world containers.
- [Sitting & leaning](https://open2077.net/docs/furniture.md) — Enable seating on supported city furniture, check your venue's seats and approaches, and call seating from your own Lua resources.

#### Roleplay jobs

- [Roleplay jobs](https://open2077.net/docs/rp-jobs.md) — Seven ready-made Night City jobs for a roleplay server: what each one plays like, how they connect, the setup steps they share, the settings to run them all on one server and the platform features they rely on.
- [Ripperdoc: Viktor's clinic](https://open2077.net/docs/rp-job-ripperdoc.md) — Run a ripperdoc at Viktor's clinic: quotes the patient accepts on an offer card, the operating chair, real cyberware installed and paid once, what happens on a cancel or a restart, chrome from the Kabuki dealer, server setup, configuration and console commands.
- [CHOOH2: from the corn rows to the fuel depot](https://open2077.net/docs/rp-job-chooh2.md) — Run the CHOOH2 fuel chain: harvest Biotechnica corn into crates, truck them to the Petrochem refinery in the North Sunrise Oil Field, ferment canisters, sell them at the depot or refuel a car, with server setup, configuration and console commands.
- [Techie: the Kabuki west workshop](https://open2077.net/docs/rp-job-techie.md) — Run a techie workshop in Kabuki: buy salvage from a dealer or a scrapper, strip it and assemble weapon mods and cyberware components at two benches sharing a tray, sell to Wilson, Dr. Osei or players with consent, every stage paid and created once, server setup, mint rules, configuration and console commands.
- [Netrunner: breach contracts from the Afterlife](https://open2077.net/docs/rp-job-netrunner.md) — Run netrunner contracts from the Afterlife's back room: fixers post breach contracts on a board, a netrunner breaches a real terminal against the clock, a failure is traced by NCPD, the data shard is paid once on delivery with the fixer's cut, plus quickhacks, server setup, configuration and console commands.
- [Fixer: the contract board](https://open2077.net/docs/rp-job-fixer.md) — Run the fixers' contract board at the Afterlife, Wakako's and Regina's corner: deliveries, retrievals, job orders and cyberpsycho bounties with escrowed rewards, reputation tiers, payouts and refunds made once, what survives a restart, server setup, configuration and console commands.
- [Trauma Team: plans, calls and evacuations](https://open2077.net/docs/rp-job-trauma-team.md) — Run Trauma Team International: Silver, Gold and Platinum plans sold at kiosks, downed players held where they fell and a call raised with their plan's priority, medics who fly the AV out, stabilise and evacuate to Vik's clinic, every charge paid once, what survives a disconnect or a restart, server setup, configuration and console commands.
- [Nomad smuggler: Aldecaldos runs](https://open2077.net/docs/rp-job-nomad.md) — Run Aldecaldos smuggling runs: take a run at the camp's board, carry crates into a truck bed, get past the NCPD checkpoints and hand the crates to a fence or a player, paid once with the camp's cut, plus officer searches, what survives a restart, server setup, configuration and console commands.

#### WebUI & HUD

- [Pause menu customization](https://open2077.net/docs/pause-menu.md) — Customize the OPEN//77 Escape menu with a Lua accent color and a server logo beside OPEN//77. Manifest setup, local assets, image URLs, permissions and resource cleanup.
- [Custom loading screens](https://open2077.net/docs/loading-screens.md) — Detect native loading cycles, display a branded WebUI with progress, and hide or restore HUDs during teleport loads.
- [The UI kit](https://open2077.net/docs/ui-kit.md) — `open77_uikit`: progress bars, text hints, alerts, input forms, context and keyboard menus and a radial wheel drawn from one page — with the focus rule that outranks the whole feature list.
- [Shared WebUI assets with open77://](https://open2077.net/docs/webui-assets.md) — Share images, CSS, JavaScript, fonts and data between resource interfaces with open77://. Provider and consumer manifests, URL rules, caching, limits and troubleshooting.
- [WebUI budget and performance](https://open2077.net/docs/webui-budget.md) — What one WebUI page costs at each resolution, the 24-per-resource limit, the 32 pages a server's resources share and the separate pool for built-in pages, reading a `webui_surface_quota` refusal, `WebUI.budget()`, and keeping a big server inside the budget.
- [Notifications](https://open2077.net/docs/notifications.md) — Reusable WebUI toasts: local client notifications, server-targeted delivery, updating and dismissing, queue limits, positions and progress.
- [HUD visibility](https://open2077.net/docs/hud-visibility.md) — Show and hide native HUD components, or mask the HUD, watermark and nonfocused resource interfaces with cinematic letterbox bars. Resource-owned claims, cleanup and native notifications.
- [World screens](https://open2077.net/docs/world-screens.md) — Put a WebUI page or a NativeUI document on any mesh as its real material: a television with YouTube and 3D sound, monitors, billboards. Open77.screens, attaching to props and entities, and the scripted-camera mode to click on it.

#### NativeUI

- [NativeUI](https://open2077.net/docs/native-ui.md) — Draw HUDs and menus on the game's own GPU with RML, RCSS and Lua: when to choose it over WebUI, requirements, a first HUD resource, layers, lifecycle, errors and developer tools.
- [NativeUI documents, data and events](https://open2077.net/docs/native-ui-documents.md) — Every Open77.ui call: create options, data models and bindings, model and DOM events, changing the document, reading layout, keyboard and mouse focus, images and fonts.
- [RML and RCSS for web developers](https://open2077.net/docs/native-ui-rml.md) — What differs from HTML and CSS in NativeUI documents: the body is the document, dp units, flexbox pitfalls, events, styles, assets, large lists and debugging.
- [NativeUI motion](https://open2077.net/docs/native-ui-motion.md) — Native springs and tweens with ui:animate, keyframes, presets and easings, exit animations and automatic layout transitions.
- [NativeUI effects, shapes and themes](https://open2077.net/docs/native-ui-effects.md) — The built-in Open77 design system, GPU filters, animated fx() decorators, custom HLSL effects, vector paths and shapes, runtime themes and fonts.
- [NativeUI 3D HUDs](https://open2077.net/docs/native-ui-3d.md) — Tilt a whole document in perspective with projection, float a HUD with the camera, and angle single panels with RCSS 3D transforms.
- [NativeUI 3D scenes and minimaps](https://open2077.net/docs/native-ui-scenes.md) — scene3d viewports: meshes, instanced markers, lines, views and cameras, object styles, draped areas and zones, road routes and anchored markers.
- [NativeUI recipes](https://open2077.net/docs/native-ui-recipes.md) — Complete NativeUI examples: a keyboard and mouse menu, notification toasts, a keyboard-only composer, a server-driven scoreboard and a recolourable theme.
- [Building preflight screens](https://open2077.net/docs/preflight-screens.md) — NativeUI Lite, the RML and RCSS subset the OPEN//77 launcher renders before the game starts: elements, styles, design tokens, data binding, events, inputs, images, limits, layout tips and a complete welcome and rules example.

#### Input & controls

- [WebUI keyboard input](https://open2077.net/docs/webui-input.md) — Capture Escape and navigation keys in a WebUI while keeping movement and mouse look active. Supported keys, permissions, errors and focus cleanup.
- [WebUI cursor](https://open2077.net/docs/webui-cursor.md) — Hide the Open77 cursor and draw your own in a page, follow the page's CSS cursor or force a shape, read and move the pointer: page:setCursor, Open77.cursor, window.Open77.cursor and held-key release.
- [Key bindings](https://open2077.net/docs/keybindings.md) — Register and rebind keyboard, mouse, wheel and gamepad actions, optionally consume native input, and read devices directly.
- [Blocking player input](https://open2077.net/docs/input-blocking.md) — Take a named input away from the player and give it back — per-action blocks or the whole control stream — with the ownership and release rules that stop a crashed resource stranding somebody.
- [Clipboard](https://open2077.net/docs/clipboard.md) — The write-only client clipboard API, the clipboard.write permission, failure reasons, and the built-in /pos and /rot transform commands.

#### Cameras & cinematics

- [First and third person](https://open2077.net/docs/perspective.md) — Player preferences, server policy and camera ownership.
- [Scripted cameras](https://open2077.net/docs/cameras.md) — Put the view where your resource wants it: create, place, follow, look at and shake a camera, spectate a player from the server, and the release rules that guarantee the player gets their own view back.
- [Third-person camera styles](https://open2077.net/docs/third-person-camera.md) — Client-side customization of the playable F7 camera: classic or over-the-shoulder framing, custom pivots, movement FOV and temporary shake; not a scripted camera.
- [Remote cameras & screens](https://open2077.net/docs/remote-camera.md) — Independent render cameras: client Lua API, server access and streaming, HUD/WebUI feeds, world screens, placement editor and experimental limits.
- [Native fades & transitions](https://open2077.net/docs/screen-transitions.md) — Use Cyberpunk's native screen fades from client Lua: fade out, hold and return, color options, completion events, resource ownership and automatic recovery. Includes server-coordinated transitions and HUD limitations.
- [Braindance](https://open2077.net/docs/braindance.md) — The braindance launch as a scriptable sequence (white-out flash, black screen to hide a teleport or a set change, the braindance look, events for every phase) and the VISUAL, THERMAL and AUDIO vision layers with a native heatmap of every body around the player: open77_braindance exports and Open77.braindance.
- [Photo mode](https://open2077.net/docs/photo-mode.md) — Control photo mode from a client resource, with exclusive ownership and cleanup.
- [Screenshots and mugshots](https://open2077.net/docs/screenshots.md) — In-process captures of the game's own frame, the upload to a URL, the server-requested picture a client delivers, and the scripted-camera mugshot: what is captured, the surface formats, the in-flight rule, and where the picture goes.

#### Audio & communication

- [Package audio: 2D & 3D](https://open2077.net/docs/package-audio.md) — Play package MP3/WAV assets locally or through server-owned network sounds. Control playback, pause, seek, volume and position with resource-owned handles, routing buckets and late-join timelines.
- [Resource audio](https://open2077.net/docs/sound.md) — `open77_sound` plays an audio file your resource ships: spatial behaviour, server twins, ownership and lifetime, why files are never shared across resources, and how it differs from the game's own `Open77.sfx` events.
- [Chat](https://open2077.net/docs/chat.md) — Chat messages, slash commands, completion, and the built-in developer commands available to every player.

#### Voice

- [Voice chat overview](https://open2077.net/docs/voice.md) — Native proximity voice with acoustics, server configuration, the open77_voice and open-voice packages, reliability, bandwidth and diagnostics.
- [Voice channels](https://open2077.net/docs/voice-channels.md) — Server voice channels: radio, phone, party, admin and spatial modes, memberships and permissions, mutes, proximity control, global policy and ownership.
- [Voice audio graph](https://open2077.net/docs/voice-graph.md) — Loudspeakers, microphones, direct listeners and synchronised music from the server: anchors, connections and intents, file playback, acoustics, events and limits.
- [Voice effects and presets](https://open2077.net/docs/voice-effects.md) — The 36-parameter voice effect rack and its 12 built-in presets, custom presets, per-listener effects, targets, composition order and ownership.
- [Client voice API](https://open2077.net/docs/voice-client.md) — Client voice controls: devices and gains, push-to-talk and voice activation, transmit intents, reach requests, talkers, events, local music and generated PCM.
- [Voice recipes](https://open2077.net/docs/voice-recipes.md) — Complete server voice examples: radio frequencies, phone calls, a vehicle megaphone, a club with music and a DJ, an intercom, voice disguise and admin broadcasts.
- [Voice lipsync](https://open2077.net/docs/voice-lipsync.md) — Enable voice-driven mouths on male/female player heads, locally and on replicated players, with resource-owned Lua controls and diagnostics.
- [Guided voice setup API](https://open2077.net/docs/guided-voice-setup.md) — Build optional microphone onboarding with device selection, a live meter, local playback, keyboard and mouse push-to-talk bindings, and shared pause-menu preferences.

### Références et outils

#### Mods & game data

- [Mods and custom assets](https://open2077.net/docs/mods.md) — Install and declare stack, world and personal mods.
- [Mods your server requires](https://open2077.net/docs/server-mods.md) — Requiring a mod for your world: whether the content needs a mod at all or is really a Lua resource, the trust verdicts and the capability cap that lets unverified packages carry inert data only, importing from the Warden Mods tab, who hosts the bytes, requesting a review, bandwidth, and removing a mod without stranding players.
- [Game data reference](https://open2077.net/docs/data-reference.md) — NPC templates, vehicle records, seat and door constants, items, weapons, effects, sounds, animations and sprite catalogues, with explicit support levels.
- [Game data catalogues](https://open2077.net/docs/data-catalogues.md) — `Open77.data` turns a TweakDB record string into the facts a resource can show — display name, class, manufacturer, seats, quality — the fields each runtime can answer, and the one asymmetry on `npc`.

#### API & developer tools

- [Lua API reference](https://open2077.net/docs/api.md) — Searchable reference for every registered OPEN//77 Lua function, separated by runtime so a client projection is never mistaken for server authority.
- [Complete server Lua API](https://open2077.net/docs/server-api.md) — Every dedicated-server global, Open77 namespace, permission, constant and result shape: scheduler, events, commands, players, combat, routing buckets, loot, vehicles, NPCs, elevators, database and logging.
- [Build resources with an AI agent](https://open2077.net/docs/agents.md) — Install the OPEN//77 Devkit MCP in one line and give Claude Code, Codex, Cursor or VS Code every Lua native, permission, event and game-data name, pinned to the server build you run.
- [RTTI calls and hooks](https://open2077.net/docs/rtti.md) — Call native and scripted game functions from Lua, obtain GameInstance and game systems, install hooks and manage reference lifetimes.
- [Test RTTI calls in the running game](https://open2077.net/docs/rtti-testing.md) — Run the RTTI fixture with agent-play and the debug bridge, interpret results, check stop and reload cleanup, and test both client resource hosts through the server laboratory.
- [Privileged debug runtime](https://open2077.net/docs/debug-runtime.md) — The ACL-gated in-game Lua laboratory: client execution, native lab commands, the REDscript bridge, and the limits that keep it out of gameplay code.
- [Autonomous agent testing](https://open2077.net/docs/agent-testing.md) — The MCP server and Claude Code skill that let an AI agent test OPEN//77 in the running game: the tool list, the human-path connection, repeatable scenarios, installation, and the rules that keep a verdict honest.

## Index détaillé des espaces de noms Lua



### Server runtime (1109 functions)

- [Globals](https://open2077.net/docs/api/server/globals.md) — 230 functions.
- [Namespace sans nom](https://open2077.net/docs/api/server/unnamed.md) — 1 function.
- [.request](https://open2077.net/docs/api/server/request.md) — 1 function.
- [Citizen](https://open2077.net/docs/api/server/citizen.md) — 8 functions.
- [db](https://open2077.net/docs/api/server/db.md) — 11 functions.
- [json](https://open2077.net/docs/api/server/json.md) — 2 functions.
- [open77_doors](https://open2077.net/docs/api/server/resource-open77-doors.md) — 18 functions.
- [Open77.abilities](https://open2077.net/docs/api/server/open77-abilities.md) — 5 functions.
- [Open77.access](https://open2077.net/docs/api/server/open77-access.md) — 24 functions.
- [Open77.acl](https://open2077.net/docs/api/server/open77-acl.md) — 8 functions.
- [Open77.animations](https://open2077.net/docs/api/server/open77-animations.md) — 11 functions.
- [Open77.appearance](https://open2077.net/docs/api/server/open77-appearance.md) — 2 functions.
- [Open77.audio](https://open2077.net/docs/api/server/open77-audio.md) — 10 functions.
- [Open77.callbacks](https://open2077.net/docs/api/server/open77-callbacks.md) — 4 functions.
- [Open77.chat](https://open2077.net/docs/api/server/open77-chat.md) — 7 functions.
- [Open77.clothing](https://open2077.net/docs/api/server/open77-clothing.md) — 5 functions.
- [Open77.combat](https://open2077.net/docs/api/server/open77-combat.md) — 11 functions.
- [Open77.consumables](https://open2077.net/docs/api/server/open77-consumables.md) — 9 functions.
- [Open77.convars](https://open2077.net/docs/api/server/open77-convars.md) — 4 functions.
- [Open77.cyberware](https://open2077.net/docs/api/server/open77-cyberware.md) — 14 functions.
- [Open77.dash](https://open2077.net/docs/api/server/open77-dash.md) — 6 functions.
- [Open77.data](https://open2077.net/docs/api/server/open77-data.md) — 5 functions.
- [Open77.database](https://open2077.net/docs/api/server/open77-database.md) — 10 functions.
- [Open77.effects](https://open2077.net/docs/api/server/open77-effects.md) — 17 functions.
- [Open77.elevators](https://open2077.net/docs/api/server/open77-elevators.md) — 10 functions.
- [Open77.environment](https://open2077.net/docs/api/server/open77-environment.md) — 8 functions.
- [Open77.events](https://open2077.net/docs/api/server/open77-events.md) — 7 functions.
- [Open77.EventVerdict](https://open2077.net/docs/api/server/open77-eventverdict.md) — 3 functions.
- [Open77.exports](https://open2077.net/docs/api/server/open77-exports.md) — 2 functions.
- [Open77.hacking](https://open2077.net/docs/api/server/open77-hacking.md) — 12 functions.
- [Open77.heldItems](https://open2077.net/docs/api/server/open77-helditems.md) — 4 functions.
- [Open77.http](https://open2077.net/docs/api/server/open77-http.md) — 15 functions.
- [Open77.interest](https://open2077.net/docs/api/server/open77-interest.md) — 3 functions.
- [Open77.io](https://open2077.net/docs/api/server/open77-io.md) — 12 functions.
- [Open77.kvp](https://open2077.net/docs/api/server/open77-kvp.md) — 11 functions.
- [Open77.locomotion](https://open2077.net/docs/api/server/open77-locomotion.md) — 6 functions.
- [Open77.log](https://open2077.net/docs/api/server/open77-log.md) — 4 functions.
- [Open77.loot](https://open2077.net/docs/api/server/open77-loot.md) — 5 functions.
- [Open77.metrics](https://open2077.net/docs/api/server/open77-metrics.md) — 3 functions.
- [Open77.motion](https://open2077.net/docs/api/server/open77-motion.md) — 4 functions.
- [Open77.net](https://open2077.net/docs/api/server/open77-net.md) — 9 functions.
- [Open77.notifications](https://open2077.net/docs/api/server/open77-notifications.md) — 5 functions.
- [Open77.npcs](https://open2077.net/docs/api/server/open77-npcs.md) — 45 functions.
- [Open77.npcs.tasks](https://open2077.net/docs/api/server/open77-npcs-tasks.md) — 28 functions.
- [Open77.perspective](https://open2077.net/docs/api/server/open77-perspective.md) — 2 functions.
- [Open77.playerInteractions](https://open2077.net/docs/api/server/open77-playerinteractions.md) — 6 functions.
- [Open77.players](https://open2077.net/docs/api/server/open77-players.md) — 88 functions.
- [Open77.preflight](https://open2077.net/docs/api/server/open77-preflight.md) — 10 functions.
- [Open77.Promise](https://open2077.net/docs/api/server/open77-promise.md) — 5 functions.
- [Open77.props](https://open2077.net/docs/api/server/open77-props.md) — 15 functions.
- [Open77.ragdoll](https://open2077.net/docs/api/server/open77-ragdoll.md) — 4 functions.
- [Open77.ready](https://open2077.net/docs/api/server/open77-ready.md) — 5 functions.
- [Open77.reflex](https://open2077.net/docs/api/server/open77-reflex.md) — 6 functions.
- [Open77.resource](https://open2077.net/docs/api/server/open77-resource.md) — 8 functions.
- [Open77.routingBuckets](https://open2077.net/docs/api/server/open77-routingbuckets.md) — 6 functions.
- [Open77.runtime](https://open2077.net/docs/api/server/open77-runtime.md) — 6 functions.
- [Open77.sound](https://open2077.net/docs/api/server/open77-sound.md) — 6 functions.
- [Open77.state](https://open2077.net/docs/api/server/open77-state.md) — 9 functions.
- [Open77.stats](https://open2077.net/docs/api/server/open77-stats.md) — 20 functions.
- [Open77.statuses](https://open2077.net/docs/api/server/open77-statuses.md) — 3 functions.
- [Open77.time](https://open2077.net/docs/api/server/open77-time.md) — 3 functions.
- [Open77.tunables](https://open2077.net/docs/api/server/open77-tunables.md) — 6 functions.
- [Open77.vehicles](https://open2077.net/docs/api/server/open77-vehicles.md) — 175 functions.
- [Open77.vehicles.ai](https://open2077.net/docs/api/server/open77-vehicles-ai.md) — 15 functions.
- [Open77.voice](https://open2077.net/docs/api/server/open77-voice.md) — 18 functions.
- [Open77.weapons](https://open2077.net/docs/api/server/open77-weapons.md) — 19 functions.
- [Open77.world](https://open2077.net/docs/api/server/open77-world.md) — 7 functions.
- [Open77.zones](https://open2077.net/docs/api/server/open77-zones.md) — 5 functions.
- [promise](https://open2077.net/docs/api/server/promise.md) — 1 function.
- [r](https://open2077.net/docs/api/server/r.md) — 16 functions.
- [Redis](https://open2077.net/docs/api/server/redis.md) — 8 functions.
- [Redis.connect](https://open2077.net/docs/api/server/redis-connect.md) — 1 function.
- [session](https://open2077.net/docs/api/server/session.md) — 11 functions.
- [SQLite](https://open2077.net/docs/api/server/sqlite.md) — 5 functions.
- [SQLite.open](https://open2077.net/docs/api/server/sqlite-open.md) — 1 function.

### Client runtime (825 functions)

- [Globals](https://open2077.net/docs/api/client/globals.md) — 81 functions.
- [Citizen](https://open2077.net/docs/api/client/citizen.md) — 8 functions.
- [open77_doors](https://open2077.net/docs/api/client/resource-open77-doors.md) — 3 functions.
- [Open77.abilities](https://open2077.net/docs/api/client/open77-abilities.md) — 8 functions.
- [Open77.anchors](https://open2077.net/docs/api/client/open77-anchors.md) — 6 functions.
- [Open77.animations](https://open2077.net/docs/api/client/open77-animations.md) — 16 functions.
- [Open77.appearance](https://open2077.net/docs/api/client/open77-appearance.md) — 8 functions.
- [Open77.assets](https://open2077.net/docs/api/client/open77-assets.md) — 2 functions.
- [Open77.audio](https://open2077.net/docs/api/client/open77-audio.md) — 10 functions.
- [Open77.blips](https://open2077.net/docs/api/client/open77-blips.md) — 24 functions.
- [Open77.callbacks](https://open2077.net/docs/api/client/open77-callbacks.md) — 4 functions.
- [Open77.camera](https://open2077.net/docs/api/client/open77-camera.md) — 29 functions.
- [Open77.character](https://open2077.net/docs/api/client/open77-character.md) — 31 functions.
- [Open77.chute](https://open2077.net/docs/api/client/open77-chute.md) — 3 functions.
- [Open77.clipboard](https://open2077.net/docs/api/client/open77-clipboard.md) — 1 function.
- [Open77.consumables](https://open2077.net/docs/api/client/open77-consumables.md) — 9 functions.
- [Open77.cyberware](https://open2077.net/docs/api/client/open77-cyberware.md) — 20 functions.
- [Open77.dash](https://open2077.net/docs/api/client/open77-dash.md) — 1 function.
- [Open77.data](https://open2077.net/docs/api/client/open77-data.md) — 5 functions.
- [Open77.debug](https://open2077.net/docs/api/client/open77-debug.md) — 3 functions.
- [Open77.debugDraw](https://open2077.net/docs/api/client/open77-debugdraw.md) — 2 functions.
- [Open77.doors](https://open2077.net/docs/api/client/open77-doors.md) — 25 functions.
- [Open77.elevators](https://open2077.net/docs/api/client/open77-elevators.md) — 4 functions.
- [Open77.environment](https://open2077.net/docs/api/client/open77-environment.md) — 6 functions.
- [Open77.equipment](https://open2077.net/docs/api/client/open77-equipment.md) — 11 functions.
- [Open77.events](https://open2077.net/docs/api/client/open77-events.md) — 3 functions.
- [Open77.exports](https://open2077.net/docs/api/client/open77-exports.md) — 2 functions.
- [Open77.gizmos](https://open2077.net/docs/api/client/open77-gizmos.md) — 14 functions.
- [Open77.hacking](https://open2077.net/docs/api/client/open77-hacking.md) — 6 functions.
- [Open77.hud](https://open2077.net/docs/api/client/open77-hud.md) — 11 functions.
- [Open77.input](https://open2077.net/docs/api/client/open77-input.md) — 18 functions.
- [Open77.inspector](https://open2077.net/docs/api/client/open77-inspector.md) — 3 functions.
- [Open77.json](https://open2077.net/docs/api/client/open77-json.md) — 2 functions.
- [Open77.kvp](https://open2077.net/docs/api/client/open77-kvp.md) — 11 functions.
- [Open77.locomotion](https://open2077.net/docs/api/client/open77-locomotion.md) — 1 function.
- [Open77.log](https://open2077.net/docs/api/client/open77-log.md) — 4 functions.
- [Open77.loot](https://open2077.net/docs/api/client/open77-loot.md) — 6 functions.
- [Open77.map](https://open2077.net/docs/api/client/open77-map.md) — 21 functions.
- [Open77.markers](https://open2077.net/docs/api/client/open77-markers.md) — 7 functions.
- [Open77.media](https://open2077.net/docs/api/client/open77-media.md) — 5 functions.
- [Open77.motion](https://open2077.net/docs/api/client/open77-motion.md) — 7 functions.
- [Open77.movement](https://open2077.net/docs/api/client/open77-movement.md) — 5 functions.
- [Open77.nameplates](https://open2077.net/docs/api/client/open77-nameplates.md) — 10 functions.
- [Open77.net](https://open2077.net/docs/api/client/open77-net.md) — 6 functions.
- [Open77.network](https://open2077.net/docs/api/client/open77-network.md) — 7 functions.
- [Open77.npcs](https://open2077.net/docs/api/client/open77-npcs.md) — 6 functions.
- [Open77.pauseMenu](https://open2077.net/docs/api/client/open77-pausemenu.md) — 4 functions.
- [Open77.perspective](https://open2077.net/docs/api/client/open77-perspective.md) — 7 functions.
- [Open77.photoMode](https://open2077.net/docs/api/client/open77-photomode.md) — 4 functions.
- [Open77.playerInteractions](https://open2077.net/docs/api/client/open77-playerinteractions.md) — 9 functions.
- [Open77.players](https://open2077.net/docs/api/client/open77-players.md) — 52 functions.
- [Open77.prediction](https://open2077.net/docs/api/client/open77-prediction.md) — 2 functions.
- [Open77.prevention](https://open2077.net/docs/api/client/open77-prevention.md) — 4 functions.
- [Open77.Promise](https://open2077.net/docs/api/client/open77-promise.md) — 5 functions.
- [Open77.props](https://open2077.net/docs/api/client/open77-props.md) — 14 functions.
- [Open77.puppets](https://open2077.net/docs/api/client/open77-puppets.md) — 6 functions.
- [Open77.reflex](https://open2077.net/docs/api/client/open77-reflex.md) — 3 functions.
- [Open77.remoteCamera](https://open2077.net/docs/api/client/open77-remotecamera.md) — 15 functions.
- [Open77.resource](https://open2077.net/docs/api/client/open77-resource.md) — 7 functions.
- [Open77.rtti](https://open2077.net/docs/api/client/open77-rtti.md) — 5 functions.
- [Open77.runtime](https://open2077.net/docs/api/client/open77-runtime.md) — 6 functions.
- [Open77.screen](https://open2077.net/docs/api/client/open77-screen.md) — 14 functions.
- [Open77.session](https://open2077.net/docs/api/client/open77-session.md) — 18 functions.
- [Open77.settings](https://open2077.net/docs/api/client/open77-settings.md) — 3 functions.
- [Open77.sfx](https://open2077.net/docs/api/client/open77-sfx.md) — 6 functions.
- [Open77.state](https://open2077.net/docs/api/client/open77-state.md) — 7 functions.
- [Open77.stats](https://open2077.net/docs/api/client/open77-stats.md) — 5 functions.
- [Open77.time](https://open2077.net/docs/api/client/open77-time.md) — 1 function.
- [Open77.travel](https://open2077.net/docs/api/client/open77-travel.md) — 11 functions.
- [Open77.vehicles](https://open2077.net/docs/api/client/open77-vehicles.md) — 51 functions.
- [Open77.vfx](https://open2077.net/docs/api/client/open77-vfx.md) — 12 functions.
- [Open77.voice](https://open2077.net/docs/api/client/open77-voice.md) — 18 functions.
- [Open77.wardrobe](https://open2077.net/docs/api/client/open77-wardrobe.md) — 3 functions.
- [Open77.weapons](https://open2077.net/docs/api/client/open77-weapons.md) — 22 functions.
- [Open77.webui](https://open2077.net/docs/api/client/open77-webui.md) — 5 functions.
- [Open77.world](https://open2077.net/docs/api/client/open77-world.md) — 16 functions.
- [Open77.zones](https://open2077.net/docs/api/client/open77-zones.md) — 3 functions.
- [promise](https://open2077.net/docs/api/client/promise.md) — 1 function.
- [WebUI.Page](https://open2077.net/docs/api/client/webui-page.md) — 11 functions.
