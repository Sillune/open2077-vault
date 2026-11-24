# Open77.acl (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-acl.md) · [Page web](https://open2077.net/docs/api/server/open77-acl)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

8 functions.

## addRole

```lua
Open77.acl.addRole(userId, role)
```

`NETWORK`: Uses the network backend.

Puts a user in an ACL role at runtime.

FiveM's `add_principal`. Requires `acl.grant:<pattern>`, and the check is on the role's **entire reach**, not on its name: every permission behind it must be inside the scope or the call is refused `role_exceeds_scope`. Without that rule a resource scoped to `job.police.*` would collect `command.*` by adding the operator's `moderator` role, which is the same escalation arriving one indirection later. The role may be one this resource defined with `definePermission` or one the operator defined in `acl.jsonc`. Idempotent. Shows up in `Open77.acl.roles`. Publishes `onAclChanged`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `userId` | `string (uuid)` | required | — |
| `role` | `string` | required | — |

Returns: `true, or nil`, `reason: role_exceeds_scope, role_not_defined, role_reserved, operator_principal, permission_denied:acl.grant, scope_not_delegable, invalid_user_id, invalid_role, runtime_acl_limit, acl_persistence_failed`

```lua
-- Promote a player to the "cop" role defined by this resource (needs acl.grant:job.police.*)
RegisterCommand("hire-cop", function(source, args)
    local identity = Open77.players.identity(tonumber(args[1]))
    if identity then Open77.acl.addRole(identity.userId, "cop") end
end, true)
```

[Commands & ACL guide](../../01%20Documentation/server-acl.md)

Registered in `LuaResourceRuntime.Acl.cs` (line 264) as `LuaResourceRuntime.Bootstrap`.

## definePermission

```lua
Open77.acl.definePermission(role, permissions)
```

`NETWORK`: Uses the network backend.

Defines, retunes or deletes what a runtime ACL role may do.

FiveM's `add_ace` against a group. Requires the **second** capability, `acl.define:<pattern>`, because retuning a role changes the rights of everyone holding it at once and retroactively -- a resource that may promote people should not also decide what the promotion is worth. Every permission, and every permission the role *currently* holds, must be inside that scope. Passing nil or an empty list deletes the role and takes it off every subject holding it, rather than leaving a name that resolves to nothing. A role the operator defined in `acl.jsonc` is theirs: `role_defined_by_operator`. `owner` and `admin` are reserved. Publishes `onAclChanged`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `role` | `string` | required | — |
| `permissions` | `array of string, or nil to delete` | required | — |

Returns: `true, or nil`, `reason: permission_denied:acl.define, scope_not_delegable, permission_not_delegable, role_defined_by_operator, role_reserved, not_defined, invalid_role, invalid_permission, runtime_acl_limit, acl_persistence_failed`

```lua
-- Define what a cop may do (needs acl.define:job.police.*)
Open77.acl.definePermission("cop", { "job.police.arrest", "job.police.fine" })
```

[Commands & ACL guide](../../01%20Documentation/server-acl.md)

Registered in `LuaResourceRuntime.Acl.cs` (line 274) as `LuaResourceRuntime.Bootstrap`.

## grant

```lua
Open77.acl.grant(userId, permission)
```

`NETWORK`: Uses the network backend.

Grants one permission to a user id at runtime, persisted.

