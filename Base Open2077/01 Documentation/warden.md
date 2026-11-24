# Warden

> Source officielle : [Markdown](https://open2077.net/docs/warden.md) · [Page web](https://open2077.net/docs/warden)

Manage the server from a browser: console, resources, players, configuration and operator permissions.

## Overview

Warden is bundled with the dedicated server and disabled by default. Many operations apply immediately; configuration and preload changes can require a restart. Check each operation's status before treating it as active.

## Enabling Warden

Enable the warden block in server.jsonc, then restart the server.

```jsonc
{
  "warden": {
    // Off by default. Set to true to serve the admin panel.
    "enabled": true,

    // The address the panel is served on. Default 11780. Keep it bound to
    // localhost (or behind a VPN / SSH tunnel) unless you know what you are
    // exposing — see the security note below.
    "bind": "127.0.0.1:11780"
  }
}
```

> The default local address is http://localhost:11780. Access remote installations through a VPN or SSH tunnel; avoid exposing the administration port publicly.

## First run: the setup PIN

On initial setup, Warden prints a one-time PIN in the server log. Use it to create the first administrator account.

01. **Read the PIN from the log**: On the first launch with Warden enabled, the server log prints a one-time setup PIN. It is shown once, in the console output you already have in front of you.
02. **Open the panel and enter it**: Browse to the Warden address (http://<host>:11780 by default), and it asks for that PIN. Entering it proves you are the operator and unlocks account creation.
03. **Create your admin account**: Set your admin username and password. This becomes the owner account for the panel, with full access to everything Warden can do. The setup PIN is spent and will not work again.
04. **Sign in from anywhere**: From then on you log in with that account. You can invite more admins and give each of them a role, so your moderators get exactly the access they need and no more.

## What you can do

Warden provides the following administrative tools. Restart requirements depend on the operation.

- **Live console & commands**: Watch the server's console stream in real time, and type any server command straight into it: the same commands you would run at the machine, from wherever you are. It is your primary window into what the server is doing.
- **Hot-reload Lua resources**: Edit a resource, then reload it from the panel and the server picks up the change on the spot. No restart, no dropping the players who are connected. Start, stop and restart individual resources to iterate on a game mode while it is live.
- **Player moderation**: See who is connected and act on them: warn, kick, or ban a player, and manage your ban list, all from the panel. The moderation actions that used to mean digging through commands are buttons next to each player.
- **Announcements**: Push a message to everyone on the server at once (a scheduled restart warning, an event kickoff, or a rules reminder) without joining the game yourself.
- **Config & identity**: Edit the server's configuration and identity (its name, visibility and public presentation) from the panel, so tuning the server does not mean editing files over SSH and restarting. Upload the server's icon and banner here too: the panel checks the size and shape before it accepts them, and a new image goes live without a restart.
- **Access & roles**: Add more admins and give each one a role, so your staff get scoped access: a moderator who can kick and ban without being able to rewrite the config, for instance. You decide who can do what.

## Keeping Warden secure

Warden is complete control of your server, so treat access to it accordingly.

- **Off unless you enable it**: Warden does nothing until you set warden.enabled to true. A server with no Warden block runs exactly as before.
- **Bind it privately**: Keep the panel on localhost, or behind a VPN or SSH tunnel, rather than exposing its port to the internet. Anyone who can reach the panel and hold an account can run your server.
- **The setup PIN is one-time**: The PIN only exists to bootstrap the first admin, and it is spent the moment that account is created. After that, access is your admin accounts and their roles.
