# RBAC — Role-Based Access Control

> Maintained by `security` agent. Update after every role, permission, or user assignment change.

## Roles

| Role | ID (prefix) | Type | Purpose | Assigned users |
|---|---|---|---|---|
| `InterfaceAdmin` | `0C697616...` | Custom | Full access to InterfaceCockpit — view, execute, edit app and BO | `marco.spoel@t-systems.com` |
| `InterfaceOperator` | `C6F890F1...` | Custom | Read-only access — view interface status and details only | (assign as needed) |
| `AppInterfaceCockpit` | `ED59F256...` | Auto-generated | Platform-managed app role — required for BO execution check | all app users (auto-assigned at login) |

> **Note (2026-06-28):** The root cause of the original "Missing permissions" error was `anonymousAccessEnabled: true` — the app was deployed without any roles assigned. The fix was assigning InterfaceAdmin, InterfaceOperator, and AppInterfaceCockpit to the app via `POST /UserInterface/api/app-roles/InterfaceCockpit`, then redeploying. The AppInterfaceCockpit auto-generated role has bInterfaceStatus execute+view permissions as a secondary safeguard.

## Permission matrix

| Resource | Characteristic | InterfaceAdmin | InterfaceOperator |
|---|---|---|---|
| App: InterfaceCockpit | execute | ✓ | ✓ |
| App: InterfaceCockpit | view | ✓ | ✓ |
| App: InterfaceCockpit | edit | ✓ | ✗ |
| App: InterfaceCockpit | delete | ✗ | ✗ |
| App: InterfaceCockpit | releasemeta | ✗ | ✗ |
| BO: bInterfaceStatus | execute | ✓ | ✓ |
| BO: bInterfaceStatus | view | ✓ | ✓ |
| BO: bInterfaceStatus | edit | ✗ | ✗ |
| BO: bInterfaceStatus | delete | ✗ | ✗ |

## Functional access (derived from permissions + app design)

| Function | InterfaceAdmin | InterfaceOperator |
|---|---|---|
| View interface list (Cockpit) | ✓ | ✓ |
| Filter by status / scope | ✓ | ✓ |
| Refresh cockpit | ✓ | ✓ |
| Navigate to interface detail | ✓ | ✓ |
| View interface detail | ✓ | ✓ |
| Trigger retry | ✓ | ✗ (BO permission denied) |
| Edit app screen design | ✓ | ✗ |

## Assignment policy

- Users are assigned to roles in the Simplifier User Management UI (`/UserInterface/index.html#/Role`)
- A user may hold multiple roles — effective permissions are the union of all role permissions
- Role assignments are reviewed quarterly by the application owner
- The `security` agent must update this document after any role or assignment change
