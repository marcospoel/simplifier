# Project Plan — WP11d (Low-Code Interfaces) & WP13 (Interface Development)

**Project:** ReFIT — Replace Fleet IT  
**Scope:** Interface delivery for Low-Code/Simplifier and SAP/OFI integration  
**Deployment target:** GCP  
**Sources:** Engineering tasks document + Confluence interface requirements  
**Last updated:** 2026-06-27

---

## 1. Guiding Principles

- **TARDIS first** — all Low-Code → SAP / internal system calls route through StarGate; no direct connections
- **SAP Clean Core** — no Z-tables in production; ABAP Cloud only; standard APIs only
- **Single Version of the Truth** — SAP/OFI is the authoritative data source; Low-Code caches only
- **GCP deployment** — Low-Code runtime on GCP; deployment via CI/CD pipeline
- **Loose coupling** — interface unavailability must not crash the application; graceful degradation required
- **Contract-first** — interface contracts (OpenAPI / OData EDMX / WSDL) are defined and approved before implementation starts

---

## 2. Phases Overview

| Phase | Focus | Weeks | Key gate |
|---|---|---|---|
| **P0 — Mobilisation** | Stakeholders, inventory, workshops | 1–4 | Signed workshop minutes + dependency register |
| **P1 — Architecture & Contracts** | Architecture, data model, interface contracts, TARDIS onboarding | 3–8 | Approved architecture document + approved interface contracts |
| **P2 — Foundation Build** | Connector library, TARDIS registration, network access, SAP customising | 7–12 | All connectors deployable; SAP customising in Dev (XF4) |
| **P3 — Interface Implementation** | All individual interface integrations in parallel streams | 10–20 | All interfaces passing developer tests in Dev |
| **P4 — Cockpit & Resilience** | Interface Cockpit app, graceful degradation, notifications | 14–20 | Cockpit live showing all interface statuses |
| **P5 — Testing** | Automated tests, integration tests, availability tests | 18–22 | All critical/high defects resolved |
| **P6 — Release** | GCP deployment, operations manual, transport package, release approval | 21–24 | Go-live approval granted |

Phases P2, P3, and P4 run in parallel once P1 is complete.

---

## 3. Phase P0 — Mobilisation (Weeks 1–4)

### P0-T1 · Interface Stakeholder Matrix `LC-INT-01`

**Owner:** Solution Architect + Project Manager  
**Duration:** Week 1–2  
**Depends on:** —

**Tasks:**
1. Identify all systems: SAP/OFI, Low-Code/Simplifier, e:c:car, Driver App, Interface Cockpit, Fleetpoint, AskT, HR Data Highway, fuel card providers, KBA, DAT, High Mobility, Abiscon
2. For each system: name the system owner, business owner, technical owner, security contact, test contact, and escalation path
3. Confirm communication channel per interface (team email, Jira project, Confluence space)
4. Identify the **17 Low-Code-scoped interfaces** (see `docs/requirements/interfaces-wp11d-wp13.md`) and assign owners

**Deliverable:** `docs/artifacts/interface-stakeholder-matrix.md`  
**Acceptance criteria:** Every interface has named owner, named test contact, and agreed communication channel

---

### P0-T2 · Interface Documentation Inventory `SAP-INT-01`

**Owner:** Solution Architect  
**Duration:** Week 1–2  
**Depends on:** —

**Tasks:**
1. Collect existing documentation: DT IT interfaces, SAP FIT interface programs, current YZA IDoc specs, eccar interface specs, fuel-card provider specs, external partner specs
2. Classify each interface: `reuse` | `adapt` | `implement-new` | `delivered-by-Olympus` | `delivered-by-Low-Code` | `delivered-by-fleet-AddOn` | `out-of-scope`
3. Cross-reference with the 55-entry interface master list in `docs/requirements/interfaces-wp11d-wp13.md`

**Deliverable:** `docs/artifacts/interface-inventory.md`  
**Acceptance criteria:** Every interface has a classification and a reference to existing documentation (or `none`)

---

### P0-T3 · Requirements Workshops `LC-INT-02` + `SAP-INT-02`

**Owner:** Solution Architect (facilitator) + Business Analyst  
**Duration:** Weeks 2–4 (6 half-day workshops + Low-Code interface workshops with FO, FpM, TAM, MFZ)  
**Depends on:** P0-T1, P0-T2

