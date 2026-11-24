# Host your own server

> Source officielle : [Markdown](https://open2077.net/docs/host-a-server.md) · [Page web](https://open2077.net/docs/host-a-server)

Anyone can download and run a dedicated server in Open Alpha. Create a license with a verified account, configure reachable endpoints and invite your community. Cyberpunk 2077 is not required on the host.

## Overview

The dedicated server is a self-contained Windows or Linux application. It uses a platform license to register with the Master and publish its name, endpoint and status. Public, eligible servers appear in the directory; private or hidden servers do not.

## From download to open doors

01. **Get the build**: Download the official dedicated server for your platform from the Host a Server page. Windows ships as a .zip, Linux as a self-contained .tar.gz (built for Debian and compatible distributions). Every archive lists its SHA-256 so you can verify exactly what you run.
02. **Mint a license key**: In your account, open the keymaster and create a license key. It ties the server to your account and is shown once. Copy it now, because the platform only stores a fingerprint and can never show it again.
03. **Unpack and run the first-run setup**: Unzip (Windows) or untar (Linux) into a folder of your choice. The first time you launch with no server.jsonc present, the server opens a first-run setup, a small wizard in your browser that walks you through naming the server, pasting your license key and picking visibility, then writes the config for you.
04. **Check server.jsonc**: Whether the wizard wrote it or you prefer to edit by hand, confirm the handful of fields that matter: the server's name and visibility, its public address, and that the master server is enabled. The defaults already point at the production master.
05. **Set the license key in the environment**: Export OP77_LICENSE_KEY so the server can present it to the master on start-up. Keeping the key in the environment rather than the file means server.jsonc stays safe to copy, back up or share.
06. **Run it, and it appears automatically**: Start the server. It enrols, registers and begins heartbeating, and within a heartbeat it is live in the launcher and on the public server list under your account. Players can join straight away.

## What the host needs

The host is deliberately light. You do not install Cyberpunk 2077, REDengine or any game content on the server machine. The server ships everything it needs.

- **The self-contained server package**: Both Windows x64 and Linux x64 downloads include the .NET runtime, native dependencies, Freeroam and its system resources. Extract the archive and use Start.cmd on Windows or ./start.sh on Linux; no separate .NET installation is needed.
- **A public address**: A hostname or IP, and the server's port, that players on the internet can reach. This is what goes in network.publicEndpoint, and it is how the launcher tells people where to connect. Behind a router or firewall you forward the port; on a hosting box it is usually the public IP directly.
- **A license key**: One op77_live_ key minted in the keymaster. No key means no enrolment and no listing. Anonymous servers are not part of the platform.

## Get the build

Head to the Host a Server page and download the archive for your platform. Both builds contain the same server; they differ only in packaging and how you launch them.

| Platform | Archive | Run |
| --- | --- | --- |
| Windows (x64) | .zip | `Open77.Server.exe (or dotnet Open77.Server.dll)` |
| Linux (x64, Debian) | .tar.gz, self-contained | `./Open77.Server` |

Only ever download the server from the Host a Server page or the official CDN, nowhere else. Each release publishes a SHA-256 for every archive; compare it against the file you downloaded before you run it. Stable is the default. For testing, expand ‘Looking for a preview build?’ on /host and acknowledge the warning to reveal Unstable downloads. Back up configuration, resources and databases and extract into a separate folder. Players must select Unstable in launcher Settings and use the compatible client version listed with the server. Do not overwrite a live server with a preview build.

## First-run setup

If server.jsonc is absent on first launch, a browser-based setup wizard creates it.

Enter the server name, license key, public endpoint and visibility. Review the generated server.jsonc before starting. Later changes can be made in the file or through Warden.

> To skip the wizard, place a valid server.jsonc beside the server executable before launch.

## Configure server.jsonc

server.jsonc supports JSON with // comments. Configure the following fields for enrollment and directory visibility.

```jsonc
{
  // How your server presents itself in the launcher and server list.
  "identity": {
    "name": "Neon District RP",
    "visibility": "public"          // "public" to be listed; "private" to hide
  },

  // The address players connect to. Use a hostname or public IP plus the port
  // players can reach — not a LAN address.
  "network": {
    "publicEndpoint": "play.example.com:27015"
  },

  // The platform link. Leave enabled so the server enrols and gets listed;
  // the url already defaults to the production master, so you rarely touch it.
  "masterServer": {
    "enabled": true,
    "url": "https://master.open2077.net/",

    // Prefer the OP77_LICENSE_KEY environment variable and leave this null so
    // the file stays safe to share. Set it here only for a private deployment.
    "licenseKey": null
  }
}
```

- **identity.name**: The name players see in the launcher and on the server list. Make it recognisable. It is your server's front door.
- **identity.visibility**: Set to "public" to appear in the launcher and on open2077.net. "private" keeps the server enrolled and licensed but off the public list, for a closed group.
- **network.publicEndpoint**: The address the launcher hands to players. It must be reachable from the internet: a public hostname or IP and the server's port, not a 192.168.x address.
- **masterServer.enabled**: Leave this true. It is the switch that makes the server enrol with the platform and get listed. Its url already points at the production master, so you almost never change it.

## Set the license key

Set your license key in the environment before you start the server. This keeps the key out of server.jsonc, so the config stays safe to copy or back up.

```bash
# Linux / macOS
export OP77_LICENSE_KEY="op77_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# Windows (PowerShell)
$env:OP77_LICENSE_KEY = "op77_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

## Run it

With the key set and server.jsonc in place, start the server the way its platform expects:

```bash
# Windows, from the unpacked folder
Open77.Server.exe
#   …or, against an installed .NET 10 runtime:
dotnet Open77.Server.dll

# Linux, from the unpacked folder
./Open77.Server
```

[Startup arguments, log levels and hosting-panel configuration](server-startup.md)

## It appears automatically

On startup, the server enrolls and registers automatically:

1. **It enrols with the master**: The server presents your license key. The master verifies it, binds the server to your account, and issues a short-lived run lease that is renewed on every heartbeat.
2. **It registers itself**: The server publishes its identity (name, public endpoint, player count and icon) to the master. There is no separate listing form; the config is the listing.
3. **It appears in the launcher**: Within a heartbeat the server shows up in the in-game launcher and on the public server list at open2077.net, under your account. Players pick it and connect. Stop the server, or revoke its key, and it drops off the list just as automatically.

> If the server is missing from the directory, check its license, public visibility, master connection and endpoint. LAN and loopback endpoints are not public listings. A platform administrator can also hide a server. For connection failures, check external port reachability.

## Next: run it live with Warden

Use Warden to view logs, manage players and control resources. Some configuration and preload changes require a server restart.
