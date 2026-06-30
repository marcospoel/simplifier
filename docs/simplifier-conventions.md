# Simplifier Development Conventions

Extracted from project rule files and skills. These conventions apply to all Simplifier work on this project.

---

## Naming Conventions

### Business Objects

| Prefix | Meaning | Example |
|--------|---------|---------|
| `b` | App-specific server BO | `bVehicle`, `bAuditTrail`, `bMockData` |
| `c` | Client BO (app-specific) | `cCalcBO`, `cCalc_validation_and_formater` |
| `SF_` | Standard/reusable (framework) | `SF_User`, `SF_Login`, `SF_PopupTemplates` |
| `TMS_` | Domain-specific (TMS fleet) | `TMS_Kalkulator`, `TMS_ElementHelper` |

### Connectors

| Prefix | Meaning | Example |
|--------|---------|---------|
| `c` | App-specific SQL connector | `cCalcMasterData`, `cAuditTrail`, `cMockData` |
| `SF_` | Standard/framework connector | `SF_Login`, `SF_Workflow_Runtime` |

### BO Function Naming

| Pattern | Use case | Example |
|---------|----------|---------|
| `get{Data}` | Read/fetch | `getVehicles`, `getMaintenanceTableData` |
| `add{Entity}` / `create{Entity}` | Insert | `addVehicle`, `createVehicleExtension` |
| `edit{Entity}` / `update{Entity}` | Update | `editDuration`, `updateVehicleOption` |
| `delete{Entity}` | Delete | `deleteVehicle` |
| `validate{Rule}` | Validation | `validateVehicleTaxConflicts` |
| `calculate{What}` | Computation | `calculateMobilitaetsrate` |
| `count{Entity}Usage` | Dependency check | `countDurationUsage` |

### Schemas and Entity Sets

- Schema names: `s` prefix → `sCalcMasterData`, `sCalculation`, `sMockData`
- Table names: UpperCamelCase, singular noun → `TireSize`, `Vehicle`
- Entity set names: `es` prefix, lowerCamelCase, plural → `esTireSizes`, `esVehicles`

### Process Stories

| Pattern | Use case | Example |
|---------|----------|---------|
| `(MBI-XXXX) [Category] Title` | Feature stories | `(MBI-1079) [Master Data Management] Vehicles` |
| `00.X Title` | Wizard-generated | `00.2 Wizard Login` |
| `00.3 Navigation` | Shared navigation events | all screen nav goes here |
| `XX.X Title` | Calculation flows | `02.1 Calculation` |
| `Z_HelperName` | Helper/utility | `Z_Transport_Helper` |
| `99 Title` | Technical cross-cutting | `99 Track execution time` |

---

## Screen Patterns

### Standard Screen Set

| Screen | Purpose |
|--------|---------|
| `Splash` | Loading/initialization |
| `Login` | Authentication |
| `Launchpad` | Navigation hub (`FlexBox` + `GenericTile` widgets) |
| Main feature screen | Feature implementation |
| `MasterData` | CRUD for reference data (tabbed `IconTabBar`) |
| `Popups` | All reusable `Dialog` widgets in one dedicated screen |

### Launchpad Pattern

```
FlexBox (centered)
└── GenericTile (per section)
    ├── header / subheader / headerImage (sap-icon)
    └── press → navigate to target screen
Bar (customHeader)
├── contentLeft: Image (logo)
├── contentMiddle: Title
└── contentRight: Avatar + Text (username) + Button (logout)
```

### Tabbed CRUD Screen Pattern

```
Screen
├── customHeader: Bar
└── ScreenContent: IconTabBar
    ├── IconTabFilter "Entity A"
    │   └── TableEnhanced + toolbar (Add / Edit / Delete)
    └── IconTabFilter "Entity B"
        └── TableEnhanced + toolbar
```

- Tab selection triggers data load for that tab.
- Toolbar buttons open dialogs on the `Popups` screen.

### Dialog/Popup Pattern

- All dialogs live on the dedicated `Popups` screen.
- Naming: `{Action}{Entity}Dialog` → `AddVehicleTaxDialog`, `DeleteFuelTypeDialog`
- Each dialog: form inputs + footer Save/Cancel buttons.
- Process stories open/close dialogs via `UIAction` nodes.

### Table Pattern (`TableEnhanced`)

- `columns` → `Column` widgets with `Label` headers
- `items` → `ColumnListItem` template
- Cells: `Text` (read-only), `Input`/`Select` (editable)
- Inline editing: wrap `Input` + `Title` in `FlexBox`; toggle visibility by edit state

### Form Pattern

