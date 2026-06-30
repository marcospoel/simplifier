# Bug Report: `getItemValue` catch block silently swallows control lookup failures

**Product:** Simplifier Low-Code Platform  
**Version:** 2606.30 (MC 26-06.30)  
**Component:** Process Designer — compiled application controller (`Component-preload.js`)  
**Severity:** Medium — data loss (filter/search inputs silently ignored)  
**Reported by:** Marco Spoel, Deutsche Telekom / T-Systems  
**Date:** 2026-06-30

---

## Summary

The `getItemValue` helper compiled into every Simplifier app's controller is missing a `return` statement in its catch block. When the property getter lookup throws for any reason, the function returns `undefined` instead of the property value. Callers receive `undefined` (coerced to empty string), so any process story that reads a widget property via `ScreenItem` source silently sends an empty value to the Business Object.

The most visible symptom: a `Select` dropdown with a `change` event story appears to fire correctly (the BO is called), but the BO always receives an empty filter value and returns all rows — the table never filters.

---

## Affected Code

**File:** `Component-preload.js` (compiled per app, served at `/appDirect/{AppName}/Component-preload.js`)  
**Function:** `getItemValue` on the controller prototype

### Buggy compiled output (2606.30)

```javascript
getItemValue: function(e, t, i) {
  try {
    var n = this.getScreen(e).byId(t);
    var s = n.getMetadata().getProperty(i)._sGetter;
    return n[s]();
  } catch (n) {
    this.getScreen(e).byId(t).getProperty(i)  // ← missing return
  }
}
```

### Expected correct implementation

```javascript
getItemValue: function(e, t, i) {
  try {
    var n = this.getScreen(e).byId(t);
    var s = n.getMetadata().getProperty(i)._sGetter;
    return n[s]();
  } catch (n) {
    return this.getScreen(e).byId(t).getProperty(i)  // ← return required
  }
}
```

---

## How to Reproduce

### Prerequisites

- Simplifier platform version 2606.30
- A deployed app with at least one `Select` widget on a screen
- A process story wired to the `change` event of that `Select`

### Steps

1. Create a Simplifier app with a `Select (1.96)` widget named `FilterStatusSelect` on a screen named `Cockpit`.
2. Add items to the Select: `OptAll` (key `""`), `OptOK` (key `"OK"`), `OptErr` (key `"ERROR"`).
3. Create a process story with:
   - Subscribe node: `ItemEvent` / `change` on `FilterStatusSelect`
   - DataObject node: any Business Object call
   - SubProcess Input: `ScreenItem(Cockpit, FilterStatusSelect, selectedKey)` → `Parameter(filterValue)`
4. Deploy the app.
5. Open the app in a browser, open DevTools Network tab.
6. Change the `FilterStatusSelect` dropdown to `"OK"`.
7. Observe the POST request to `/client/1.0/executeBO`.

### Expected result

Request body contains `"filterValue": "OK"`.

### Actual result

Request body contains `"filterValue": ""` (empty string) or the parameter is marked `empty: true` in the server logs.

### Verification via Simplifier logs

In **Monitoring → Logging**, find the `Exec_BusinessObject` log entry created when the dropdown changed. The details JSON shows:

```json
{
  "parameters": {
    "filterValue": {
      "dataType": "String",
      "empty": true       ← value was not read from the widget
    }
  }
}
```

---

## Root Cause Analysis

### Call chain

When the `change` event fires, the compiled controller calls:

```
FilterStatusSelect_change(event)
  → getItemValue("Cockpit", "FilterStatusSelect", "selectedKey")
      → getScreen("Cockpit")            // returns sap.ui.getCore().byId("Cockpit")
      → screen.byId("FilterStatusSelect") // scoped lookup
      → control.getMetadata().getProperty("selectedKey")._sGetter
      → control[getter]()               // e.g. control.getSelectedKey()
```

### When the try block succeeds

Everything works. `getSelectedKey()` returns the current key and the value reaches the BO.

### When the try block throws

If the `getMetadata().getProperty(i)._sGetter` access throws (e.g. because `_sGetter` is undefined for a non-standard property, or because `getMetadata()` is not available on the resolved control instance), execution falls to the catch block.

