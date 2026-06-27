# TARDIS — Telekom Architecture for Decoupling and Integration of Services

**Source:** https://mdx.pages.devops.telekom.de/docs/tools/middleware/tardis/  
**Note:** The DX Documentation site is reaching end-of-life; new content is being migrated to https://docs.devops.telekom.de/

---

## Overview

TARDIS is DT-IT's standard API management and integration platform. It is a suite of components that together provide API gateway, authentication, event streaming, monitoring, and tracing for all services in the Telekom ecosystem.

**Rule for Simplifier:** All calls from Simplifier to SAP or any Telekom internal system must route through TARDIS (StarGate). Direct calls from Simplifier to SAP are not permitted.

---

## Components

| Component | Role |
|---|---|
| **StarGate** | Managed API gateway — routes all inbound and outbound API traffic |
| **Iris** | M2M Identity Provider — OAuth 2.0 Client Credentials authentication |
| **Rover / roverctl** | Configuration control plane — declarative YAML-based API registration |
| **Horizon** | Pub/Sub event bus — async event delivery between systems |
| **Chevron** | RBAC permission resolver — maps user roles to fine-grained permissions |
| **Raccoon** | Metrics and monitoring — Grafana dashboards |
| **Drax** | Distributed tracing — OpenTelemetry-compatible, visualised in Jaeger/Grafana |
| **Spectre** | API wiretapping — auditable traffic capture with provider+consumer consent |
| **MissionControl** | UI portal — manage teams, view APIs, approve subscriptions |

---

## StarGate

StarGate is the gateway through which all API traffic flows. Key properties:

- **Gateway Mesh** — location-transparent: a service on AWS, CaaS, GCP, or on-premise can be consumed by any consumer without knowing where the service runs
- **OAuth2 mandatory** — every API on StarGate is automatically secured with OAuth2 Client Credentials; there is no unauthenticated access
- **Request size limit** — 10 MB per request; header size limit 16 KB total
- **Gateway timeout** — 60 seconds, not configurable
- **Zone selection** — consumers must use the StarGate zone nearest to their application:
  - `aws` — for applications on AWS
  - `cetus` — for applications on CaaS, AppAgile, Biere, Hitnet
  - `gaia` — for applications on GCP

### Standard headers set by StarGate

| Header | Purpose |
|---|---|
| `X-Forwarded-*` | Forwarding info (IP, host, proto) |
| `X-B3-*` | Distributed trace IDs |
| `Realm` | IDP realm |
| `Environment` | Environment of the API (playground/preprod/prod) |

### Authentication flow (M2M)

1. Consumer requests access token from **Iris** using `client_id` / `client_secret` (Client Credentials flow)
2. Consumer caches the token and includes it as `Authorization: Bearer <token>` on every request to StarGate
3. StarGate validates the token (signature, expiry, issuer, consumer whitelist)
4. StarGate forwards request to upstream with a **Gateway Token** in the `Authorization` header (Last Mile Security)
5. Upstream service must validate the Gateway Token

### Last Mile Security (Gateway Token)

StarGate replaces the consumer's access token with a Gateway Token when forwarding to the upstream. The upstream **must** validate this token:

Required validation steps:
1. Verify issuer (`iss`) against trusted StarGate issuer for the zone
2. Verify token signature
3. Verify token not expired (`exp`)
4. Verify `operation` matches the HTTP method used
5. Verify `requestPath` starts with the API's registered `basePath`

Gateway Token example:
```json
{
  "clientId": "eni--hyperion--petstore",
  "sub": "pets",
  "azp": "stargate",
  "typ": "Bearer",
  "env": "preprod",
  "operation": "GET",
  "requestPath": "/petstore/v1/pets",
  "iss": "https://stargate-preprod.live.dhei.telekom.de/auth/realms/default",
  "exp": 1619099274,
  "iat": 1619098974
}
```

---

## Iris — M2M Identity Provider

Iris is the OAuth 2.0 server for machine-to-machine communication.

### Token request (preferred: BASIC auth method)

```
POST https://iris-<env>.live.dhei.telekom.de/auth/realms/default/protocol/openid-connect/token
Authorization: Basic base64(client_id:client_secret)
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
```

**Important rules:**
- Cache and reuse the token until it expires — do not request a new token for every call
- Do not include any `scope` in the token request (Iris does not support client scopes)
- Refresh tokens are not issued (disabled since May 2022)
- Use the same `clientAuthMethod` consistently for a given `client_id` (BASIC or BODY, never mixed)

### Client credentials location

Client `client_id` and `client_secret` are found in **MissionControl** under Application Details.

---

## Rover — Configuration Control Plane

Rover is the declarative tool that registers APIs and subscriptions with TARDIS. Configuration is expressed in a `rover.yaml` file and applied via the `roverctl` CLI (typically as a CI/CD pipeline step).

### rover.yaml structure

```yaml
apiVersion: tcp.ei.telekom.de/v1
kind: Rover
metadata:
  name: my-application          # unique application name: hub--team--appname format
spec:
  zone: cetus                   # stargate zone nearest to application
  icto: icto-12345              # mandatory ICTO number (lowercase)
  exposures:
    - basePath: '/hub/service/v1'          # must match OAS basePath
      upstream: 'https://my-service.internal'
      approval: 'SIMPLE'        # SIMPLE (default) | AUTO | FOUREYES
      visibility: ENTERPRISE    # ENTERPRISE (default) | ZONE | WORLD
  authentication:
    m2m:
      clientAuthMethod: basic   # basic (default) | body
  subscriptions:
    - basePath: '/other/service/v1'
```