**Workshop agenda per session:**
- Functional requirements per interface group
- Data objects and field-level structure
- Process triggers (event that initiates the interface call)
- Error scenarios and retry expectations
- Availability/SLA requirements
- Security requirements (auth method, data sensitivity)
- Open questions → owners + due dates

**Workshop sessions (suggested grouping):**
| Session | Interfaces covered |
|---|---|
| WS-1 | Calculation tool, e:c:car ↔ Low-Code (12a, 12b, 37a, 37b, 37c, 37d) |
| WS-2 | SAP HR interfaces (20a–20e, HR Data Highway) |
| WS-3 | Fleetpoint + assessor + freight forwarder (42, 43, 45, 46) |
| WS-4 | Identity / roles (CAIMAN, Entra ID) (21, 22) |
| WS-5 | Company Bike, replacement mobility, chatbot (1, 2, 29, 30, 31, 32, 34) |
| WS-6 | SAP YZA IDoc programs, EDI, manufacturer data (3–11, 28, 36–41) |

**Deliverable:** Signed workshop minutes per session + requirements backlog in Jira  
**Acceptance criteria:** Every requirement has: source system, target system, data object, trigger, protocol, security requirement, acceptance criterion, and named owner

---

### P0-T4 · Dependency and Risk Register `SAP-INT-20`

**Owner:** Solution Architect + Project Manager  
**Duration:** Week 1 (initial), maintained throughout  
**Depends on:** —

**Track the following customer/DT IT dependencies:**
- SAP/OFI infrastructure readiness per environment (XF4, ZF4, AF4)
- API information from Abiscon (Process API contracts)
- TARDIS network / firewall clearance requests
- API keys, OAuth credentials, certificates (Iris client_id / client_secret per environment)
- HR Data Highway API access
- KBA GKS large-customer access (XAdES certificate)
- Shell, Aral, Total, UTA API credentials
- GCP environment provisioning
- DT IT effort contribution (≥90 person-days assumed)

**Deliverable:** `docs/plans/dependency-risk-register.md` (maintained live)  
**Acceptance criteria:** Every dependency has: owner, due date, impact on delivery, mitigation, current status

---

## 4. Phase P1 — Architecture & Contracts (Weeks 3–8)

### P1-T1 · Low-Code Integration Architecture `LC-INT-03`

**Owner:** Solution Architect  
**Duration:** Weeks 3–6  
**Depends on:** P0-T2, P0-T3 (WS-1 at minimum)

**Tasks:**
1. Design the full Low-Code integration topology: Simplifier → TARDIS StarGate → SAP/Abiscon/external systems
2. Define which Simplifier artifact tier handles each interface (Connector, BO, Workflow, Scheduled Job)
3. Map all 17 Low-Code-scoped interfaces to Simplifier artifact types
4. Define the TARDIS zone for each integration (CETUS for on-premise/CaaS; check GCP zone)
5. Design the Interface Cockpit architecture: data model, update mechanism, alert routing
6. Design error-handling topology: retry queues, dead-letter handling, fallback messages, alert paths
7. Document security boundaries: which data classifications pass through which interfaces

**Deliverable:** `docs/designs/low-code-integration-architecture.md`  
**Acceptance criteria:** Architecture shows SAP/OFI, Low-Code, e:c:car, Driver App, Interface Cockpit, DT network, security boundaries, protocols, error-handling flows, and TARDIS routing

---

### P1-T2 · Canonical Data Source Concept `LC-INT-04`

**Owner:** Solution Architect  
**Duration:** Weeks 4–6  
**Depends on:** P0-T3, P1-T1

**Tasks:**
1. For every Low-Code data object, document:
   - Authoritative source system (SAP = truth for financial/HR; Abiscon = truth for fleet)
   - Refresh logic (event-driven vs. scheduled polling vs. on-demand)
   - Transformation / mapping rules
   - Consuming application(s)
   - Allowed caching duration in Low-Code
2. Define which data Low-Code is allowed to persist (intermediate only) vs. must immediately pass to SAP
3. Align with ADR-001 (preliminary calculation persistence: Abiscon OData preferred)

**Deliverable:** `docs/designs/canonical-data-model.md`  
**Acceptance criteria:** Every Low-Code data object has: authoritative source, refresh logic, transformation rule, consuming app, and caching policy

---

### P1-T3 · Interface Contracts `LC-INT-05` + `SAP-INT-03`

**Owner:** Solution Architect  
**Duration:** Weeks 5–8  
**Depends on:** P0-T3 (all workshops), P1-T1, P1-T2

