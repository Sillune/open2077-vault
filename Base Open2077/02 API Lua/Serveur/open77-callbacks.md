# Open77.callbacks (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-callbacks.md) · [Page web](https://open2077.net/docs/api/server/open77-callbacks)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

4 functions.

## callClient

```lua
Open77.callbacks.callClient(playerId, name, [...])
```

`NETWORK`: Uses the network backend.

Alias of Open77.net.callClient.

Asks one client a question and returns an Open77.Promise, or nil, reason when the request is refused before it leaves.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `name` | `string|table` | required | — |
| `...` | `any` | optional | — |

Returns: `Open77.Promise, or nil, reason`

```lua
-- Ask one client to confirm a purchase (alias of Open77.net.callClient)
CreateThread(function()
    local pending = Open77.callbacks.callClient(source, "confirmPurchase", "ammo_box", 50)
    local confirmed = pending and pending:await()
    if confirmed then sell(source, "ammo_box") end
end)
```

[Network callbacks guide](../../01%20Documentation/callbacks.md)

Registered in `LuaResourceRuntime.Callbacks.cs` (line 501) as `LuaResourceRuntime.Bootstrap`.

## callClientAwait

```lua
Open77.callbacks.callClientAwait(playerId, name, [...])
```

`NETWORK`: Uses the network backend.

Alias of Open77.net.callClientAwait.

Awaits the promise, so it needs a CreateThread or another managed task.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `name` | `string|table` | required | — |
| `...` | `any` | optional | — |

Returns: `the client handler's return values, or nil, reason`

```lua
-- The same question, awaited in one line
CreateThread(function()
    local confirmed = Open77.callbacks.callClientAwait(source, "confirmPurchase", "ammo_box", 50)
    if confirmed then sell(source, "ammo_box") end
end)
```

[Network callbacks guide](../../01%20Documentation/callbacks.md)

Registered in `LuaResourceRuntime.Callbacks.cs` (line 501) as `LuaResourceRuntime.Bootstrap`.

## register

```lua
Open77.callbacks.register(name, handler)
```

`NETWORK`: Uses the network backend.

Alias of Open77.net.register.

Open77.callbacks is the same four callback functions under a second name, for resources that prefer to read the callback surface as its own namespace. Pick one spelling per resource; these are the same functions, not copies.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `handler` | `function` | required | — |

Returns: `true, or nil, reason`

```lua
-- Answer a client's question with the authenticated source (alias of Open77.net.register)
Open77.callbacks.register("getStock", function(source, item)
    return stock[item] or 0, prices[item]
end)
```

[Network callbacks guide](../../01%20Documentation/callbacks.md)

Registered in `LuaResourceRuntime.Callbacks.cs` (line 500) as `LuaResourceRuntime.Bootstrap`.

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
-- The shop closed for the night: stop answering
Open77.callbacks.unregister("getStock")
```

[Network callbacks guide](../../01%20Documentation/callbacks.md)

Registered in `LuaResourceRuntime.Callbacks.cs` (line 500) as `LuaResourceRuntime.Bootstrap`.
