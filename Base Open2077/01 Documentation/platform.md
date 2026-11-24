# How OPEN//77 works

> Source officielle : [Markdown](https://open2077.net/docs/platform.md) · [Page web](https://open2077.net/docs/platform)

Connect players to community-operated Cyberpunk 2077 servers and build gameplay with Lua resources.

## Overview

OPEN//77 is a multiplayer platform for Cyberpunk 2077. Each community operates its own dedicated server, gamemode and rules.

### OPEN//77 is not

- One official multiplayer server run by us
- A fixed game mode you have to play
- A peer-to-peer co-op session mod
- A product of CD PROJEKT RED

### OPEN//77 is

- The infrastructure that lets anyone run a Cyberpunk 2077 server
- A client that discovers and connects you to community servers
- A creator toolkit for building custom game modes and systems
- Common ground for players, server owners and developers

## How it works (for players)

1. **Own the game**: OPEN//77 requires your own legal copy of Cyberpunk 2077, at build 2.5, with Phantom Liberty DLC. The platform never distributes game content. It builds on the game you bought. Pirated or cracked copies are not supported and not welcome.
2. **Install the client**: The OPEN//77 client runs alongside your installation. Your single-player game, saves and mods stay untouched.
3. **Browse the servers**: Open the server browser, filter by game mode or language, read a server's page, and pick the world you want to live in tonight.
4. **Connect & play**: The client fetches that server's resource set, verifies its signature and content hashes, and drops you into Night City alongside everyone else on that server.

## Requirements

Players need a legal copy of Cyberpunk 2077 at the exact supported game build, with Phantom Liberty installed. Newer game builds are not automatically compatible; see the requirements below.

### To play

- **Your own legal copy of Cyberpunk 2077**: OPEN//77 never distributes the game or any of its assets. It builds on the installation you already own: bought, not pirated. Cracked copies are not supported and not welcome.
- **Game build 2.5**: This exact build, not a minimum. The client hooks the game engine at addresses established for it and refuses to load when they do not match.
- **Phantom Liberty DLC**: The expansion ships as EP1, and the client needs it: the world it loads when you connect to a server is an EP1 save.
- **Windows 10 or 11, 64-bit**: The client is a native Windows plugin that loads inside the game process.

### To host a server

The dedicated server runs independently of the game and does not require game assets.

- **The game is not needed**: A dedicated server runs independently of Cyberpunk 2077, REDengine and the client plugin. It never loads game content and never needs a copy installed.
- **Windows x64 or Linux x64**: Both official server packages include the .NET runtime and native networking dependencies. No separate .NET installation or copy of the game is needed on the host.
- **Your players still need all of the above**: Everyone who connects needs Cyberpunk 2077 2.5 + Phantom Liberty DLC on their own machine. Hosting changes nothing about that.

## Dedicated servers

A dedicated server owns shared world state and accepts player connections. OPEN//77 does not use peer-to-peer sessions.

- **Persistent worlds**: The world keeps running when you log off. Economies, factions and stories continue. The server remembers.
- **Authoritative state**: Positions, inventories, vehicles, loot, time and weather: the server decides what is true, and clients render approved state. That is what makes real economies and fair PvP possible.
- **Operated by communities**: Anyone can download the Windows or Linux server, create a license with a verified account and build their own worlds. Open Alpha needs no access application or approval.

## Resources and scripting

Gameplay is packaged as resources: self-contained directories with a manifest, Lua scripts, declared permissions, dependencies and optional web interfaces. The server picks the resource set for a session; connecting clients download it, verify its signature and content hashes, and activate it before entering the world.

Documented systems:

- **Vehicles** (/docs/vehicles): network identity, streaming, authority leases, seats, doors and damage
- **NPCs** (/docs/npcs): server-owned templates, streaming, task queues and life state
- **Loot** (/docs/loot): authoritative ground drops and validated pickups
- **Time and weather** (/docs/weather): synchronised session time and weather presets
- **Elevators** (/docs/elevators): server-authoritative native lifts with late-join catch-up
- **Interactions** (/docs/interactions): contextual world and NPC prompts with action keys
- **Custom UI** (/docs/notifications): WebUI pages, toasts, blips and map pins
- **Identity and ACL** (/docs/server-acl): durable player ids, whitelists and restricted commands

Resource manifest (`resources/hello/open77.lua`):

```lua
resource "hello"
version "1.0.0"
auto_start true

client_script "client/main.lua"
server_script "server/main.lua"

permissions { "network.events", "world.loot" }
```

Server entry point (`resources/hello/server/main.lua`):

```lua
RegisterCommand("hello", function(source, args)
    print(("player %d said hello"):format(source))
end, false)
```

Every registered function is listed in the Lua API reference at /docs/api, separated by runtime so a client projection is never mistaken for server authority. APIs can change during Alpha; read each guide's limitations before depending on a feature.

## Roadmap

Alpha members can play, download the server and build custom gamemodes now. Development priorities include reliability, compatibility and the resource ecosystem; release dates are not guaranteed.

### NOW (ALPHA): Build and test today

Launcher and Windows/Linux server packages, Freeroam, a live server browser, WebUI and documented client/server Lua APIs. Everyone can start building in Open Alpha without an application or approval.

### NEXT: Reliability and developer feedback

Broader fresh-install and multiplayer validation, reconnect and animation fixes, clearer diagnostics and improved hosting workflows.

### THEN: Compatibility and capacity

Measure representative server workloads and hardware, reduce breaking changes, and document supported combinations as they are validated.

### BEYOND: The ecosystem

Featured communities, server reputation, resource sharing between servers: the parts that only matter once real worlds exist. The community will shape these.

## FAQ

### Is OPEN//77 official? Is CD PROJEKT RED involved?

No. OPEN//77 is an independent community project. It is not affiliated with, endorsed by, or supported by CD PROJEKT RED. Cyberpunk 2077 is the property of CD PROJEKT S.A. We build alongside the game, not on their behalf.

### Do I need to own Cyberpunk 2077?

Yes, always. OPEN//77 never distributes the game or its assets. You need your own legal copy of Cyberpunk 2077 on 64-bit Windows, at game build 2.5, with Phantom Liberty DLC installed. The platform adds multiplayer infrastructure on top of it.

### Does OPEN//77 work with pirated or cracked copies?

No. A legal copy of Cyberpunk 2077 and its required expansion is mandatory. Pirated or cracked installations are unsupported.

### Is Phantom Liberty DLC required, or only recommended?

Required. The expansion ships as the EP1 content set, and the world the client loads when you connect to a server is an EP1 save. Without the expansion installed there is nothing for it to load. The base game on its own is not enough.

### Do I need Cyberpunk 2077 to host a server?

No. The dedicated server runs independently of Cyberpunk 2077 on Windows x64 or Linux x64. Both published packages include the .NET runtime. Everyone who connects still needs their own Cyberpunk 2077 2.5 installation with Phantom Liberty DLC.

### So is this one big multiplayer server?

No, and this is the core idea. OPEN//77 is the platform underneath many servers. Communities run their own independent worlds with their own game modes and rules; the client lets you browse and join them. If FiveM's model for GTA V is familiar, that is the shape.

### Can I play it right now?

Yes. Open Alpha is available to everyone. Download the launcher and sign in with your free OPEN//77 account to join a server. No access application or approval is needed. Alpha can still have bugs, crashes and incomplete features; this is not a stable release.

### Can I host my own server?

Yes. Anyone can download the Windows or Linux server and start developing. Downloads are public; a free account with a verified email lets you create a license key. The Freeroam template and system resources are included. Follow the hosting guide, configure your license and reachable endpoints, and invite your community.

### What can server creators actually customize?

Servers run resources: packages of server-side and client-side Lua with a manifest, declared permissions and optional web interfaces. The documented systems today include vehicles, NPCs, loot, time and weather, elevators, contextual interactions, chat, notifications, blips and visual effects. A pure racing server and a hardcore roleplay city can both be OPEN//77 servers.

### Will it break my single-player game or saves?

The client is designed to run alongside your installation without touching your single-player saves or mod setup. Multiplayer state lives on the server you join.

### How much will it cost?

The platform is intended to be free to use. Players must purchase Cyberpunk 2077 and the required expansion; server owners cover their hosting costs.
