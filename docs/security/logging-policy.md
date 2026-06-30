# Logging Policy

> Maintained by `security` agent.

## What to log (using `Simplifier.Log.*` in server BOs)

| Event | Log level | Example |
|---|---|---|
| Access denied (wrong role) | Warning | `"Unauthorized triggerRetry by user X"` |
| Input validation failure | Warning | `"Invalid interfaceId: null from user X"` |
| Unhandled exception in BO | Error | `"bInterfaceStatus.triggerRetry: " + e.message` |
| Successful retry trigger (audit) | Info | `"triggerRetry called for interfaceId X by user Y"` |

## What must NEVER be logged

- Session tokens (`SimplifierToken`)
- Passwords or API keys
- Full HTTP request/response bodies (may contain credentials)
- PII (if introduced in future features)

## Log access

Simplifier platform logs are accessible to platform administrators and the application owner. They are not exposed to end users.

## Retention

Per T-Systems platform policy (typically 90 days for application logs). Confirm with platform admin.
