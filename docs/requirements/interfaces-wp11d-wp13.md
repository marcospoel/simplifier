# Interface Requirements — WP11d (Low-Code Interfaces) & WP13 (Interface Development)

**Sources:**
- [Low Code – Interfaces Workshop](https://wiki.telekom.de/spaces/MBI/pages/3275715849/Low+Code+-+Interfaces+Workshop)
- [ReFIT Interfaces](https://wiki.telekom.de/spaces/MBI/pages/3368080860/ReFIT+Interfaces)
- Engineering Tasks document: `C:\Users\A110231610\Downloads\crawler\engineering_tasks_wp11d_wp13.md`

**Status:** Requirements extracted — interface contracts and technical design pending  
**Deployment target:** GCP (not Tanzu/FCI)

---

## Vocabulary

| Term | Meaning |
|---|---|
| **Interface** | Communication with systems **outside** the SAP landscape |
| **Integration** | Communication between SAP modules **inside** the SAP landscape |
| **FMO** | Future Mode of Operation — target technology for the interface |
| **APC** | Abiscon Process Cockpit |

---

## Interface Master List

Columns: **#** | **Name / Description** | **Scope** | **Source** | **Target** | **Current tech** | **FMO tech** | **Notes / Open points**

### SAP Integrations (IDoc-based)

| # | Interface | Scope | Source | Target | Current | FMO | Notes |
|---|---|---|---|---|---|---|---|
| 3 | **YZA_XV01** — Create master data and transactional data; error logging / restart points | fleet | — | SAP | IDoc | fleet | Core fleet IDoc |
| 4 | **YZA_XV02** — Restart of YZA_XV01 | fleet | — | SAP | IDoc | fleet | BAPI `YZA_MP_IMP_ORDER` |
| 5–11 | **YZA_XV03–09** — Data correction programs | — | — | — | — | — | Scope TBD |
| 1 | **yza_xv10** — Upload bike order | Low-Code | Excel / CSV | APC | IDoc | OData | Dirk L. to check API req. with Company Bike; follow-up September |
| 2 | **yza_xv11** — Upload leasing bike | Low-Code | Portal (tbc) | APC | IDoc | REST | Investigate source portal |

---

### Low-Code Interfaces (WP11d primary scope)

| # | Interface | Scope | Source | Target | FMO tech | Notes |
|---|---|---|---|---|---|---|
| 12a | **Calculation / Configuration Tool** — calculation data when car is ordered | Low-Code | Low-Code | APC | OData | Covers preliminary calculation data at order time |
| 12b | **Calculation / Configuration Tool** — in-app running calculator | Low-Code | e:c:car | Low-Code | REST | Live rate calculation during e:c:car configuration session |
| 13 | **Customer App (Kunden App)** — if not provided by fleet solution | Low-Code | Low-Code | APC | OData | |
| 16c | **Electronic document delivery — Invoice AI validation + repair cost calc** | Low-Code | Low-Code | APC | — | AI-based invoice validation; open points on AWS Textract vs. smart.process |
| 20e | **Actual calculation — GU-specific fields** | fleet | fleet | Low-Code | — | Batch triggered by yearly contract event; GU = Gesellschaftsfahrzeug (company vehicle); Marc to clarify GU and total products |
| 21 | **Identity / Roles / Authorisation** | Low-Code | — | Low-Code | REST | Internal: Entra ID + CAIMAN → role in Low-Code. External: OpenID / ICU. CAIMAN process too complex for external; Patrick to clarify |
| 26a | **Vermarktungsportal** (vehicle sales portal) | fleet | fleet | Low-Code | — | Start discussion on future portal (replacement); open |
| 26b | **Vermarktungsportal** | Low-Code | Low-Code | External App | — | TBD |
| 34 | **AskT internal chatbot** | Low-Code | — | — | — | Chatbot factory with RAG/long-term memory. Marc + Patrick to decide. See: [Chatbot Factory docs](https://mdx.pages.devops.telekom.de/docs/tools/ai/chatbotfactory/) |
| 37a | **Manufacturer — JATO code per order** | e:c:car | e:c:car (JATO code) | Low-Code | — | Built code (Ausbaukode) under discussion (Pavol for order); stickers/racks etc. |
| 37d | **Manufacturer buildability change — taxation recalculation** | Low-Code | Alt: change JSON of car | — | — | Recalculate after buildability change; see 37c |
| 42 | **Fleetpoint — appointment / termination tool** | Low-Code | — | APC | — | Incl. handover protocol; Fleetpoint sets Wareneingang; needs e2e process in APC; portal for externals + Car App for drivers |
| 43 | **Fleetpoint / logistics — inventory lists / yard management** | Low-Code | — | APC | — | Location of car (license plate referenced) |
| 45 | **Assessor (Gutachter) connection** | Low-Code | — | APC | — | FTU over EDI; follows PM module replacing MM-based processes; also covers logistics orders |
| 46 | **Freight forwarder (Spedition) connection** | Low-Code | — | APC | — | Same as 43; portal in Low-Code |
| 52 | **Workshop data entry** | Low-Code | — | — | — | Agile approach: start with data entry point and single version of truth |
| HR-DH | **HR SAP Data Highway** | Low-Code / fleet | — | — | REST | API for detailed HR employee data. See: [DataHighway documentation](https://documentserp.telekom.de:4443/wiki.aspx?title=EDM:EDM-HR-Minimaster_Overview/en) |

---

### SAP HR Interfaces

| # | Interface | Scope | Source | Target | FMO | Notes |
|---|---|---|---|---|---|---|
| 20a | **Driver master data (Fahrerstammdaten)** | OFi | — | — | — | Ilia: 80% in HR-Mini stam; 20% not available (e.g. home address). Olympus API details pending (Monika/Ilia) |
| 20b | **Order / driving auth, GWM, contract changes, chargeable costs, annual settlement, terminations, deregistration (Abmeldung Versteuerung)** | fleet | — | — | — | Multiple HR sub-interfaces |
| 20c | **Terminate taxation (Abmeldung Versteuerung)** | fleet | fleet | APC | — | Einsteuerung start: Order → FleetPoint pre-tax → final value at order+price → DTSE + driver PDF email |
| 20d | **GU registration (Registrierung GU) + annual adjustment + final settlement** | fleet | fleet | APC | — | HR dept receives batch triggered by year event of contract |

---

### SAP OFI / Finance Interfaces

| # | Interface | Scope | Source | Target | FMO | Notes |
|---|---|---|---|---|---|---|
| 15 | **Document management incl. archiving** — transfer to Telekom archiving system (Image Master) after defined retention period | fleet | fleet | SAP | — | ArchiveLink interface |
| 16a | **Electronic document delivery** (fleet assets, drivers/users) | fleet | fleet | SAP | — | |
| 16b | **Electronic document delivery** (invoices) | OFi | OFi | ImageMaster | — | |
| 17 | **E-invoice delivery (E-Rechnung)** | OFi | — | — | — | |
| 18 | **OCR tool** — incoming invoices, appraisals/Gutachten; interpretation and clustering | OFi | — | — | — | If not integrated already |
| 19 | **SAP One ERP** — master data (debtors/creditors/framework contracts/material master) + FI CO data | OFi | — | — | — | Note: insurance and remarketing debtors currently only in SAP FIT — must be migrated to SAP Telekom Group system before main migration |
| INTRA | **INTRASTAT** — EU cross-border sales reporting (remarketing) | OFi | OFi | SAP | — | General OFI functionality; Business Epic @ Olympus: BEGHSFI-177; addressed in Olympus workshops |
| 22 | **Identity / Roles (OFi/fleet/INStra/SAP)** | OFi | — | SAP | — | Roles managed in OFi SAP role shop; contact within TelIT/TMS needed; currently maintained by Juliane (TelIT) |
| 23 | **IC-4 — Intercompany platform** (IC invoices, open items) | **decom.** | — | — | — | No interface needed — fleet is on OFi; CO details to be clarified; standard approach preferred |
| 24 | **Payment Factory** — banking interface (DTSE executes payments for TMS) | OFi | — | — | — | |
| 25 | **SMART accounting tool** (Abgrenzungstool) — if accounting is in fleet system | OFi | — | — | — | To be followed up: Marc |
| 28 | **EDI — BMW, VW, Carglass** (orders + invoices) | OFi | — | — | C-PeX / e-Best / XI | BTP eBilling standard; C-PeX being replaced; migration end of 2026; Juliane to clarify C-PeX replacement; Monika to follow up with Olympus team |
| 48 | **Supplier qualification** — NCL, Financial Scorecard, Compliance, CSR | OFi | — | SAP | — | |
| 50 | **Collective invoice** (e.g. car wash) | SAP | — | — | — | Part of PM workshop |

---

### Manufacturer / OEM Interfaces

| # | Interface | Scope | Source | Target | FMO | Notes |
|---|---|---|---|---|---|---|
| 36a | **Model codes** — standard/special equipment incl. codes, package components incl. BLP, price changes | e:c:car | — | e:c:car | — | |
| 36b | **Re-import at first registration** — verify config data vs. registration data | fleet | — | — | — | In case BLP ≠ invoice; CO2 must be accounted for (KBA data includes CO2); GFZ = electric only → no CO2 |
| 37a | **JATO code (config ID) per order** | e:c:car | JATO code | Low-Code | — | Built code under discussion |
| 37b | **Communicate the build (DFz)** | — | Built code | — | — | Cover in calculator + configurator workshop |
| 37c | **Buildability changes** — config ID adjustment + customer re-approval | e:c:car | OEM email | — | — | Options: A) accept + update order (config change in e:c:car + tax recalc) B) cancel + new order |
| 37d | **Buildability changes — taxation recalculation** | Low-Code | Alt: change car JSON | — | — | |
| 37e | **Final details — DAT interface** | fleet | Final data | — | — | |
| 38 | **Service data / mileage (km Stand)** | fleet | — | fleet | — | High Mobility — Andreas + Marc to discuss pricing / ROI |
| 39 | **Vehicle service appointments** | fleet | — | fleet | — | High Mobility or PM maintenance plan |
| 40a | **Order, order confirmation (AB), buildability info, delivery dates** | OFi | — | SAP | — | Follows EDI (#28) discussion |
| 40b | **Buildability info** | e:c:car | — | — | — | Cover in calc & conf workshop |
| 41 | **Recall notices (Rückrufaktionen)** | fleet | OEM (email/mass format) | — | — | List of affected cars; mass technical data format |

---

### Vehicle Registration

| # | Interface | Scope | Source | Target | FMO | Notes |
|---|---|---|---|---|---|---|
| 35 | **KBA data import + translation** | fleet | KBA | fleet | — | DTFS classification per model; KBA offered as standard; CSV bridge possible to start earlier; i-KFZ: https://www.kba.de/gks |
| 47 | **Registration authority (Zulassungsstelle)** — currently IKOL | fleet | — | fleet | — | i-KFZ; KBA Großkunden |
| 54 | **VIN → license plate** exchange | fleet | — | — | — | Email to OEMs: VIN to plate + i-KFZ |
| 14 | **HZA Hauptzollamt** — online retrieval of vehicle tax notices (3.5.9) | — | — | — | — | Marc to clarify what needs to be exchanged |

---

### Fuel Cards / Charging

| # | Interface | Scope | Source | Target | FMO | Notes |
|---|---|---|---|---|---|---|
| 27 | **Fuel providers** — Shell, Aral, Total, UTA + electric charging cards: card order, import tank/charge data, SmartPay | fleet | — | SAP | — | Shell Recharge (public) to check; home charging is a separate topic; Marc to check provider impact |

---

### Replacement Mobility / Additional Tools

| # | Interface | Scope | Source | Target | Notes |
|---|---|---|---|---|---|
| 29 | **Company Bike** | Low-Code | — | — | See #1 and #2 (yza_xv10/xv11) |
| 30 | **Car Rental Portal — Fleetster** (5.2) | — | — | — | Invoice or just linking to order replacement vehicle; clarification: Marc |
| 31 | **Shuttle App** (5.3) | — | — | — | Flat-rate billing or one-click link; clarification: Ingo + Marc |
| 32 | **Chauffeur Dispo Tool** (5.4) | — | — | — | Invoice data exchange; clarification: Ingo |

---

### Technical / Assessment Systems

| # | Interface | Scope | Source | Target | Notes |
|---|---|---|---|---|---|
| 44 | **audatec / TecRMI** (3.5.2) | fleet | — | — | To be replaced by DAT — handed to fleet workshop |

---

### Reporting / Analytics

| # | Interface | Scope | Notes |
|---|---|---|---|
| 33 | **BI tool / CSBI** (central reporting) | — | Reporting layers: 1) Fleet addon reports 2) S/4HANA Embedded Analytics / CDS Views 3) CSBI or SAC for non-fleet/OFI data 4) SAP Databricks (future, presumably out of scope). Patrick to clarify in reporting workshop |
| 53 | **Reporting DTT / DTA data exchange** | — | Juliane; needs to go to reporting workstream |
| 55 | **Telemetry** | — | Open |

