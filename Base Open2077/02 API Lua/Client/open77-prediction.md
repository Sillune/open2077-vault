# Open77.prediction (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-prediction.md) · [Page web](https://open2077.net/docs/api/client/open77-prediction)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

2 functions.

## setPolicy

```lua
Open77.prediction.setPolicy([policy])
```

`GAME`: Requires a live game instance.

Sets the prediction policy this client applies before the server's verdict.

Requires `prediction.policy`. Sets the per-family switches and the round-trip ceiling this client applies to presentation-only predictions. It changes only what this client shows before the server's verdict, never the verdict. `policy` is `{ families = { [name] = boolean, ... }, maxPingMs = integer }`. A family absent from `families` is on, and a name this build does not know is ignored. `maxPingMs` is 0–5000 ms: a round trip above it starts no new prediction, 0 removes the ceiling, and an absent value keeps the compiled 250 ms default. `nil` restores the compiled defaults. The host releases the policy when the owning resource stops. Families: `melee`, `slam`, `hack`, `door`, `blast`, `carContact`, `playerContact`. The bundled `open77_prediction` resource calls this with the policy the server publishes; a gamemode narrows that policy with the resource's `restrict` export instead. See [Prediction](../../01%20Documentation/prediction.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `policy` | `table { families?, maxPingMs? } or nil` | optional | — |

Returns: `true on success`, `nil and a reason on refusal`

```lua
-- open77.lua: permissions { "prediction.policy" }; normally only open77_prediction does this.
AddEventHandler("onClientResourceStart", function(name)
    if name ~= GetCurrentResourceName() then return end
    local ok, reason = Open77.prediction.setPolicy({ families = { blast = false }, maxPingMs = 200 })
    if not ok then print("prediction policy refused: " .. tostring(reason)) end
end)
```

[Prediction guide](../../01%20Documentation/prediction.md)

Registered in `ResourceHost.cpp` (line 24900) as `LuaPredictionSetPolicy`.

## stats

```lua
Open77.prediction.stats()
```

`GAME`: Requires a live game instance.

Reads the live prediction policy, this client's round trip and per-family counters.

No permission. Returns `{ maxPingMs, ping, owner?, families }`: the ceiling in force, this client's measured round trip in milliseconds (0 or less while unknown), the resource whose policy is applied (absent under the compiled defaults), and one `{ enabled, predicted, adopted, refuted, skippedDisabled, skippedLatency }` table per family. `predicted` counts predictions that started, `adopted` those the server's own reaction took over or confirmed, `refuted` those it refused or never answered in time, and `skippedDisabled` / `skippedLatency` those the policy stopped before they started. Counters are per game process; reading them sends no network traffic. See [Prediction](../../01%20Documentation/prediction.md).

Returns: `{ maxPingMs, ping, owner?, families }`, `nil and a reason on refusal`

```lua
RegisterCommand("predstats", function()
    local stats, reason = Open77.prediction.stats()
    if not stats then print(reason); return end
    print(("ping %d ms, ceiling %d ms"):format(stats.ping, stats.maxPingMs))
    for name, family in pairs(stats.families) do
        print(name, family.enabled, family.predicted, family.adopted, family.refuted)
    end
end, false)
```

[Prediction guide](../../01%20Documentation/prediction.md)

Registered in `ResourceHost.cpp` (line 24901) as `LuaPredictionStats`.
