# Business Objects

> Maintained by `doc-writer` agent. Updated after every BO or function change.

<!-- Add entries as BOs are created. Format:

## <BOName>

**Project:** <Simplifier project>
**Purpose:** <one sentence>

### Functions

#### `<functionName>(param1: Type, param2: Type) → ReturnType`
<What it does. What connector calls it makes. What it returns.>

**Error cases:** <list known error conditions>
-->

## bInterfaceStatus

**Project:** InterfaceCockpit  
**Purpose:** Server-side BO for interface monitoring — loads interface status lists, retrieves detail records, and triggers manual retries.  
**Jira:** MBI-2055

### Functions

#### `getInterfaceStatuses(filterStatus: String, filterScope: String) → { interfaces: sInterfaceStatus_col, totalCount: Integer, errorCount: Integer, warningCount: Integer }`

Retrieves all interface status records, optionally filtered by status and/or scope. Returns the full list plus summary counts.

**Inputs:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `filterStatus` | String | No | Filter by status value; empty = all statuses |
| `filterScope` | String | No | Filter by scope (e.g. "Low-Code", "fleet"); empty = all |

**Outputs:**

| Parameter | Type | Description |
|---|---|---|
| `interfaces` | sInterfaceStatus_col | List of matching interface status objects |
| `totalCount` | Integer | Total matched interfaces |
| `errorCount` | Integer | Count with status = ERROR |
| `warningCount` | Integer | Count with status = WARNING |

**Used by stories:** [Cockpit] Load Data, [Cockpit] Refresh, [Cockpit] Filter

---

#### `getInterfaceDetail(interfaceId: String) → { interface: sInterfaceStatus, interfaceId: String }`

Retrieves the full detail record for a single interface by its ID.

**Inputs:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `interfaceId` | String | Yes | Unique interface identifier |

**Outputs:**

| Parameter | Type | Description |
|---|---|---|
| `interface` | sInterfaceStatus | Full interface detail object |
| `interfaceId` | String | Echo of the input ID (for downstream variable binding) |

**Used by stories:** [InterfaceDetail] Load Data

---

#### `triggerRetry(interfaceId: String) → { interfaceId: String, message: String, newRetryStatus: String }`

Triggers a manual retry for the specified interface. Returns the updated retry status.

**Inputs:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `interfaceId` | String | Yes | Interface to retry |

**Outputs:**

| Parameter | Type | Description |
|---|---|---|
| `interfaceId` | String | Echo of the input ID |
| `message` | String | Human-readable result message |
| `newRetryStatus` | String | Updated retry status after triggering |

**Used by stories:** [InterfaceDetail] Trigger Retry

---

## bLicensePlateCheck

**Project:** InterfaceCockpit  
**Purpose:** Server-side BO for Kennzeichen (license plate) availability checks — calls the external kennzeichen connector, persists results to `sInterfaceCockpit`, and retrieves check history.  
**Jira:** MBI-2055

### Functions

#### `checkAndSave(district: String, letters: String, digits: String) → { success: Boolean, result: Object }`

Calls the `kennzeichen` connector to check availability of a license plate combination, then saves the result to `sInterfaceCockpit.es_license_plate_checks`.

**Inputs:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `district` | String | Yes | District code (Kennzeichen prefix) |
| `letters` | String | Yes | Letter combination |
| `digits` | String | Yes | Digit combination |

**Outputs:**

| Parameter | Type | Description |
|---|---|---|
| `success` | Boolean | Whether the check and save succeeded |
| `result` | Object | Full check result including availability status |

---

#### `saveCheckResult(district: String, letters: String, digits: String, available: Boolean, rawResponse: String) → { success: Boolean }`

Manually saves a pre-computed check result to `sInterfaceCockpit.es_license_plate_checks`.

**Inputs:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `district` | String | Yes | District code |
| `letters` | String | Yes | Letter combination |
| `digits` | String | Yes | Digit combination |
| `available` | Boolean | Yes | Availability result to persist |
| `rawResponse` | String | No | Raw API response payload |

**Outputs:**

| Parameter | Type | Description |
|---|---|---|
| `success` | Boolean | Whether the insert succeeded |

---

#### `listCheckResults(limit: Integer) → { results: Collection, success: Boolean }`

Returns recent license plate check results from `sInterfaceCockpit.es_license_plate_checks`, ordered by `checked_at` DESC.

**Inputs:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `limit` | Integer | No | Max rows to return; defaults to 50 |

**Outputs:**

| Parameter | Type | Description |
|---|---|---|
| `results` | Collection | List of check result records |
| `success` | Boolean | Whether the query succeeded |

---

## cLicensePlateCheck

**Project:** InterfaceCockpit  
**Purpose:** Client-side BO that exposes license plate availability checking to the UI — delegates to `bLicensePlateCheck.checkAndSave`.  
**Jira:** MBI-2055

### Functions

#### `checkAvailability(district: String, letters: String, digits: String) → void`

Delegates to `bLicensePlateCheck.checkAndSave` with the plate components as input. Calls `fnSuccess` with the result on success, `fnError` on failure.

**Signature:** `(oEvent, oPayload, fnSuccess, fnError)`

**Inputs (via oPayload):**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `district` | String | Yes | District code |
| `letters` | String | Yes | Letter combination |
| `digits` | String | Yes | Digit combination |

**Note:** Every code path calls either `fnSuccess` or `fnError` — never both, never neither.
