# Changelog

## [2026-06-30] Filter dropdown bug fix + conventions update

**Changed:** Fixed `FilterStatusSelect` not filtering `InterfaceTable`; added filter dropdown pattern to conventions, agents, and memory  
**Why:** MBI-2055 — status filter was silently sending empty `filterStatus` to the BO on every change

### Root cause

The compiled `getItemValue` in the Simplifier controller has a broken catch block (missing `return`). When the control lookup throws, `undefined` is returned silently → BO receives empty string → returns all rows.

Additionally, `varFilterStatus`/`varFilterScope` were never written back by the Filter story, so Refresh always reset to "show all" regardless of the dropdown state.

### Fixes applied

| Fix | What changed |
|---|---|
| BO `getInterfaceStatuses` | Added `echoFilterStatus`/`echoFilterScope` optional output params that echo back the applied filter values |
| Filter story SubProcess Output | Added mappings: `echoFilterStatus → varFilterStatus`, `echoFilterScope → varFilterScope` |
| Refresh story | Already reading from `varFilterStatus`/`varFilterScope` variables (correct) |

### Documentation updated

- `docs/simplifier-conventions.md` — new section "Filter dropdown to table pattern" with full checklist
- `agents/simplifier-app-builder.md` — filter pattern checklist added to Workflow
- `agents/simplifier-bo-developer.md` — filter BO pattern (echo outputs) added
- Memory `feedback_simplifier_process_stories.md` — filter pattern + diagnosis steps added

---

## [2026-06-30] Tile filter stories created

**Changed:** Added 4 process stories wiring tile press to `getInterfaceStatuses` with hardcoded `filterStatus` constants  
**Why:** MBI-2055 — TileTotal/OK/Warning/Error tiles now filter the InterfaceTable when pressed

| Story | filterStatus constant |
|---|---|
| `[Cockpit] Tile — Total (show all)` | `""` (empty = all) |
| `[Cockpit] Tile — OK` | `"OK"` |
| `[Cockpit] Tile — Warning` | `"WARNING"` |
| `[Cockpit] Tile — Error` | `"ERROR"` |

---

## [2026-06-28] InterfaceCockpit — data binding fixes + startup load investigation

**Changed:** Fixed tile and table bindings so data renders after Refresh; documented Simplifier process story event trigger behaviour  
**Why:** MBI-2055 — tiles showed 0 and table was empty despite BO returning 13 interfaces with correct data

### Root causes fixed

| Symptom | Root cause | Fix |
|---|---|---|
| Tiles show 0 (varTotalCount etc.) | Bindings used `{varTotalCount}` — resolves against default model (null); variables live in `variableHolder` named model | Changed to `{variableHolder>/varTotalCount}` |
| Table shows template row but no data | `InterfaceTable` had no `itemsPath`/`itemsTemplate` set | Set `itemsTemplate: true` + `itemsPath: "variableHolder>/varInterfaceList"` |
| Row cells empty despite items binding | Cells used `{name}` etc. — default model relative; binding context is in `variableHolder` model | Changed all cells to `{variableHolder>name}` prefix |
| AddInterfaceBackBtn aggregation crash | Button/Title placed directly in Page `customHeader` aggregation (requires Bar) | Added `AddInterfaceHeaderBar` (Bar) to customHeader; moved Button to contentLeft, Title to contentMiddle |

### Platform behaviour confirmed (Simplifier 2606.30)

Investigated why `BuiltInEvent/appInit` and `ScreenEvent/show`/`afterInit`/`beforeFirstShow` stories do not fire on page load. Full investigation documented in `docs/simplifier-conventions.md` → "Process Story Event Trigger Behaviour".

Key findings:
- **Only `ItemEvent` (widget events) compile executable code** — all ScreenEvent and BuiltInEvent stories produce empty stubs in lifecycle hooks
- `builtInEvent_appInit` IS compiled correctly in Globals with the right BO call, but `bindBuiltInEvents` in Component only wires orientation/online/offline — never calls `appInit`
- Manual call `globals.builtInEvent_appInit.call(ctrl)` works and fires `executeBO` correctly
- App shows 13 interfaces with all data when Refresh is clicked (widget event → works)
- Auto-load on startup is a known platform limitation in this version

### Current state

- App fully functional: tiles show correct counts (13/10/2/1), table shows 13 interfaces with all fields
- Data loads on Refresh button press and filter changes
- Auto-load on initial startup not yet working (platform limitation)

