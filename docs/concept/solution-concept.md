# Solution Concept: DT Mobility Fleet Management Platform

**Sources:**
- [NEW Design / Implementation / Concept](https://wiki.telekom.de/spaces/MBI/pages/3691940648/NEW+Design+Implementation+Concept) — DT Mobility Confluence Space (Dec 18, 2025)
- [Architecture](https://wiki.telekom.de/spaces/MBI/pages/3216569738/Architecture) — DT Mobility Confluence Space

**Status:** Living document — reflects implementation in progress

---

## 1. Executive Summary

Deutsche Telekom is building a next-generation fleet management platform for its mobility division (DT Mobility). The solution replaces and extends legacy fleet management processes by integrating five core platform components into a coherent, end-to-end system. The architecture is deliberately split across best-of-breed components, each covering the domain it is best suited for, connected through well-defined interfaces.

The five components and their primary roles:

| Component | Primary Role |
|---|---|
| **SAP OFI** | Financial backbone — billing, invoicing, accounts payable/receivable |
| **Abiscon Fleet + INStra Add-ons** | Fleet domain platform — vehicle master data, processes, mobility cards, partner management |
| **Abiscon Process Cockpit** | Process orchestration — manages, monitors, and drives all fleet business processes |
| **Simplifier Low-Code Platform** | User interfaces and integration glue — driver-facing apps, calculation tools, data entry screens |
| **e:c:car Car Configurator** | Vehicle configuration and ordering — entitlement-driven car selection and configuration |

---

## 2. Component Roles

### 2.1 SAP OFI — Financial Backbone

SAP OFI (Operations, Financials & Invoicing) is the financial system of record. It handles all monetary transactions, contract billing, and accounts-payable processing in the fleet lifecycle.

**Key responsibilities:**
- Creation of billing documents for fleet sales contracts (SD — Sales & Distribution)
- Processing of incoming invoices (integration with OFI VIM OpenText for vendor invoice management)
- Equipment master data as SAP equipment objects
- SAP Archive Link integration for document storage
- Frame/purchasing contract management (SAP transaction ME31K)
- Maintenance planning (SAP transactions IP01 / IP11)
- EDI exchange with external partners (INTRASTAT reporting)
- Receiving cost centre framework data from external systems (EDMtoTMS interface)

**Integration position:** SAP OFI sits downstream of both Abiscon and Low-Code. Abiscon triggers billing document creation and equipment updates via SAP Standard interfaces. OFI does not drive fleet processes — it receives results and produces financial documents.

---

### 2.2 Abiscon Fleet + INStra Add-ons — Fleet Domain Platform

Abiscon is the core fleet management platform. It holds the authoritative master data for all fleet entities and executes the specialised fleet business logic that SAP standard does not cover. The INStra add-ons extend Abiscon with insurance management, mobility card management, and other fleet-specific capabilities.

**Key responsibilities:**

**Vehicle master data**
- Vehicle management (full vehicle lifecycle records)
- Vehicle classification: corporate group, brand, model line, model variant, wheel variant
- Vehicle enrollment to and offboarding from the fleet portfolio
- License plate management (KBA interface via GKS iKFZ SOAP)
- Vehicle sales portal and fleet sales contract management

**Person and partner data**
- Person management (drivers, contacts)
- Fleet business partner management (companies, customer hierarchy)
- Organisational unit management

**Mobility card management**
- Fleet card management (Shell, ARAL, UTA, Total mobility cards)
- Order mobility card, block mobility card
- Import mobility card movement/transaction data (fuel card import monitor)

**Insurance (INStra)**
- Insurance policy management
- Insurance claim management

**Process execution (via Process Cockpit)**
- Onboarding vehicle (triggers DAT SilverDAT for maintenance plans and VIN data, Shell B2B Mobility Card order)
- Offboarding vehicle
- Change customer/fleet category of vehicle
- Incoming traffic offence handling
- Maintenance process
- Incoming invoice processing
- Cash expenses ("Barauslagen")
- Request additional vehicles
- Person org data changes

**Administrative tools**
- Electronic file (smart.files) — document management at the browser/fleet UI
- Delivery dates Excel upload (Liefer-Avis)
- Generic data import monitor
- E-mail protocol
- Sales dashboard
- Maintenance bundling (Mischpult)
- Reports

**External interfaces operated by Abiscon:**
- Shell B2B Mobility Card (order card, update status, transaction data)
- ARAL fleet card API (bp.com)
- UTA, Total fleet cards
- KBA GKS iKFZ (SOAP) — vehicle registration authority
- DAT SilverDAT — maintenance plans and vehicle master data via VIN
- High Mobility Auto API — vehicle telematics data
- Driver licence data via EU Digital Wallet (TSI)
- SAP Standard interfaces: Billing Document Create, Equipment, Archive Link

---

### 2.3 Abiscon Process Cockpit — Process Orchestration Engine

The Abiscon Process Cockpit is the central process management and orchestration layer. It does not simply store data — it actively drives multi-step business processes by managing tasks, monitoring progress, triggering next steps, and coordinating between human actors and automated system calls.

**Key responsibilities:**
- Define and execute fleet business processes as structured workflows with discrete tasks and outcomes
- Assign tasks to people or roles (Task Cockpit — the to-do list for fleet agents)
- Trigger automated steps (e.g. call an external interface, create an SAP document)
- Expose process state to the Simplifier Low-Code front-end via the **Process API**:
  - `Process start` — initiate a new process instance
  - `Process read` — query current state of a process
  - `Process execute task` — advance a process by completing a task step
  - `Process Event Triggers` — scheduled job that polls for events and triggers process transitions

**Processes managed by the Process Cockpit** (selected):
- Vehicle enrollment to portfolio
- Onboarding vehicle (multi-step: VIN lookup → maintenance plan setup → mobility card order → SAP equipment creation)
- Offboarding vehicle
- Change customer/fleet category
- Incoming traffic offence
- Maintenance
- Incoming invoice
- Request additional vehicles
- Order/block mobility card
- Person org data change

**Integration position:** The Process Cockpit is the nerve centre. Simplifier Low-Code calls it to start and advance processes. SAP Scheduled Jobs trigger event-based process transitions. Abiscon's fleet UI (smart.process at Outlook) also surfaces tasks and processes for back-office users.

---

### 2.4 Simplifier Low-Code Platform — UIs and Integration Glue

Simplifier is the low-code development platform used to build all driver-facing and configuration UIs, as well as the integration logic connecting the Process Cockpit, e:c:car, and external systems. It is built on **OpenUI5 1.96.40**.

**Key responsibilities:**

**Driver and fleet user UIs (Low-Code UIs):**
- **Driver App** — mobile/browser app for drivers; covers entitlements, vehicle requests, documents
- **Preliminary calculation** — pre-lease cost calculation screen for fleet managers/HR
- **Calculation master data management** — maintenance of rate and cost master data
- **Ticket tool** — internal support/task tracking UI

**Integration and orchestration role:**
Simplifier Business Objects act as the server-side integration layer between UIs, the Abiscon Process Cockpit, e:c:car, and external services. Key integration flows:

| Flow | Direction | Purpose |
|---|---|---|
| Receive customer vehicle entitlement | PEGA → Low-Code | PEGA sends employee vehicle entitlement data; Low-Code stores and forwards to process |
| Calculate mobility rate | ec:car → Low-Code → Abiscon | ec:car triggers rate calculation; Low-Code fetches model variant cost data from Abiscon |
| Receive Referenzliste | ec:car → Low-Code → Abiscon | ec:car sends reference list; Low-Code starts Abiscon process |
| Receive configured car | ec:car → Low-Code → Abiscon | ec:car delivers car configuration; Low-Code executes Abiscon process task |
| Synchronise person data | Low-Code Scheduled Job | Keeps person data in sync across systems |
| Set person data for manual person creation | Low-Code Scheduled Job | Triggers Abiscon person creation process |
| Check for missing JATO data | Low-Code Scheduled Job | Data quality check on vehicle variant master data |
| AI-Scan incoming e-mails | Low-Code Scheduled Job (pending decision) | AWS Textract to extract text from incoming e-mails and route to Abiscon processes |

**Abiscon API calls made from Low-Code:**
- `fleet available bridging cars read`
- `fleet business partner` read/write
- `Person` management
- `Process start`, `Process read`, `Process execute task`
- `Get vehicle model variant data` (costs, wheels, fleet cards)
- `Get Products`
- `Get Order Vehicle Entitlement Categories`

**Other external interfaces from Low-Code:**
- AvD (roadside assistance trigger — pending TMS decision)
- City of Bonn — license plate registration
- Company bike partner
- AWS Textract (AI text extraction — pending architecture board)
- EmployeeDataAPI (Cayman — HR data)
- PEGA Monetization (create monetization record)

---

### 2.5 e:c:car Car Configurator — Vehicle Configuration and Ordering

e:c:car is the vehicle configuration and ordering portal. It presents the vehicle catalogue to eligible drivers based on their entitlement, allows them to configure their vehicle (model, colour, wheels, options), and submits the configured order into the fleet process.

**Key responsibilities:**
- Present the available vehicle models and variants to the driver based on entitlement category
- Drive the car configuration experience (model selection, options, colours, extras)
- Provide the reference list (Referenzliste) of orderable vehicles to the Low-Code platform
- Submit the configured car into the Low-Code → Abiscon Process Cockpit flow
- Trigger mobility rate calculation via the Low-Code platform

**Integration flows operated by e:c:car:**

| Interface | Direction | Description |
|---|---|---|
| Calculate mobility rate | ec:car → Low-Code | Requests rate calculation for a selected vehicle; Low-Code queries Abiscon for cost data |
| Receive Referenzliste | ec:car → Low-Code | Delivers the reference list of orderable vehicles; Low-Code starts Abiscon process |
| Receive configured car | ec:car → Low-Code | Delivers the final vehicle configuration; Low-Code executes the Abiscon ordering task |

**Integration position:** e:c:car is the entry point for the driver-facing ordering journey. It sits in front of the Low-Code platform, which translates the configuration events into Abiscon process calls. e:c:car does not connect to SAP OFI or Abiscon directly.

---

## 3. System Interaction Map

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              Driver / Employee                                   │
└──────────────────┬──────────────────────────────────┬───────────────────────────┘
                   │                                  │
                   ▼                                  ▼
        ┌──────────────────┐              ┌───────────────────────┐
        │   e:c:car        │              │  Simplifier Low-Code   │
        │  Car Configurator│◄────────────►│  (Driver App, Calc,   │
        │                  │  Rate calc,  │   Ticket Tool, etc.)  │
        └──────────────────┘  Ref-Liste,  └──────────┬────────────┘
                               Config              │
                                              ┌────▼─────────────────────────────┐
                                              │  Abiscon Process Cockpit          │
                                              │  (Process start / read /          │
                                              │   execute task / event triggers)  │
                                              └────┬─────────────────────────────┘
                                                   │
                              ┌────────────────────┼────────────────────┐
                              │                    │                    │
                              ▼                    ▼                    ▼
                   ┌──────────────────┐  ┌─────────────────┐  ┌────────────────┐
                   │  Abiscon Fleet + │  │    SAP OFI      │  │ External       │
                   │  INStra          │  │  (Billing, AP,  │  │ Systems        │
                   │  (Vehicle, Person│  │   Equipment,    │  │ (Shell, KBA,   │
                   │   Cards, Claims) │  │   Contracts)    │  │  DAT, PEGA,    │
                   └──────────────────┘  └─────────────────┘  │  High Mobility)│
                                                               └────────────────┘

All Simplifier → SAP / external traffic routes via TARDIS gateway (mandatory).
External API integrations route via Abiscon Cloud hub (see ADR-002).
```

---

## 4. Key Data Flows

### 4.1 Vehicle Ordering (end-to-end)
1. PEGA sends employee vehicle entitlement to **Simplifier Low-Code** (`Receive customer order vehicle entitlement`)
2. Driver opens **e:c:car** and selects a vehicle; e:c:car calls **Low-Code** to calculate the mobility rate (Low-Code queries Abiscon for model variant cost data)
3. e:c:car delivers the Referenzliste to **Low-Code**; Low-Code starts an **Abiscon Process** (vehicle ordering process)
4. Driver finalises configuration in e:c:car; configured car sent to **Low-Code**; Low-Code executes the Abiscon process task
5. **Abiscon Process Cockpit** drives the remaining steps: SAP equipment creation, mobility card order to Shell, KBA registration

### 4.2 Vehicle Onboarding
1. Fleet manager triggers onboarding in **Abiscon** or via **Simplifier**
2. **Abiscon** calls DAT SilverDAT to receive VIN-based vehicle master data and maintenance plans
3. **Abiscon** orders Shell B2B Mobility Card
4. **Abiscon** creates/updates SAP Equipment object in **SAP OFI**
5. Process Cockpit advances through all steps, assigning user tasks as needed

### 4.3 Incoming Invoice Processing
1. Invoice arrives; **SAP OFI VIM / OpenText** receives the document
2. **Abiscon Process Cockpit** picks up the invoice event and starts an incoming invoice process
3. Process tasks are assigned for approval/coding; completed via **Abiscon UI** or **Simplifier**
4. Approved invoice posted in **SAP OFI**

### 4.4 Person Data Synchronisation
1. **Simplifier Low-Code Scheduled Job** periodically calls the EmployeeDataAPI (Cayman/HR) or HR Data Highway REST API
2. Changed person data triggers Abiscon `Persons org data changed` process
3. Abiscon updates person records and propagates to SAP business partner if needed

---

## 5. Interface Overview

### Low-Code ↔ Abiscon Process API
| Interface | Purpose |
|---|---|
| `Process start` | Initiate a new Abiscon process instance |
| `Process read` | Query current state and tasks of a running process |
| `Process execute task` | Complete a task step to advance a process |
| `Get vehicle model variant data` | Retrieve costs, wheel variants, fleet card data for a model |
| `Get Products` | Retrieve product/vehicle catalogue |
| `Get Order Vehicle Entitlement Categories` | Retrieve entitlement categories for a person |
| `fleet available bridging cars read` | List available bridge/loaner vehicles |
| `fleet business partner` | Read/write fleet business partner records |
| `Person` | Read/write person records |

### e:c:car ↔ Low-Code
| Interface | Direction | Purpose |
|---|---|---|
| Calculate mobility rate | ec:car → Low-Code | Rate calculation for a vehicle selection |
| Receive Referenzliste | ec:car → Low-Code | Reference list of orderable vehicles |
| Receive configured car | ec:car → Low-Code | Final vehicle configuration for ordering |

### Abiscon ↔ External Systems
| System | Interface | Type | Notes |
|---|---|---|---|
| Shell | B2B Mobility Card (order, status update, transaction data) | REST | Card ordering + fuel transaction import |
| ARAL | Fleet Card API (bp.com) | REST | Fuel card management |
| UTA | Fleet card | REST | Fuel card management |
| Total | Fleet card | REST | Fuel card management |
| KBA | GKS iKFZ Großkundenschnittstelle | SOAP | Vehicle registration authority; **XAdES signature required** |
| DAT | SilverDAT (VIN lookup, maintenance plans, residual value) | REST | Vehicle master data enrichment |
| High Mobility | Auto API (telematics) | REST | Live vehicle telematics |
| TSI | EU Digital Wallet — driver licence data | — | Driver licence verification |
| BMW / VW / Carglass | EDI via C-PeX → eBilling BTP | EDI | Order/invoice exchange with OEM partners |
| EntraID / CAIMAN | Identity | REST | Authentication and employee identity |
| AskT | Internal chatbot | REST | Support routing |

### SAP OFI Interfaces
| Interface | Direction | Purpose |
|---|---|---|
| Billing Document Create (SD) | Abiscon → SAP | Create billing documents for fleet sales contracts |
| Equipment (PM) | Abiscon → SAP | Create/update SAP PM equipment records |
| Archive Link | → SAP | Document archiving (OpenText VIM) |
| IDoc YZA | SAP ↔ Abiscon | Core fleet data exchange IDocs |
| EDI / INTRASTAT | SAP ↔ External | Trade statistics reporting |
| Kostenstellenrahmen (EDMtoTMS) | External → OFI | Cost centre framework data |
| Frame / Purchasing Contract (ME31K) | Internal | SAP procurement — purchasing contracts |
| Maintenance Plan (IP01 / IP11) | Internal | Preventive maintenance scheduling |
| OFI VIM / OpenText | → SAP | Incoming vendor invoice management |

### Low-Code ↔ HR and Identity Systems
| Interface | Direction | Purpose |
|---|---|---|
| HR Data Highway | HR → Low-Code | SAP REST API — employee master data sync |
| EmployeeDataAPI (Cayman) | HR → Low-Code | HR data for person sync scheduled job |
| PEGA — Receive entitlement | PEGA → Low-Code | Employee vehicle entitlement data |
| PEGA Monetization | Low-Code → PEGA | Create monetization record for ordered vehicle |

---

## 6. Responsibilities by UI Category

| UI Type | Technology | Who uses it | Managed by |
|---|---|---|---|
| Abiscon fleet UIs (smart.files, Process Cockpit, Task Cockpit, Vehicle Mgmt, etc.) | Abiscon native | Fleet managers, back-office | Abiscon |
| smart.process at Outlook | Abiscon / Outlook Add-in | Back-office agents | Abiscon / TMS |
| Low-Code UIs (Driver App, Preliminary Calc, Calc Master Data, Ticket Tool) | Simplifier / OpenUI5 1.96.40 | Drivers, HR, fleet coordinators | TSI / Simplifier |
| e:c:car | e:c:car platform | Drivers ordering vehicles | DTSE / TMS |
| SAP Standard (ME31K, IP01, IP11) | SAP GUI / Fiori | SAP power users | TSI / SAP basis |
| Other (EU Digital Wallet App, Central BP, Central Material) | iOS/Android / SAP | Drivers, SAP users | Various |

---

## 7. Open Points and Decisions Pending

- **OFI VIM / OpenText integration** for incoming invoices — approach to be clarified
- **Warranty contract management** — on hold pending DTIT CR approval decision
- **AI-Scan incoming e-mails** (AWS Textract → Abiscon processes) — pending architecture board decision; alternative is smart.process AI at Outlook
- **Order confirmations upload** — UI/approach undecided
- **Vehicle sales portal** — potentially via Low-Code if TSI implements
- **AvD integration** (roadside assistance trigger) — pending TMS decision
- **License plate registration** (City of Bonn API) — pending TMS decision
- **Fleet sales contract management** — decision needed on how order-to-cash is handled
- **Insurance management** (policy + claims) — scheduled for Q4 2026 / Q1 2027
- **ADR-001 Gate A/B** — preliminary calculation persistence layer: decision point at Gate A (whether Abiscon OData path is feasible) and Gate B (whether Simplifier/GCP no-sync path is acceptable)

---

## 8. Delivery Timeline (indicative, from Confluence overview)

| Area | Target |
|---|---|
| Vehicle management (Abiscon) | Beginning Q2 2026 |
| Vehicle enrollment, Process Cockpit, Task Cockpit | Beginning Q2 2026 |
| Electronic file (smart.files) | Beginning Q2 2026 |
| Vehicle classification management | Beginning Q2 2026 |
| Person management, License plate management | Beginning Q3 2026 |
| Fleet card management | Beginning Q4 2026 |
| Insurance claim/policy management | Q4 2026 – Q1 2027 |

---

## 9. Architecture Principles

### 9.1 SAP Clean Core

The SAP OFI implementation follows the **Clean Core** principle mandated for the ReFIT programme:

- **No Z-tables in production** — all custom persistence must use standard SAP objects or BTP-approved extension patterns
- **ABAP Cloud only** — custom ABAP development (if any) is restricted to ABAP Cloud APIs; no classic ABAP extensions
- **Standard APIs only** — integration uses SAP-released OData, SOAP, or REST APIs; no direct database table reads from external systems
- **Consumer vs. AddOn namespace** — Abiscon Fleet operates as a SAP AddOn (own namespace); integrations from external systems use the Consumer namespace (standard released APIs only)

This constraint directly influences ADR-001: the preferred persistence layer for the preliminary calculation uses Abiscon's own OData APIs backed by SAP-standard objects, rather than Z-tables.

### 9.2 TARDIS Integration Gateway

**All integration between Simplifier and SAP (or any T-Systems internal system) must route through the TARDIS gateway.** Direct calls from Simplifier to SAP are not permitted.

- TARDIS is the mandatory API gateway for the Deutsche Telekom platform
- Simplifier Connectors must be configured to target TARDIS endpoints, not SAP directly
- TARDIS provides: authentication, rate limiting, observability, and policy enforcement

### 9.3 BTP Integration Layer

SAP Business Technology Platform (BTP) is used for:
- **Cloud Connector** — secure tunnel from BTP to on-premise SAP systems
- **Integration Suite** — message transformation and routing (where required)
- **eBilling** — EDI processing for OEM partner invoices (BMW, VW, Carglass via C-PeX)

### 9.4 Microservice Architecture

Abiscon Fleet on OFI follows a microservice architecture:
- Fleet domain logic is encapsulated in the Abiscon AddOn, not spread across SAP customising
- External API integrations are routed via the **Abiscon Cloud hub** (see ADR-002) — a Java/Quarkus application running on StackIT Germany
- The Abiscon Cloud hub provides: PSA (privacy and security assessment), observability (metrics, logs, traces), SLA management, runbooks, and an exit concept
- All processing in the Abiscon Cloud hub is **transient** — no data persistence; data sovereignty stays in SAP

---

## 10. SAP Environments

The ReFIT programme uses the following SAP landscape:

| Environment | System ID | Purpose | Notes |
|---|---|---|---|
| Sandbox | TF4 | Prototype / spike environment | Independent sandbox |
| Development | XF4 | Active development | Primary build environment |
| Quality Assurance | ZF4 | QA testing | Current integration QA |
| QA (new) | AF4 | Integration QA (from Jan 2027) | Replaces ZF4 for integration testing |
| Migration | WF4 | Data migration testing | Used for migration rehearsals |

**Transport route:** `XF4 → ZF4 → AF4`

Changes are developed in XF4, tested in ZF4, and promoted to AF4 for integration QA before production release. TF4 and WF4 are isolated and do not participate in the transport route.

---

## 11. Simplifier Integration Guidelines

The architecture Confluence pages define the following constraints for the Simplifier platform's role in the overall system:

### 11.1 Scope boundaries
- Simplifier's scope is **limited to filling gaps in the Abiscon Process Manager** that Abiscon cannot cover natively
- Simplifier does not replace Abiscon's own fleet UI — it complements it with driver-facing and calculation UIs
- Any new UI requirements that Abiscon can cover should be addressed in Abiscon, not duplicated in Simplifier

### 11.2 Integration gateway rule
- **Simplifier must always use TARDIS** as the integration gateway — no direct SAP calls
- Connector configurations must point to TARDIS-registered endpoints

### 11.3 Data sovereignty and persistence
- **Data sovereignty lies in SAP** — Simplifier is not the system of record for any fleet or financial data
- Simplifier may **cache data** for performance (e.g. vehicle model variants for the preliminary calculation screen)
- Persistent storage in Simplifier is **intermediate only** — data must be synchronised to SAP as soon as the relevant process step completes
- Long-term storage of business data in Simplifier without SAP synchronisation is not permitted (this is the constraint driving ADR-001)

### 11.4 Multi-platform delivery
- Simplifier UIs must support **portal / web / mobile** — the Driver App is accessed from both desktop browsers and mobile devices
- OpenUI5 1.96.40 responsive design patterns must be applied consistently

---

## 12. Architecture Decision Records

### ADR-001: Persistent Layer for Preliminary Calculation

**Question:** Where is the preliminary fleet cost calculation data persisted?

The preliminary calculation screen allows fleet managers and HR to estimate the total mobility cost for a vehicle before an entitlement or order is created. This data must be stored somewhere between sessions.

**Options evaluated:**

| Option | Approach | Status |
|---|---|---|
| **Option 1 — Z-tables (interim)** | Store calculation data in SAP Z-tables (custom tables) | Interim only — violates Clean Core for production |
| **Option 2 — Abiscon OData + SAP standard** | Use Abiscon's own OData APIs backed by SAP-standard objects | **Preferred (Gate A)** — Clean Core compliant |
| **Option 3 — Simplifier / GCP (no-sync fallback)** | Store in Simplifier or GCP with no SAP synchronisation | Fallback (Gate B) — only if Option 2 is not feasible |

**Decision:** Proceed with Option 2 if the Abiscon OData path is technically feasible (Gate A decision). If not feasible, fall back to Option 3 (Gate B). Option 1 is permitted as an interim measure during development only.

**Key constraint:** The Clean Core principle forbids Z-tables in the production landscape. Any interim Z-table usage must have a migration path to Option 2 or 3 before go-live.

---

### ADR-002: External API Integration Hub

**Question:** How should external API integrations (KBA, Shell, ARAL, DAT, High Mobility, etc.) be connected to the platform?

**Options evaluated:**

| Option | Approach |
|---|---|
| **Option A — SAP Integration Suite (direct)** | Route all external API calls through SAP BTP Integration Suite |
| **Option B — Abiscon Cloud hub** | Route external API calls through a dedicated Java/Quarkus microservice on StackIT Germany |

**Decision: Option B — Abiscon Cloud hub** is recommended.

**Key drivers:**
- **XAdES signature requirement** for KBA GKS iKFZ (German vehicle registration SOAP interface) — this XML-level signature cannot be handled by SAP Integration Suite without significant custom development; Abiscon Cloud hub handles it natively
- Abiscon already manages Shell, ARAL, DAT, and High Mobility integrations — centralising these in the Abiscon Cloud hub reduces integration sprawl
- The hub provides built-in PSA, observability, SLA management, runbooks, and an exit concept

**Safeguards required for Option B:**
- Privacy and Security Assessment (PSA) must be completed for the Abiscon Cloud hub
- Observability: metrics, structured logs, and distributed tracing must be in place
- SLA agreement with Abiscon
- Runbooks for incident response
- Exit concept: documented path to migrate integrations away from Abiscon Cloud if needed

**Infrastructure:** Abiscon Cloud runs on **StackIT Germany** (T-Systems cloud) using Java/Quarkus. All processing is transient — no data is persisted in Abiscon Cloud.
