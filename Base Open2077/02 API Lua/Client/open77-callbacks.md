# Open77.callbacks (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-callbacks.md) · [Page web](https://open2077.net/docs/api/client/open77-callbacks)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

4 functions.

## call

```lua
Open77.callbacks.call(name, [...])
```

`NETWORK`: Uses the network backend.

Alias of Open77.net.call.

Open77.callbacks is the same four functions under a second name, for resources that prefer to read the callback surface as its own namespace. Pick one spelling per resource; the two are the same functions, not copies.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string|table` | required | — |
| `...` | `any` | optional | — |

Returns: `Open77.Promise, or nil, reason`

```lua
-- Ask the server for the shop stock (alias of Open77.net.call)
CreateThread(function()
    local pending = Open77.callbacks.call("getStock", "ammo_box")
    local count, price = pending and pending:await()
    print(count, price)
end)
```

[Network callbacks guide](../../01%20Documentation/callbacks.md)

Registered in `ResourceHost.cpp` (line 23188) as `LuaPrelude`.

## callAwait

```lua
Open77.callbacks.callAwait(name, [...])
```

`NETWORK`: Uses the network backend.

Alias of Open77.net.callAwait.

Awaits the promise, so it needs a CreateThread or another managed task.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string|table` | required | — |
| `...` | `any` | optional | — |

Returns: `the handler's return values, or nil, reason`

```lua
-- The same question, awaited in one line
CreateThread(function()
    local count, price = Open77.callbacks.callAwait("getStock", "ammo_box")
    print(count, price)
end)
```

[Network callbacks guide](../../01%20Documentation/callbacks.md)

Registered in `ResourceHost.cpp` (line 23188) as `LuaPrelude`.

## register

```lua
Open77.callbacks.register(name, handler)
```

`NETWORK`: Uses the network backend.

Alias of Open77.net.register.

Answers a question the server asks with Open77.net.callClient. The handler receives the call's arguments and no source, because the caller is always the server.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `handler` | `function` | required | — |

Returns: `true, or nil, reason`

```lua
-- Answer the server's "confirm this purchase?" question with a dialog result
Open77.callbacks.register("confirmPurchase", function(item, price)
    return confirmDialog(("Buy %s for %d?"):format(item, price))
end)
```

[Network callbacks guide](../../01%20Documentation/callbacks.md)

Registered in `ResourceHost.cpp` (line 23187) as `LuaPrelude`.

## unregister

```lua
Open77.callbacks.unregister(name)
```

`NETWORK`: Uses the network backend.

Alias of Open77.net.unregister.

Stops answering that name; a later request is refused with callback_not_found on the next frame.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |

Returns: `true, or false, reason`

```lua
-- The shop UI closed: stop answering
Open77.callbacks.unregister("confirmPurchase")
```

[Network callbacks guide](../../01%20Documentation/callbacks.md)

Registered in `ResourceHost.cpp` (line 23187) as `LuaPrelude`.
