# Open77.notifications (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-notifications.md) · [Page web](https://open2077.net/docs/api/server/open77-notifications)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

5 functions.

## broadcast

```lua
Open77.notifications.broadcast(definition)
```

`NETWORK`: Uses the network backend.

Sends one notification to every player.

`send` with a player id of `-1`. The returned id addresses the whole broadcast, so one `update` or `dismiss` reaches everyone who received it. Delivered through `TriggerClientEvent`, so it needs `network.events` and works from the top level of a script as well as from handlers (unlike the `Open77.chat` facades, which publish on the host bus and answer `resource_preparing` until the chunk has loaded). Declare `dependency "open77_notifications"` in the manifest: a server whose load list lacks the package then refuses to start the resource instead of dropping every toast.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `definition` | `table` | required | — |

Returns: `owner-local broadcast id, or nil`, `reason`

```lua
-- Everybody sees the storm warning for ten seconds
Open77.notifications.broadcast({
    type = "warning",
    title = "Weather",
    message = "A sandstorm is rolling in.",
    durationMs = 10000,
})
```

Registered in `LuaResourceRuntime.cs` (line 8234) as `LuaResourceRuntime.Bootstrap`.

## clear

```lua
Open77.notifications.clear([playerId])
```

`NETWORK`: Uses the network backend.

Clears this resource's notifications for one player or for everyone.

Omitting the player clears this owner's notifications everywhere. It never touches another resource's notifications, so a gamemode clearing its own HUD does not wipe a chat or admin toast.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | optional | — |

Returns: `boolean`, `reason`

```lua
-- The heist is over: drop every toast this resource still shows
Open77.notifications.clear()
```

Registered in `LuaResourceRuntime.cs` (line 8237) as `LuaResourceRuntime.Bootstrap`.

## dismiss

```lua
Open77.notifications.dismiss(id)
```

`NETWORK`: Uses the network backend.

Dismisses one notification this resource sent.

Only the owning resource may dismiss it. Dismissing one that has already expired succeeds.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `notification id` | required | — |

Returns: `boolean`, `reason`

```lua
-- Remove the "waiting for players" toast once the round starts
local waiting = Open77.notifications.send(playerId, { message = "Waiting for players...", durationMs = 0 })
AddEventHandler("arena:roundStarted", function()
    Open77.notifications.dismiss(waiting)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8236) as `LuaResourceRuntime.Bootstrap`.

## send

```lua
Open77.notifications.send(playerId, definition)
```

`NETWORK`: Uses the network backend.

Sends a notification to one player.

Routes to the official `open77_notifications` client package. Ids and mutation rights are isolated per calling server resource, so one resource cannot dismiss another's notification. See [notifications](https://open2077.net/docs/api/server/notifications.md) for every definition field and the queue limits. Delivered through `TriggerClientEvent`, so it needs `network.events` and works from the top level of a script as well as from handlers (unlike the `Open77.chat` facades, which publish on the host bus and answer `resource_preparing` until the chunk has loaded). No dependency is checked at the call: this is a prelude function whose only gate is `network.events`, and the toast is drawn by the `open77_notifications` client script that every shipped load list auto-starts, so a resource without `dependency "open77_notifications"` still delivers. The line is a guard: with it, the package starts first and a server whose load list lacks it refuses to start your resource instead of dropping every toast on clients that never received the package.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `definition` | `table` | required | — |

Returns: `owner-local notification id, or nil`, `reason`

```lua
-- Paid for a completed delivery
RegisterNetEvent("delivery:done", function()
    Open77.notifications.send(source, {
        type = "success",
        title = "Delivery complete",
        message = "You earned 350 eddies.",
        icon = "E$",
        durationMs = 6000,
    })
end)
```

Registered in `LuaResourceRuntime.cs` (line 8233) as `LuaResourceRuntime.Bootstrap`.

## update

```lua
Open77.notifications.update(id, patch)
```

`NETWORK`: Uses the network backend.

Patches a notification this resource sent.

Sparse patch. Use it for a live counter or a progress line rather than dismissing and resending, which would restart the entry's animation and its place in the queue.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `notification id` | required | — |
| `patch` | `table` | required | — |

Returns: `boolean`, `reason`

```lua
-- A live progress counter, patched in place instead of re-sent
CreateThread(function()
    local id = Open77.notifications.send(playerId, { message = "Loading 0 %", durationMs = 0 })
    for percent = 10, 100, 10 do
        Wait(500)
        Open77.notifications.update(id, { message = ("Loading %d %%"):format(percent) })
    end
    Open77.notifications.dismiss(id)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8235) as `LuaResourceRuntime.Bootstrap`.
