# Simplifier Apps

> Maintained by `doc-writer` agent. Updated after every app or screen change.

<!-- Add entries as apps are created on the platform. Format:

## <App Name>

**Project:** <Simplifier project>
**Purpose:** <one sentence>
**Screens:** [list]

### Screens

#### <Screen Name>
See [../screens/<screen-name>.md](../screens/<screen-name>.md)
-->

## InterfaceCockpit

**Project:** InterfaceCockpit  
**App ID:** `5D18C05B4C741790C73E81A989D7877A5011EF00759F52BC37249D07C170728A`  
**Purpose:** Real-time monitoring cockpit for all connected system interfaces — shows availability, last success/failure, error details, retry status, and business function. Allows manual retry triggering.  
**Jira:** MBI-2055 (epic MBI-2032 / WP11d — LC-INT-08)  
**Deployed to:** tsystems-eval.simplifier.cloud

### Screens

#### Cockpit (start screen)
See [../screens/InterfaceCockpit-Cockpit.md](../screens/InterfaceCockpit-Cockpit.md)

#### InterfaceDetail
See [../screens/InterfaceCockpit-InterfaceDetail.md](../screens/InterfaceCockpit-InterfaceDetail.md)

#### Popups
Reserved for dialogs (none implemented yet).

### App Variables

| Variable | Type | Purpose |
|---|---|---|
| `varInterfaceList` | sInterfaceStatus_col | Loaded interface list bound to Cockpit table |
| `varSelectedInterface` | sInterfaceStatus | Full detail object for selected interface |
| `varSelectedInterfaceId` | String | ID of selected interface (set on detail load; used by Trigger Retry) |
| `varFilterStatus` | String | Current status filter value |
| `varFilterScope` | String | Current scope filter value |
| `varTotalCount` | Integer | Total interface count for summary tile |
| `varErrorCount` | Integer | Error count for summary tile |
| `varWarningCount` | Integer | Warning count for summary tile |

### Process Stories

| Story | Trigger | BO called |
|---|---|---|
| `(MBI-2055) [Cockpit] Load Data` | Cockpit screen show | `bInterfaceStatus.getInterfaceStatuses` |
| `(MBI-2055) [Cockpit] Refresh` | HeaderRefreshBtn press | `bInterfaceStatus.getInterfaceStatuses` |
| `(MBI-2055) [Cockpit] Filter` | FilterStatusSelect / FilterScopeSelect change | `bInterfaceStatus.getInterfaceStatuses` |
| `00.3 Navigation` | InterfaceRow / CellDetailBtn press | — (navigate to InterfaceDetail) |
| `(MBI-2055) [InterfaceDetail] Load Data` | InterfaceDetail screen show | `bInterfaceStatus.getInterfaceDetail` |
| `(MBI-2055) [InterfaceDetail] Trigger Retry` | TriggerRetryBtn press | `bInterfaceStatus.triggerRetry` |
| `(MBI-2055) [InterfaceDetail] Back` | DetailBackBtn press | — (navigate back to Cockpit) |
