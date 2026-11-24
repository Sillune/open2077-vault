# Open77.npcs (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-npcs.md) · [Page web](https://open2077.net/docs/api/server/open77-npcs)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

45 functions.

## all

```lua
Open77.npcs.all([bucket])
```

`NETWORK`: Uses the network backend.

Every authoritative NPC, optionally filtered to one routing bucket.

Requires `world.npcs`. Returns the whole registry, whichever resource created each NPC.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | optional | — |

Returns: `array of NPC snapshots`

```lua
-- /npcs: how many server NPCs are alive right now
RegisterCommand("npcs", function(source)
    local alive = 0
    for _, npc in ipairs(Open77.npcs.all()) do
        if npc.health > 0 then alive = alive + 1 end
    end
    Open77.chat.send(source, ("%d NPCs alive"):format(alive))
end, true)
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9336) as `LuaResourceRuntime.Bootstrap`.

## applyCrossfireDamage

```lua
Open77.npcs.applyCrossfireDamage(victimId, attackerId, amount, [source], [cause])
```

`NETWORK`: Uses the network backend.

Applies managed NPC-to-NPC crossfire with difficulty.

Requires `world.npcs`; both victim and attacker must belong to the caller. Applies attacker damageMultiplier and victim healthMultiplier to the supplied damage through the NPC damage policy. This is distinct from exact `applyDamage`; it does not change unrelated PvP damage.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `victimId` | `NPC id` | required | — |
| `attackerId` | `NPC id` | required | — |
| `amount` | `number` | required | — |
| `source` | `string` | optional | — |
| `cause` | `string` | optional | — |

Returns: `boolean`, `reason`

```lua
local ok, reason = Open77.npcs.applyCrossfireDamage(victimId, attackerId, 10, "script", "generic")
if not ok then print(reason) end
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9638) as `LuaResourceRuntime.Bootstrap`.

## applyDamage

```lua
Open77.npcs.applyDamage(id, amount, [source], [cause])
```

`NETWORK`: Uses the network backend.

Applies authoritative damage to an NPC.

Requires `world.npcs`. `source` and `cause` are recorded with the hit so a kill can be attributed. The damage still passes the NPC's damage policy, so an `invulnerable` NPC takes nothing.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `amount` | `number` | required | — |
| `source` | `integer|string` | optional | — |
| `cause` | `string` | optional | — |

Returns: `boolean`

```lua
-- A trap deals 30 damage to the guard who stepped on it
Open77.npcs.applyDamage(guardId, 30, nil, "trap")
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9342) as `LuaResourceRuntime.Bootstrap`.

## capacity

```lua
Open77.npcs.capacity()
```

`NETWORK`: Uses the network backend.

Reads allocated NPC counts and admission limits.

Requires `world.npcs`. Takes no arguments. `resourceUsed` counts every NPC still owned by the calling resource, including retained corpses and persistent entries; `globalUsed` counts all allocated NPCs across resources. `resourceLimit` is 512 and `globalLimit` is 4096. These are allocation counts, not counts of living or currently visible NPCs. Returns nil, reason when permission or the NPC subsystem is unavailable.

Returns: `{ resourceUsed, resourceLimit, globalUsed, globalLimit }, or nil`, `reason: permission_denied:world.npcs, npcs_unavailable`

```lua
-- Check both allocation limits before admitting another NPC
local capacity, reason = Open77.npcs.capacity()
if capacity and capacity.resourceUsed < capacity.resourceLimit and capacity.globalUsed < capacity.globalLimit then
    admitNpc()
end
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9332) as `LuaResourceRuntime.Bootstrap`.

## create

```lua
Open77.npcs.create(def)
```

`NETWORK`: Uses the network backend.

Creates an authoritative NPC.

Requires `world.npcs`. Build one from `record = "Character.*"` directly -- no catalogue entry is needed -- or from the legacy `template` alias. Definition fields are `record`, `template`, `position`, `yaw`, `bucket`, `appearance`, `loadout`, `behavior`, `aiMode`, `damagePolicy`, `health`, `maxHealth`, `streamingRadius`, `streamingHysteresis`, `despawnWhenUnobserved`, `persistent` and `authorityPolicy`. `aiMode` and `damagePolicy` are **integers** from the constant tables `Open77.npcs.ai` (`tasks = 0`, `frozen = 1`, `native = 2`) and `Open77.npcs.damage` (`mortal = 0`, `immortal = 1`, `invulnerable = 2`; default `immortal`); a string such as `"invulnerable"` raises a Lua argument error (`number expected, got string`) in `CreateNpc`. `authorityPolicy` is `"nearest"` (default) or `"balanced"` (the client simulating the fewest NPCs, with a tenure-gated rebalance); anything else returns `nil, "npc_authority_policy_invalid"`. `behavior.observerReplay = true` makes clients that do not simulate the NPC replay the simulator's motion on the shared server clock. The id comes back at once, but engine readiness is asynchronous: a freshly created NPC is not yet a body in the world. See [NPCs](https://open2077.net/docs/api/server/npcs.md). Stable 147 also accepts body and equipment from a saved player look; body.family selects the matching Open77 player template. Do not combine it with a legacy template or an unrelated record. See [Character previews](https://open2077.net/docs/api/server/character-previews.md). Stable 156 snapshots include armor. Server create accepts armor (default 0), update preserves it when omitted; finite 0..100000 points absorb canonical damage before health. Matching protocol 1.50 is required. See [NPC armor](../../01%20Documentation/npcs.md#armor).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `def` | `table` | required | — |