This is the **critical deliverable** — implementation cannot start without approved contracts.

**Tasks per interface:**
1. Choose contract format:
   - REST → OpenAPI 3.0 spec (`.yaml`)
   - OData → EDMX metadata document
   - SOAP → WSDL
   - SAP IDoc/BAPI → SAP documentation + field mapping table
2. Define for each interface:
   - Request / response schema
   - Mandatory vs. optional fields
   - Field-level mapping (source field → target field, type, transformation)
   - Authentication method (OAuth2 via TARDIS / API key / certificate)
   - Error response structure (HTTP status codes, error payload format)
   - Timeout behaviour (client timeout, server SLA)
   - Retry behaviour (retry count, backoff strategy)
   - Version number and versioning strategy
3. Get sign-off from interface provider team before implementation starts

**Priority order for contract delivery:**
| Priority | Interfaces | Reason |
|---|---|---|
| P1 (weeks 5–6) | 12a/b (Calc tool), 37a (JATO), HR Data Highway | Blocks core ordering flow |
| P2 (weeks 6–7) | 42/43 (Fleetpoint), 21 (Identity/CAIMAN), 20a–20e (HR) | Blocks key Low-Code flows |
| P3 (weeks 7–8) | 1/2 (Company Bike), 34 (AskT), 45/46 (Assessor/Spedition) | Secondary flows |
| P4 (weeks 8+) | 26a/b (Vermarktungsportal), 30–32 (replacement mobility) | Pending scope decisions |

**Deliverables:**
- `docs/designs/contracts/` — one contract file per interface
- `docs/designs/interface-contract-pack.md` — index with sign-off status per interface

**Acceptance criteria:** Each interface contract has: schema, field mapping, auth, error structure, timeouts, retry rules, version number, and provider sign-off

---

### P1-T4 · TARDIS Onboarding & Network Access `LC-INT-06`

**Owner:** Solution Architect + DevOps / Network engineer  
**Duration:** Weeks 5–8  
**Depends on:** P1-T1

**Tasks:**
1. Self-onboard Simplifier application in **MissionControl** — create Hub + Team (`mbi--refit--simplifier` or equivalent)
2. Create `rover.yaml` for the Simplifier Low-Code application as TARDIS **consumer**:
   - Subscribe to each required API basePath (one entry per interface)
   - Zone: confirm whether CETUS or GCP zone applies
   - `clientAuthMethod: basic`
3. Apply `rover.yaml` via CI/CD pipeline (GitLab) for each environment: Playground → Preprod → Prod
4. Retrieve `client_id` and `client_secret` per environment from MissionControl
5. Store credentials in GCP Secret Manager (not in code or .env files)
6. Coordinate firewall clearance requests for each required endpoint
7. Validate VPN/proxy settings from GCP runtime to TARDIS endpoints
8. Test connectivity: Low-Code runtime → TARDIS → each target system
9. Store and rotate SSL certificates, API keys per agreed security standard

**Deliverable:** 
- `docs/designs/tardis-rover.yaml` — Simplifier consumer registration
- Connectivity test report per environment
- Credentials stored in GCP Secret Manager (evidence screenshot)

**Acceptance criteria:** Connection tests pass from GCP Low-Code runtime to every required endpoint; all secrets and certificates stored in GCP Secret Manager

---

## 5. Phase P2 — Foundation Build (Weeks 7–12)

### P2-T1 · Reusable Connector Library `LC-INT-07`

**Owner:** Simplifier developer (simplifier-connector-manager agent)  
**Duration:** Weeks 7–12 (builds incrementally as contracts are approved)  
**Depends on:** P1-T3 (contract per interface), P1-T4 (TARDIS access)

**Tasks per connector:**
1. Create Simplifier Connector with:
   - Base URL = TARDIS StarGate endpoint for the zone/environment
   - Auth: OAuth2 Login (Iris token endpoint, `client_id`, `client_secret` from GCP Secret Manager)
   - Environment-specific configurations (Dev XF4, QA ZF4, Prod AF4)
2. Create Connector Calls for each operation defined in the interface contract
3. Implement token caching in a shared BO (`TokenCacheBO`) — cache token until `exp`, refresh on 401
4. Create a wrapper BO per system that encapsulates all calls to that system
5. Apply consistent naming: `{SystemName}Connector`, `{SystemName}BO`, connector calls as `{Operation}_{Entity}`
6. Document each connector in `docs/artifacts/connectors.md`

