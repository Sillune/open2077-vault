# Open77.ragdoll (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-ragdoll.md) · [Page web](https://open2077.net/docs/api/server/open77-ragdoll)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

4 functions.

## impulse

```lua
Open77.ragdoll.impulse(playerId, { impulse, point?, body? })
```

`SERVER`: Available in server resources only.

`true`. Push a body that is down; at most 20 per second per player.

Open77.ragdoll grants ragdoll leases: the player's body falls as a physically simulated ragdoll
that still reacts (it clutches a wound, flails, rolls, braces), on the owner's screen and on every
observer's, and the player gets up where the body ended. Requires players.ragdoll and client
and server 2.50.0-unstable+op77.252 or later; on older servers Open77.ragdoll is nil. The
clients apply leases through the open77_ragdoll resource: without it loaded, a lease ends with
timeout after two seconds. Failures are nil, reason.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `not specified` | required | — |
| `{ impulse, point?, body? }` | `table` | required | — |

Returns: ``true`. Push a body that is down; at most 20 per second per player.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#active-ragdoll) (line 1010).

## start

```lua
Open77.ragdoll.start(playerId, { profile?, impulse?, point?, body?, durationMs?, getUp?, tension? }?)
```

`SERVER`: Available in server resources only.

The lease's serial. `profile` defaults to `"stumble"`; `impulse` is a velocity change in m/s, each component within ±30; `durationMs` (default 3000, at most 30000, `0` = until stopped) counts from the moment the owner's ragdoll simulates. A new `start` replaces a lease (`replaced`).

Open77.ragdoll grants ragdoll leases: the player's body falls as a physically simulated ragdoll
that still reacts (it clutches a wound, flails, rolls, braces), on the owner's screen and on every
observer's, and the player gets up where the body ended. Requires players.ragdoll and client
and server 2.50.0-unstable+op77.252 or later; on older servers Open77.ragdoll is nil. The
clients apply leases through the open77_ragdoll resource: without it loaded, a lease ends with
timeout after two seconds. Failures are nil, reason.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `not specified` | required | — |
| `{ profile?, impulse?, point?, body?, durationMs?, getUp?, tension? }` | `table` | optional | — |

Returns: `The lease's serial. `profile` defaults to `"stumble"`; `impulse` is a velocity change in m/s, each component within ±30; `durationMs` (default 3000, at most 30000, `0` = until stopped) counts from the moment the owner's ragdoll simulates. A new `start` replaces a lease (`replaced`).`

Documented in [server-api.md](../../01%20Documentation/server-api.md#active-ragdoll) (line 1009).

## state

```lua
Open77.ragdoll.state(playerId)
```

`SERVER`: Available in server resources only.

`{ serial, phase, profile, startedAtMs, durationMs, getUp }` (`phase` is `pending` or `active`), or `nil`.

Open77.ragdoll grants ragdoll leases: the player's body falls as a physically simulated ragdoll
that still reacts (it clutches a wound, flails, rolls, braces), on the owner's screen and on every
observer's, and the player gets up where the body ended. Requires players.ragdoll and client
and server 2.50.0-unstable+op77.252 or later; on older servers Open77.ragdoll is nil. The
clients apply leases through the open77_ragdoll resource: without it loaded, a lease ends with
timeout after two seconds. Failures are nil, reason.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `not specified` | required | — |

Returns: ``{ serial, phase, profile, startedAtMs, durationMs, getUp }` (`phase` is `pending` or `active`), or `nil`.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#active-ragdoll) (line 1012).

## stop

```lua
Open77.ragdoll.stop(playerId)
```

`SERVER`: Available in server resources only.

`true`. Ends the lease with `stopped`.

Open77.ragdoll grants ragdoll leases: the player's body falls as a physically simulated ragdoll
that still reacts (it clutches a wound, flails, rolls, braces), on the owner's screen and on every
observer's, and the player gets up where the body ended. Requires players.ragdoll and client
and server 2.50.0-unstable+op77.252 or later; on older servers Open77.ragdoll is nil. The
clients apply leases through the open77_ragdoll resource: without it loaded, a lease ends with
timeout after two seconds. Failures are nil, reason.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `not specified` | required | — |

Returns: ``true`. Ends the lease with `stopped`.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#active-ragdoll) (line 1011).