Returns: `NPC id, or nil`, `reason`

```lua
-- A shopkeeper standing behind the counter, who never fights back
local shopkeeper, reason = Open77.npcs.create({
    record = "Character.Judy",
    position = { x = -1442.2, y = 127.4, z = 18.0 },
    yaw = 180.0,
    damagePolicy = Open77.npcs.damage.invulnerable,   -- an integer (2), never the string
    persistent = true,
})
if not shopkeeper then print("could not spawn the shopkeeper: " .. reason) end
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9266) as `LuaResourceRuntime.Bootstrap`.

## get

```lua
Open77.npcs.get(id)
```

`NETWORK`: Uses the network backend.

Reads one NPC's canonical snapshot.

Requires `world.npcs`. Answers `nil` for an unknown id. The snapshot is the server's record, so it is available before the body has streamed to anyone. Stable 156 snapshots include armor. Server create accepts armor (default 0), update preserves it when omitted; finite 0..100000 points absorb canonical damage before health. Matching protocol 1.50 is required. See [NPC armor](../../01%20Documentation/npcs.md#armor).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |

Returns: `NPC snapshot, or nil`

```lua
-- Is the boss still alive?
local boss = Open77.npcs.get(bossId)
if boss and boss.health <= 0 then
    Open77.chat.broadcast("The boss is down!")
end
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9331) as `LuaResourceRuntime.Bootstrap`.

## getArmor

```lua
Open77.npcs.getArmor(npcId)
```

`NETWORK`: Uses the network backend.

Reads an owned NPC's armor points.

Requires `world.npcs` and resource ownership. Returns the canonical armor value, or nil for an unavailable/unowned NPC. On the client use `Open77.npcs.get(id).armor` with `npcs.read`. See [NPC armor](../../01%20Documentation/npcs.md#armor).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `npcId` | `integer` | required | — |

Returns: `number, or nil`

```lua
local armor = Open77.npcs.getArmor(npcId)
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9524) as `LuaResourceRuntime.Bootstrap`.

## getAttitude

```lua
Open77.npcs.getAttitude(id)
```

`NETWORK`: Uses the network backend.

Reads back every attitude row an NPC states.

Requires `world.npcs`, and only for an NPC this resource created. Returns a table with `default`, `players`, `npcs`, `groups` and `relations`; `relations` is the row projected by `setRelationship` and is written by the platform, not by `setAttitude`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |

Returns: `table, or nil`

```lua
-- Debug: print how the guard regards everyone
local attitude = Open77.npcs.getAttitude(guardId)
if attitude then print("default attitude:", attitude.default) end
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9319) as `LuaResourceRuntime.Bootstrap`.

## getBehavior

```lua
Open77.npcs.getBehavior(id)
```

`NETWORK`: Uses the network backend.

Reads the canonical behavior policy of an owned NPC.

Requires `world.npcs`. Returns a detached table with `aiEnabled`, `combatEnabled`, `perceptionEnabled`, `voiceEnabled`, `vehicleContactEnabled` and `hitPricing`. Client inspection uses `Open77.npcs.get(id).behavior` with `npcs.read`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `table behavior, or nil`

```lua
-- Only re-enable combat if it was actually off
local behavior = Open77.npcs.getBehavior(guardId)
if behavior and not behavior.combatEnabled then
    Open77.npcs.setCombatEnabled(guardId, true)
end
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9285) as `LuaResourceRuntime.Bootstrap`.

## getBody

```lua
Open77.npcs.getBody(id)
```

`SERVER`: Available in server resources only.

Read the stored player body of an NPC.

Requires world.npcs. Returns the stored body snapshot or nil when the NPC is unknown, owned by another resource, or has no player body. This describes canonical data, not native rendering readiness. See [Character previews](https://open2077.net/docs/api/server/character-previews.md#networked-npc).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `table body, or nil`

```lua
local body = Open77.npcs.getBody(npcId)
```

[Character previews & player-shaped NPCs](../../01%20Documentation/character-previews.md)

Registered in `LuaResourceRuntime.cs` (line 9469) as `LuaResourceRuntime.Bootstrap`.

## getDifficulty

```lua
Open77.npcs.getDifficulty(id)
```

`NETWORK`: Uses the network backend.

Reads an owned NPC's difficulty multipliers.

Requires `world.npcs`. Returns `{healthMultiplier, damageMultiplier}` or `nil, reason`. The same canonical settings appear in `Open77.npcs.get(id).difficulty`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |

Returns: `table or nil`, `reason`

```lua
local difficulty, reason = Open77.npcs.getDifficulty(npcId)
if difficulty then print(difficulty.healthMultiplier, difficulty.damageMultiplier) else print(reason) end
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9574) as `LuaResourceRuntime.Bootstrap`.

## getEquipment

```lua
Open77.npcs.getEquipment(id)
```

`NETWORK`: Uses the network backend.

The clothing set an owned NPC is dressed in, slot to record.

Requires `world.npcs`. Reads `loadout.equipment` back as a table; empty when the NPC wears nothing of ours, `nil` for an id this resource does not own. The same object is `json.decode(Open77.npcs.get(id).loadout).equipment`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |

Returns: `slot-to-record table, or nil`

```lua
-- Swap the jacket, keep the rest.
local worn = Open77.npcs.getEquipment(clerkId) or {}
worn.OuterChest = "Items.Jacket_02_basic_01"
Open77.npcs.setEquipment(clerkId, worn)
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.Npcs6.cs` (line 251) as `LuaResourceRuntime.Bootstrap`.

## getGroup

```lua
Open77.npcs.getGroup(id)
```

`NETWORK`: Uses the network backend.

The relationship group an NPC belongs to.

Requires `world.npcs`. Returns the `combat.group` string, or nil when the NPC has none.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |

Returns: `string, or nil`

```lua
-- Gang members share loot with their own group only
if Open77.npcs.getGroup(npcId) == "maelstrom" then
    dropGangLoot(npcId)
end
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9321) as `LuaResourceRuntime.Bootstrap`.

## getRelationship

```lua
Open77.npcs.getRelationship(groupA, groupB)
```

`NETWORK`: Uses the network backend.

Reads back a declared group relationship.

Requires `world.npcs`. Returns the attitude token, or nil when this resource has declared nothing for the pair.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `groupA` | `string` | required | — |
| `groupB` | `string` | required | — |

Returns: `string, or nil`

```lua
-- Check before declaring: are the two gangs already at war?
if Open77.npcs.getRelationship("maelstrom", "valentinos") ~= "hostile" then
    Open77.npcs.setRelationship("maelstrom", "valentinos", "hostile")
end
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9327) as `LuaResourceRuntime.Bootstrap`.

## kill

```lua
Open77.npcs.kill(id)
```

`NETWORK`: Uses the network backend.

Kills an NPC authoritatively.

Requires `world.npcs`. Goes through the NPC life authority rather than setting health to zero, so the death is replicated once and consistently.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |

Returns: `boolean`

```lua
-- /killnpc <id>
RegisterCommand("killnpc", function(source, args)
    local npcId = tonumber(args[1])
    if npcId then Open77.npcs.kill(npcId) end
end, true)
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9340) as `LuaResourceRuntime.Bootstrap`.

## owner

```lua
Open77.npcs.owner(id)
```

`NETWORK`: Uses the network backend.

Who is simulating an NPC, and since when.

Requires `world.npcs`. The same shape as `Open77.vehicles.owner`, so a script reads both alike: `authorityPlayerId` and `physicsOwner` carry the same value and are `0` when nobody simulates the NPC; `epoch` is the authority epoch; `since` is when the authority last changed on the `GetGameTimer()` clock and `ageMs` how long ago; `reason` is why (`projectionready`, `bettercandidate`, `leaseexpired`, `playerdisconnected`, `streamedout`, `npcremoved`, `validationfailed`, `serverrevoke`, or `none`); `leaseMs` is how long the current lease still has, when there is one; `readyClients` is how many clients have reported a ready projection -- the election's candidates. There is no request counterpart on purpose: an NPC's authority is elected from the ready clients, and a script has no better information than the election about which of them should simulate a body; a grant to a client that never reported ready would be a lease nobody renews.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |

Returns: `{ authorityPlayerId, physicsOwner, epoch, readyClients, since, ageMs, reason, leaseMs? }, or nil`, `reason: invalid_npc_id, npc_not_found`

```lua
-- Why is the guard standing still? Nobody is simulating him yet.
local owner = Open77.npcs.owner(guardId)
if owner and owner.authorityPlayerId == 0 then
    print(("guard %d has no authority: %d ready client(s), last reason %s"):format(guardId, owner.readyClients, owner.reason))
end
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.Vehicles6.cs` (line 276) as `LuaResourceRuntime.Bootstrap`.

## presence

```lua
Open77.npcs.presence(id)
```

`NETWORK`: Uses the network backend.

Where an NPC is and who simulates it, in one O(1) read.

Requires `world.npcs`, and `npcs.foreign` for an NPC another resource created (the same rule as `Open77.npcs.all({ includeForeign = true })`, because where somebody else's NPC stands is roster information). The read a rule needs when a client acts FOR an NPC: `authorityPlayerId` is the client simulating it (`0` when nobody), `epoch` the authority epoch that client holds, and `leaseMs` how long its lease still has -- present only while there is a live lease. `alive` is the canonical life (alive flag and health above zero). `x`, `y`, `z` and `bucket` are the canonical place: the transform the simulator last reported and the server accepted under its anti-teleport budget. `owned` says whether the caller created it; `resource` names who did. Unlike `get` it builds one table, not a snapshot of the whole NPC, and unlike `all` it never walks the roster. The door service uses it to admit an NPC's passage only from the client that simulates that NPC.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |

Returns: `{ id, resource, owned, bucket, x, y, z, alive, authorityPlayerId, epoch, leaseMs? }, or nil`, `reason: invalid_npc_id, npc_not_found, npcs_unavailable, permission_denied:world.npcs, permission_denied:npcs.foreign`

```lua
-- Let a client speak for an NPC only while it simulates it, near the target.
local function speaksFor(player, npcId, target)
    local p = Open77.npcs.presence(npcId)
    if not p or not p.alive or p.authorityPlayerId ~= player or not p.leaseMs then return false end
    local dx, dy, dz = p.x - target.x, p.y - target.y, p.z - target.z
    return dx * dx + dy * dy + dz * dz <= 36