**Priority build order:**
| Week | Connectors |
|---|---|
| 7–8 | AbisconProcessApiConnector (Process start/read/execute) |
| 8–9 | CalculationToolConnector (OData, preliminary calc) |
| 9–10 | HrDataHighwayConnector (REST, employee data) |
| 10–11 | FleetpointConnector (appointment + yard management) |
| 11–12 | IdentityConnector (CAIMAN/Entra ID roles) |
| 12+ | CompanyBikeConnector, AssessorConnector, AskTConnector |

**Deliverable:** Reusable connector library in Simplifier; `docs/artifacts/connectors.md` updated  
**Acceptance criteria:** Connectors are reusable, consistently named, versioned, documented, and callable from workflows/apps without duplicating auth/routing logic

---

### P2-T2 · SAP Customising `SAP-INT-04`

**Owner:** SAP developer  
**Duration:** Weeks 7–9  
**Depends on:** P1-T3 (SAP interface contracts)

**Tasks:**
1. Configure SAP/OFI settings required by the technical implementation concept (SAP-INT-03)
2. Transport customising to Dev environment (XF4)
3. Developer verification test

**Deliverable:** SAP customising transport requests (XF4)  
**Acceptance criteria:** Customising is in XF4 and verified by developer test

---

### P2-T3 · SAP Inbound Processing Programs `SAP-INT-05` + `SAP-INT-06`

**Owner:** SAP developer / ABAP developer  
**Duration:** Weeks 8–14  
**Depends on:** P2-T2

**Tasks:**
1. Develop / adapt SAP programs for inbound processing:
   - YZA_XV01: vehicle master data + transactional data creation; error logging + restart points
   - YZA_XV02: restart program for YZA_XV01 (BAPI `YZA_MP_IMP_ORDER`)
   - Adapt existing SAP FIT programs to S/4HANA + SAP/OFI
2. Each program must include: validation, transformation, target table persistence, application logging, error handling, reprocessing support
3. Invalid records rejected with clear error messages, without corrupting valid records

**Deliverable:** SAP development transport requests (XF4)  
**Acceptance criteria:** Inbound processing works end-to-end; invalid records rejected cleanly; reprocessing works

---

## 6. Phase P3 — Interface Implementation (Weeks 10–20)

Interfaces are implemented in parallel streams once their contracts are approved and connectors are ready.

### Stream A — Vehicle Ordering & Configuration

**Interfaces:** 12a, 12b, 37a, 37b, 37c, 37d  
**Owner:** simplifier-app-builder + simplifier-bo-developer  
**Weeks:** 10–14

| Task | Interface | Deliverable |
|---|---|---|
| A1 | 12a — Calculation tool → APC (OData) | BO: `CalculationBO.sendCalculationData()` |
| A2 | 12b — e:c:car → Low-Code calculator (REST) | BO: `EccarCalculationBO.calculateRate()` |
| A3 | 37a — JATO code (e:c:car → Low-Code) | BO: `ManufacturerBO.receiveJatoCode()` |
| A4 | 37d — Buildability change → tax recalc | BO: `ManufacturerBO.recalculateTaxation()` |

**SAP-INT-08** (eccar ↔ Low-Code integration): covers A1–A4 together  
**Acceptance criteria:** Vehicle configuration data can be exchanged and traced end-to-end

---

### Stream B — HR & Person Data

**Interfaces:** 20a, 20b, 20c, 20d, 20e, HR Data Highway  
**Owner:** simplifier-bo-developer + SAP HR developer (`SAP-INT-10`)  
**Weeks:** 10–16

| Task | Interface | Deliverable |
|---|---|---|
| B1 | HR Data Highway — employee data sync | BO: `HrDataHighwayBO.syncEmployeeData()` (scheduled job) |
| B2 | 20c — Terminate taxation → APC | BO: `TaxationBO.terminateTaxation()` + Workflow |
| B3 | 20d — GU registration / annual settlement → APC | BO: `GuBO.registerGu()` + batch job |
| B4 | 20e — GU calc fields → Low-Code | BO: `GuBO.getCalculationFields()` |
| B5 | **SAP-INT-11** — Geldwertmeldung | SAP: monetary benefit notification program |
| B6 | **SAP-INT-12** — Gehaltsbuchungen | SAP: salary posting interface |

**Acceptance criteria:** Each HR interface has confirmed field mapping, test data, successful inbound/outbound test, documented error handling

---

### Stream C — Fleetpoint & Logistics

