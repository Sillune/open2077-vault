# Open77.database (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-database.md) · [Page web](https://open2077.net/docs/api/server/open77-database)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

10 functions.

## insert

```lua
Open77.database.insert(sql, [params], [callback])
```

`NETWORK`: Uses the network backend.

Runs an INSERT and returns the inserted id.

Requires `database.access`. Same callback and `.await` forms as `query`. Limits, checked before the request leaves and refused whole rather than trimmed: 64 KiB of SQL, 64 parameters per statement (positional and named together -- a multi-row `INSERT ... VALUES (?,...),(?,...)` binds every `?`, so 50 rows of 6 columns is 300 and refused with `too_many_parameters`), a parameter table with no `nil` holes (`invalid_parameters`; write `NULL` in the SQL text instead), and as many positional values as `?` placeholders (`parameter_count_mismatch`). A refusal returns `false, reason`, schedules the callback with `(nil, reason)` and makes `.await` raise `reason`; it is not logged, so read the second callback argument. See [Limits of one request](https://open2077.net/docs/api/server/server-api.md#limits-of-one-request).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `sql` | `string` | required | — |
| `params` | `table` | optional | — |
| `callback` | `function(insertId, reason)` | optional | — |

Returns: `inserted id or result`

```lua
-- Create the player's row on first join and keep the id
CreateThread(function()
    local id = Open77.database.insert.await("INSERT INTO players (identifier, name) VALUES (?, ?)", { identifier, name })
    print("new player row", id)
end)
```

[SQL database setup](../../01%20Documentation/database.md)

Registered in `LuaResourceRuntime.cs` (line 6634) as `LuaResourceRuntime.Bootstrap`.

## isReady

```lua
Open77.database.isReady()
```

`NETWORK`: Uses the network backend.

Whether the database has answered yet.

Requires `database.access`. `true` once the host's probe has opened a connection and got an answer out of it; otherwise `false` and the reason -- `database_connecting` while the first connection is still being attempted, `database_unreachable` for a database that is configured but not answering (still retried), `database_unavailable` for a server that has no database bridge at all, and `permission_denied:database.access` for a resource that may not query one. Readiness is one-way: a later outage surfaces as a failing request, not as un-readiness, because every request already reports its own failure.

Returns: `true, or false`, `reason`

```lua
-- Refuse the shop until the database answers
RegisterCommand("shop", function(source)
    local ready, reason = Open77.database.isReady()
    if not ready then return Open77.chat.send(source, "Shop unavailable: " .. reason) end
    openShop(source)
end, false)
```

[SQL database setup](../../01%20Documentation/database.md)

Registered in `LuaResourceRuntime.cs` (line 6658) as `LuaResourceRuntime.Bootstrap`.

## prepare

```lua
Open77.database.prepare(sql, [params], [callback])
```

`NETWORK`: Uses the network backend.

Alias of `Open77.database.query`.

Requires `database.access`. Present for oxmysql compatibility: `prepare` and `query` are the same function, so an ported resource that calls one behaves exactly as one that calls the other. Limits, checked before the request leaves and refused whole rather than trimmed: 64 KiB of SQL, 64 parameters per statement (positional and named together -- a multi-row `INSERT ... VALUES (?,...),(?,...)` binds every `?`, so 50 rows of 6 columns is 300 and refused with `too_many_parameters`), a parameter table with no `nil` holes (`invalid_parameters`; write `NULL` in the SQL text instead), and as many positional values as `?` placeholders (`parameter_count_mismatch`). A refusal returns `false, reason`, schedules the callback with `(nil, reason)` and makes `.await` raise `reason`; it is not logged, so read the second callback argument. See [Limits of one request](https://open2077.net/docs/api/server/server-api.md#limits-of-one-request).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `sql` | `string` | required | — |
| `params` | `table` | optional | — |
| `callback` | `function(rows, reason)` | optional | — |

Returns: `rows`

```lua
-- oxmysql-style code works unchanged (await needs a managed task: a thread, a command or an event handler)
RegisterCommand("mycars", function(source)
    local rows = Open77.database.prepare.await("SELECT plate FROM vehicles WHERE owner = ?", { Open77.players.identifier(source) })
    Open77.chat.send(source, ("You own %d cars"):format(#rows))
end, false)
```

[SQL database setup](../../01%20Documentation/database.md)

Registered in `LuaResourceRuntime.cs` (line 6638) as `LuaResourceRuntime.Bootstrap`.

## query

```lua
Open77.database.query(sql, [params], [callback])
```

`NETWORK`: Uses the network backend.

Runs a SQL query and returns every row.

Requires `database.access`. `Open77.database` and the global `MySQL` are the same oxmysql-compatible table. Two forms: pass a callback, or use `Open77.database.query.await(sql, params?)` from a managed coroutine. Either way the continuation resumes on the owning resource's scheduler, never on the database worker, so the rest of the resource's state is safe to touch. `prepare` is an alias. Rows past `database.maxRows` (default 10 000) are dropped and the server logs a `WRN`; page with `LIMIT ?` bound to an integer. Limits, checked before the request leaves and refused whole rather than trimmed: 64 KiB of SQL, 64 parameters per statement (positional and named together -- a multi-row `INSERT ... VALUES (?,...),(?,...)` binds every `?`, so 50 rows of 6 columns is 300 and refused with `too_many_parameters`), a parameter table with no `nil` holes (`invalid_parameters`; write `NULL` in the SQL text instead), and as many positional values as `?` placeholders (`parameter_count_mismatch`). A refusal returns `false, reason`, schedules the callback with `(nil, reason)` and makes `.await` raise `reason`; it is not logged, so read the second callback argument. See [Limits of one request](https://open2077.net/docs/api/server/server-api.md#limits-of-one-request).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `sql` | `string` | required | — |
| `params` | `table` | optional | — |
| `callback` | `function(rows, reason)` | optional | — |

Returns: `rows (await form), or nothing (callback form)`

```lua
-- Every car the player owns
CreateThread(function()
    local rows = Open77.database.query.await("SELECT plate, props FROM vehicles WHERE owner = ?", { identifier })
    for _, row in ipairs(rows) do print(row.plate) end
end)
```

[SQL database setup](../../01%20Documentation/database.md)

Registered in `LuaResourceRuntime.cs` (line 6631) as `LuaResourceRuntime.Bootstrap`.

## rawExecute

```lua
Open77.database.rawExecute(sql, [params], [callback])
```

`NETWORK`: Uses the network backend.

Alias of `Open77.database.update`.

Requires `database.access`. Present for oxmysql compatibility; it is the same function as `update`, not a way around parameter binding. Limits, checked before the request leaves and refused whole rather than trimmed: 64 KiB of SQL, 64 parameters per statement (positional and named together -- a multi-row `INSERT ... VALUES (?,...),(?,...)` binds every `?`, so 50 rows of 6 columns is 300 and refused with `too_many_parameters`), a parameter table with no `nil` holes (`invalid_parameters`; write `NULL` in the SQL text instead), and as many positional values as `?` placeholders (`parameter_count_mismatch`). A refusal returns `false, reason`, schedules the callback with `(nil, reason)` and makes `.await` raise `reason`; it is not logged, so read the second callback argument. See [Limits of one request](https://open2077.net/docs/api/server/server-api.md#limits-of-one-request).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `sql` | `string` | required | — |
| `params` | `table` | optional | — |
| `callback` | `function(affectedRows, reason)` | optional | — |

Returns: `affected-row result`

```lua
-- oxmysql-style code works unchanged
RegisterNetEvent("garage:sell", function(plate)
    Open77.database.rawExecute.await("DELETE FROM vehicles WHERE plate = ? AND owner = ?", { plate, Open77.players.identifier(source) })
end)
```

[SQL database setup](../../01%20Documentation/database.md)

Registered in `LuaResourceRuntime.cs` (line 6639) as `LuaResourceRuntime.Bootstrap`.

## ready

```lua
Open77.database.ready(handler)
```

`NETWORK`: Uses the network backend.

Runs a handler once the database is reachable, or refuses when it never will be.

Requires `database.access`. oxmysql's boot gate: register schema creation or a migration here rather than racing it against the first connection. `ready` distinguishes three states a boolean would hide. A server with **no database configured** answers `false, "database_unavailable"` and the handler never runs -- the connection it would promise will never come. A database still connecting, or configured but **not answering**, queues the handler and keeps polling, so one that comes up late still fires. And a handler registered **after** readiness runs anyway, on the next tick, so a resource never has to ask whether it missed the moment. Handlers always run on this resource's own scheduler, never on the database worker. Refusals: `invalid_argument`, `database_unavailable`, `permission_denied:database.access`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `handler` | `function()` | required | — |

Returns: `true when the handler was run or queued, or false`, `reason`

```lua
-- Create the table once the database is reachable
Open77.database.ready(function()
    Open77.database.update.await([[
        CREATE TABLE IF NOT EXISTS players (
            identifier VARCHAR(64) PRIMARY KEY,
            money INT NOT NULL DEFAULT 500
        )
    ]])
end)
```

[SQL database setup](../../01%20Documentation/database.md)

Registered in `LuaResourceRuntime.cs` (line 6664) as `LuaResourceRuntime.Bootstrap`.

## scalar

```lua
Open77.database.scalar(sql, [params], [callback])
```

`NETWORK`: Uses the network backend.

Runs a SQL query and returns one value.

Requires `database.access`. The first column of the first row -- a count, a sum, an id. Same callback and `.await` forms as `query`. Limits, checked before the request leaves and refused whole rather than trimmed: 64 KiB of SQL, 64 parameters per statement (positional and named together -- a multi-row `INSERT ... VALUES (?,...),(?,...)` binds every `?`, so 50 rows of 6 columns is 300 and refused with `too_many_parameters`), a parameter table with no `nil` holes (`invalid_parameters`; write `NULL` in the SQL text instead), and as many positional values as `?` placeholders (`parameter_count_mismatch`). A refusal returns `false, reason`, schedules the callback with `(nil, reason)` and makes `.await` raise `reason`; it is not logged, so read the second callback argument. See [Limits of one request](https://open2077.net/docs/api/server/server-api.md#limits-of-one-request).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `sql` | `string` | required | — |
| `params` | `table` | optional | — |
| `callback` | `function(value, reason)` | optional | — |

Returns: `one value or nil`

```lua
-- One value: the player's money
RegisterCommand("balance", function(source)
    local money = Open77.database.scalar.await("SELECT money FROM players WHERE identifier = ?", { Open77.players.identifier(source) })
    Open77.chat.send(source, ("Balance: %d eddies"):format(money or 0))
end, false)
```

[SQL database setup](../../01%20Documentation/database.md)

Registered in `LuaResourceRuntime.cs` (line 6633) as `LuaResourceRuntime.Bootstrap`.

## single

```lua
Open77.database.single(sql, [params], [callback])
```

`NETWORK`: Uses the network backend.

Runs a SQL query and returns its first row.

Requires `database.access`. Same callback and `.await` forms as `query`; answers `nil` when nothing matched, which is the difference from `query` returning an empty table. Limits, checked before the request leaves and refused whole rather than trimmed: 64 KiB of SQL, 64 parameters per statement (positional and named together -- a multi-row `INSERT ... VALUES (?,...),(?,...)` binds every `?`, so 50 rows of 6 columns is 300 and refused with `too_many_parameters`), a parameter table with no `nil` holes (`invalid_parameters`; write `NULL` in the SQL text instead), and as many positional values as `?` placeholders (`parameter_count_mismatch`). A refusal returns `false, reason`, schedules the callback with `(nil, reason)` and makes `.await` raise `reason`; it is not logged, so read the second callback argument. See [Limits of one request](https://open2077.net/docs/api/server/server-api.md#limits-of-one-request).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `sql` | `string` | required | — |
| `params` | `table` | optional | — |
| `callback` | `function(row, reason)` | optional | — |

Returns: `one row or nil`

```lua
-- One row, or nil when the player is unknown
AddEventHandler("onPlayerReady", function(playerId)
    local identifier = Open77.players.identifier(playerId)
    local row = Open77.database.single.await("SELECT * FROM players WHERE identifier = ?", { identifier })
    if not row then createPlayer(identifier) end
end)
```

[SQL database setup](../../01%20Documentation/database.md)

Registered in `LuaResourceRuntime.cs` (line 6632) as `LuaResourceRuntime.Bootstrap`.

## transaction

```lua
Open77.database.transaction(statements, [callback])
```

`NETWORK`: Uses the network backend.

Runs several statements as one transaction.

Requires `database.access`. `statements` is an array of `{ query, values }` entries; all of them commit or none do. Same callback and `.await` forms as `query`. Limits: 1 to 64 statements (`invalid_transaction`), and each statement carries the per-statement caps of `query` (64 KiB of SQL, 64 parameters, no `nil` holes, matching `?` count). A refusal before the request leaves returns `false, reason` and schedules the callback with `(nil, reason)`; `.await` raises for a refusal and returns `false, reason` for a rollback. The callback receives `(true, nil)` on commit. See [Limits of one request](https://open2077.net/docs/api/server/server-api.md#limits-of-one-request).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `statements` | `table` | required | — |
| `callback` | `function(ok, reason)` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- Move money between two players, or not at all
RegisterNetEvent("bank:transfer", function(toId, amount)
    local from, to = Open77.players.identifier(source), Open77.players.identifier(toId)
    local ok = Open77.database.transaction.await({
        { query = "UPDATE players SET money = money - ? WHERE identifier = ?", values = { amount, from } },
        { query = "UPDATE players SET money = money + ? WHERE identifier = ?", values = { amount, to } },
    })
    if not ok then Open77.chat.send(source, "Transfer failed.") end
end)
```

[SQL database setup](../../01%20Documentation/database.md)

Registered in `LuaResourceRuntime.cs` (line 6636) as `LuaResourceRuntime.Bootstrap`.

## update

```lua
Open77.database.update(sql, [params], [callback])
```

`NETWORK`: Uses the network backend.

Runs a statement and returns the affected-row result.

Requires `database.access`. Use it for UPDATE and DELETE. `rawExecute` is an alias for the same method. Same callback and `.await` forms as `query`. Limits, checked before the request leaves and refused whole rather than trimmed: 64 KiB of SQL, 64 parameters per statement (positional and named together -- a multi-row `INSERT ... VALUES (?,...),(?,...)` binds every `?`, so 50 rows of 6 columns is 300 and refused with `too_many_parameters`), a parameter table with no `nil` holes (`invalid_parameters`; write `NULL` in the SQL text instead), and as many positional values as `?` placeholders (`parameter_count_mismatch`). A refusal returns `false, reason`, schedules the callback with `(nil, reason)` and makes `.await` raise `reason`; it is not logged, so read the second callback argument. See [Limits of one request](https://open2077.net/docs/api/server/server-api.md#limits-of-one-request).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `sql` | `string` | required | — |
| `params` | `table` | optional | — |
| `callback` | `function(affectedRows, reason)` | optional | — |

Returns: `affected-row result`

```lua
-- Save the balance with the callback form; the second argument is the refusal or database reason
Open77.database.update("UPDATE players SET money = ? WHERE identifier = ?", { money, identifier }, function(affected, reason)
    if reason then print("save failed: " .. reason); return end
    print("rows affected", affected)
end)
```

[SQL database setup](../../01%20Documentation/database.md)

Registered in `LuaResourceRuntime.cs` (line 6635) as `LuaResourceRuntime.Bootstrap`.
