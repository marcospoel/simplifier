# Solution Concept: DT Mobility Fleet Management Platform

**Source:** [NEW Design / Implementation / Concept](https://wiki.telekom.de/spaces/MBI/pages/3691940648/NEW+Design+Implementation+Concept) — DT Mobility Confluence Space  
**Last updated in source:** Dec 18, 2025  
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
1. **Simplifier Low-Code Scheduled Job** periodically calls the EmployeeDataAPI (Cayman/HR)
2. Changed person data triggers Abiscon `Persons org data changed` process
3. Abiscon updates person records and propagates to SAP business partner if needed

---

## 5. Interface Overview

### Low-Code ↔ Abiscon (internal platform API)
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
| System | Interface | Type |
|---|---|---|
| Shell | B2B Mobility Card (order, status update, transaction data) | REST |
| ARAL | Fleet Card API (bp.com) | REST |
| KBA | GKS iKFZ Großkundenschnittstelle | SOAP |
| DAT | SilverDAT (VIN lookup, maintenance plans) | — |
| High Mobility | Auto API (telematics) | REST |
| TSI | EU Digital Wallet driver licence data | — |

### SAP OFI Interfaces
| Interface | Direction | Purpose |
|---|---|---|
| Billing Document Create | Abiscon → SAP | Create billing documents for fleet sales contracts |
| Equipment | Abiscon → SAP | Create/update SAP PM equipment records |
| Archive Link | → SAP | Document archiving |
| EDI/INTRASTAT | SAP ↔ External | Trade statistics reporting |
| Kostenstellenrahmen (EDMtoTMS) | External → OFI | Cost centre framework data |

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
