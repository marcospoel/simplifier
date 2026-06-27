# Architecture

## Platform overview

This project builds enterprise applications on the **Simplifier Low-Code Platform** (SAP ecosystem), hosted at `https://tsystems-eval.simplifier.cloud`.

## Layers

```
┌─────────────────────────────────────────────────┐
│  UI Layer                                        │
│  OpenUI5 1.96.40 widgets, screens, themes        │
│  Built in Simplifier Screen Designer             │
├─────────────────────────────────────────────────┤
│  Logic Layer                                     │
│  Business Objects (Node.js JS)                   │
│  Events, conditions, scripting, Process Designer │
├─────────────────────────────────────────────────┤
│  Integration Layer                               │
│  Connectors: REST, SOAP, SQL, SAP RFC, OData     │
│  Connector Calls, Logins, Filters                │
├─────────────────────────────────────────────────┤
│  Data Layer                                      │
│  Data Types: Domain, Struct, Collection          │
│  Database Designer (MySQL 8.4 / Oracle)          │
└─────────────────────────────────────────────────┘
```

## Key concepts

| Concept | Description |
|---|---|
| **App** | Container of Screens; runs in browser or Simplifier Mobile Client |
| **Screen** | Single view built with OpenUI5 1.96.40 widgets |
| **Business Object (BO)** | Server-side Node.js functions; callable from apps and other BOs |
| **Connector** | Integration definition (REST, SOAP, SQL, SAP RFC, OData V2/V4) |
| **Connector Call** | A specific operation on a connector with defined parameters |
| **Data Type** | Domain (single value), Struct (record), Collection (list) |
| **Workflow** | Process automation: Automated, User, and Notification tasks |
| **Module** | Reusable app component with a defined interface |
| **Project** | Organizational container grouping all artifacts |

## Integration gateway: TARDIS

All Simplifier → SAP / Telekom internal API calls must route through **TARDIS StarGate**. See [`docs/tardis.md`](tardis.md) for:
- Component overview (StarGate, Iris, Rover, Horizon, Chevron, Raccoon, Drax)
- OAuth2 M2M auth flow (Iris Client Credentials)
- Rover `rover.yaml` reference and onboarding steps
- Simplifier-specific wiring: connector base URL, token caching, subscription registration

## MCP server topology

```
Claude Code
  ├── simplifier-mcp  →  https://tsystems-eval.simplifier.cloud  (platform API)
  ├── Telecontext     →  https://telecontext.trap.ng.telekom.net  (Jira + Confluence)
  ├── ui5-mcp-server  →  https://ui5.sap.com/1.96.40             (UI5 docs/lint)
  ├── figma           →  http://127.0.0.1:3845                   (Figma desktop)
  ├── playwright      →  Chrome browser                          (UI testing)
  └── chrome-devtools →  Chrome browser                          (debugging)
```

## Agent orchestration

See [../AGENTS.md](../AGENTS.md) for the full agent index and orchestration patterns.

## Artifact inventory

See `docs/artifacts/` for the current state of all Simplifier artifacts:
- [apps.md](artifacts/apps.md)
- [business-objects.md](artifacts/business-objects.md)
- [connectors.md](artifacts/connectors.md)
- [workflows.md](artifacts/workflows.md)
- [data-types.md](artifacts/data-types.md)
