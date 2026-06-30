# Security Overview

> Maintained by `security` agent. Updated after every architecture or integration change.

## Threat model

**Application:** InterfaceCockpit (and future Simplifier apps in this project)  
**Platform:** Simplifier Low-Code on `tsystems-eval.simplifier.cloud` (T-Systems cloud)  
**Classification:** T-Systems internal — interfaces monitoring data, no direct PII in v1

### Trust boundaries

```
[Browser / User]
      │  HTTPS (TLS)
      ▼
[Simplifier Platform — tsystems-eval.simplifier.cloud]
      │  SimplifierToken session auth
      ▼
[Server Business Objects — Node.js / Simplifier JS runtime]
      │  Connector Logins (stored credentials)
      ▼
[Backend Systems: SAP RFC / OData / REST APIs]
```

### Key security decisions

| Decision | Rationale |
|---|---|
| RBAC enforced at BO level, not only UI | UI controls can be bypassed; server-side enforcement is authoritative |
| SimplifierToken stored in `.env` only | Prevents accidental commit to git |
| SQL connectors use named bind params only | Prevents SQL injection |
| Connector base URLs are fixed values | Prevents SSRF via user input |
| `Simplifier.Log.*` used for security events | Centralised audit trail on the platform |

## Component versions

See [components.md](components.md).

## RBAC

See [rbac.md](rbac.md).

## PSA

See [psa.md](psa.md).
