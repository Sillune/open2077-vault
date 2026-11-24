# Open77.clipboard (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-clipboard.md) · [Page web](https://open2077.net/docs/api/client/open77-clipboard)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

1 function.

## setText

```lua
Open77.clipboard.setText(text)
```

`GAME`: Requires a live game instance.

Writes UTF-8 text to the operating-system clipboard.

Client-only and requires `clipboard.write` on the resource whose VM performs the call. Text is limited to 256 KiB, must be valid UTF-8 and cannot contain an embedded NUL. The operation is write-only: resources cannot inspect the user's existing clipboard. A temporarily owned clipboard returns `false, clipboard_busy`; callers should let the player retry instead of polling every frame.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `text` | `string` | required | — |

Returns: `true on success, otherwise false`, `reason for the refusal`

```lua
-- /pos copies the player's coordinates for the mapper
RegisterCommand("pos", function()
    local x, y, z = Open77.character.position()
    Open77.clipboard.setText(("{ x = %.1f, y = %.1f, z = %.1f }"):format(x, y, z))
    Open77.hud.notify("Position copied")
end, false)
```

Registered in `ResourceHost.cpp` (line 24763) as `LuaClipboardSetText`.
