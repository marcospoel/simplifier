# Security Test Checklist

> Produced by `security` agent. Executed by `security-tester` (API/backend tests) and `ui-tester` (UI security behaviour tests).

## InterfaceCockpit v1 — MBI-2055

### For `security-tester` (API / BO / backend)

| # | Test | OWASP | Expected result | Status |
|---|---|---|---|---|
| ST-01 | Call `getInterfaceStatuses` without a session token | A01 | 401 / redirect to login | TODO |
| ST-02 | Call `getInterfaceDetail` without a session token | A01 | 401 / redirect to login | TODO |
| ST-03 | Call `triggerRetry` without a session token | A01 | 401 / redirect to login | TODO |
| ST-04 | Call `triggerRetry` authenticated as InterfaceOperator | A01 | Access denied / error response | TODO |
| ST-05 | Send SQL injection payload in `filterStatus` param | A03 | Empty result or validation error — no DB error, no stack trace | TODO |
| ST-06 | Send SQL injection payload in `filterScope` param | A03 | Empty result or validation error | TODO |
| ST-07 | Send null / missing `interfaceId` to `triggerRetry` | A03 | Validation error — not a crash | TODO |
| ST-08 | Send 10,000-character string as `filterStatus` | A03 | Validation error — not a crash | TODO |
| ST-09 | Check HTTP response headers for CSP, X-Frame-Options, HSTS | A05 | All three present | TODO |
| ST-10 | Check CORS header on API endpoints | A05 | `Access-Control-Allow-Origin` not `*` | TODO |
| ST-11 | Inspect BO response bodies for token/credential content | A07 | No tokens, passwords, or secrets in any response | TODO |
| ST-12 | Attempt SSRF: pass `http://localhost/admin` as a connector input (if applicable) | A10 | Request rejected at input validation | TODO |

### For `ui-tester` (UI security behaviour)

| # | Test | OWASP | Expected result | Status |
|---|---|---|---|---|
| UT-S01 | Load app without login — verify redirect to login page | A01 | Redirect to Simplifier login | TODO |
| UT-S02 | Log in as InterfaceOperator — verify TriggerRetryBtn is absent or disabled | A01 | Button not visible or disabled | TODO |
| UT-S03 | Inspect browser console for token/credential leakage | A07 | No tokens in console output | TODO |
| UT-S04 | Inspect DOM for hidden sensitive fields containing credentials | A07 | No credential data in DOM | TODO |
| UT-S05 | Verify network responses do not expose internal error stack traces | A05 | Error responses contain only user-safe messages | TODO |

## How to update this checklist

After every new feature, the `security` agent adds test cases to this file, then briefs `security-tester` and `ui-tester` to run the new cases. Results are recorded in `docs/security/test-results-<date>.md`.