**Interfaces:** 42, 43, 45, 46  
**Owner:** simplifier-bo-developer  
**Weeks:** 11–16

| Task | Interface | Deliverable |
|---|---|---|
| C1 | 42 — Fleetpoint appointment tool → APC | BO: `FleetpointBO.createAppointment()` + portal screen |
| C2 | 43 — Fleetpoint yard / inventory → APC | BO: `FleetpointBO.getVehicleLocation()` |
| C3 | 45 — Assessor → APC | BO: `AssessorBO.submitAssessment()` |
| C4 | 46 — Freight forwarder → APC | BO: `SpeditionBO.notifyFreightForwarder()` + portal |

**Acceptance criteria:** Data exchange works bidirectionally/unidirectionally as agreed; failure handled with cockpit visibility

---

### Stream D — Identity & Authorisation

**Interface:** 21  
**Owner:** simplifier-connector-manager + Platform engineer  
**Weeks:** 10–13

| Task | Deliverable |
|---|---|
| D1 — Entra ID integration (internal) | IdentityConnector: Entra ID OIDC flow → Low-Code role assignment |
| D2 — CAIMAN role mapping | BO: `IdentityBO.mapCaimanRoles()` — CAIMAN roles → Simplifier project roles |
| D3 — External user flow (Patrick to confirm) | OpenID / ICU integration (pending decision) |

**Acceptance criteria:** Internal users (Entra ID) get correct Low-Code roles; CAIMAN role changes propagate within agreed SLA

---

### Stream E — Manufacturer Data (`SAP-INT-13`)

**Interfaces:** 36a, 36b, 37c, 38, 39, 40a, 40b, 41  
**Owner:** SAP developer + simplifier-bo-developer  
**Weeks:** 12–18