---

## [2026-06-28] sInterfaceCockpit schema + persistent layer migration

**Changed:** Created `sInterfaceCockpit` schema and migrated `bInterfaceStatus` from hardcoded mock data to real SQL persistence  
**Why:** MBI-2055 — production-ready persistence layer for InterfaceCockpit; all InterfaceCockpit connectors now write to `sInterfaceCockpit`

### Schema: `sInterfaceCockpit`
| Table | Purpose |
|---|---|
| `es_interfaces` | Interface master data (id, name, system, technology, direction, scope, business_function, active) |
| `es_interface_status` | Current operational status per interface (1:1 with es_interfaces) |
| `es_interface_calls` | Full call/transaction audit log with OData forwarding status |
| `es_license_plate_checks` | Kennzeichen API check results + OData forwarding tracking |

### Business Objects updated
| BO | Function | Change |
|---|---|---|
| `bInterfaceStatus` | `getInterfaceStatuses` | Replaced hardcoded array with JOIN query on `es_interfaces` + `es_interface_status` |
| `bInterfaceStatus` | `getInterfaceDetail` | Replaced in-memory filter with direct SQL query by `interfaceId` |
| `bInterfaceStatus` | `triggerRetry` | Replaced stub with read-before-write UPDATE + audit INSERT to `es_interface_calls` |
| `bLicensePlateCheck` | `checkAndSave` | Changed write target from `sMockData` to `sInterfaceCockpit.es_license_plate_checks` |
| `bLicensePlateCheck` | `saveCheckResult` | Changed write target from `sMockData` to `sInterfaceCockpit.es_license_plate_checks` |
| `bLicensePlateCheck` | `listCheckResults` | Changed read target from `sMockData` to `sInterfaceCockpit.es_license_plate_checks` |

### New Business Object: `bLicensePlateCheck`
| Function | Purpose |
|---|---|
| `checkAndSave` | Calls `kennzeichen` connector, saves result to `sInterfaceCockpit` |
| `saveCheckResult` | Manual save of a check result to `sInterfaceCockpit` |
| `listCheckResults` | Returns recent check results ordered by `checked_at` DESC |

### New Client BO: `cLicensePlateCheck`
| Function | Purpose |
|---|---|
| `checkAvailability` | Delegates to `bLicensePlateCheck.checkAndSave` with plate components as input |

### Seed data
12 interface records seeded into `es_interfaces` + `es_interface_status` (LC-01..LC-08, SAP-01/02, OFI-01/02)

---

## [2026-06-28] InterfaceCockpit RBAC — AppInterfaceCockpit BO permission fix

**Changed:** Added `bInterfaceStatus execute + view` permissions to the auto-generated `AppInterfaceCockpit` role  
**Why:** Simplifier checks BO execute permission against the app-bound auto-generated role, not just custom roles. Marco received "Missing permissions to execute business object bInterfaceStatus" because `AppInterfaceCockpit` only had App permissions.

| Role | Permission added |
|---|---|
| `AppInterfaceCockpit` (auto-generated) | Business Object: bInterfaceStatus / Execute: true |
| `AppInterfaceCockpit` (auto-generated) | Business Object: bInterfaceStatus / View: true |

**Affected user:** marco.spoel@t-systems.com — already a member of `AppInterfaceCockpit` (auto-assigned at deploy time).

---

## [2026-06-28] Security agent infrastructure

**Changed:** Added `security` and `security-tester` agents; created `docs/security/` documentation scaffold  
**Why:** Enforce OWASP Top-10, RBAC, and Deutsche Telekom PSA requirements across all project artifacts

- Added `agents/security.md` — RBAC enforcement, OWASP Top-10 patterns, CVE research, PSA assistance
- Added `agents/security-tester.md` — API/backend security test execution (access control, injection, headers, SSRF)
- Created `docs/security/` with: overview, rbac, psa, components, cve-watchlist, logging-policy, test-checklist
- Updated `AGENTS.md` and `CLAUDE.md` with security agent orchestration triggers and patterns
- `security` runs parallel with all builders; `security-tester` runs after builders before `code-reviewer`
- UI security tests (hidden controls, DOM leakage) remain with `ui-tester` to preserve role separation

---

## [2026-06-28] InterfaceCockpit RBAC — InterfaceAdmin & InterfaceOperator roles

**Changed:** Created two Simplifier roles for RBAC on the InterfaceCockpit app  
**Why:** MBI-2055 security requirement — role-based access control for cockpit operators vs. admins