The catch block calls `this.getScreen(e).byId(t).getProperty(i)` — the `sap.ui.ManagedObject#getProperty` fallback. This is correct, but the result is **not returned**. JavaScript `catch` blocks without a `return` produce `undefined`, which propagates as the return value of `getItemValue`.

The caller assigns this to the BO input parameter:

```javascript
e["filterStatus"] = this.getItemValue("Cockpit", "FilterStatusSelect", "selectedKey");
// e["filterStatus"] === undefined  →  BO receives empty string
```

### Why it triggers for `Select.selectedKey`

`sap.m.Select` in OpenUI5 1.96 exposes `selectedKey` as a regular property with a `_sGetter` of `"getSelectedKey"`. In most cases the try block succeeds. However, if the screen has not fully rendered when the event fires, or if the control's metadata chain differs from what the compiled code expects (e.g. wrapped in a Simplifier widget proxy), the metadata lookup can fail, routing through the broken catch path.

In the InterfaceCockpit app on 2606.30, the `Exec_ConnectorCall` log consistently shows `filterStatus: empty: true` for all UI-triggered calls, confirming the catch path is always taken for this control.

---

## Impact

| Scenario | Impact |
|---|---|
| Filter dropdown (`Select`) wired to a table via process story | Table never filters — always shows all rows |
| Search input (`Input`) read via `ScreenItem` source | Search query silently ignored — always shows all results |
| Any ScreenItem source reading a widget property in SubProcess Input | Property value is empty; BO processes as if no input was provided |

The story fires correctly (the BO IS called). The symptom is purely data: the BO receives empty values and behaves as if no filter was applied.

---

## Workaround

Until the platform is patched, use the **echo-output pattern**:

### 1. BO echoes back the applied filter value

Add optional output parameters `echoFilterStatus` (String) and `echoFilterScope` (String) to the BO function. The BO reads `input.filterStatus`, normalises it, uses it for filtering, and writes it back to `output.echoFilterStatus`:

```javascript
var fs = (input.filterStatus && input.filterStatus !== '') ? input.filterStatus : '';
// ... filter logic ...
output.echoFilterStatus = fs;
```

### 2. Filter story output maps echo values to app variables

In the SubProcess Output of the filter story's DataObject node, add:

```
Parameter(echoFilterStatus) → Variable(varFilterStatus)
Parameter(echoFilterScope)  → Variable(varFilterScope)
```

### 3. Refresh and subsequent calls read from Variables, not ScreenItems

The Refresh story and any other story that re-calls the same BO should read from `Variable(varFilterStatus)` instead of `ScreenItem(FilterStatusSelect.selectedKey)`. Since the variable is written by the echo-output mapping, it correctly reflects the last user selection.

### Why this works despite the bug

When `getItemValue` fails, the BO receives `filterStatus: ""` and returns all rows. But `echoFilterStatus: ""` is then written to `varFilterStatus`. On the next Refresh, `varFilterStatus = ""` → returns all rows. When the dropdown fires again and by chance succeeds (or after a platform patch), `varFilterStatus` gets the correct value and Refresh re-applies it.

This pattern is also correct by design: app variables are the right place to persist UI state across multiple BO calls.

---

## Suggested Platform Fix

In the code generator that emits `getItemValue`, add `return` to the catch block:

```javascript
// Before (buggy):
} catch (n) {
  this.getScreen(e).byId(t).getProperty(i)
}

// After (fixed):
} catch (n) {
  return this.getScreen(e).byId(t).getProperty(i)
}
```

Alternatively, consolidate both paths:

```javascript
getItemValue: function(screenName, itemId, propName) {
  var ctrl = this.getScreen(screenName).byId(itemId);
  try {
    var getter = ctrl.getMetadata().getProperty(propName)._sGetter;
    return ctrl[getter]();
  } catch (e) {
    return ctrl.getProperty(propName);
  }
}
```

---

## References

- Simplifier Logging entry format: `Exec_BusinessObject` / `Exec_ConnectorCall` (Monitoring → Logging)
- Platform version: MC 26-06.30 (tile in Simplifier dashboard → Instance Information)
- App where confirmed: `InterfaceCockpit` on `tsystems-eval.simplifier.cloud`
- Related conventions: `docs/simplifier-conventions.md` → "Filter dropdown to table pattern"
