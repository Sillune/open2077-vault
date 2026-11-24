# Open77.perspective (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-perspective.md) · [Page web](https://open2077.net/docs/api/server/open77-perspective)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

2 functions.

## policy

```lua
Open77.perspective.policy()
```

`NETWORK`: Uses the network backend.

The perspective policy **this** resource set.

Requires `network.events`. Returns `policy, perspective, declared` for the calling VM only -- it is not a read of the server's effective policy, which is whatever was written last by any resource.

Returns: `policy`, `perspective`, `declared (boolean)`

```lua
-- What did this resource set?
local policy, perspective, declared = Open77.perspective.policy()
if declared then print("we set", policy, perspective) end
```

Registered in `LuaResourceRuntime.cs` (line 8266) as `LuaResourceRuntime.Bootstrap`.

## setPolicy

```lua
Open77.perspective.setPolicy(policy, [perspective])
```

`NETWORK`: Uses the network backend.

Sets whether players on this server may use third person.

Requires `network.events`. `policy` is `disabled`, `allowed`, `default` or `forced`; `perspective` is `fpp` or `tps` and is required for `forced`, because a pin with nothing pinned would quietly become first person. The policy is broadcast to every client and answered again to anyone who joins later. Call it from the resource's **start path**: a reload replaces the VM, and the policy lives in it. If two resources declare a policy the last write wins, so one owner -- normally the gamemode -- is the remedy.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `policy` | `string (disabled|allowed|default|forced)` | required | — |
| `perspective` | `string (fpp|tps)` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- A hardcore server: first person only, set on the resource's start path
Open77.perspective.setPolicy("forced", "fpp")
```

Registered in `LuaResourceRuntime.cs` (line 8265) as `LuaResourceRuntime.Bootstrap`.