end
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.Vehicles6.cs` (line 284) as `LuaResourceRuntime.Bootstrap`.

## remove

```lua
Open77.npcs.remove(id)
```

`NETWORK`: Uses the network backend.

Removes an authoritative NPC.

Requires `world.npcs`. Despawns the body for every viewer and drops the registry entry, including its queued tasks.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |

Returns: `boolean`

```lua
-- Clean up every NPC this resource spawned when the event ends
AddEventHandler("event:ended", function()
    for _, npcId in ipairs(spawned) do Open77.npcs.remove(npcId) end
    spawned = {}
end)
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9330) as `LuaResourceRuntime.Bootstrap`.

## revive

```lua
Open77.npcs.revive(id, [health])
```

`NETWORK`: Uses the network backend.

Revives an NPC, at full health by default.

Requires `world.npcs`. `health` defaults to 100. Reviving an NPC that is alive is accepted and simply sets its health.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `health` | `number` | optional | — |

Returns: `boolean`

```lua
-- The medic revives the fallen bodyguard at half health
Open77.npcs.revive(bodyguardId, 50)
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9341) as `LuaResourceRuntime.Bootstrap`.

## setAIEnabled

```lua
Open77.npcs.setAIEnabled(id, enabled)
```

`NETWORK`: Uses the network backend.

Pauses or resumes a server-owned NPC's native AI and Open77 tasks.

Requires world.npcs and ownership. Does not change aiMode or the other behavior options. Disabling revokes the simulation lease; enabling resumes according to the selected mode.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `boolean success, optional string reason`

```lua
-- Pause every NPC while the round is paused, resume after
for _, npc in ipairs(Open77.npcs.all(ARENA_BUCKET)) do
    Open77.npcs.setAIEnabled(npc.id, false)
end
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9286) as `LuaResourceRuntime.Bootstrap`.

## setAiMode

```lua
Open77.npcs.setAiMode(id, mode)
```

`NETWORK`: Uses the network backend.

Chooses who drives an NPC.

Requires `world.npcs`. Modes are in `Open77.npcs.ai`: `tasks` (0, the Open77 task queue), `frozen` (1) and `native` (2, vanilla AI). This is a different axis from `setAIEnabled`, which pauses whichever driver is selected without changing the mode.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `mode` | `integer|string` | required | — |

Returns: `boolean`, `reason`

```lua
-- Let the engine's own AI drive this civilian instead of the task queue
Open77.npcs.setAiMode(npcId, "native")
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9280) as `LuaResourceRuntime.Bootstrap`.

## setAppearance

```lua
Open77.npcs.setAppearance(id, appearance)
```

`NETWORK`: Uses the network backend.

Convenience patch for an NPC's appearance.

Requires `world.npcs`. Exactly `update(id, { appearance = appearance })`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `appearance` | `string` | required | — |

Returns: `boolean`, `reason`

```lua
-- Swap the bartender's outfit for the night shift
Open77.npcs.setAppearance(bartenderId, "judy_night")
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9274) as `LuaResourceRuntime.Bootstrap`.

## setArmor

```lua
Open77.npcs.setArmor(npcId, points)
```

`NETWORK`: Uses the network backend.

Sets an owned NPC's armor points.

Requires `world.npcs` and ownership of the NPC. Sets finite armor points from 0 to 100000; equivalent to `update(id, { armor = points })`. Updates omitting armor preserve it; creation defaults to zero. Canonical damage consumes armor before health, while invulnerability preserves both. Damage event amounts remain health damage, including zero for an entirely absorbed hit. Armor is replicated to current observers and late joiners with protocol 1.50. See [NPC armor](../../01%20Documentation/npcs.md#armor).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `npcId` | `integer` | required | — |
| `points` | `number` | required | — |

Returns: `boolean`, `reason`

```lua
assert(Open77.npcs.setArmor(npcId, 50))
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9520) as `LuaResourceRuntime.Bootstrap`.

## setAttitude

```lua
Open77.npcs.setAttitude(id, attitude, [options])
```

`NETWORK`: Uses the network backend.

Makes an NPC hostile, neutral or friendly toward a target.

Requires `world.npcs`. Attitude is **per NPC and per target**, because that is the only shape the engine expresses per incarnation: `gameAttitudeAgent.SetAttitudeTowards(agent, attitude)` is a pair override. `options.towards` selects the row -- a player id, `{ npcId = n }` (which must belong to this resource) or `{ group = "name" }` -- and omitting it sets the default that covers everyone without a row. Resolution order is the per-entity row, then the per-group row, then the resource relationship row, then the default. Passing `nil` for `attitude` removes the row. An NPC that states nothing behaves exactly as it did before this call existed. Eligibility is a hard cap on players: a dead player, or one in another routing bucket, is never made a target by a directive. See the [NPC guide](https://open2077.net/docs/api/server/npcs.md#attitude-and-relationship-groups).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `attitude` | `string `friendly`|`neutral`|`hostile`, or nil to clear` | required | — |
| `options` | `table { towards? }` | optional | — |

Returns: `boolean`, `reason`

```lua
-- The guard becomes hostile toward the one player who broke in
Open77.npcs.setAttitude(guardId, "hostile", { towards = intruderId })
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9305) as `LuaResourceRuntime.Bootstrap`.

