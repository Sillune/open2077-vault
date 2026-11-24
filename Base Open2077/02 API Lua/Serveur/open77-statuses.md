# Open77.statuses (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-statuses.md) · [Page web](https://open2077.net/docs/api/server/open77-statuses)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

3 functions.

## apply

```lua
Open77.statuses.apply(playerId, targetId, definitionId, gradeId)
```

`NETWORK`: Uses the network backend.

Applies a configured, bounded status to a target using a registered grade.

Requires `players.statuses.apply`. The grade's `kind`, duration and effect come from the definition; canonical eligibility (alive, ready, not protected) is checked the same way an upload's impact is. No upload runs and no damage is dealt: this is the status half of a hack on its own, for a scripted consequence such as a story beat or a penalty.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `targetId` | `integer` | required | — |
| `definitionId` | `string` | required | — |
| `gradeId` | `string` | required | — |

Returns: `{ ok = true, actionId }, or nil`, `reason on failure`

```lua
-- A story beat blinds the player for the grade's duration, no upload, no damage
Open77.statuses.apply(narratorId, playerId, "myserver.short_circuit", "training")
```

Registered in `LuaResourceRuntime.cs` (line 8060) as `LuaResourceRuntime.Bootstrap`.

## list

```lua
Open77.statuses.list(playerId)
```

`NETWORK`: Uses the network backend.

Lists the owned statuses currently affecting a player.

Requires `players.hacking.read`. Every status any provider applied -- kind, provider, action id, deadline -- the same answer as `Open77.hacking.statuses`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `array of status tables`, `nil, reason on failure`

```lua
-- Show a player their active statuses
for _, status in ipairs(Open77.statuses.list(playerId) or {}) do
    Open77.chat.send(playerId, ("%s until %d"):format(status.kind, status.deadline))
end
```

Registered in `LuaResourceRuntime.cs` (line 8059) as `LuaResourceRuntime.Bootstrap`.

## remove

```lua
Open77.statuses.remove(actionId)
```

`NETWORK`: Uses the network backend.

Removes this provider's status without reversing the damage it did.

Requires `players.statuses.purge`. Only a status this resource applied can be removed; the victim's client is told to end its native presentation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `actionId` | `string` | required | — |

Returns: `{ ok = true }, or nil`, `reason on failure`

```lua
-- The medic cures the status this resource applied
Open77.statuses.remove(actionId)
```

Registered in `LuaResourceRuntime.cs` (line 8063) as `LuaResourceRuntime.Bootstrap`.
