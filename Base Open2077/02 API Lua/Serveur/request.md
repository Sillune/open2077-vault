# .request (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/request.md) · [Page web](https://open2077.net/docs/api/server/request)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

1 function.

## await

```lua
.request.await(options)
```

`SERVER`: Available in server resources only.

Outbound HTTP: await.

Open77.http calls web APIs, webhooks and file hosts. It requires the http.request
permission and a server that enabled outbound HTTP with an allow-list. The request leaves the
tick thread at once, and the callback (or .await) resumes on the owning resource's own tick.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `not specified` | required | — |

Documented in [server-api.md](../../01%20Documentation/server-api.md#outbound-http) (line 3025).