Use `sap.ui.layout.form.Form` with `ColumnLayout` (M=2, L=3, XL=4 columns).  
Only use `SimpleForm` when explicitly requested.  
Labels and fields alternate in `content` aggregation.

---

## Server-Side Business Object JavaScript

### Core Rules

- Execution is **synchronous** — no callbacks, no `fnSuccess`, no `fnError`.
- Read all inputs from the global `input` object: `input.parameterName`.
- Write all outputs to the global `output` object: `output.parameterName = value`.
- **Never** use `return` to return data. The runtime appends `return output` automatically.
- **Never** use `oPayload` — that is client-side pattern only.

### Required Output Shape

```javascript
// Success
output.success = true;
output.message = "Operation completed successfully";
output.data = result;

// Failure
output.success = false;
output.error = error.message;
output.errorCode = "OPERATION_ERROR";
```

Always set `output.success`. Use specific `errorCode` values.

### Error Handling Template

```javascript
var myFunction = function() {
  try {
    if (!input.requiredValue) {
      throw new Error("requiredValue is required");
    }

    Simplifier.Log.info("Starting myFunction");

    var result = { value: input.requiredValue };

    output.success = true;
    output.data = result;
    output.message = "Operation completed successfully";

  } catch (error) {
    output.success = false;
    output.error = error.message;
    output.errorCode = "MY_FUNCTION_ERROR";
    Simplifier.Log.error("myFunction failed", error.message);
  }
};
```

### Logging

```javascript
Simplifier.Log.debug(message, details);
Simplifier.Log.info(message, details);
Simplifier.Log.warn(message, details);
Simplifier.Log.error(message, details);

// Always convert errors to string — never pass raw error objects
Simplifier.Log.error("Error occurred", error.message);
Simplifier.Log.error("Error details", JSON.stringify(error));
```

### Calling Other Artifacts (Server-Side)

```javascript
// Other server BO
var result = Simplifier.BusinessObject.OtherBO.methodName({ param: "value" });

// Current BO
var result = Simplifier.CurrentBusinessObject.anotherMethod({ param: "value" });

// Connector
var result = Simplifier.Connector.ConnectorName.callName({ param: "value" });
```

Any BO or connector referenced via `Simplifier.BusinessObject.*` or `Simplifier.Connector.*` **must** be declared as a dependency on the BO (`businessobject-update` `dependencies` array).

### Client-Side BO Differences

| Area | Server-Side | Client-Side |
|---|---|---|
| Execution | Synchronous | Often async |
| Input | `input.parameterName` | `oPayload.parameterName` |
| Output | `output.parameterName = value` | `fnSuccess(result)` |
| Return | Never | Allowed |
| Callbacks | Not used | Required for async |
| Other BO | `Simplifier.BusinessObject.BO.method(payload)` | `Simplifier.ClientsideBusinessObject.BO.method(...)` |
| Connectors | Direct synchronous | Not directly — use Process Designer connector shapes |

#### Client-Side BO Template

```javascript
// function(oEvent, oPayload, fnSuccess, fnError)
try {
    if (!oPayload.myParam) {
        fnError("myParam is required");
        return;
    }
    var oReturn = { result: oPayload.myParam };
    fnSuccess(oReturn);
} catch (e) {
    fnError(e.message || String(e));
}
```

Every code path must end with `return`, `fnSuccess(...)`, or `fnError(...)`.

#### App Variable Bridge (Server BO output → Client BO)

```javascript
var oControl = sap.ui.getCore().byId("MyControlId");
var oVarModel = oControl ? oControl.getModel("variableHolder") : null;
var vRaw = oVarModel ? oVarModel.getProperty("/myVariableName") : null;
var aItems = Array.isArray(vRaw) ? vRaw
           : (typeof vRaw === "string" ? JSON.parse(vRaw) : []);
```

---

## UI5 Data Binding

### Single Binding Source Rule

**Critical:** Bind each UI property in **exactly one place** — either the view (XML/JS) or the Process Designer. Never both.

| Approach | When to use |
|---|---|
| **View-level** (XML binding `{/path}`) | Static, design-time known binding; stable model structure |
| **Process-level** (mapping nodes) | Dynamic data from connector/BO calls; runtime conditions; data reused across screens |

### Conflict Detection Before Binding

1. Check current binding state of the screen item.
2. Check for existing process mappings to the same property.
3. If both exist, remove one before adding the other.

### Process-Level Binding Patterns

- **Variable → UI Element**: store data in app variables for later use
- **Screen Item → Connector Input**: pass user input to backend calls
- **Connector Output → Variable**: store backend response for reuse
- **Variable → Connector Input**: feed stored data into subsequent calls