Native-parity A12, FiveM's `add_ace`. Requires a scoped `acl.grant:<pattern>` in the manifest; there is no bare `acl.grant`, and `acl.grant:*` is not a wildcard scope but is dropped, leaving `scope_not_delegable`. The permission must fall inside one of the declared patterns or the call is refused `permission_not_delegable` -- that scope is the whole answer to whether a resource may grant a right it does not itself hold. The subject is a **master-issued userId**, the string `Open77.players.identity(playerId).userId` answers, never a playerId: a grant outlives the session that earned it. Writes to `acl.runtime.jsonc`, the machine-owned sibling of `acl.jsonc`, atomically through a temporary; the operator's own document is never touched and can never be reduced by this call. Takes effect on the very next `isAllowed`, with no reload. Idempotent. Publishes `onAclChanged`. See the [ACL guide](https://open2077.net/docs/api/server/server-acl.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `userId` | `string (uuid)` | required | — |
| `permission` | `string` | required | — |

Returns: `true, or nil`, `reason: permission_denied:acl.grant, scope_not_delegable, permission_not_delegable, operator_principal, invalid_user_id, invalid_permission, runtime_acl_limit, acl_persistence_failed, acl_unavailable`

```lua
-- A single right for one account, persisted across sessions
local identity = Open77.players.identity(playerId)
Open77.acl.grant(identity.userId, "job.police.arrest")
```

[Commands & ACL guide](../../01%20Documentation/server-acl.md)

Registered in `LuaResourceRuntime.Acl.cs` (line 258) as `LuaResourceRuntime.Bootstrap`.

## grants

```lua
Open77.acl.grants(userId)
```

`NETWORK`: Uses the network backend.

What the runtime overlay holds for one user id.

Requires the existing `acl.read`. Answers `{ permissions = {}, roles = {} }`, with empty tables for a subject that has no runtime grant. It reports **only** what the resource layer granted at runtime and never a principal from the operator's `acl.jsonc`, so it widens nothing a per-permission `isAllowed` could not already answer. For the effective answer, which unions both layers, call `Open77.acl.isAllowed`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `userId` | `string (uuid)` | required | — |

Returns: `table { permissions, roles }, or nil`, `reason: permission_denied:acl.read, invalid_user_id, acl_unavailable`

```lua
-- What did this resource layer give the player?
local grants = Open77.acl.grants(userId)
if grants then print(#grants.permissions, "permissions,", #grants.roles, "roles") end
```

[Commands & ACL guide](../../01%20Documentation/server-acl.md)

Registered in `LuaResourceRuntime.Acl.cs` (line 281) as `LuaResourceRuntime.Bootstrap`.

## isAllowed

```lua
Open77.acl.isAllowed(playerId, right)
```

`NETWORK`: Uses the network backend.

Whether a connected player holds an ACL right on this server.

The same access-control list the console and restricted commands consult, exposed so a resource can make its own authorization decision with the operator's configuration rather than a hard-coded admin list. `RegisterCommand(name, handler, true)` already checks `command.<name>` through it. The first argument is the session player id (`source` in a command handler), not a user id or a role name: the ACL is resolved for the identity behind that session, and the console (source 0) is not a player here.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `right` | `string` | required | — |

Returns: `boolean: allowed, or false with a reason (`permission_denied:acl.read`, `invalid_player_id`)`

```lua
-- Only cops may use /arrest
RegisterCommand("arrest", function(source, args)
    if not Open77.acl.isAllowed(source, "job.police.arrest") then
        return Open77.chat.send(source, "You are not a cop.")
    end
    arrest(source, tonumber(args[1]))
end, false)
```

[Commands & ACL guide](../../01%20Documentation/server-acl.md)

Registered in `LuaResourceRuntime.cs` (line 8407) as `LuaResourceRuntime.Bootstrap`.

## removeRole

```lua
Open77.acl.removeRole(userId, role)
```

`NETWORK`: Uses the network backend.

Takes a user out of a role the runtime put them in.

The same scope and the same `role_exceeds_scope` check as `addRole`. Removes only from the runtime overlay: a role assigned by the operator's own principal answers `operator_grant`, and one nobody assigned answers `not_granted`. Publishes `onAclChanged`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `userId` | `string (uuid)` | required | — |
| `role` | `string` | required | — |

Returns: `true, or nil`, `reason: operator_grant, not_granted, role_exceeds_scope, role_not_defined, permission_denied:acl.grant, scope_not_delegable, operator_principal, invalid_user_id, invalid_role, acl_persistence_failed`

```lua
-- Fired from the police
Open77.acl.removeRole(userId, "cop")
```

[Commands & ACL guide](../../01%20Documentation/server-acl.md)

Registered in `LuaResourceRuntime.Acl.cs` (line 267) as `LuaResourceRuntime.Bootstrap`.

## revoke

```lua
Open77.acl.revoke(userId, permission)
```

`NETWORK`: Uses the network backend.

Takes back a runtime permission grant.

FiveM's `remove_ace`. Same `acl.grant:<pattern>` scope and the same refusals as `grant`, so a resource cannot strip a right it could not have handed out -- which is what stops one resource tampering with another's job model. It removes only from the runtime overlay: a permission the operator wrote into `acl.jsonc` answers `operator_grant` rather than appearing to work, and one nobody granted answers `not_granted`. Publishes `onAclChanged`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `userId` | `string (uuid)` | required | — |
| `permission` | `string` | required | — |

Returns: `true, or nil`, `reason: operator_grant, not_granted, permission_denied:acl.grant, scope_not_delegable, permission_not_delegable, operator_principal, invalid_user_id, invalid_permission, acl_persistence_failed`

```lua
-- Take a single right back
Open77.acl.revoke(userId, "job.police.arrest")
```

[Commands & ACL guide](../../01%20Documentation/server-acl.md)

Registered in `LuaResourceRuntime.Acl.cs` (line 261) as `LuaResourceRuntime.Bootstrap`.

## roles

```lua
Open77.acl.roles(playerId)
```

`NETWORK`: Uses the network backend.

The ACL roles a connected player holds.

Reads the server's configured role assignments. Roles are operator configuration, not player state: a resource should treat them as a policy input and still validate the gameplay preconditions of whatever it is about to do.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `array of role names`

```lua
-- Greet admins differently
AddEventHandler("onPlayerReady", function(playerId)
    for _, role in ipairs(Open77.acl.roles(playerId)) do
        if role == "admin" then Open77.chat.send(tonumber(playerId), "Welcome back, admin.") end
    end
end)
```

[Commands & ACL guide](../../01%20Documentation/server-acl.md)

Registered in `LuaResourceRuntime.cs` (line 8407) as `LuaResourceRuntime.Bootstrap`.
