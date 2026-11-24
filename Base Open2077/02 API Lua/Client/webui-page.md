# WebUI.Page (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/webui-page.md) · [Page web](https://open2077.net/docs/api/client/webui-page)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

11 functions.

## destroy

```lua
WebUI.Page.destroy()
```

`SHARED`: Available without a live game instance.

Destroys the surface.

Destroys this resource-owned browser surface, removes all of its Lua event handlers and invalidates the page handle. Resource teardown performs the same cleanup automatically.

Returns: `boolean`

```lua
-- Tear the menu surface down when the player leaves the shop
AddEventHandler("shop:left", function()
    if page then page:destroy(); page = nil end
end)
```

Registered in `ResourceHost.cpp` (line 24438) as `LuaWebPageDestroy`.

## hasFocus

```lua
WebUI.Page.hasFocus()
```

`GAME`: Requires a live game instance.

Check whether this resource-owned page currently owns keyboard or cursor focus.

Call page:hasFocus(). Reads actual focus ownership and presentation availability without changing either. Returns false if another surface owns focus or this surface cannot present. Only accepts a live page handle owned by the calling resource. Combine with setFocus(false,false), hide and resource-scoped control cleanup when closing a modal. See screen-picking.md and the generic open77_contextmenu package.

Returns: `boolean`

```lua
page:show()
page:setFocus(true, true)
print(page:hasFocus())
page:setFocus(false, false)
page:hide()
```

[WebUI keyboard input guide](../../01%20Documentation/webui-input.md)

Registered in `ResourceHost.cpp` (line 24042) as `LuaWebPageHasFocus`.

## hide

```lua
WebUI.Page.hide()
```

`SHARED`: Available without a live game instance.

Hides the surface.

Makes the owned browser surface invisible without destroying its document or handlers. Visibility is client-local and can be restored with `show()`.

Returns: `boolean`

```lua
-- Hide the HUD page during the cutscene, keeping its document alive
AddEventHandler("cutscene:started", function() page:hide() end)
AddEventHandler("cutscene:ended", function() page:show() end)
```

Registered in `ResourceHost.cpp` (line 24440) as `LuaWebPageVisibility`.

## id

```lua
WebUI.Page.id()
```

`SHARED`: Available without a live game instance.

Numeric id of the surface.

Returns the opaque client-local surface id for diagnostics and host integration. It is not a network id and must not be persisted across page destruction or resource reload.

Returns: `integer`

```lua
-- Log the surface id for a bug report
print("hud surface id", page:id())
```

Registered in `ResourceHost.cpp` (line 24437) as `LuaWebPageId`.

## off

```lua
WebUI.Page.off(handlerId)
```

`SHARED`: Available without a live game instance.

Removes a handler registered with `on`.

Removes one WebUI callback id previously returned by `page:on`. The id is the only argument -- there is no event name parameter. It must belong to this page and the current resource generation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `handlerId` | `integer` | required | — |

Returns: `boolean`

```lua
-- Stop listening to the page's "buy" event after checkout
local handler = page:on("buy", function(payload) buy(payload.item) end)
AddEventHandler("shop:checkout", function() page:off(handler) end)
```

Registered in `ResourceHost.cpp` (line 24446) as `LuaWebPageOff`.

## on

```lua
WebUI.Page.on(event, handler)
```

`SHARED`: Available without a live game instance.

Listens for an event the page raised with `Open77.emit`.

Registers a Lua callback for an event emitted by this page's JavaScript bridge. Event names and handler counts are bounded, and the registration is automatically removed with the page or resource generation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `handler` | `function` | required | — |

Returns: `handler id`

```lua
-- The page's JavaScript called Open77.emit("buy", { item = "ammo" })
page:on("buy", function(payload)
    TriggerServerEvent("shop:buy", payload.item)
end)
```

Registered in `ResourceHost.cpp` (line 24445) as `LuaWebPageOn`.

## reply

```lua
WebUI.Page.reply(requestId, payload, [ok])
```

`SHARED`: Available without a live game instance.

Answers an `invoke` call made by the page.

