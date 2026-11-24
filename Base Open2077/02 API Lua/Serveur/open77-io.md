# Open77.io (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-io.md) · [Page web](https://open2077.net/docs/api/server/open77-io)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

12 functions.

## append

```lua
Open77.io.append(path, contents)
```

`NETWORK`: Uses the network backend.

Appends text to a file in this resource's data directory.

Requires `filesystem.write`. Creates the file when it does not exist. The 4 MiB ceiling applies to the resulting file, so an append-only log needs its own rotation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `path` | `string` | required | — |
| `contents` | `string` | required | — |

Returns: `true, or false`, `reason`

```lua
-- An audit log line per admin command
Open77.io.append("admin.log", ("%s %s /%s\n"):format(Open77.time.utc(), Open77.players.name(source), command))
```

Registered in `LuaResourceRuntime.cs` (line 8151) as `LuaResourceRuntime.Bootstrap`.

## copy

```lua
Open77.io.copy(source, destination, [overwrite])
```

`NETWORK`: Uses the network backend.

Copies a file inside this resource's data directory.

Requires `filesystem.read` for the source **and** `filesystem.write` for the destination -- both, because the operation reads and writes. Without `overwrite`, an existing destination answers `already_exists`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `source` | `string` | required | — |
| `destination` | `string` | required | — |
| `overwrite` | `boolean` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- Keep a backup before rewriting the config
Open77.io.copy("config.json", "config.backup.json", true)
```

Registered in `LuaResourceRuntime.cs` (line 8158) as `LuaResourceRuntime.Bootstrap`.

## exists

```lua
Open77.io.exists(path)
```

`NETWORK`: Uses the network backend.

Whether a path exists inside this resource's data directory.

Requires `filesystem.read`. Returns `nil, reason` rather than `false` when the path itself is refused, so a rejected path is never mistaken for an absent file.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `path` | `string` | required | — |

Returns: `boolean, or nil`, `reason`

```lua
-- First run: write a default config if there is none
if not Open77.io.exists("config.json") then
    Open77.io.writeJson("config.json", { startingMoney = 500 })
end
```

Registered in `LuaResourceRuntime.cs` (line 8152) as `LuaResourceRuntime.Bootstrap`.

## list

```lua
Open77.io.list([directory])
```

`NETWORK`: Uses the network backend.

Lists one directory inside this resource's data directory.

Requires `filesystem.read`. Non-recursive and sorted, at most 2,048 entries. Each entry is `{ name, type, size? }`. Omitting the argument lists the data directory itself.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `directory` | `string` | optional | — |

Returns: `array of { name, type, size? }, or nil`, `reason: not_a_directory, too_many_entries, ...`

```lua
-- Load every house file in data/houses/
for _, entry in ipairs(Open77.io.list("houses") or {}) do
    if entry.type == "file" then loadHouse(json.decode(Open77.io.read("houses/" .. entry.name))) end
end
```

Registered in `LuaResourceRuntime.cs` (line 8153) as `LuaResourceRuntime.Bootstrap`.

## makeDirectory

```lua
Open77.io.makeDirectory(path)
```

`NETWORK`: Uses the network backend.

Creates a directory and any missing parents.

Requires `filesystem.write`. Creating a directory that already exists succeeds: the caller asked for it to be there, and it is.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `path` | `string` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Make sure the folder exists before the first save
Open77.io.makeDirectory("houses")
```

Registered in `LuaResourceRuntime.cs` (line 8155) as `LuaResourceRuntime.Bootstrap`.

## move

```lua
Open77.io.move(source, destination, [overwrite])
```

`NETWORK`: Uses the network backend.

Moves or renames a file inside this resource's data directory.

Requires `filesystem.write`. Both ends are checked against the same path rules. Without `overwrite`, an existing destination answers `already_exists` rather than being replaced.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `source` | `string` | required | — |
| `destination` | `string` | required | — |
| `overwrite` | `boolean` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- Rotate the log when it gets big
local info = Open77.io.stat("admin.log")
if info and info.size > 1024 * 1024 then
    Open77.io.move("admin.log", "admin.old.log", true)