### Roles created

| Role | ID (prefix) | Purpose |
|---|---|---|
| `InterfaceAdmin` | `0C697616...` | Full access — view, execute, edit app + BO |
| `InterfaceOperator` | `C6F890F1...` | Read-only — execute + view app + BO only |

### InterfaceAdmin permissions
| Resource | Characteristic | Value |
|---|---|---|
| App: InterfaceCockpit | execute | true |
| App: InterfaceCockpit | edit | true |
| App: InterfaceCockpit | view | true |
| Business Object: bInterfaceStatus | execute | true |
| Business Object: bInterfaceStatus | view | true |

**Assigned users:** `marco.spoel@t-systems.com`

### InterfaceOperator permissions
| Resource | Characteristic | Value |
|---|---|---|
| App: InterfaceCockpit | execute | true |
| App: InterfaceCockpit | view | true |
| App: InterfaceCockpit | edit | false |
| App: InterfaceCockpit | delete | false |
| App: InterfaceCockpit | releasemeta | false |
| Business Object: bInterfaceStatus | execute | true |
| Business Object: bInterfaceStatus | view | true |
| Business Object: bInterfaceStatus | edit | false |
| Business Object: bInterfaceStatus | delete | false |

**Assigned users:** none (assign operators as needed)

---

## [2026-06-27] InterfaceCockpit app — MBI-2055 / LC-INT-08

**Changed:** Created InterfaceCockpit Simplifier app for real-time interface monitoring  
**Why:** WP11d delivery — Simplifier Low-Code interface cockpit (LC-INT-08)  
**Agents involved:** simplifier-app-builder, simplifier-bo-developer

### Artifacts created

**App:** InterfaceCockpit (`5D18C05B...`)
- 3 screens: Cockpit (start), InterfaceDetail, Popups
- 8 app variables: varInterfaceList, varSelectedInterface, varSelectedInterfaceId, varFilterStatus, varFilterScope, varTotalCount, varErrorCount, varWarningCount
- 7 process stories fully wired with BO inputs/outputs and variable mappings

**Business Object:** `bInterfaceStatus`
- `getInterfaceStatuses(filterStatus, filterScope)` — filtered list + counts
- `getInterfaceDetail(interfaceId)` — single interface full record
- `triggerRetry(interfaceId)` — manual retry trigger

**Data Types:**
- `sInterfaceStatus` — 16-field struct (id, name, system, technology, direction, scope, businessFunction, status, availability, lastSuccess, lastFailure, errorCode, errorReason, messageCount, retryCount, retryStatus)
- `sInterfaceStatus_col` — collection of sInterfaceStatus

### Process story mapping summary

| Story | Input mapping | Output mapping |
|---|---|---|
| [Cockpit] Load Data | — | interfaces→varInterfaceList, counts→vars |
| [Cockpit] Refresh | varFilterStatus→filterStatus, varFilterScope→filterScope | interfaces→varInterfaceList, counts→vars |
| [Cockpit] Filter (status) | FilterStatusSelect.selectedKey→filterStatus | interfaces→varInterfaceList, counts→vars |
| [Cockpit] Filter (scope) | FilterScopeSelect.selectedKey→filterScope | interfaces→varInterfaceList, counts→vars |
| 00.3 Navigation | — | Navigate to InterfaceDetail |
| [InterfaceDetail] Load Data | InterfaceRow.id→interfaceId | interface→varSelectedInterface, interfaceId→varSelectedInterfaceId |
| [InterfaceDetail] Trigger Retry | varSelectedInterfaceId→interfaceId | — |
| [InterfaceDetail] Back | — | Navigate back to Cockpit |

---

## [2026-06-27] Initial agent infrastructure

**Changed:** Project setup — agents, MCP config, documentation scaffold
**Why:** Bootstrap Claude Code agent infrastructure for Simplifier development
**Agents involved:** (initial setup, no agents run yet)

- Added 11 specialized agents: planner, simplifier-app-builder, simplifier-bo-developer, simplifier-connector-manager, simplifier-workflow-builder, ui5-developer, figma-inspector, ui-tester, debugger, code-reviewer, retrospective, doc-writer
- Configured 5 MCP servers: simplifier-mcp, ui5-mcp-server, figma, playwright, chrome-devtools
- Created docs scaffold: architecture.md, setup.md, changelog.md, artifacts/