---

### Decommissioned

| # | Interface | Note |
|---|---|---|
| 23 | IC-4 Intercompany platform | No longer needed — fleet is on OFi |
| 49 | Interface Agreement PC3 to TMS | Ingo to confirm (cost centre hierarchy HR/SD/CO) |

---

## Interfaces by Scope — Summary

### Low-Code / Simplifier scope (WP11d)

| # | Interface | FMO tech | Status |
|---|---|---|---|
| 1 | yza_xv10 — Upload bike order | OData | Follow-up Sep |
| 2 | yza_xv11 — Upload leasing bike | REST | Investigate source |
| 12a | Calculation tool → APC | OData | |
| 12b | e:c:car → Low-Code calculator | REST | |
| 13 | Customer App → APC | OData | |
| 16c | AI invoice validation → APC | — | Arch board pending |
| 20e | GU calculation data → Low-Code | — | Marc to clarify |
| 21 | Identity / Roles (Entra ID, CAIMAN) | REST | Patrick to clarify external |
| 26a/b | Vermarktungsportal | — | Open |
| 34 | AskT chatbot | — | Marc + Patrick decide |
| 37a | JATO code e:c:car → Low-Code | — | Built code TBD |
| 37d | Buildability change → tax recalc | — | |
| 42 | Fleetpoint appointments → APC | — | e2e process in APC needed |
| 43 | Fleetpoint yard management → APC | — | |
| 45 | Assessor → APC | — | PM module follow |
| 46 | Freight forwarder → APC | — | Portal |
| HR-DH | HR Data Highway | REST | |

