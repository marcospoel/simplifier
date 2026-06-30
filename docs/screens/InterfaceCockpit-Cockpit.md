# Screen: Cockpit — InterfaceCockpit

**App:** InterfaceCockpit  
**Screen UUID:** `1e9987d6-5071-46f9-8a86-55f3b2ed3910`  
**Purpose:** Main monitoring view — lists all connected system interfaces with status, last success/failure, error details, and retry status. Entry point for detail navigation.  
**Jira:** MBI-2055 / LC-INT-08

---

## Widget Inventory

| Name | Type | UUID | Key Properties | Notes |
|---|---|---|---|---|
| `HeaderBar` | Bar (1.96) | — | — | Screen header |
| `HeaderTitle` | Title (1.96) | — | text: "Interface Cockpit" | |
| `HeaderRefreshBtn` | Button (1.96) | `1db67386-472c-484e-9d17-8f6135428ddb` | icon: sap-icon://refresh | Triggers Refresh story |
| `SummaryHBox` | HBox (1.96) | — | — | Tile row |
| `TileTotalCard` | GenericTile (1.96) | — | header: "Total" | |
| `TileTotalNumeric` | NumericContent (1.96) | — | value bound via varTotalCount | |
| `TileErrorCard` | GenericTile (1.96) | — | header: "Errors" | |
| `TileErrorNumeric` | NumericContent (1.96) | — | value bound via varErrorCount | |
| `TileWarningCard` | GenericTile (1.96) | — | header: "Warnings" | |
| `TileWarningNumeric` | NumericContent (1.96) | — | value bound via varWarningCount | |
| `FilterToolbar` | Toolbar (1.96) | — | — | Filter bar above table |
| `FilterStatusSelect` | Select (1.96) | `c92006a5-c420-4900-aacf-712b990342a9` | — | Status filter; triggers Filter story on change |
| `FilterScopeSelect` | Select (1.96) | `dba41b23-e3c1-4626-8c13-eb40bb127c1c` | — | Scope filter; triggers Filter story on change |
| `InterfaceTable` | Table (1.96) | `df9a429a-6592-466f-a5d5-2f3b962fea4f` | alternateRowColors, growing, noDataText | Items bound to varInterfaceList |
| `InterfaceRow` | ColumnListItem (1.96) | `d54dd914-ed82-4097-b3f5-66285f833f1f` | type: Navigation | Row template; press triggers Navigation story |
| `CellNameVBox` | VBox (1.96) | `84f5b658-d34c-4d84-a29c-20d1ee40a331` | — | |
| `CellName` | Text (1.96) | `8108b1db-b411-4924-ab79-ea7a88a6d85b` | text: `{name}` | |
| `CellBizFunc` | Text (1.96) | `eb20f878-2f62-4c2f-ac11-ff6664f176c4` | text: `{businessFunction}` | |
| `CellSystem` | Text (1.96) | `40e03f5f-653b-4398-96c6-18e9d9a3cb48` | text: `{system}` | |
| `CellTechVBox` | VBox (1.96) | `ad36a389-7de6-4a9c-80da-7da083a67faa` | — | |
| `CellTech` | Text (1.96) | `d1198abe-9102-42df-8cc6-4ada4e5edb19` | text: `{technology}` | |
| `CellDir` | Text (1.96) | `ad151e4d-6d92-442e-b906-a2e432b4854e` | text: `{direction}` | |
| `CellStatus` | ObjectStatus (1.96) | `78c1a8fc-d9d3-4f14-814a-a277470b1691` | text: `{status}` | |
| `CellLastSuccess` | Text (1.96) | `cb297684-d856-4621-90c3-bf19362e8220` | text: `{lastSuccess}` | |
| `CellErrorVBox` | VBox (1.96) | `1d22d31d-1042-4ee7-901b-5d0a4c45aedf` | — | |
| `CellErrorCode` | Text (1.96) | `88e0eff2-1eb8-4f3c-8499-1ee1fb236217` | text: `{errorCode}` | |
| `CellErrorReason` | Text (1.96) | `9201ae54-3c01-494a-93f0-26311cb4e838` | text: `{errorReason}` | |
| `CellRetryStatus` | ObjectStatus (1.96) | `7026cf81-51c5-4338-8a63-5612daa4d3a9` | text: `{retryStatus}` | |
| `CellDetailBtn` | Button (1.96) | `63078576-6979-44aa-871d-0aa92d4cd9c9` | icon: sap-icon://detail-view, type: Transparent | Press triggers Navigation story |

---

## Data Bindings

| Target | Source | Set by Story |
|---|---|---|
| `InterfaceTable.items` | `varInterfaceList` | [Cockpit] Load Data, [Cockpit] Refresh, [Cockpit] Filter |
| Tile total count | `varTotalCount` | [Cockpit] Load Data, [Cockpit] Refresh, [Cockpit] Filter |
| Tile error count | `varErrorCount` | [Cockpit] Load Data, [Cockpit] Refresh, [Cockpit] Filter |
| Tile warning count | `varWarningCount` | [Cockpit] Load Data, [Cockpit] Refresh, [Cockpit] Filter |
| Row cells | `{name}`, `{system}`, `{technology}`, `{direction}`, `{status}`, `{lastSuccess}`, `{errorCode}`, `{errorReason}`, `{retryStatus}` | Template binding within row context |

---

## Events and Process Stories

| Widget | Event | Story triggered |
|---|---|---|
| Screen | show | `(MBI-2055) [Cockpit] Load Data` |
| `HeaderRefreshBtn` | press | `(MBI-2055) [Cockpit] Refresh` |
| `FilterStatusSelect` | change | `(MBI-2055) [Cockpit] Filter` |
| `FilterScopeSelect` | change | `(MBI-2055) [Cockpit] Filter` |
| `InterfaceRow` | press | `00.3 Navigation` |
| `CellDetailBtn` | press | `00.3 Navigation` |

---

## Process Story Wiring Summary

### `(MBI-2055) [Cockpit] Load Data`
- Subscribe: Cockpit screen `show`
- BO: `bInterfaceStatus.getInterfaceStatuses(filterStatus="", filterScope="")`
- Output: `interfaces → varInterfaceList`, `totalCount → varTotalCount`, `errorCount → varErrorCount`, `warningCount → varWarningCount`

### `(MBI-2055) [Cockpit] Refresh`
- Subscribe: `HeaderRefreshBtn` press
- BO: `bInterfaceStatus.getInterfaceStatuses(filterStatus=varFilterStatus, filterScope=varFilterScope)`
- Output: same as Load Data

### `(MBI-2055) [Cockpit] Filter`
- Subscribe (path 1): `FilterStatusSelect` change → BO with `FilterStatusSelect.selectedKey → filterStatus`
- Subscribe (path 2): `FilterScopeSelect` change → BO with `FilterScopeSelect.selectedKey → filterScope`
- Both paths output: `interfaces → varInterfaceList`, counts → vars

### `00.3 Navigation`
- Subscribe (path 1): `InterfaceRow` press → Navigate to InterfaceDetail
- Subscribe (path 2): `CellDetailBtn` press → Navigate to InterfaceDetail
- The selected row's binding context `id` field is read by the InterfaceDetail Load Data story via `ScreenItem(Cockpit, InterfaceRow, id)`