## setBehavior

```lua
Open77.npcs.setBehavior(id, behavior)
```

`NETWORK`: Uses the network backend.

Changes a server-owned NPC's native AI, combat, perception and voice policy.

Requires `world.npcs` and ownership. Partial options: the booleans `aiEnabled`, `combatEnabled`, `perceptionEnabled` and `voiceEnabled` (default `true`) and `vehicleContactEnabled` (default `false`), and `hitPricing` (`"resource"`, the default, or `"platform"`). The same object is accepted as `behavior` by `create` and `update`. Replicated to observers and late joiners. Malformed values return `npc_behavior_invalid`; unknown option names return `npc_behavior_unknown_option`. See [NPC AI, combat and voice control](../../01%20Documentation/npc-behavior.md) for engine semantics and boundaries.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `behavior` | `table` | required | — |

Returns: `boolean success, optional string reason`

```lua
-- A passive quest giver: no combat, no barks, still walks
Open77.npcs.setBehavior(questGiverId, { combatEnabled = false, voiceEnabled = false })
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9281) as `LuaResourceRuntime.Bootstrap`.

## setBody

```lua
Open77.npcs.setBody(id, body)
```

`SERVER`: Available in server resources only.

Replace a player-shaped NPC body.

Requires world.npcs and a body snapshot from appearance.captureBody. Replaces the complete body while preserving other loadout fields. The body family must match the NPC template; recreate the NPC to change family. Unrelated Character records cannot project player body groups. NPC ownership and the complete loadout 8 KiB limit apply. See [Character previews](https://open2077.net/docs/api/server/character-previews.md#networked-npc).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `body` | `table` | required | — |

Returns: `boolean accepted`, `reason on failure`

```lua
local ok, reason = Open77.npcs.setBody(npcId, savedLook.body)
```

[Character previews & player-shaped NPCs](../../01%20Documentation/character-previews.md)

Registered in `LuaResourceRuntime.cs` (line 9465) as `LuaResourceRuntime.Bootstrap`.

## setBucket

```lua
Open77.npcs.setBucket(id, bucket)
```

`NETWORK`: Uses the network backend.

Moves an NPC to another routing bucket.

Requires `world.npcs`. Changes who can see it: only players in the same bucket stream the NPC.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `bucket` | `integer` | required | — |

Returns: `boolean`

```lua
-- Move the arena boss into the arena instance
Open77.npcs.setBucket(bossId, ARENA_BUCKET)
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9273) as `LuaResourceRuntime.Bootstrap`.

## setCombatEnabled

```lua
Open77.npcs.setCombatEnabled(id, enabled)
```

`NETWORK`: Uses the network backend.

Allows or suppresses a server-owned NPC's combat behavior.

Requires world.npcs and ownership. False clears threats, disables hostile acquisition/reactions and blocks outgoing hit processing without disabling scripted locomotion.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `boolean success, optional string reason`

```lua
-- Truce: the gang stops fighting during the negotiation
for _, npcId in ipairs(gang) do
    Open77.npcs.setCombatEnabled(npcId, false)
end
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9290) as `LuaResourceRuntime.Bootstrap`.

## setDamagePolicy

```lua
Open77.npcs.setDamagePolicy(id, policy)
```

`NETWORK`: Uses the network backend.

Sets whether an NPC can be hurt or killed.

Requires `world.npcs`. Policies are in `Open77.npcs.damage`: `mortal` (0), `immortal` (1, can be hurt but not killed) and `invulnerable` (2, takes nothing).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `policy` | `integer|string` | required | — |

Returns: `boolean`, `reason`

```lua
-- The quest giver can be hurt but never killed
Open77.npcs.setDamagePolicy(questGiverId, "immortal")
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9279) as `LuaResourceRuntime.Bootstrap`.

## setDifficulty

```lua
Open77.npcs.setDifficulty(id, difficulty)
```

`NETWORK`: Uses the network backend.

Patches an owned NPC's difficulty multipliers.

Requires `world.npcs`. Accepts a partial table of finite `healthMultiplier` (0.01..100) and `damageMultiplier` (0..100); omitted values stay unchanged. Invalid keys, types or ranges return `false, "npc_difficulty_invalid"` without mutation. H divides admitted native body and admitted player body/weakspot fractions after admission, without rescaling authored part-break bonuses. D is used by resource-authored NPC-to-player damage and managed crossfire. Health fraction and phase are unchanged; exact scripted damage and setHealth are not scaled.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `difficulty` | `table` | required | — |

Returns: `boolean`, `reason`

