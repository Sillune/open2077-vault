# Open77.net (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-net.md) · [Page web](https://open2077.net/docs/api/client/open77-net)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

6 functions.

## call

```lua
Open77.net.call(name, [...])
```

`NETWORK`: Uses the network backend.

Asks the server a question and returns a promise for the answer.

Requires network.events. The name alone targets the handler this same resource registered on the server; the table form { resource = "other", name = "x", timeout = ms } targets another resource or sets a deadline (default 10000 ms, range 100..120000). Resolves with the handler's return values, or rejects with a reason such as callback_not_found, callback_timeout or callback_target_stopped. Up to 29 arguments and 48 KiB per frame; 256 questions open per resource. Open77.net.callAwait(...) is the same call followed by :await(), for use inside a CreateThread. Aliased as Open77.callbacks.call. See the Network callbacks guide.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string|table` | required | — |
| `...` | `any` | optional | — |

Returns: `Open77.Promise, or nil, reason`

```lua
-- Ask the server for the stock and price of an item
CreateThread(function()
    local pending, reason = Open77.net.call("getStock", "medkit")
    if not pending then return print(reason) end
    local count, price = pending:await()
    print(count, price)
end)
```

Registered in `ResourceHost.cpp` (line 24654) as `LuaCallbackCall`.

## callAwait

```lua
Open77.net.callAwait(name, [...])
```

`NETWORK`: Uses the network backend.

Asks the server a question and waits for the answer.

Requires network.events. Open77.net.call followed by :await() on the promise it returns, so it must run inside a CreateThread or another managed task; outside one the await answers nil, await_requires_scheduler_coroutine instead of blocking the frame. Same targeting, timeouts, limits and failure reasons as Open77.net.call. Defined by the client's callback prelude rather than by a native. Aliased as Open77.callbacks.callAwait. See the Network callbacks guide.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string|table` | required | — |
| `...` | `any` | optional | — |

Returns: `the handler's return values, or nil, reason`

```lua
CreateThread(function()
    local count, price = Open77.net.callAwait("getStock", "medkit")
end)
```

Registered in `ResourceHost.cpp` (line 23181) as `LuaPrelude`.

## emitServer

```lua
Open77.net.emitServer(event, [...])
```

`NETWORK`: Uses the network backend.

Sends an event to the server.

Alias of `TriggerServerEvent`: sends a bounded, serializable event payload to the authenticated server session. The current resource must declare `network.events`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `...` | `any (up to 32 values: strings, numbers, booleans, tables)` | optional | — |

Returns: `boolean`

```lua
-- Namespaced spelling of TriggerServerEvent
Open77.net.emitServer("shop:buy", { item = "ammo_box" })
```

Registered in `ResourceHost.cpp` (line 24651) as `LuaTriggerServerEvent`.

## on

```lua
Open77.net.on(event, handler)
```

`NETWORK`: Uses the network backend.

Listens for an event coming from the server.

Registers an authenticated incoming server event and its callback for the current resource generation. It is the namespaced equivalent of `RegisterNetEvent` and requires `network.events`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `handler` | `function` | required | — |

Returns: `handler id`

```lua
-- Namespaced spelling of RegisterNetEvent
Open77.net.on("bank:deposit", function(amount)
    Open77.hud.notify(("Deposited %d eddies"):format(amount))
end)
```

Registered in `ResourceHost.cpp` (line 24650) as `LuaRegisterNetEvent`.

## register

```lua
Open77.net.register(name, handler)
```

`NETWORK`: Uses the network backend.

Answers a callback the server sends with Open77.net.callClient.

Requires network.events. The handler receives the call's arguments (the caller is always the server, so there is no source) and its return values become the answer; raising rejects the caller's promise with the error text. Callback names are per resource: the server reaches this handler by naming this resource. Up to 256 names and 256 in-flight calls per resource; stopping the resource answers what it was still serving with callback_target_stopped. Aliased as Open77.callbacks.register. See the Network callbacks guide.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `handler` | `function` | required | — |

Returns: `true, or nil, reason`

```lua
-- Answer the server's "confirm this purchase?" with a dialog result
Open77.net.register("confirmPurchase", function(item, price)
    return confirmDialog(("Buy %s for %d?"):format(item, price))
end)
```

Registered in `ResourceHost.cpp` (line 24652) as `LuaCallbackRegister`.

## unregister

```lua
Open77.net.unregister(name)
```

`NETWORK`: Uses the network backend.

Stops answering a callback name registered with Open77.net.register.

Requires network.events. A later request for that name is refused with callback_not_found on the next frame rather than waiting for a timeout. Aliased as Open77.callbacks.unregister.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |

Returns: `true, or false, reason`

```lua
-- Stop answering the server's question once the dialog is gone
Open77.net.unregister("confirmPurchase")
```

Registered in `ResourceHost.cpp` (line 24653) as `LuaCallbackUnregister`.
