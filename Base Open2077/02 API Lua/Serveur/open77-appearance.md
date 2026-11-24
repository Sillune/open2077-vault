# Open77.appearance (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-appearance.md) · [Page web](https://open2077.net/docs/api/server/open77-appearance)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

2 functions.

## apply

```lua
Open77.appearance.apply(playerId, snapshot, [options])
```

`NETWORK`: Uses the network backend.

Restores a saved appearance snapshot on a player, without the client asking.

Asynchronous, and relayed exactly like `Open77.clothing`: the addressed client owns the REDengine player object, so the server asks and the client answers. Requires `players.appearance.relay` in the manifest AND `network.events`; the request is relayed to the official `open77_appearance` client package. Returns a request id prefixed with the resource name and an `appearance` segment, so an appearance answer can never complete a clothing or weapon request and only this resource's VM can match the completion. Listen for the resource-local `open77:appearance:relayCompleted` event with `playerId, requestId, operation, accepted, reason, result`; an unanswered request completes with `request_timeout` after ten seconds rather than leaving the caller waiting. The client refuses while its appearance is busy (`appearance_busy` -- an editor, the character creator or a fitting-room preview is open), before the player is in the world (`player_not_ready`) or after a failed restore (`appearance_restore_failed`). `players.appearance.relay` is a permission of its own, not a corner of `player.appearance.edit` (which is a client-side grant over the local player) or of clothing (garments a player can see and take off): a snapshot is the output of the character creator and the closest thing the platform holds to a likeness of the person playing. Nothing here writes a snapshot to a log. The snapshot must be the table `capture` returned. It does not reach the native layer behind `open77_appearance`'s back: it becomes that player's new appearance **record** -- the server's word about what they wear -- and the resource's ordinary reconciliation puts it on the body. That is the same road a stored appearance takes at world entry, so a relayed face survives death, a bucket change and a reload instead of being quietly undone by the next reconcile. It does **not** change the body family: a different family is a player *reload*, not an appearance change, and a snapshot whose gender disagrees with the body on the player is refused with `body_family_mismatch`. The platform's stored appearance row is not written -- a resource that wants the change to outlive the session owns that persistence itself. Completion is the client's acceptance, reported once the player is observed wearing the snapshot; `apply_timeout` means the record was set but the body had not settled in eight seconds, and `appearance_superseded` means a newer apply replaced it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `snapshot` | `table` | required | — |
| `options` | `table` | optional | — |

Returns: `request id, or nil`, `reason: permission_denied:players.appearance.relay, invalid_player, invalid_snapshot, dispatch_failed`

```lua
-- Restore a saved look on a player who just spawned
RegisterNetEvent("character:loaded", function()
    local snapshot = savedLooks[Open77.players.identifier(source)]
    if snapshot then Open77.appearance.apply(source, snapshot) end
end)
```

Registered in `LuaResourceRuntime.cs` (line 8274) as `LuaResourceRuntime.Bootstrap`.

## capture

```lua
Open77.appearance.capture(playerId)
```

`NETWORK`: Uses the network backend.

Asks a player's client for its current appearance snapshot.

Asynchronous, and relayed exactly like `Open77.clothing`: the addressed client owns the REDengine player object, so the server asks and the client answers. Requires `players.appearance.relay` in the manifest AND `network.events`; the request is relayed to the official `open77_appearance` client package. Returns a request id prefixed with the resource name and an `appearance` segment, so an appearance answer can never complete a clothing or weapon request and only this resource's VM can match the completion. Listen for the resource-local `open77:appearance:relayCompleted` event with `playerId, requestId, operation, accepted, reason, result`; an unanswered request completes with `request_timeout` after ten seconds rather than leaving the caller waiting. The client refuses while its appearance is busy (`appearance_busy` -- an editor, the character creator or a fitting-room preview is open), before the player is in the world (`player_not_ready`) or after a failed restore (`appearance_restore_failed`). `players.appearance.relay` is a permission of its own, not a corner of `player.appearance.edit` (which is a client-side grant over the local player) or of clothing (garments a player can see and take off): a snapshot is the output of the character creator and the closest thing the platform holds to a likeness of the person playing. Nothing here writes a snapshot to a log. The result is `{ snapshot, family, revision, characterKey }`, where `snapshot` is the network form -- schema version, game build, catalogue digest, gender and the option identities and values -- with the editor's local metadata (editable/active/censored) stripped. It is the same shape `apply` takes, so a capture can be stored and handed straight back later.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `request id, or nil`, `reason: permission_denied:players.appearance.relay, invalid_player, dispatch_failed`

```lua
-- Ask the client for its look and store the answer
Open77.appearance.capture(playerId)
AddEventHandler("open77:appearance:relayCompleted", function(playerId, requestId, operation, accepted, reason, result)
    if operation == "capture" and accepted then savedLooks[Open77.players.identifier(playerId)] = result end
end)
```

Registered in `LuaResourceRuntime.cs` (line 8273) as `LuaResourceRuntime.Bootstrap`.