```lua
local ok, reason = Open77.npcs.setDifficulty(npcId, { healthMultiplier = 2 })
if not ok then print(reason) end
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9570) as `LuaResourceRuntime.Bootstrap`.

## setEquipment

```lua
Open77.npcs.setEquipment(id, set)
```

`NETWORK`: Uses the network backend.

Sets the canonical clothing set of an owned NPC, slot to `Items.*` record. Whether a rig renders it is the client's answer, not this call's.

Requires `world.npcs` and ownership. FiveM's `SetPedComponentVariation` on a ped, through the same puppet equipment path the loadout's `weapon` key already uses. The table is the WHOLE desired set: keys are the nine clothing slots (`Head`, `Face`, `InnerChest`, `OuterChest`, `Legs`, `Feet`, `Outfit`, `UnderwearTop`, `UnderwearBottom` -- the held weapon stays on `setLoadout`), values are `Items.*` record names, and a slot left out is a slot you want empty; `{}` undresses. Only a slot Open77 itself filled is ever emptied, so a character's own look (its appearance) is never stripped. Published as one revision on `loadout.equipment`, reaches late joiners, and survives a later `setLoadout` that does not mention `equipment`. Existence of the record and whether the garment binds to that character's rig are decided on each client against the live game -- the client never runs `InitializeSlots` for clothing, and refuses a slot the `Character.*` record does not declare with a log line (`slot_unavailable`). Refusals: `npc_equipment_invalid`, `npc_equipment_slot_invalid`, `npc_equipment_record_invalid`, `npc_not_found`, `npc_not_owned`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `set` | `table { slot = "Items.*", ... }` | required | — |

Returns: `true on success, otherwise nil`, `reason`

```lua
-- Canonical state, replicated and kept for late joiners. Whether the clerk is
-- SEEN in these clothes is decided per client against the live game: a
-- `Character.*` record that does not declare the slot refuses it with
-- `slot_unavailable`, and measured on 2.31 a Character rig renders none of it.
-- Read it back with getEquipment; do not treat a `true` here as a visible outfit.
local ok, reason = Open77.npcs.setEquipment(clerkId, {
    OuterChest = "Items.Jacket_01_basic_01",
    Legs = "Items.Pants_01_basic_01",
    Feet = "Items.Shoes_01_basic_01",
})
if not ok then print("could not dress the clerk: " .. reason) end
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.Npcs6.cs` (line 246) as `LuaResourceRuntime.Bootstrap`.

## setGroup

```lua
Open77.npcs.setGroup(id, group)
```

`NETWORK`: Uses the network backend.

Puts an NPC in a relationship group.

Requires `world.npcs`. Writes the pre-existing `combat.group` loadout field, so it keeps its existing meaning -- two NPCs sharing a non-empty group are allies -- and additionally picks up whatever `setRelationship` has declared for that group. Group names are at most 64 characters of `A-Z a-z 0-9 _ - .`; `nil` removes the group. Unlike the attitude rows, a group does **not** survive a wholesale `setLoadout`, because it is an ordinary loadout field.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `group` | `string, or nil to clear` | required | — |

Returns: `boolean`, `reason`

```lua
-- Two spawned gang members become allies through a shared group
Open77.npcs.setGroup(firstId, "maelstrom")
Open77.npcs.setGroup(secondId, "maelstrom")
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9320) as `LuaResourceRuntime.Bootstrap`.

## setHealth

```lua
Open77.npcs.setHealth(id, health, [maxHealth])
```

`NETWORK`: Uses the network backend.

Convenience patch for an NPC's health and maximum.

Requires `world.npcs`. Passing `maxHealth` raises or lowers the canonical maximum at the same time; omitting it leaves the maximum alone.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `health` | `number` | required | — |
| `maxHealth` | `number` | optional | — |

Returns: `boolean`, `reason`

```lua
-- The boss has 500 health, out of 500
Open77.npcs.setHealth(bossId, 500, 500)
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9276) as `LuaResourceRuntime.Bootstrap`.

## setLoadout

```lua
Open77.npcs.setLoadout(id, loadout)
```

`NETWORK`: Uses the network backend.

Convenience patch for an NPC's loadout.

Requires `world.npcs`. Exactly `update(id, { loadout = loadout })`. Stable 147 supports player body snapshots in the NPC loadout. update(id, {body=...}) replaces the body while retaining other loadout data. Weapon-only loadout changes preserve the body and equipment. The complete loadout remains limited to 8 KiB. See [Character previews](https://open2077.net/docs/api/server/character-previews.md#networked-npc).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `loadout` | `table|string` | required | — |

Returns: `boolean`, `reason`

```lua
-- Arm the guard with a Lexington
Open77.npcs.setLoadout(guardId, { weapon = "Items.Preset_Lexington_Default" })
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9275) as `LuaResourceRuntime.Bootstrap`.

## setPerceptionEnabled

```lua
Open77.npcs.setPerceptionEnabled(id, enabled)
```

`NETWORK`: Uses the network backend.

Controls autonomous sensory acquisition for a server-owned NPC.

Requires world.npcs and ownership. False disables senses, reactions and automatic threat seeding; does not cancel an existing explicit target. Combat or AI disabled takes precedence.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `boolean success, optional string reason`

```lua
-- A stealth mission: the guard only fights if a script tells it to
Open77.npcs.setPerceptionEnabled(guardId, false)
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9294) as `LuaResourceRuntime.Bootstrap`.

## setRagdoll

```lua
Open77.npcs.setRagdoll(id, enabled)
```

`NETWORK`: Uses the network backend.

Sets an NPC's canonical ragdoll state.

Requires `world.npcs`. Exactly `update(id, { ragdoll = enabled == true })`; the state is replicated, so every viewer agrees.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `boolean`, `reason`

```lua
-- Knocked out by a punch: the body goes limp for three seconds
Open77.npcs.setRagdoll(npcId, true)
SetTimeout(3000, function() Open77.npcs.setRagdoll(npcId, false) end)
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9329) as `LuaResourceRuntime.Bootstrap`.

## setRelationship

