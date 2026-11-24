# Open77.heldItems (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-helditems.md) · [Page web](https://open2077.net/docs/api/server/open77-helditems)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

4 functions.

## hold

```lua
Open77.heldItems.hold(playerId, record, [options])
```

`NETWORK`: Uses the network backend.

Hold a native item on the presented player body, replicated to nearby clients.

WeaponRight (default) uses resource-owned replicated native item props and the authored AttachmentSlots.WeaponRight transform. Requires world.props, players.life.read and network.events. Returns a request ID; observe open77:helditem:completed. Native results include propId, record, slot, rendered and attachmentStatus. hidden_first_person is accepted presentation intent with rendered=false, not visual success. Initial failures/timeouts remove only that requested item. Another resource's occupied hand is refused. Hold before starting a walking RP animation to supply an inventory item instead of the animation default; release on your action completion. No job dependency or inventory grant/consumption. Records must have a usable native item factory and a grip compatible with the animation. Other nine equipment slots retain their legacy relay. See attachments.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `record` | `string` | required | — |
| `options` | `table { slot? }` | optional | — |

Returns: `request id, or nil`, `reason`

```lua
local request, reason = Open77.heldItems.hold(playerId, "Items.crowd_soda_can_a")
if not request then print(reason) end
```

[World props guide](../../01%20Documentation/props.md#attachment)

Registered in `LuaResourceRuntime.cs` (line 8381) as `LuaResourceRuntime.Bootstrap`.

## release

```lua
Open77.heldItems.release(playerId, [options])
```

`NETWORK`: Uses the network backend.

Empties one of a player's equipment slots of the item this API put there.

WeaponRight removes only this server resource's held native item from the replicated registry; other resources' items are untouched. Completion is asynchronous. Native item props are also removed on death, disconnect, bucket change and resource stop. Other equipment slots keep their existing relay. See attachments.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `options` | `table { slot? }` | optional | — |

Returns: `request id, or nil`, `reason`

```lua
-- The courier delivered: empty the hand
Open77.heldItems.release(courierId, { slot = "WeaponRight" })
```

[World props guide](../../01%20Documentation/props.md#attachment)

Registered in `LuaResourceRuntime.cs` (line 8394) as `LuaResourceRuntime.Bootstrap`.

## requestSnapshot

```lua
Open77.heldItems.requestSnapshot(playerId)
```

`NETWORK`: Uses the network backend.

Asks a player's client what its ten equipment slots currently hold.

Asynchronous completion carries the ten slot states. The current native hand item is read from its projected snapshot, including attachmentStatus and whether it renders; clothing uses the local equipment registry. Neither a queued item nor a hidden first-person item is reported as a visibly occupied hand.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `request id, or nil`, `reason`

```lua
-- What is the player holding right now?
Open77.heldItems.requestSnapshot(playerId)
AddEventHandler("open77:helditem:completed", function(playerId, requestId, operation, accepted, reason, result)
    if operation == "snapshot" and accepted then print(json.encode(result)) end
end)
```

[World props guide](../../01%20Documentation/props.md#attachment)

Registered in `LuaResourceRuntime.cs` (line 8402) as `LuaResourceRuntime.Bootstrap`.

## slots

```lua
Open77.heldItems.slots()
```

`NETWORK`: Uses the network backend.

The ten equipment slot names Open77.heldItems accepts.

Sorted. These are the `AttachmentSlots.*` TweakDB records, the only closed and validated slot vocabulary in the tree -- `entSlotComponent` slot names are authored per entity and cannot be enumerated ahead of time, which is why the effects surface (`Open77.effects.attach`) takes a free string and validates it against the target instead.

Returns: `array of strings`

```lua
-- Print the slot names this API accepts
for _, slot in ipairs(Open77.heldItems.slots()) do print(slot) end
```

[World props guide](../../01%20Documentation/props.md#attachment)

Registered in `LuaResourceRuntime.cs` (line 8374) as `LuaResourceRuntime.Bootstrap`.