Completes a pending JavaScript request identified by its request id with a JSON-serializable Lua value. The payload is the SECOND argument and the success flag the third: `page:reply(requestId, { accepted = true })` resolves the promise, and a third argument of `false` rejects it. Success defaults to true when omitted. The payload is positional and cannot be skipped -- an absent third argument fails with `invalid_webui_reply`, though an explicit `nil` is accepted. Request ids are page-local and invalid or unsupported payloads are rejected.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `requestId` | `integer` | required | — |
| `payload` | `table` | required | — |
| `ok` | `boolean` | optional | `true` |

Returns: `boolean`

```lua
-- Answer an invoke() from the page with the player's balance
page:on("getBalance", function(payload, requestId)
    page:reply(requestId, { balance = balance })
end)
```

Registered in `ResourceHost.cpp` (line 24447) as `LuaWebPageReply`.

## send

```lua
WebUI.Page.send(event, [payload])
```

`SHARED`: Available without a live game instance.

Sends an event to the page.

Sends a named event and JSON-serializable payload from Lua to this page's JavaScript runtime. This is local UI IPC, not a network event, and both event names and payloads are bounded.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `payload` | `table` | optional | — |

Returns: `boolean`

```lua
-- Push the new balance to the page
RegisterNetEvent("bank:balance", function(amount)
    page:send("balance", { amount = amount })
end)
```

Registered in `ResourceHost.cpp` (line 24444) as `LuaWebPageSend`.

## setConsumedKeys

```lua
WebUI.Page.setConsumedKeys(keys)
```

`GAME`: Requires a live game instance.

Consume selected navigation keys while preserving gameplay input.

Call page:setConsumedKeys(keys) before page:setFocus(true,false,true). Client-local, per-surface policy: selected keys still reach the browser but not Cyberpunk Raw Input or Open77 pause/scoreboard shortcuts. Accepts a dense array of at most 32 case-insensitive names: escape/esc, enter, tab, backspace, space, up/down/left/right (arrow aliases accepted), home, end, pageup, pagedown, insert and delete. Invalid input leaves the existing list unchanged; {} clears it for future presses. Active only while this visible, presentation-enabled page owns keyboard focus. Repeats and releases remain consumed if the keydown handler hides/destroys the page or changes focus. No arbitrary gameplay keys or controller-menu interception. No permission of its own; the keepInput argument of setFocus requires webui.keep_input. Check method availability before using this mode. See webui-input.md for the complete lifecycle and error contract.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `keys` | `string[]` | required | — |

Returns: `boolean`, `string? reason`

```lua
-- Manifest: permissions { 'webui.keep_input' }
assert(page:setConsumedKeys({'escape'}))
assert(page:setFocus(true, false, true))
-- Close the menu:
page:setFocus(false, false)
page:hide()
```

[WebUI keyboard input guide](../../01%20Documentation/webui-input.md)

Registered in `ResourceHost.cpp` (line 24442) as `LuaWebPageConsumedKeys`.

## setFocus

```lua
WebUI.Page.setFocus(keyboard, [cursor], [keepInput])
```

`SHARED`: Available without a live game instance.

Gives the surface focus, cursor and keyboard separately.

The FIRST argument is keyboard capture, not a general `focused` flag, and the second is the cursor: `page:setFocus(true, true)` takes both, `page:setFocus(false, true)` takes only the mouse. Keyboard capture is asked for separately so a page can take the mouse without depriving the game of the movement keys. The optional third argument keeps game input alive while the page holds focus and requires `webui.keep_input`; without that permission it fails with `permission_denied:webui.keep_input`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `keyboard` | `boolean` | required | — |
| `cursor` | `boolean` | optional | — |
| `keepInput` | `boolean` | optional | — |

Returns: `boolean`

```lua
-- Open the menu: take the mouse, leave the keyboard to the game
page:show()
page:setFocus(false, true)
```

[WebUI keyboard input guide](../../01%20Documentation/webui-input.md)

Registered in `ResourceHost.cpp` (line 24441) as `LuaWebPageFocus`.

## show

```lua
WebUI.Page.show()
```

`SHARED`: Available without a live game instance.

Shows the surface.

Avoid calling this right after `create` — see the race described in `Open77.webui.create`.

Returns: `boolean`

```lua
-- Show the menu surface (created hidden) when the key is pressed
RegisterKeyMapping("shop_menu", "Shop menu", "F3", function()
    page:show()
    page:setFocus(true, true)
end)
```

Registered in `ResourceHost.cpp` (line 24439) as `LuaWebPageVisibility`.