```lua
Open77.npcs.setRelationship(groupA, groupB, attitude)
```

`NETWORK`: Uses the network backend.

Declares how two of this resource's groups regard each other.

Requires `world.npcs`. The matrix is **per resource** -- one resource's groups never reach another's bodies, exactly as its NPC reads and writes never do -- and it is symmetric: declaring `a` against `b` also declares `b` against `a`. The resulting row is projected onto every owned NPC in either group and replicated, so observers resolve it without asking the server; an NPC that joins the group later picks up the standing row. `nil` for `attitude` removes the declaration. The matrix dies with the resource that declared it. This is NOT the engine's global attitude matrix: `gameCAttitudeManager.SetAttitudeRelation` is process-wide and shared with vanilla content, and is deliberately not used -- see `docs/research/npc-behavior-control.md`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `groupA` | `string` | required | — |
| `groupB` | `string` | required | — |
| `attitude` | `string `friendly`|`neutral`|`hostile`, or nil to clear` | required | — |

Returns: `boolean`, `reason`

```lua
-- Gang war: every Maelstrom NPC this resource owns hates every Valentino
Open77.npcs.setRelationship("maelstrom", "valentinos", "hostile")
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9326) as `LuaResourceRuntime.Bootstrap`.

## setTransform

```lua
Open77.npcs.setTransform(id, def)
```

`NETWORK`: Uses the network backend.

Sets an NPC's canonical position and yaw.

Requires `world.npcs`. A teleport, not a walk: for movement that the AI performs, enqueue a `moveTo` task instead.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `def` | `table { position, yaw? }` | required | — |

Returns: `boolean`, `reason`

```lua
-- Teleport the shopkeeper back behind the counter
Open77.npcs.setTransform(shopkeeperId, { position = counter, yaw = 180.0 })
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9266) as `LuaResourceRuntime.Bootstrap`.

## setVoiceEnabled

```lua
Open77.npcs.setVoiceEnabled(id, enabled)
```

`NETWORK`: Uses the network backend.

Controls voice lines and barks of a server-owned NPC.

Requires world.npcs and ownership. Controls native voiceset lines/grunts (including combat/search barks), suppresses scripted voiceovers and filters native dialogue routed through subtitles. Replicated per NPC, not a global sound/SFX mute or a guarantee to stop audio already playing.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `boolean success, optional string reason`

```lua
-- Silence the bartender's barks during the cutscene
Open77.npcs.setVoiceEnabled(bartenderId, false)
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9298) as `LuaResourceRuntime.Bootstrap`.

## speak

```lua
Open77.npcs.speak(id, voice, options)
```

`NETWORK`: Uses the network backend.

Queues one voice-over line from an NPC's own voiceset on every client streaming it. `voice` is a `voContext` name (`greeting`, `fear_beg`), never a sentence: the NPC cannot say arbitrary text. `true` is the line queued, not the line heard.

Requires `world.npcs` and ownership. FiveM's `PlayPedAmbientSpeechNative`: `voice` is a `voContext` name -- the same word vanilla passes to `GameObject.PlayVoiceOver` (`greeting`, `fear_beg`, `rep_ask_to_leave`) -- resolved by the engine against the puppet's voiceset (`Character.*.voiceTag`), not a Wwise event. `Open77.npcs.voices()` lists the generic barks that can be pointed at in the game's 2.31 sources; any identifier (letters, digits, underscore, at most 64) is accepted, because a record may carry lines no vanilla script calls. **A name the voiceset does not have is silent and nothing can report it** -- `true` means queued on every viewer, not heard. Audibility was not measured in the test loop (no audio capture; the plumbing is proven as far as the `SoundPlayVo` event reaching each viewer's puppet): test the lines you ship by ear, on the records you ship. Rides the existing effect one-shot to the NPC's routing bucket; the bundled `open77_effects` resource queues the engine's own `SoundPlayVo` event on each viewer's puppet through `Open77.sfx.playVoice`. `options.ignoreFrustum` (default true) speaks off-screen; `options.ignoreDistance` (default false) skips the engine's distance cull. Refusals: `invalid_voice`, `invalid_npc_id`, `npc_not_found`, `npc_not_owned`, `npc_dead`, `voice_disabled` (`setVoiceEnabled(id, false)` is in force), `npc_not_streamed` (no client has reported the body ready, so nobody could hear it), `npc_voice_busy` (a line was accepted on this NPC less than 400 ms ago).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `voice` | `string (voContext name, not text)` | required | — |
| `options` | `table { ignoreFrustum?, ignoreDistance? }` | required | — |

Returns: `true on success, otherwise nil`, `reason`

```lua
-- The clerk greets whoever uses the counter prompt.
AddEventHandler("onNpcInteracted", function(npcId, playerId)
    if npcId == clerkId then
        local ok, reason = Open77.npcs.speak(clerkId, "greeting")
        if not ok then print("clerk stayed silent: " .. reason) end
    end
end)
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.Npcs6.cs` (line 204) as `LuaResourceRuntime.Bootstrap`.

## target

```lua
Open77.npcs.target(id)
```

`NETWORK`: Uses the network backend.

Who an NPC is currently fighting.

Requires `world.npcs`, and only for an NPC this resource created. Returns `{ kind = "none"|"player"|"npc", id = n }`, mirroring the last reading its lease holder reported. It is the engine's own target tracker, not a record of what a script asked for, so an NPC that acquires a target by perception shows up here too. `onNpcTargetChanged` fires on every change to this value.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |

Returns: `table { kind, id }, or nil`

```lua
-- Is the guard fighting a player right now?
local target = Open77.npcs.target(guardId)
if target and target.kind == "player" then
    Open77.chat.send(target.id, "The guard is after you!")
