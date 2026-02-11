# PR1.02 — Integrations, Auth Model, and Data Usage

## Integration inventory (current state)

| Provider | Legacy location(s) | Protocol | Auth mechanism | Retry / idempotency behavior | Secret location pattern |
|---|---|---|---|---|---|
| HPCI (payments) | `LandcorSystem/Landcor.Integration/Billing/HPCIPaymentProcessor.cs` | Provider API (service adapter) | API credentials/certs (legacy config-bound) | Appears synchronous; idempotency controls need explicit keying in modernization | Legacy `Web.config` / `App.config` patterns |
| BC Online | `LandcorSystem/BCOnlineService/*`, `LandcorSystem/Landcor.Integration/*` | Service/API integration | Credential-based integration account | Worker-style retry likely in service loop; verify backoff/dead-letter behavior | Service config files |
| NetSuite | `LandcorSystem/NetsuiteService/*` | Extract/integration job | Service account/API credentials | Batch retry semantics need explicit outbox/inbox tracking in target state | Service config files / scheduled runtime context |
| Solidifi | `LandcorSystem/Landcor.Integration/*Solidifi*` | Provider API | Credential-based | Retry/idempotency contract requires confirmation | Legacy config |
| ConstantContact | `LandcorSystem/Landcor.Integration/*ConstantContact*` | Provider API | API key/token | Typically eventual, low-criticality; still needs deterministic retry policy | Legacy config |
| Email/notifications | `LandcorSystem/*` (mail sender components) | SMTP/provider API | SMTP/API credentials | Usually best-effort in legacy stacks; target should use queued reliable delivery | App/service config |

## Internal coupling map

| Producer | Consumer | Coupling type | Risk |
|---|---|---|---|
| UI and WebService entry points | `Landcor.Service.Business*` | Direct assembly dependency | Medium |
| Business services | `Landcor.DataAccess` | Shared mapper + sproc access | High |
| UI process/auth orchestration | Account services + shared DB model | Shared library and schema coupling | High |
| Workers | `Landcor.Integration` + `Landcor.DataAccess` | Shared libraries across runtimes | High |

## Authentication and authorization

### Current state (legacy)

- Custom account/session-centric auth flows integrated into UI and service logic.
- Request-pipeline session checks and login-stamp style validation in web runtime.
- SOAP endpoint authentication appears account/service-call based (not token-based OAuth boundary).

### Target AuthN/AuthZ model (PR1 proposal)

#### Tenant / org model
- **Platform tenant:** Landcor platform boundary.
- **Organization tenant:** customer organizations (e.g., realtor/broker/customer orgs) isolated by `OrganizationId`/`TenantId` claim.
- **Internal tenant realm:** Landcor internal operations and support users with elevated operational scopes.

#### RBAC shape
- **Roles (coarse):** `OrgUser`, `OrgManager`, `Appraiser`, `BillingOperator`, `IntegrationOperator`, `SupportAdmin`, `PlatformAdmin`.
- **Permissions (fine):** verb/resource permissions (e.g., `orders.read`, `orders.submit`, `reports.generate`, `integrations.retry`, `users.manage`).
- **Scopes (token/API):** API scopes mapped per module (`property.read`, `order.write`, `report.read`, `admin.ops`).
- **Policy model:** role grants default permissions; explicit deny/elevation for admin operations.

#### Migration approach
1. **Phase 1 (coexistence):** legacy session auth remains for legacy UI; introduce token validation at new .NET 8 API boundary.
2. **Phase 2 (dual auth):** token exchange/bridge issues modern tokens from validated legacy session for migrated paths.
3. **Phase 3 (cutover):** migrated modules require token-based auth; legacy session reduced to compatibility facade only.
4. **Phase 4 (retire):** remove session bridge after UI and SOAP clients are fully migrated.

## Database usage patterns

1. Stored procedure-centric transactional workflows.
2. Shared mapper layer with raw SQL/stored procedures in `Landcor.DataAccess`.
3. Shared schema access by web, services, and workers.
4. Mixed direct SQL/ORM usage depending on feature area.

## Data flows by app/service

| App / host | Inbound | Core processing | Outbound |
|---|---|---|---|
| Store (consumer flow) | Web request | Search → cart/order → payment → report generation | DB writes, payment provider calls, notifications |
| Appraiser | Web/internal request | Assignment/review/report workflows | DB updates, partner callbacks |
| RealEstate (realtor) | Web request + user auth | Property search, order/report lifecycle | DB reads/writes, public/internal service calls |
| Admin | Internal web/admin request | User/admin/config operations | DB admin updates, audit/log outputs |
| WebService (public SOAP) | Partner SOAP request | Authenticate → execute operations | SOAP response + DB/service interactions |
| WindowsService jobs | Service timers / scheduled triggers | Batch sync/extract/reconcile | External APIs + DB state transitions |