---

## Connector Call Rules

### SQL Parameterization

Always use named bind parameters — never string concatenation (SQL injection risk).

```sql
-- Correct
SELECT * FROM Orders WHERE CustomerId = :CustomerId: AND Status = :Status:

-- Wrong — never do this
SELECT * FROM Orders WHERE CustomerId = '" + customerId + "'
```

Use explicit column names, never `SELECT *`. Include aliases for returned fields.

### NULL-Safe SQL Patterns

```sql
-- NULL-safe equality
AND (
  (:ProductId: IS NULL AND fvc.ProductId IS NULL)
  OR
  (:ProductId: IS NOT NULL AND fvc.ProductId = :ProductId:)
)

-- Optional filter — null means "return all", including "applies-to-all" rows
WHERE (
    :productId: IS NULL
    OR fvc.ProductId = :productId:
    OR fvc.ProductId IS NULL
)

-- NULL-safe typed INSERT
CASE
  WHEN :ProductId: IS NULL OR :ProductId: = ''
  THEN NULL
  ELSE CAST(:ProductId: AS INTEGER)
END
```

### Parameter `optional` Flag

- `optional: true`: parameter can be null or empty at runtime
- `optional: false`: parameter always required, never null
- `validateIn: true` treats both `null` and `""` as missing for non-optional params
- Mark autoincrement/PK columns as output only, never input

### Process Node Mapping `IsOptional`

All three gates must agree for nullable values to pass through:
- Connector call `optional` flag
- BO function parameter `isOptional` flag
- Process node input-mapping destination `IsOptional` flag

### Testing

Use `businessobject-function-test` for full chain testing. `connector-call-test` strips empty strings from the HTTP body before sending — a "Mandatory parameter missing" error may be a test tool artifact, not a real runtime error.

### Connector Skill Process (when creating/modifying a connector call)

1. Check MCP server is active
2. Check project exists; create if needed
3. Check connector exists in project; create if needed
4. Determine create vs modify
5. If modify: **read before write** — `simplifier://connector/{Connector}/call/{Call}` — resend all existing parameters, only change what is intended
6. Create or update the connector call
7. Test via `businessobject-function-test`

---

## Validation Strategy

### Client-Side (immediate feedback)

- Format validation (number, date, regex)
- Required field checks
- Range checks (min/max, decimal places)
- Cross-field consistency (ValidFrom ≤ ValidTo)
- Input sanitization (strip non-numeric for money fields)

### Server-Side (data integrity)

- Uniqueness/duplicate checks before insert
- Referential integrity (FK exists?)
- Business rule validation (date overlaps, dependency conflicts)
- Overlap resolution with user confirmation flow

### Overlap Handling Pattern

1. Server BO calls `check{Entity}Conflicts` connector call
2. Returns overlap type: `NONE`, `OVERLAP_BEGINNING`, `OVERLAP_END`, `SPLIT`, `COVER_EXISTING`, etc.
3. Client BO builds warning message
4. UI shows confirmation dialog with planned changes
5. User confirms → re-call with `confirmed: true` flag to execute resolution

### Overlap Handler Output Contract

```javascript
output.success = false;          // not saved yet
output.message = "";
output.savedId = null;
output.requiresWarning = false;  // true = show confirmation dialog
output.plannedChangesText = "";
output.actions = [];
// entity-specific:
output.vehicleTaxOverlapWarning = false;
output.vehicleTaxOverlapMessage = "";
```

---

## Audit Trail Pattern

For master data changes, use the three-layer audit stack:

| BO | Role |
|----|------|
| `bAuditTrail` | Low-level insert into AuditTrail table |
| `bAuditTrailDynamic` | Smart wrapper that auto-diffs before/after states |
| `bAuditTrailClient` | Client-side entry point from process stories |

### Audit Entry Fields

| Field | Description |
|-------|-------------|
| `objectType` | Entity changed (e.g. "Vehicle", "Duration") |
| `actionType` | CREATE, UPDATE, DELETE, DEACTIVATE |
| `actorId` | User login name |
| `businessKeyInt` / `businessKey` | Record identifier |
| `fieldName` | Which field changed |
| `beforeValue*` / `afterValue*` | Typed before/after (String, Integer, Number, Boolean, Date) |
| `source` | Where change originated (default: "UI") |
| `timestampUtc` | When it happened |

---

## Authentication and Login

- Use `SF_Login` standard BO for authentication service discovery.
- Support OAuth2 and SAML via `loginAuthService`.
- After login, fire custom event `checkPermission` for role-based UI adjustment.
- Token-based session management handled by platform.

