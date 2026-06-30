# Component Inventory

> Maintained by `security` agent. Update when platform or library versions change.

| Component | Version | Vendor | License | Known CVEs | Status |
|---|---|---|---|---|---|
| OpenUI5 | 1.96.40 | SAP | Apache 2.0 | Check NVD for `openui5` | Review needed |
| Simplifier Platform | (check instance version) | Simplifier AG | Proprietary | Check vendor advisories | Review needed |
| Node.js (BO runtime) | (platform-managed) | OpenJS Foundation | MIT | Check NVD for `node.js` | Review needed |

## How to check for CVEs

```
NVD search: https://nvd.nist.gov/vuln/search?query=<component>
SAP Security Notes: https://support.sap.com/en/my-support/knowledge-base/security-notes-news.html
MITRE CVE: https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=<component>
```

Update the Status column after each review:
- **OK** — no relevant CVEs for our version
- **Monitor** — CVE exists but not applicable to our usage
- **Action required** — CVE affects our version and usage; remediation planned