| Task | Interface | Deliverable |
|---|---|---|
| E1 | 36a — Model codes, equipment, BLP, price changes | e:c:car integration (e:c:car-owned, coordinated) |
| E2 | 36b — Re-import at first registration | fleet integration (fleet-owned) |
| E3 | 37c — Buildability change → e:c:car config update | e:c:car process; Low-Code taxation recalc (37d) |
| E4 | 38/39 — Service data / service appointments | fleet integration via High Mobility (pending ROI decision) |
| E5 | 40a — Order confirmation + delivery dates | OFI/SAP — follows EDI (#28) |
| E6 | 41 — Recall notices | fleet: mass processing of OEM data file |

---

### Stream F — Fuel Cards (`SAP-INT-09`)

**Interface:** 27  
**Owner:** fleet AddOn developer  
**Weeks:** 12–17

| Task | Deliverable |
|---|---|
| F1 — Shell B2B: card order, status, transaction import | Shell B2B Mobility Card integration (via Abiscon Cloud hub) |
| F2 — Aral bp.com: fleet card | ARAL fleet card API integration |
| F3 — Total / UTA: fleet cards | Total + UTA integrations |
| F4 — Electric charging card data import | Charging card integration (scope TBD: Shell Recharge etc.) |

---

### Stream G — Company Bike (`SAP-INT-13` partial)

**Interfaces:** 1 (yza_xv10), 2 (yza_xv11), 29  
**Owner:** simplifier-bo-developer  
**Weeks:** 13–17 (pending Company Bike API details from Dirk L.)

| Task | Deliverable |
|---|---|
| G1 | yza_xv10: Upload bike order (OData → APC) | BO: `CompanyBikeBO.uploadBikeOrder()` |
| G2 | yza_xv11: Upload leasing bike (REST → APC) | BO: `CompanyBikeBO.uploadLeasingBike()` |

**Note:** Blocked until Company Bike API requirements confirmed (Dirk L., September 2026 follow-up)

---

### Stream H — EDI `SAP-INT-09` partial / `28`

**Interface:** 28  
**Owner:** OFI/SAP developer  
**Weeks:** 14–18

| Task | Deliverable |
|---|---|
| H1 | EDI BMW / VW / Carglass: migrate C-PeX → BTP eBilling | SAP BTP eBilling configuration (Juliane to confirm C-PeX replacement; target end 2026) |

---

### Stream I — Fleetpoint `SAP-INT-14` + Reporting `SAP-INT-16`

| Task | Interface | Deliverable |
|---|---|---|
| I1 | 42/43 Fleetpoint (SAP side) | SAP integration for appointment + inventory (see Stream C) |
| I2 | Reporting: Fleet addon reports + S/4HANA Embedded Analytics | CDS views for fleet reporting (Patrick: reporting workshop) |
| I3 | AskT chatbot (34) | Chatbot Factory RAG integration (Marc + Patrick decision pending) |

---

### Stream J — Workshop / Assessor `SAP-INT-15`

| Task | Deliverable |
|---|---|
| J1 — Cost estimates, incoming invoices | SAP PM module integration (follows MM-based process replacement) |
| J2 — Supplier self-disclosure + qualification | SAP supplier qualification integration (48) |
| J3 — audatec / TecRMI | Handed to fleet workshop; replace with DAT if confirmed |

---

## 7. Phase P4 — Cockpit & Resilience (Weeks 14–20)

### P4-T1 · Interface Cockpit Application `LC-INT-08`

**Owner:** simplifier-app-builder + simplifier-bo-developer  
**Duration:** Weeks 14–18  
**Depends on:** P2-T1 (connector library exists)

**Tasks:**
1. Define data model: `InterfaceStatus` struct — {interfaceId, systemName, status: UP|DOWN|DEGRADED, lastSuccessAt, lastFailAt, lastErrorMessage, retryStatus, affectedBusinessFunction}
2. Implement status collection: each connector BO writes status events to a `CockpitStatusBO`
3. Build the Cockpit screen (OpenUI5 1.96.40):
   - Status grid: one tile per interface, colour-coded (green/amber/red)
   - Detail panel: last success, last failure, error message, retry count, affected business function
   - Filter by scope (Low-Code / fleet / OFi) and status
4. Implement alert routing: cockpit failures trigger notifications to the relevant owner team
5. Build a health-check scheduled job that polls each interface's health endpoint

**Deliverable:** Interface Cockpit app in Simplifier; `docs/artifacts/apps.md` updated  
**Acceptance criteria:** Users can see whether each system/interface is available; failures are visible centrally and linked to technical details; cockpit updates within ≤5 minutes of a status change

---

### P4-T2 · Graceful Degradation & Resilience `LC-INT-09`

**Owner:** simplifier-bo-developer  
**Duration:** Weeks 15–18  
**Depends on:** P2-T1

**Tasks:**
1. Implement retry queue BO: failed calls stored with payload, retry count, next-retry timestamp
2. Implement circuit-breaker pattern in each connector BO: after N consecutive failures, mark interface DOWN and stop retrying until health check passes
3. Define fallback messages per interface ("Funktion vorübergehend nicht verfügbar")
4. Route all fallback/error states to the Cockpit status BO
5. Implement deferred processing: if an interface is down, queue the operation and process it when recovered

**Deliverable:** Error-handling and resilience implementation; test evidence  
**Acceptance criteria:** Simulated interface outage does not crash the application; user receives controlled message; cockpit shows the affected interface and status

---

### P4-T3 · Driver App Status Notifications `LC-INT-10`

**Owner:** simplifier-workflow-builder  
**Duration:** Weeks 16–18  
**Depends on:** P4-T1, P4-T2

**Tasks:**
1. Implement notification workflow: trigger on interface DOWN event from cockpit BO
2. Notify relevant functional area (fleet team / HR team / driver if applicable)
3. Implement recovery notification: trigger on interface RECOVERED event
4. Log all notifications with timestamp, recipient, and message content

**Deliverable:** Notification workflow in Simplifier  
**Acceptance criteria:** Outage and recovery notifications sent to correct audience; notifications are traceable in logs

---

## 8. Phase P5 — Testing (Weeks 18–22)

### P5-T1 · Automated Test Suite `LC-INT-11`

**Owner:** ui-tester agent + simplifier developer  
**Duration:** Weeks 18–20  
**Depends on:** P3 (all streams), P4

**Coverage required:**
- Critical connector calls (happy path + error path)
- Field-level mapping correctness
- Token caching and refresh logic
- Interface cockpit status updates on failure
- Outage notifications
- Retry queue processing
- Graceful degradation messages

**Deliverable:** Automated test suite (Playwright E2E + BO unit tests); test execution report  
**Acceptance criteria:** Critical flows, field mappings, error scenarios, cockpit updates, and notifications covered

---

### P5-T2 · Availability & Integration Tests `LC-INT-12`

**Owner:** Solution Architect + test team  
**Duration:** Weeks 20–22  
**Depends on:** P5-T1; all interfaces deployed to QA (ZF4)

**Tests per interface:**
- Connectivity from GCP runtime to TARDIS to target system
- Authentication (token acquisition and validation)
- Happy-path end-to-end call
- Error response handling (simulate 400, 401, 500, timeout)
- Retry behaviour
- Cockpit status reflects outcome correctly

**Deliverable:** Availability and integration test report  
**Acceptance criteria:** All interfaces meet agreed availability, response time, auth, and error-handling expectations before production release

---

### P5-T3 · SAP Developer Tests & Defect Fix `SAP-INT-17`

**Owner:** SAP developer  
**Duration:** Weeks 20–22

**Deliverable:** Developer test evidence and defect log  
**Acceptance criteria:** All critical and high defects resolved or formally accepted; no unresolved defect blocks release

---

## 9. Phase P6 — Release (Weeks 21–24)

### P6-T1 · GCP Deployment Package `LC-INT-13`

**Owner:** DevOps / Simplifier platform engineer  
**Duration:** Weeks 21–22  
**Depends on:** P5

**Tasks:**
1. Package Simplifier Low-Code components (apps, BOs, connectors, workflows) as a deployable artifact
2. Document GCP deployment runbook:
   - Prerequisites (GCP project, Secret Manager secrets, TARDIS credentials)
   - Deployment steps
   - Post-deployment smoke tests
   - Rollback procedure
3. Execute deployment to QA, verify, then production
4. CI/CD pipeline configured for repeatable deployment

**Deliverable:** GCP deployment package + runbook  
**Acceptance criteria:** Deployment can be repeated in test and production using documented steps; rollback procedure tested

---

### P6-T2 · SAP Transport & Deployment Package `SAP-INT-18`

**Owner:** SAP developer + Basis  
**Duration:** Weeks 21–22

**Each transport includes:** content description, dependency order, target environment, test evidence, rollback note  
**Transport route:** XF4 → ZF4 → AF4

**Deliverable:** Transport and deployment package  
**Acceptance criteria:** Every transport has content description, dependency, target environment, test evidence, and rollback note

---

### P6-T3 · Operations Manual `LC-INT-14`

**Owner:** Solution Architect + doc-writer agent  
**Duration:** Weeks 22–23

**Manual must cover:**
- Monitoring: which Grafana / Raccoon dashboards to watch per interface
- Cockpit interpretation: status codes, thresholds, alert escalation
- Known error codes and resolution steps per interface
- Restart / retry procedures (how to manually trigger the retry queue)
- Certificate renewal: Iris client secret rotation, KBA XAdES certificate renewal
- API key renewal: Shell, Aral, Total, UTA, DAT, High Mobility
- Escalation contacts per interface (from P0-T1 stakeholder matrix)

**Deliverable:** `docs/operations/interface-operations-manual.md`  
**Acceptance criteria:** Manual includes monitoring steps, cockpit interpretation, known error codes, restart/retry procedures, credential renewal steps, and escalation contacts

---

### P6-T4 · Release Approval Support `SAP-INT-19`

**Owner:** Solution Architect + Project Manager  
**Duration:** Week 23–24

**Release approval package includes:**
- Implementation concept summary
- Test evidence (P5-T1, P5-T2, P5-T3 results)
- Transport and deployment list
- Open defect list with status
- Operational notes and known limitations
- GCP deployment runbook reference

**Deliverable:** Release approval support package  
**Acceptance criteria:** Approvers receive all listed materials; approval gate cleared within 1 project day

---

## 10. Key Dependencies Summary

| Dependency | Provider | Due | Impact if late |
|---|---|---|---|
| Abiscon Process API contracts | Abiscon | Week 5 | Blocks connector build (P2-T1) |
| TARDIS StarGate access + firewall clearance | TelIT / DT IT | Week 7 | Blocks all integration testing |
| Iris credentials (client_id / secret) per environment | TARDIS team via MissionControl | Week 7 | Blocks authentication |
| HR Data Highway API documentation | DT IT (Ilia / Monika) | Week 5 | Blocks HR interfaces |
| KBA GKS large-customer XAdES certificate | KBA / DT IT | Week 8 | Blocks vehicle registration |
| Shell / Aral / Total / UTA API credentials | Providers / Abiscon Cloud | Week 10 | Blocks fuel card integration |
| GCP environment provisioned for Low-Code runtime | GCP / Platform team | Week 7 | Blocks integration tests from GCP |
| Company Bike API requirements | Dirk L. / Company Bike partner | September 2026 | Blocks yza_xv10/xv11 |
| C-PeX → eBilling migration decision | Juliane / OFI team | End 2026 | Blocks EDI (#28) |
| AskT chatbot integration decision | Marc + Patrick | Week 4 | Blocks Stream I chatbot work |
| Fleetpoint e2e process definition in APC | Abiscon / TMS | Week 8 | Blocks Stream C |
| GU scope clarification | Marc | Week 4 | Blocks Stream B GU tasks |
| External identity/CAIMAN approach for externals | Patrick | Week 4 | Blocks Stream D external flow |
| DT IT effort contribution (≥90 person-days) | DT IT | Ongoing | Assumption; track in risk register |

---

## 11. Deliverables Registry

| ID | Deliverable | Location | Phase | Status |
|---|---|---|---|---|
| D-01 | Interface stakeholder matrix | `docs/artifacts/interface-stakeholder-matrix.md` | P0 | — |
| D-02 | Interface documentation inventory | `docs/artifacts/interface-inventory.md` | P0 | — |
| D-03 | Workshop minutes (6 sessions) | Confluence / `docs/requirements/` | P0 | — |
| D-04 | Dependency & risk register | `docs/plans/dependency-risk-register.md` | P0 | — |
| D-05 | Low-Code integration architecture | `docs/designs/low-code-integration-architecture.md` | P1 | — |
| D-06 | Canonical data model | `docs/designs/canonical-data-model.md` | P1 | — |
| D-07 | Interface contract pack | `docs/designs/contracts/` + index | P1 | — |
| D-08 | TARDIS rover.yaml + connectivity evidence | `docs/designs/tardis-rover.yaml` | P1 | — |
| D-09 | Reusable connector library | Simplifier + `docs/artifacts/connectors.md` | P2 | — |
| D-10 | SAP customising transports (XF4) | SAP TMS | P2 | — |
| D-11 | SAP development transports (XF4) | SAP TMS | P2/P3 | — |
| D-12 | Stream A–J implementations | Simplifier + SAP | P3 | — |
| D-13 | Interface Cockpit application | Simplifier + `docs/artifacts/apps.md` | P4 | — |
| D-14 | Resilience / graceful degradation | Simplifier BOs | P4 | — |
| D-15 | Notification workflows | Simplifier workflows | P4 | — |
| D-16 | Automated test suite + report | Playwright / test runner | P5 | — |
| D-17 | Availability & integration test report | `docs/tests/` | P5 | — |
| D-18 | SAP developer test evidence | SAP test logs | P5 | — |
| D-19 | GCP deployment package + runbook | `docs/operations/gcp-deployment-runbook.md` | P6 | — |
| D-20 | SAP transport + deployment package | SAP TMS | P6 | — |
| D-21 | Interface operations manual | `docs/operations/interface-operations-manual.md` | P6 | — |
| D-22 | Release approval package | `docs/plans/release-approval.md` | P6 | — |

---

## 12. Architect Checklist — Week-by-Week

| Week | Must have done |
|---|---|
| 1 | Start P0-T1 (stakeholder matrix), P0-T2 (doc inventory), P0-T4 (risk register) |
| 2 | Stakeholder matrix draft circulated; WS-1 scheduled |
| 3 | WS-1 complete (calc tool, e:c:car); start P1-T1 (architecture) |
| 4 | All 6 workshops complete; open points owner-assigned; AskT + GU + external CAIMAN decisions due |
| 5 | Architecture draft complete; P1 priority-1 interface contracts started (12a/b, 37a, HR Data Highway) |
| 6 | Priority-1 contracts sent for provider sign-off; TARDIS MissionControl onboarding started |
| 7 | P1 priority-2 contracts complete (Fleetpoint, Identity, HR); TARDIS rover.yaml applied to Playground |
| 8 | All P1 deliverables complete; P2 connector build started (AbisconProcessApiConnector first) |
| 9 | P3 Stream A + B started; SAP customising in XF4 |
| 10 | TARDIS access validated from GCP; P3 streams C + D started |
| 12 | Core connector library complete; P3 Stream E + F started |
| 14 | P4 Interface Cockpit design started; Streams A + B complete |
| 16 | Cockpit screen live in Dev; all streams in flight |
| 18 | P5 automated test suite writing starts alongside remaining P3 stream completion |
| 20 | All interfaces deployed to ZF4; availability test execution |
| 21 | All P5 tests passed; P6 GCP deployment package prepared |
| 22 | Production deployment executed; operations manual complete |
| 23–24 | Release approval package submitted and cleared |
