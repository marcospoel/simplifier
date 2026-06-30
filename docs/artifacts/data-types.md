# Data Types

> Maintained by `doc-writer` agent. Updated after every data type change.

<!-- Add entries as data types are created. Format:

## <TypeName>

**Kind:** Domain | Struct | Collection
**Used by:** <BO functions or connector calls that reference this type>

### Fields (Struct only)

| Field | Type | Description |
|---|---|---|

### Domain values (Domain only)

| Value | Label |
|---|---|
-->

## sInterfaceStatus

**Kind:** Struct  
**Project:** InterfaceCockpit  
**Used by:** `bInterfaceStatus.getInterfaceStatuses` (list items), `bInterfaceStatus.getInterfaceDetail` (detail object), `varSelectedInterface` (app variable)

### Fields

| Field | Type | Description |
|---|---|---|
| `id` | String | Unique interface identifier |
| `name` | String | Interface display name |
| `system` | String | Source/target system name |
| `technology` | String | Integration technology (e.g. OData, REST, IDoc) |
| `direction` | String | Data flow direction (Inbound / Outbound / Bidirectional) |
| `scope` | String | Interface scope (e.g. Low-Code, fleet, OFi) |
| `businessFunction` | String | Business function this interface serves |
| `status` | String | Current status (OK, WARNING, ERROR) |
| `availability` | String | Availability percentage or indicator |
| `lastSuccess` | String | Timestamp of last successful message |
| `lastFailure` | String | Timestamp of last failed message |
| `errorCode` | String | Last error code if status is ERROR/WARNING |
| `errorReason` | String | Human-readable error description |
| `messageCount` | Integer | Total messages processed |
| `retryCount` | Integer | Number of retry attempts |
| `retryStatus` | String | Current retry status (e.g. Pending, Completed, Failed) |

---

## sInterfaceStatus_col

**Kind:** Collection  
**Item type:** sInterfaceStatus  
**Project:** InterfaceCockpit  
**Used by:** `bInterfaceStatus.getInterfaceStatuses` output `interfaces`, `varInterfaceList` (app variable bound to Cockpit table)