---

## Process Story Event Trigger Behaviour (Platform 2606.30)

This section documents confirmed runtime behaviour discovered during InterfaceCockpit development. Test every claim on a fresh deploy before trusting it.

### What compiles and fires

| Trigger category | Story node setting | Compiled into | Fires? |
|---|---|---|---|
| `ItemEvent` (widget event) | `Category: "ItemEvent"`, `Event: "press"/"change"` | `{WidgetId}_{event}:function()` in controller | ✅ Always |
| `BuiltInEvent` `appInit` | `Category: "BuiltInEvent"`, `Event: "appInit"` | `builtInEvent_appInit()` on Globals singleton | ❌ Never — `bindBuiltInEvents` only wires orientation/online/offline |
| `ScreenEvent` `show` | `Category: "ScreenEvent"`, `Event: "show"` | Empty stub in `onAfterShow` | ❌ Never |
| `ScreenEvent` `afterInit` | `Category: "ScreenEvent"`, `Event: "afterInit"` | Empty stub in `onAfterRendering` | ❌ Never |
| `ScreenEvent` `beforeFirstShow` | `Category: "ScreenEvent"`, `Event: "beforeFirstShow"` | Empty stub in `onBeforeFirstShow` | ❌ Never |

**Summary**: Only widget-event stories (`ItemEvent`) compile executable code. All `ScreenEvent` and `BuiltInEvent` stories produce empty stubs. This is a platform limitation in version 2606.30.

### How auto-load on startup actually works

The only reliable pattern for auto-loading data when the start screen opens is to call the load BO manually from JavaScript. The platform's `builtInEvent_appInit` function IS compiled correctly (with the right BO call and output mappings) but `bindBuiltInEvents` never calls it.

**Workaround — call `builtInEvent_appInit` via JS from the delegate**:

```javascript
// In a Chrome DevTools or Playwright test you can confirm this works:
const ctrl = sap.ui.getCore().byId('Cockpit').getController();
ctrl.getGlobals().builtInEvent_appInit.call(ctrl);
// → fires POST /client/1.0/executeBO and sets all variables
```

The proper fix at the platform level would be for `bindBuiltInEvents` to also call `builtInEvent_appInit` if the function exists. Until the platform is patched, the recommended approach is:

1. Keep the `BuiltInEvent/appInit` story node (correct semantic intent)
2. Wire the Refresh button story as the user-triggered fallback (this always works)
3. Document auto-load as not supported on initial startup in the app's screen spec

### Variable model — `variableHolder`

App variables are stored in a named JSONModel called `variableHolder` on the `Application` (sap.m.App) control — **not** in the default model.

```javascript
// Confirm at runtime:
sap.ui.getCore().byId('Application').getModel('variableHolder').getData()
```

All bindings that reference app variables must use the `variableHolder>` prefix:

| Binding type | Correct syntax |
|---|---|
| Top-level variable (absolute) | `{variableHolder>/varTotalCount}` |
| Relative path within row template | `{variableHolder>name}` |
| Computed expression | `{= ${variableHolder>/varTotalCount} - ${variableHolder>/varErrorCount}}` |

**Never** use `{varTotalCount}` — that resolves against the default model (null at runtime).

### Table binding pattern (Simplifier-specific)

Simplifier tables do **not** use the standard UI5 `items="{modelPath}"` aggregation binding syntax. Use these two separate properties instead:

```
itemsTemplate: true          ← use first child widget as the row template
itemsPath: "variableHolder>/varMyList"   ← absolute path in variableHolder model
```

Setting `items:` as a string in Simplifier properties will be ignored or produce errors.

### Row cell bindings within templates

Within a table row template, cell bindings must also use the `variableHolder>` prefix — even though they are relative paths:

```
CellName.text → {variableHolder>name}          ✅
CellName.text → {name}                          ❌ resolves against default model (null)
```

Confirmed via:
```javascript
const row = items[0];
row.getBindingContext('variableHolder')  // → path: /varInterfaceList/0
row.getBindingContext()                  // → null (default model has no binding context)
```

### `customHeader` aggregation

The `Page` widget's `customHeader` aggregation requires a `Bar` element. Placing a `Button` or `Title` directly in `customHeader` causes:

```
Tried to add an array of controls to a single aggregation
```

Correct structure:
```
Page
└── customHeader: Bar
    ├── contentLeft: Button (back button)
    └── contentMiddle: Title
```

### `screen-item-update` always times out