### Key rover.yaml rules

- `basePath` must follow the pattern `/{HubName}/[serviceName]/[majorVersion]`
- `icto` must be provided in lowercase for every exposure and subscription
- `visibility: WORLD` exposes the API on SpaceGate (internet-accessible)
- Rover is **declarative** — removing an entry from the file and re-applying deletes it
- Secrets (e.g. `clientSecret`) should use placeholder syntax `${ENV_VAR_NAME}` and be injected via CI/CD variables

### roverctl commands

```bash
roverctl apply -f rover.yaml           # create/update exposures and subscriptions
roverctl apply -f apispec/             # upload OpenAPI specification
roverctl delete -f rover.yaml          # delete entire application
roverctl get-info -f rover.yaml --output=out.yaml --format YAML
roverctl reset-secret -a <app-name>    # rotate client secret
```

### Onboarding steps

1. Self-onboard via MissionControl — create Hub and Team
2. Receive Rover token (from MissionControl → Team Management → Rover token)
3. Set `ROVER_TOKEN` CI/CD variable in GitLab
4. Create `rover.yaml` and `.gitlab-ci.yml`
5. Apply via pipeline: `roverctl apply -f rover.yaml`

---

## Horizon — Pub/Sub Event Bus

Horizon provides asynchronous event delivery between systems via REST APIs. It is the Pub/Sub integration pattern within TARDIS — **not a Kafka-as-a-service**.

**Use Horizon when:**
- A provider publishes events and multiple consumers subscribe to them
- The provider does not need to know who receives the events or in what order

**Do not use Horizon when:**
- Acknowledgement of delivery to specific consumers is required
- Guaranteed processing time is required

### Key limitations

- Events are delivered **at-least-once** (duplicates possible but infrequent)
- Order is **not guaranteed**
- Maximum retention: **7 days** — events older than 7 days are deleted
- No Kafka or exactly-once delivery

### Consumer requirements

A consumer must either:
- Expose a **callback endpoint** that supports `POST` (receive event) and `HEAD` (health check), OR
- Open an **SSE stream** to Horizon's SSE component

The `HEAD` health check is mandatory for callback consumers — Horizon uses it to determine whether to deliver events.

---

## Environments

| Environment | Purpose |
|---|---|
| Playground | Sandbox — safe to experiment, no partner system connections |
| Preprod | Integration/testing with partner systems |
| Prod | Production |
| Virtual (AV, FRV, RV, CIT2, CIT4, SIT, Bond) | OSS/BSS release testing and reference environments |

Iris base URL pattern: `https://iris-<env>.live.dhei.telekom.de/auth/realms/default`  
StarGate URL and trusted issuers per zone/environment: see TARDIS Environment page in MissionControl.

---

## Relevance for Simplifier (ReFIT project)

### How Simplifier connects to SAP OFI / Abiscon

All Simplifier Connectors that call SAP or Abiscon APIs must be configured to call the **TARDIS StarGate endpoint**, not the SAP or Abiscon host directly.

Connector configuration in Simplifier:
- **Base URL:** StarGate URL for the relevant zone and environment (e.g. CETUS for on-premise/CaaS-hosted services)
- **Authentication:** OAuth2 Client Credentials — Iris token endpoint, `client_id`, `client_secret` from MissionControl
- **Headers:** `Authorization: Bearer <iris_token>` on every request

### Registration requirement

Before Simplifier can call any API via TARDIS, the Simplifier application must be **registered in Rover** as a consumer:

```yaml
# rover.yaml for the Simplifier Low-Code application
apiVersion: tcp.ei.telekom.de/v1
kind: Rover
metadata:
  name: simplifier-refit          # application name
spec:
  zone: cetus                     # zone where Simplifier is hosted
  icto: icto-<assigned-icto>
  authentication:
    m2m:
      clientAuthMethod: basic
  subscriptions:
    - basePath: '/abiscon/processapi/v1'
    - basePath: '/sap/ofi/billingdocs/v1'
    # ... one entry per API being consumed
```

Subscription requires **approval** from the API provider team before calls succeed.

### Token caching in Simplifier Business Objects

Simplifier BOs that call TARDIS APIs must:
1. Request an Iris token once and cache it (in BO state or a cache BO) for the token's lifetime
2. Detect token expiry (HTTP 401 from StarGate) and refresh the token
3. Never request a new token for every individual API call — this puts excessive load on Iris

### Gateway Token validation (if Simplifier exposes APIs via TARDIS)

If Simplifier exposes APIs for other systems to consume via StarGate, the Simplifier BO receiving the call must validate the Gateway Token forwarded by StarGate (verify issuer, signature, expiry, operation, requestPath).

---

## MissionControl

MissionControl is the TARDIS web UI:
- View and manage teams, applications, and API subscriptions
- Approve or reject incoming subscription requests
- Retrieve `client_id`, `client_secret`, and Rover token
- Monitor application API usage

Access: https://developer.telekom.de (login with EMEA1/EMEA2 or Collaboration Network account)
