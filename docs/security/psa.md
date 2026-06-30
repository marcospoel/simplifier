# PSA — Privacy & Security Assessment (Deutsche Telekom)

> Maintained by `security` agent. This document feeds the official T-Systems PSA process.  
> Update after every architecture change, new integration, role change, or data flow change.

**Application:** InterfaceCockpit  
**Project:** MBI-2055 / WP11d / LC-INT-08  
**Owner:** Marco Spoel (marco.spoel@t-systems.com)  
**Last updated:** 2026-06-28  
**Status:** Draft

---

## 1. Asset inventory

| Asset | Description | Classification | Stored where |
|---|---|---|---|
| Interface monitoring data | Status, availability, error codes, retry counts per integration | Internal | Simplifier platform DB / SAP backend |
| User session tokens | SimplifierToken for authenticated sessions | Confidential | `.env` file (local, gitignored) / browser session |
| Connector credentials | Login credentials for backend system connectors | Confidential | Simplifier Connector Login store (encrypted at rest) |
| Application code | BO JavaScript, screen definitions | Internal | Simplifier platform + git repository |

**PII processed:** None in v1 — interface monitoring data does not include personal data.

---

## 2. Authentication & Authorisation

**Authentication mechanism:** SimplifierToken (session token) obtained from T-Systems Simplifier instance. Expires on logout or session timeout.

**Authorisation mechanism:** Simplifier role-based access control (RBAC). Roles enforced at both platform and BO level.

**Role matrix:** See [rbac.md](rbac.md).

**Key controls:**
- Tokens stored in `.env` only — never committed to git
- BO-level role checks enforce authorisation independent of UI controls
- Session tokens are not logged

---

## 3. Data flows

| Flow | From | To | Data | Classification | Transport |
|---|---|---|---|---|---|
| Interface list load | Browser → Simplifier BO | SAP / interface monitoring backend | Filter params → interface status list | Internal | HTTPS |
| Interface detail load | Browser → Simplifier BO | SAP / interface monitoring backend | Interface ID → full detail record | Internal | HTTPS |
| Trigger retry | Browser → Simplifier BO (Admin only) | SAP / interface monitoring backend | Interface ID → retry command | Internal | HTTPS |
| Authentication | Browser → Simplifier platform | — | Username + password → session token | Confidential | HTTPS |

---

## 4. Encryption

| Layer | Control |
|---|---|
| Transport | TLS 1.2+ enforced on `tsystems-eval.simplifier.cloud` |
| Connector credentials at rest | Encrypted by Simplifier platform (AES) |
| Session tokens | Transmitted over HTTPS only; not persisted client-side beyond session |
| Git repository | No secrets committed (enforced by `.gitignore` and pre-commit checks) |

---

## 5. Third-party components

See [components.md](components.md) for full inventory with versions and CVE status.

Key components:
- OpenUI5 1.96.40 (SAP, Apache 2.0)
- Simplifier Low-Code Platform (Simplifier AG, proprietary)
- Node.js (runtime for server BOs, version determined by Simplifier platform)

---

## 6. Logging & monitoring

| What is logged | Where | Who has access | Retention |
|---|---|---|---|
| Security events (access denied, invalid input) | Simplifier platform log (`Simplifier.Log.*`) | Application owner, platform admin | Per platform policy |
| Application errors | Simplifier error log | Application owner | Per platform policy |
| User actions (BO calls) | Simplifier audit log | Platform admin | Per platform policy |

**What must NOT be logged:** Session tokens, passwords, PII, connector credentials.

See [logging-policy.md](logging-policy.md) for full policy.

---

## 7. Incident response

Security incidents are escalated via the T-Systems standard incident response process:
1. Detect: monitoring alert or user report
2. Contain: disable affected role / revoke token via Simplifier User Management
3. Investigate: review Simplifier audit logs
4. Escalate: notify application owner (Marco Spoel) and T-Systems security team
5. Remediate: patch BO code or connector config; re-test with `security-tester`
6. Document: update this PSA and `docs/security/cve-watchlist.md`

---

## 8. Vulnerability management

- CVE watch is maintained in [cve-watchlist.md](cve-watchlist.md)
- Critical CVEs: patch within 5 business days
- High CVEs: patch within 30 days
- Medium/Low: patch in next scheduled release
- Responsible: `security` agent flags CVEs; application owner approves remediation

---

## 9. Privacy (GDPR)

**PII processed in v1:** None.  
Interface monitoring data (system names, error codes, timestamps) does not constitute personal data under GDPR.

If future features introduce user-identifiable data (e.g. user IDs linked to retry actions), this section must be updated with:
- Lawful basis for processing
- Data minimisation measures
- Retention and deletion policy
- Data subject rights procedure

---

## 10. Open items / risks

| # | Risk | Severity | Status | Owner |
|---|---|---|---|---|
| R-01 | SimplifierToken expiry not enforced server-side | Medium | Open | Platform (Simplifier AG) |
| R-02 | No HTTP CSP header on Simplifier platform | Medium | Open | Platform admin |
| R-03 | BO-level role check not yet implemented in triggerRetry | High | Open | simplifier-bo-developer |
