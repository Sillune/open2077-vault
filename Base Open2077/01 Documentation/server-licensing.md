# Server licensing

> Source officielle : [Markdown](https://open2077.net/docs/server-licensing.md) · [Page web](https://open2077.net/docs/server-licensing)

A platform license associates a dedicated server with its owner's account. Any free account with a verified email can create a key in Open Alpha. Configure it on the server and authenticate with the Master at startup; no access application or staff approval is needed.

## Overview

Servers need a valid license for Master enrollment, directory registration and run leases. Manage keys in your account. An invalid or revoked key prevents enrollment.

## From zero to a listed server

01. **Create your account**: Register on open2077.net with an e-mail and password, then verify your e-mail. One account covers everything: the server browser, your license keys, and your game identities. E-mail verification is required before you can create a key.
02. **Mint a license key**: In your account, open Server license keys and create one. Give it a label so you can tell your servers apart. The key is shown once. Copy it now; the platform only stores a fingerprint and can never show it again.
03. **Give the key to your server**: Point your server config at the key, through the OP77_LICENSE_KEY environment variable (recommended) or the masterServer.licenseKey field. Keep the key out of any file you commit or share.
04. **Start the server**: On boot the server presents its key to the master, is authorised, and appears in the browser under your account. Revoke the key later and the server drops off the platform.

## Your license key

A license key looks like op77_live_ followed by 43 characters. Treat it like a password: anyone holding it can enrol a server against your account. If a key leaks, revoke it in the keymaster and mint a new one.

- **Shown once**: The full key is displayed only at creation. The platform stores a one-way fingerprint, so support can never recover it. Losing it means revoke and replace.
- **One key, many servers**: A single key can enrol more than one server. Use separate keys per community or per environment so you can revoke one without taking the others down.
- **Up to twenty active keys**: Each account can hold twenty active license keys at a time. Revoked keys do not count against the limit.
- **Revocable at any time**: Revoking a key immediately cuts off every server enrolled with it. The master stops authorising them and they leave the browser.

## Linking the key to your server

Supply the key through the server environment or configuration. Prefer the environment variable to keep secrets out of shared files.

Set OP77_LICENSE_KEY in the server's environment and leave the config credential-free:

```bash
# Linux / macOS
export OP77_LICENSE_KEY="op77_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# Windows (PowerShell)
$env:OP77_LICENSE_KEY = "op77_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

Or put it in the masterServer block of server.jsonc. The masterServer.licenseKeyEnvironmentVariable field lets you rename the variable the server looks for; it defaults to OP77_LICENSE_KEY, and an explicit licenseKey wins when both are set:

```jsonc
{
  "masterServer": {
    "enabled": true,
    "url": "https://master.open2077.net/",

    // Prefer the environment variable and leave this null so the file
    // stays safe to commit. Set it here only for a private deployment.
    "licenseKey": null,
    "licenseKeyEnvironmentVariable": "OP77_LICENSE_KEY"
  }
}
```

Never commit a real key. The tracked server.jsonc should keep licenseKey null and rely on the environment variable; a leaked key should be revoked in the keymaster, not merely rotated in the file.

## How the master authorises your server

The Master checks the key at enrollment and renews the server's run lease through heartbeats.

1. **The key is verified**: The master looks up your key. An unknown key, a revoked key, or a suspended owner is refused with a single error: the server logs "Master refused the platform license key…" and does not appear in the browser.
2. **The server is bound to your license**: On success the server is tied to your account and license, and its identity key is registered. That binding is what puts the server under your name in the browser and lets you manage it from your account.
3. **It is issued a run lease**: The master hands the server a short-lived, signed run lease, renewed every heartbeat, that stands as proof the server is licensed and current. Stop heartbeating, or lose the license, and the lease is not renewed.
4. **Revocation takes effect immediately**: Revoke the key in the keymaster and the cascade is instant: the server's credential is cut, no new lease is issued, and it drops out of the browser on its next request.

> Master-side licensing controls enrollment, directory listing and run leases. Do not rely on it as the sole client-admission control; configure server access rules separately.

## Managing your keys

Your account shows each key's label, fingerprint and creation date. Revoke keys for retired servers or compromised credentials.