end
```

Registered in `LuaResourceRuntime.cs` (line 8157) as `LuaResourceRuntime.Bootstrap`.

## read

```lua
Open77.io.read(path)
```

`NETWORK`: Uses the network backend.

Reads a UTF-8 text file from this resource's data directory.

Requires `filesystem.read`. Paths are relative to the calling resource's `data/` directory; absolute paths, `..` escapes and reparse-point traversal are rejected. A file is limited to 4 MiB and a relative path to 512 UTF-8 bytes. This cannot read another resource, the server configuration, or the resource's own manifest and scripts.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `path` | `string` | required | — |

Returns: `contents, or nil`, `reason: invalid_path, path_outside_resource, reparse_point_denied, not_found, not_a_file, file_too_large, io_error`

```lua
-- Read a JSON file shipped in data/
local text, reason = Open77.io.read("prices.json")
local prices = text and json.decode(text) or {}
```

Registered in `LuaResourceRuntime.cs` (line 8149) as `LuaResourceRuntime.Bootstrap`.

## readJson

```lua
Open77.io.readJson(path)
```

`NETWORK`: Uses the network backend.

Reads a file and decodes it with the bounded Open77 JSON codec.

Requires `filesystem.read`. Inherits the codec's 48 KiB and depth limits on top of the file limits, so it is for configuration and small records, not a database. A decode failure is reported as a reason rather than raising.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `path` | `string` | required | — |

Returns: `value, or nil`, `reason: invalid_json among the read reasons`

```lua
-- Load the race course from data/courses/
local course, reason = Open77.io.readJson("courses/watson-loop.json")
if not course then print("no course: " .. reason) end
```

Registered in `LuaResourceRuntime.cs` (line 8159) as `LuaResourceRuntime.Bootstrap`.

## remove

```lua
Open77.io.remove(path)
```

`NETWORK`: Uses the network backend.

Removes one file.

Requires `filesystem.write`. Never removes a directory tree -- that is deliberate, so a path bug cannot take a resource's whole data directory with it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `path` | `string` | required | — |

Returns: `true, or false`, `reason: not_a_file among others`

```lua
-- The house was demolished: delete its file
Open77.io.remove("houses/" .. houseId .. ".json")
```

Registered in `LuaResourceRuntime.cs` (line 8156) as `LuaResourceRuntime.Bootstrap`.

## stat

```lua
Open77.io.stat(path)
```

`NETWORK`: Uses the network backend.

Reads one entry's metadata.

Requires `filesystem.read`. Returns `{ name, type, size?, modifiedUtc }`. `modifiedUtc` is the honest way to tell whether a file changed since the resource last read it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `path` | `string` | required | — |

Returns: `table { name, type, size?, modifiedUtc }, or nil`, `reason`

```lua
-- Reload the prices only if the file changed
local info = Open77.io.stat("prices.json")
if info and info.modifiedUtc ~= lastLoaded then
    prices = json.decode(Open77.io.read("prices.json"))
    lastLoaded = info.modifiedUtc
end
```

Registered in `LuaResourceRuntime.cs` (line 8154) as `LuaResourceRuntime.Bootstrap`.

## write

```lua
Open77.io.write(path, contents)
```

`NETWORK`: Uses the network backend.

Atomically replaces a UTF-8 text file in this resource's data directory.

Requires `filesystem.write`. The replacement is atomic, so a crash mid-write leaves the previous contents rather than a truncated file. Same path rules and 4 MiB limit as `read`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `path` | `string` | required | — |
| `contents` | `string` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Save a plain text MOTD
Open77.io.write("motd.txt", "Welcome to Night City RP. Be nice.")
```

Registered in `LuaResourceRuntime.cs` (line 8150) as `LuaResourceRuntime.Bootstrap`.

## writeJson

```lua
Open77.io.writeJson(path, value)
```

`NETWORK`: Uses the network backend.

Encodes a value and writes it atomically.

Requires `filesystem.write`. Encodes with the bounded Open77 JSON codec, then writes through `Open77.io.write`, so the replacement is atomic. A value the codec refuses answers a reason instead of writing a partial file.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `path` | `string` | required | — |
| `value` | `any JSON-encodable value` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Persist the house owners without a database
Open77.io.writeJson("houses.json", houses)
```

Registered in `LuaResourceRuntime.cs` (line 8160) as `LuaResourceRuntime.Bootstrap`.
