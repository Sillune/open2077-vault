# Open77.webui (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-webui.md) · [Page web](https://open2077.net/docs/api/client/open77-webui)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

5 functions.

## blocklist

```lua
Open77.webui.blocklist(rules)
```

`GAME`: Requires a live game instance.

Pushes the operator's ad-blocklist (hosts and URL tokens) to the browser host, which only ever adds them to its compiled list.

Client-only; requires `webui.policy`, a permission of its own because this is the one verb that widens what a page's browser refuses. `hosts` and `tokens` are arrays of at most 512 strings each (`hosts_out_of_range`, `hosts_must_be_strings`, and the same for `tokens`); `revision` is the operator's list revision the host will report back, `source` a label for its log. Answers `true` when the host accepted the push -- not when the rules are in force: read `Open77.webui.blocklistState()` afterwards, because a host older than the policy message accepts nothing and leaves no receipt. Refusals: `permission_denied:webui.policy`, `webui_policy_backend_unavailable`, `invalid_blocklist`, `invalid_hosts` / `invalid_tokens`, `blocklist_not_applied`. `open77_media` is the shipped caller: it applies the server's `open77:media:adblock` payload and reports the receipt back.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `rules` | `table { hosts?, tokens?, revision?, source? }` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Manifest: permissions { "webui.policy" }
RegisterNetEvent("myserver:adblock", function(payload)
    local ok, reason = Open77.webui.blocklist({
        hosts = payload.hosts or {}, tokens = payload.tokens or {},
        revision = tonumber(payload.revision) or 0, source = "myserver",
    })
    if not ok then print("blocklist refused: " .. tostring(reason)) end
end)
```

[WebUI & remote pages](../../01%20Documentation/resource-runtime.md#webui)

Registered in `ResourceHost.cpp` (line 24698) as `LuaWebUiBlocklist`.

## blocklistState

```lua
Open77.webui.blocklistState()
```

`GAME`: Requires a live game instance.

The browser host's receipt for the blocklist in force: revision, rule counts and the rules it refused.

Client-only; requires `webui.policy`. Answers `{ revision, compiledRules, hostRules, tokenRules, refused = { ... } }`, or `nil, reason` when there is nothing to describe: `no_blocklist_receipt` before any push was applied (or on a host older than the policy message), `webui_policy_backend_unavailable`, `permission_denied:webui.policy`. Compare `revision` with the one you pushed: a smaller value means the rules you sent are not the ones in force.

Returns: `table, or nil`, `reason`

```lua
-- Is the revision we pushed the one the host enforces?
local state, reason = Open77.webui.blocklistState()
if not state then print("no receipt: " .. tostring(reason)) return end
print(("revision %d: %d hosts + %d tokens compiled, %d refused"):format(
    state.revision, state.hostRules, state.tokenRules, #state.refused))
```

[WebUI & remote pages](../../01%20Documentation/resource-runtime.md#webui)

Registered in `ResourceHost.cpp` (line 24699) as `LuaWebUiBlocklistState`.

## budget

```lua
Open77.webui.budget()
```

`GAME`: Requires a live game instance.

How full the WebUI surface pool your pages are counted against is, and which resources hold it.

Client-only; no permission. Older clients do not have it: test that `Open77.webui.budget` (or `WebUI.budget`) exists before calling it. Every page is a browser of its own and the client caps how many are alive at once: 32 for the connected server's pages (all of its resources together, Open77's own server-side resources included), plus a separate pool of 8 for the client's built-in pages (shell, load screen, pause menu, blips, reticle, debug inspector), which a server can neither use nor fill. A `create` past the cap answers `nil, "webui_surface_quota"` and the client logs `Open77 WebUI surface refused for <resource>: webui_surface_quota, server 32/32: <holders>`. Answers `{ pool, used, limit, free, own, pages = { { resource, count }, ... } }` for the caller's pool: `pool` is `"server"` (or `"system"` for a built-in resource), `own` is how many of the pages are the caller's, and `pages` lists every holder, largest first. A page being destroyed is no longer counted; a hidden page is. Answers `nil, "webui_backend_unavailable"` where there is no WebUI. Counts only: no URL, size or content of anybody's page crosses this call.

Returns: `table, or nil`, `reason`

```lua
-- Create the phone only if the client still has room, and say who holds it if not.
-- Older clients have no budget(): test for it before calling.
local budget = Open77.webui.budget and Open77.webui.budget()
if budget and budget.free == 0 then
    local holders = {}
    for _, entry in ipairs(budget.pages) do
        holders[#holders + 1] = entry.resource .. " " .. entry.count
    end
    print(("no WebUI slot left (%d/%d): %s"):format(budget.used, budget.limit, table.concat(holders, ", ")))
    return
end
local page, reason = WebUI.create({ entry = "web/phone.html", visible = false })
if not page then print("phone page refused: " .. tostring(reason)) end
```

[WebUI & remote pages](../../01%20Documentation/resource-runtime.md#webui)

Registered in `ResourceHost.cpp` (line 27251) as `LuaWebUiBudget`.

## create

```lua
Open77.webui.create(options)
```

`SHARED`: Available without a live game instance.

Creates a web surface.

Creates a WebUI asynchronously. The handle returns at once and the page is built later, so `page:send` before the page's ready event is dropped; send the first state from `page:on("ui:ready", ...)`. For an initially visible surface, pass `visible = true` and let the page show or hide its own content: showing a surface again after hiding it may not repaint reliably.

Options, all optional: `entry` (default: the manifest's `web_ui_page`), `layer`, `zIndex`, `transparent`, `visible`, `width`/`height` (a logical reference size, fitted to the viewport), `fps` (clamped to 60), `policy` and `drawBounds`. Layers: `hud`, `menu`, `modal` (`webui.modal`), `system` (`webui.system`), `debug` (`webui.debug`); a layer sets composition order and takes no focus. `policy` is `strict` (default: only declared files) or `remote` (the page may fetch, embed and open sockets to any HTTP(S) host). An HTTP(S) `entry` does not require `web_files` and is always served as `remote`; bundled entries must be declared. External scripts, media, requests and embeds follow browser CORS and TLS rules. Only the top-level document of the entry origin receives the Lua bridge. An unknown `layer` or `policy` raises a Lua error. See [remote WebUI](../../01%20Documentation/resource-runtime.md#remote-pages-external-content-and-hot-reload) for runtime requirements.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table` | required | — |

Returns: `surface, or nil`, `reason`

```lua
local overlay = WebUI.create({
    entry = "web/index.html",
    layer = "hud",
    transparent = true,
    visible = true          -- the proven pattern: the page hides its own content until ready
})
overlay:on("ui:ready", function()          -- Open77.ready() in the page
    overlay:send("state", { phase = "lobby" })  -- a send before this is dropped
end)
```

[WebUI & remote pages](../../01%20Documentation/resource-runtime.md#webui)

Registered in `ResourceHost.cpp` (line 24694) as `LuaWebUiCreate`.

## default

```lua
Open77.webui.default()
```

`SHARED`: Available without a live game instance.

The surface declared by `web_ui_page` in the manifest.

Returns the auto-created page declared by `ui_page` (alias `web_ui_page`) in the current manifest, or `nil, reason` if none is live. The entry can be a declared local file or an HTTP(S) URL. Handles belong to the resource generation and become stale after destroy or reload. See [remote WebUI](../../01%20Documentation/resource-runtime.md#remote-pages-external-content-and-hot-reload).

Returns: `surface, or nil`

```lua
-- Use the page declared by web_ui_page in the manifest
local page = Open77.webui.default()
if page then page:send("hello", { name = "V" }) end
```

[WebUI & remote pages](../../01%20Documentation/resource-runtime.md#webui)

Registered in `ResourceHost.cpp` (line 24695) as `LuaWebUiDefault`.