end
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9328) as `LuaResourceRuntime.Bootstrap`.

## templates

```lua
Open77.npcs.templates()
```

`NETWORK`: Uses the network backend.

The legacy alias catalogue of NPC templates.

Requires `world.npcs`. It is a convenience list, **not** a limit: `create` accepts any `Character.*` record whether or not it appears here.

Returns: `array of template names`

```lua
-- Print the legacy alias list once at startup
for _, template in ipairs(Open77.npcs.templates()) do
    print(template.name, template.record)
end
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9343) as `LuaResourceRuntime.Bootstrap`.

## update

```lua
Open77.npcs.update(id, def)
```

`NETWORK`: Uses the network backend.

Patches an NPC's canonical state.

Requires `world.npcs`. Sparse: appearance, loadout, behavior, AI mode, damage policy, health and ragdoll can all be patched in one call, and everything omitted is left alone. The convenience setters below are thin wrappers over this. Stable 147 supports player body snapshots in the NPC loadout. update(id, {body=...}) replaces the body while retaining other loadout data. Weapon-only loadout changes preserve the body and equipment. The complete loadout remains limited to 8 KiB. See [Character previews](https://open2077.net/docs/api/server/character-previews.md#networked-npc). Stable 156 snapshots include armor. Server create accepts armor (default 0), update preserves it when omitted; finite 0..100000 points absorb canonical damage before health. Matching protocol 1.50 is required. See [NPC armor](../../01%20Documentation/npcs.md#armor).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `def` | `table (patch)` | required | — |

Returns: `boolean`, `reason`

```lua
-- Change several things in one replicated patch
Open77.npcs.update(npcId, {
    health = 100,
    damagePolicy = "mortal",
    behavior = { combatEnabled = true },
})
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.cs` (line 9252) as `LuaResourceRuntime.Bootstrap`.

## voices

```lua
Open77.npcs.voices()
```

`NETWORK`: Uses the network backend.

The documented bark vocabulary for `speak`, each name with the 2.31 source that plays it.

Requires nothing. Rows of `{ name, context, source }` where `source` is a file and line in the game's own shipped REDscript (`tools/redmod/scripts`) or `.tweak` sources (`tools/redmod/tweaks`) -- names that can be re-read rather than believed. It is documentation, not an allowlist: `speak` accepts any identifier, and whether a given NPC's voiceset carries a line is up to its record's `voiceTag`.

Returns: `array of { name, context, source }`

```lua
-- Pick a random documented bark for an idle vendor.
local rows = Open77.npcs.voices()
local pick = rows[math.random(#rows)]
Open77.npcs.speak(vendorId, pick.name)
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.Npcs6.cs` (line 219) as `LuaResourceRuntime.Bootstrap`.

## whenReady

```lua
Open77.npcs.whenReady(id, [timeoutMs])
```

`NETWORK`: Uses the network backend.

A promise that resolves once at least one client has a ready body for an NPC.

Requires `world.npcs`. The server half of the preloading helpers: FiveM code waits on the streamer with `RequestModel` / `HasModelLoaded` loops; here the server's only honest question is "does some client have a body for this NPC yet", and the answer is already on the wire -- every client reports its projection ready, `Open77.npcs.owner(id).readyClients` counts them and the election publishes `onNpcAuthorityChanged(id, player, epoch, "projectionready")`. Resolves with the `Open77.npcs.owner(id)` table as soon as `readyClients > 0` (before the call returns when one already has) and rejects with `timeout` after `timeoutMs` (default 15000, clamped to 1..120000); listens for the election event and polls the owner read every 250 ms as the safety net. The one-shot handler is removed on resolve, on timeout and with the resource. `nil, reason` only for a bad id, a bad timeout or a missing permission; an NPC the server does not know is waited for and times out, exactly like the client's `vehicles.whenStreamed` on an id it has never seen. There is deliberately no vehicle counterpart on the server: a vehicle's steward is elected by a driver's claim, never by a streamed report, so the server holds no signal that means "somebody has this car" -- `Open77.vehicles.create` already returns after the canonical create, and the client's `Open77.vehicles.whenStreamed` answers for its own projection.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `timeoutMs` | `number` | optional | — |

Returns: `Promise, or nil`, `reason: invalid_npc_id, invalid_timeout, permission_denied:world.npcs, task_limit_or_stopping`

```lua
-- Send the guard on patrol only once somebody can animate him
local guard = assert(Open77.npcs.create({ template = "civilian_female_relaxed_01", position = post }))
Open77.npcs.whenReady(guard, 10000):next(function(owner)
    Open77.npcs.tasks.patrol(guard, route, { loop = true })
end, function(reason)
    print(("guard %d never streamed: %s"):format(guard, reason))
end)
```

[NPC AI, combat & voice guide](../../01%20Documentation/npc-behavior.md)

Registered in `LuaResourceRuntime.Streaming6.cs` (line 53) as `LuaResourceRuntime.Bootstrap`.
