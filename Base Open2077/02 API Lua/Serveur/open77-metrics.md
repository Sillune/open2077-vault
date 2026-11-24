# Open77.metrics (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-metrics.md) · [Page web](https://open2077.net/docs/api/server/open77-metrics)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

3 functions.

## gauge

```lua
Open77.metrics.gauge(name, value)
```

`NETWORK`: Uses the network backend.

Sets a resource-labelled gameplay gauge.

Server only; requires `metrics.write`. Replaces the current value of `open77_gameplay_<name>`, with the host supplying this resource's label. Value must be finite with absolute value at most 1e100; negative gauges are allowed. Names must match `[a-z][a-z0-9_]*`, be 1–64 characters, and not end in `_total`, `_bucket`, `_sum`, `_count` or `_seconds`. A name's metric type is fixed across resources. Limits until server restart are 32 names per resource and 512 resource/name pairs globally. An explicit resource stop resets its gauges to zero; set them on startup and update them as gameplay changes. There are no caller-supplied labels. Returns true or false, reason. See [Metrics](https://open2077.net/docs/api/server/metrics.md#custom-gameplay-counters-from-lua).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `value` | `number` | required | — |

Returns: `boolean`, `reason: permission_denied:metrics.write, metrics_disabled, invalid_metric_arguments, invalid_metric_name, invalid_metric_value, metric_type_conflict, metric_limit`

```lua
Open77.metrics.gauge("race_participants", #participants)
```

Registered in `LuaResourceRuntime.Metrics.cs` (line 29) as `LuaResourceRuntime.Bootstrap`.

## increment

```lua
Open77.metrics.increment(name, [amount])
```

`NETWORK`: Uses the network backend.

Adds to a resource-labelled gameplay counter.

Server only; requires `metrics.write`. Adds `amount` (default 1 when omitted or nil) to `open77_gameplay_<name>_total`, with the host supplying this resource's label. Amount must be finite and between 0 and 1e100. Names must match `[a-z][a-z0-9_]*`, be 1–64 characters, and not end in `_total`, `_bucket`, `_sum`, `_count` or `_seconds`. A name's metric type is fixed across resources. Limits until server restart are 32 names per resource and 512 resource/name pairs globally. Counters survive resource restarts; do not increment during speculative hot-reload initialization because a discarded candidate's increments cannot be rolled back. Returns true or false, reason. See [Metrics](https://open2077.net/docs/api/server/metrics.md#custom-gameplay-counters-from-lua).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `amount` | `number` | optional | — |

Returns: `boolean`, `reason: permission_denied:metrics.write, metrics_disabled, invalid_metric_arguments, invalid_metric_name, invalid_metric_value, metric_type_conflict, metric_limit`

```lua
local ok, reason = Open77.metrics.increment("shop_sales")
if not ok then print(reason) end
```

Registered in `LuaResourceRuntime.Metrics.cs` (line 28) as `LuaResourceRuntime.Bootstrap`.

## observe

```lua
Open77.metrics.observe(name, seconds)
```

`NETWORK`: Uses the network backend.

Records a duration in a resource-labelled gameplay histogram.

Server only; requires `metrics.write`. Adds one observation to the `open77_gameplay_<name>_seconds` histogram family, with the host supplying this resource's label. Supply seconds, not milliseconds; the value must be finite and between 0 and 1e100. Names must match `[a-z][a-z0-9_]*`, be 1–64 characters, and not end in `_total`, `_bucket`, `_sum`, `_count` or `_seconds`. A name's metric type is fixed across resources. Limits until server restart are 32 names per resource and 512 resource/name pairs globally. Histograms survive resource restarts; do not observe during speculative hot-reload initialization because discarded observations cannot be rolled back. Returns true or false, reason. See [Metrics](https://open2077.net/docs/api/server/metrics.md#custom-gameplay-counters-from-lua).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `seconds` | `number` | required | — |

Returns: `boolean`, `reason: permission_denied:metrics.write, metrics_disabled, invalid_metric_arguments, invalid_metric_name, invalid_metric_value, metric_type_conflict, metric_limit`

```lua
Open77.metrics.observe("job_run_duration", elapsedMilliseconds / 1000)
```

Registered in `LuaResourceRuntime.Metrics.cs` (line 30) as `LuaResourceRuntime.Bootstrap`.
