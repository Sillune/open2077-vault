# Open77.pauseMenu (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-pausemenu.md) · [Page web](https://open2077.net/docs/api/client/open77-pausemenu)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

4 functions.

## getAppearance

```lua
Open77.pauseMenu.getAppearance([knownRevision])
```

`SHARED`: Available without a live game instance.

Reads the effective pause-menu branding and revision.

Read-only; no pause_menu.customize permission required. Returns revision, accentColor and logo, or nil without an error when knownRevision is unchanged. A local logo is returned as an image data URI, not a filesystem path; avoid logging or broadcasting it. See pause-menu.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `knownRevision` | `integer` | optional | — |

Returns: `table or nil`, `reason on failure`

Registered in `ResourceHost.cpp` (line 25344) as `LuaPauseGet`.

## resetAppearance

```lua
Open77.pauseMenu.resetAppearance()
```

`SHARED`: Available without a live game instance.

Releases this resource's pause-menu branding.

Requires pause_menu.customize. Removes both the accent and server-logo claim owned by the calling resource, including its manifest logo. Does not remove other resources' claims. The preceding resource's appearance or platform defaults become effective on the next menu refresh.

Returns: `true or nil`, `reason on failure`

Registered in `ResourceHost.cpp` (line 25343) as `LuaPauseReset`.

## setAccentColor

```lua
Open77.pauseMenu.setAccentColor(color)
```

`SHARED`: Available without a live game instance.

Sets this resource's pause-menu accent.

Requires pause_menu.customize. Accepts a #RRGGBB color; nil or an empty string releases this resource's accent claim. The most recent active resource claim wins and stopping the resource restores the previous claim or the default cyan. Does not recolor the OPEN//77 logo. See pause-menu.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `color` | `string or nil` | required | — |

Returns: `true or nil`, `reason on failure`

Registered in `ResourceHost.cpp` (line 25341) as `LuaPauseAccent`.

## setLogo

```lua
Open77.pauseMenu.setLogo(pathOrUrl)
```

`SHARED`: Available without a live game instance.

Adds a server logo beside the OPEN//77 pause-menu logo.

Requires pause_menu.customize. Accepts a supported image URL or a resource-relative image declared in files. nil or an empty string releases only this resource's logo claim. Branding never replaces the platform logo; stopping or reloading the owner restores the previous active claim. See pause-menu.md for formats, URL/path validation and manifest pause_menu_logo.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `pathOrUrl` | `string or nil` | required | — |

Returns: `true or nil`, `reason on failure`

Registered in `ResourceHost.cpp` (line 25342) as `LuaPauseLogo`.