### Open points / pending decisions for Low-Code

| Topic | Owner | Status |
|---|---|---|
| Company Bike API requirements | Dirk L. | Follow-up September |
| AskT chatbot RAG integration | Marc + Patrick | Decision pending |
| AI invoice validation (AWS Textract vs. smart.process AI) | Architecture board | Pending |
| Vermarktungsportal — future portal concept | — | Start discussion |
| GU-specific calculation fields scope | Marc | To clarify |
| CAIMAN process for external users | Patrick | To clarify |
| Fleetpoint e2e process definition in APC | — | Required before 42/43 |
| Built code (Ausbaukode) definition | Pavol | Under discussion |
| Shuttle App / Chauffeur Dispo billing approach | Ingo + Marc | Clarification needed |
| HZA Hauptzollamt data scope | Marc | To clarify |

---

## Notes on Engineering Tasks Corrections

From the original engineering tasks document, the following has been corrected:

| Original | Correction |
|---|---|
| LC-INT-13: "Deploy to Tanzu on FCI" | Deployment target is **GCP** |

---

## Related Documents

- Solution concept: [`docs/concept/solution-concept.md`](../concept/solution-concept.md)
- Architecture principles + ADRs: sections 9–12 of solution concept
- TARDIS integration gateway: [`docs/tardis.md`](../tardis.md)
- Engineering tasks (WP11d + WP13): `C:\Users\A110231610\Downloads\crawler\engineering_tasks_wp11d_wp13.md`