The `screen-item-update` MCP tool always reports `Command CmdApplyProperties timed out after 30000ms`. The change IS applied on the server side despite the timeout. Always verify with `screen-read` after update — never trust the error message alone.

### Process story mapping — `resolveConnectorResult`

When a BO returns data in `output.result.fieldName` (connector-style), use `resolveConnectorResult` in the process story output mapping. When a BO returns top-level fields (`output.fieldName`), the process story maps them directly by parameter name.

### Filter dropdown → table pattern

Wiring a `Select` dropdown to filter a table is a two-part design. Both parts must be correct:

#### 1. Process story (what triggers the BO call)

Use an `ItemEvent` `change` story on the Select widget. In the SubProcess Input, use a `ScreenItem` source to read `selectedKey` from the Select, plus a `Variable` source for any other active filter that must be preserved:

```
SubProcess Input for flt-bo-status:
  ScreenItem(FilterStatusSelect.selectedKey) → Parameter(filterStatus)
  Variable(varFilterScope)                   → Parameter(filterScope)

SubProcess Output:
  Parameter(interfaces)    → Variable(varInterfaceList)
  Parameter(totalCount)    → Variable(varTotalCount)
  Parameter(errorCount)    → Variable(varErrorCount)
  Parameter(warningCount)  → Variable(varWarningCount)
  Parameter(echoFilterStatus) → Variable(varFilterStatus)   ← persist for Refresh
  Parameter(echoFilterScope)  → Variable(varFilterScope)    ← persist for Refresh
```

**Critical:** The `ScreenItem` source must reference the widget by **UUID** (not name) when building graphs via MCP. Use `screen-read` to get widget UUIDs first.

**Why `varFilterStatus` must be written back:** Refresh reads `varFilterStatus` to re-apply the active filter. If the Filter story never writes the variable, Refresh always sends empty and resets to "show all".

#### 2. BO design — echo filter values as output

The BO must echo back the applied filter values so the story can persist them to variables:

```javascript
var fs = (input.filterStatus && input.filterStatus !== '') ? input.filterStatus : '';
var fsc = (input.filterScope && input.filterScope !== '') ? input.filterScope : '';

// ... filter rows in-memory ...

output.echoFilterStatus = fs;   // story maps → varFilterStatus
output.echoFilterScope = fsc;   // story maps → varFilterScope
```

Add `echoFilterStatus` and `echoFilterScope` as optional String output parameters on the BO function.

#### 3. Refresh story — reads variables, not ScreenItems

The Refresh story input must read from **variables** (not ScreenItems), so the last filter state survives:

```
SubProcess Input for Refresh BO call:
  Variable(varFilterStatus) → Parameter(filterStatus)
  Variable(varFilterScope)  → Parameter(filterScope)
```

#### 4. `getItemValue` catch-block platform bug (2606.30)

The compiled `getItemValue` has a broken catch block — missing `return` — so any exception during property lookup returns `undefined` silently. The BO then receives an empty string and returns all rows. Symptom: filter dropdown fires the story (visible in logs) but BO always receives `filterStatus: empty`.

Diagnosis: check the connector call log entry — if `filterStatus: empty: true` despite the user changing the dropdown, this is the bug. Workaround: the echo-output pattern above compensates because subsequent Refresh calls will use the variable instead of re-reading the ScreenItem.

---

## OData V2 Mock Service Pattern

When creating a Simplifier mock service via MCP:

### Fixed Rules

- Always use schema `sMockData` — never create prompt-specific schemas.
- Always prefix OData entity sets with `es` → `esCarModel`, `esVehicles`.
- Always relate to a Simplifier project — ask if not provided.
- Use connector `cMockData` for all CRUD calls.
- Use server BO `bMockData` for all CRUD functions.

### MCP Creation Sequence

1. Resolve/create project relation
2. Create or reuse schema `sMockData` via `schema-create`
3. Add table via `schema-add-table`
4. Add columns via `schema-add-column` (one per column)
5. Add indexes via `schema-add-index` for common filter fields
6. Validate: `schema-validate`
7. Diff: `schema-diff` — stop if destructive changes detected
8. Deploy: `schema-deploy` → confirm with `schema-deploy-status`
9. Seed data: `schema-data-create`
10. Verify: `schema-data-list` + `schema-data-get`
11. Create/update connector calls in `cMockData`
12. Create/update BO functions in `bMockData`

### Safety Rules

- Never delete schemas, tables, columns, or data without explicit user instruction.
- Always validate before deploy.
- Prefer `schema-data-create` over direct SQL.
- Do not invent the OData V2 service root URL — derive it from deployed schema metadata.
