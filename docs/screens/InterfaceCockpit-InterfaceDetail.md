# Screen: InterfaceDetail — InterfaceCockpit

**App:** InterfaceCockpit  
**Screen UUID:** `11a517e4-e821-468d-8024-5129141640e2`  
**Purpose:** Detail view for a single interface — shows full status, availability, message statistics, error details, retry history, and allows manual retry trigger.  
**Jira:** MBI-2055 / LC-INT-08

---

## Widget Inventory

| Name | Type | Key Properties | Notes |
|---|---|---|---|
| `DetailBar` | Bar (1.96) | — | Screen header |
| `DetailBackBtn` | Button (1.96) | icon: sap-icon://nav-back | Press triggers Back story |
| `DetailTitle` | Title (1.96) | text: "Interface Detail" | |
| `StatusStrip` | MessageStrip (1.96) | — | Feedback strip; updated by Load Data and Trigger Retry stories |
| `DetailCard` | Card (1.96) | — | Main content card |
| `DetailCardHeader` | CardHeader (1.96) | title: `{name}`, subtitle: `{system} — {technology}` | Bound to varSelectedInterface |
| `StatusSection` | VBox (1.96) | — | |
| `StatusLabel` | Label (1.96) | text: "Status" | |
| `StatusValue` | ObjectStatus (1.96) | text bound to `varSelectedInterface.status` | |
| `AvailLabel` | Label (1.96) | text: "Availability" | |
| `AvailValue` | Text (1.96) | text bound to `varSelectedInterface.availability` | |
| `LastSuccessLabel` | Label (1.96) | text: "Last Success" | |
| `LastSuccessValue` | Text (1.96) | text bound to `varSelectedInterface.lastSuccess` | |
| `LastFailureLabel` | Label (1.96) | text: "Last Failure" | |
| `LastFailureValue` | Text (1.96) | text bound to `varSelectedInterface.lastFailure` | |
| `ErrorSection` | VBox (1.96) | — | |
| `ErrCodeLabel` | Label (1.96) | text: "Error Code" | |
| `ErrCodeValue` | Text (1.96) | text bound to `varSelectedInterface.errorCode` | |
| `ErrReasonLabel` | Label (1.96) | text: "Error Reason" | |
| `ErrReasonValue` | Text (1.96) | text bound to `varSelectedInterface.errorReason` | |
| `RetrySection` | VBox (1.96) | — | |
| `RetryCountLabel` | Label (1.96) | text: "Retry Count" | |
| `RetryCountValue` | Text (1.96) | text bound to `varSelectedInterface.retryCount` | |
| `RetryStatusLabel` | Label (1.96) | text: "Retry Status" | |
| `RetryStatusValue` | ObjectStatus (1.96) | text bound to `varSelectedInterface.retryStatus` | |
| `MetaSection` | VBox (1.96) | — | |
| `BizFuncLabel` | Label (1.96) | text: "Business Function" | |
| `BizFuncValue` | Text (1.96) | text bound to `varSelectedInterface.businessFunction` | |
| `ScopeLabel` | Label (1.96) | text: "Scope" | |
| `ScopeValue` | Text (1.96) | text bound to `varSelectedInterface.scope` | |
| `MsgCountLabel` | Label (1.96) | text: "Message Count" | |
| `MsgCountValue` | Text (1.96) | text bound to `varSelectedInterface.messageCount` | |
| `TriggerRetryBtn` | Button (1.96) | text: "Trigger Retry", type: Emphasized | Press triggers Trigger Retry story |

---

## Data Bindings

| Target | Source | Set by Story |
|---|---|---|
| All detail fields | `varSelectedInterface.*` | [InterfaceDetail] Load Data |
| `varSelectedInterfaceId` | `getInterfaceDetail` output `interfaceId` | [InterfaceDetail] Load Data |
| `StatusStrip` text/type | Set via story UIAction | [InterfaceDetail] Load Data, Trigger Retry |

---

## Events and Process Stories

| Widget | Event | Story triggered |
|---|---|---|
| Screen | show | `(MBI-2055) [InterfaceDetail] Load Data` |
| `DetailBackBtn` | press | `(MBI-2055) [InterfaceDetail] Back` |
| `TriggerRetryBtn` | press | `(MBI-2055) [InterfaceDetail] Trigger Retry` |

---

## Process Story Wiring Summary

### `(MBI-2055) [InterfaceDetail] Load Data`
- Subscribe: InterfaceDetail screen `show`
- BO input: `ScreenItem(Cockpit, InterfaceRow, id) → interfaceId`
- BO output: `interface → varSelectedInterface`, `interfaceId → varSelectedInterfaceId`
- Note: `id` comes from the Cockpit row binding context at navigation time

### `(MBI-2055) [InterfaceDetail] Trigger Retry`
- Subscribe: `TriggerRetryBtn` press
- BO input: `varSelectedInterfaceId → interfaceId`
- BO output: `interfaceId → varSelectedInterfaceId`, `message`, `newRetryStatus`

### `(MBI-2055) [InterfaceDetail] Back`
- Subscribe: `DetailBackBtn` press
- Navigation: NavBack to Cockpit (Slide transition)

---

## Navigation Context

Arrived at from Cockpit via `00.3 Navigation` story (InterfaceRow press or CellDetailBtn press). The row's binding context `id` is passed to the Load Data BO at screen open. On successful load, `varSelectedInterfaceId` is populated from the BO's `interfaceId` output for use by Trigger Retry.
