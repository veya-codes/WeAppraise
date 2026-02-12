# PR1.02 — Integrations, Auth Model, and Data Usage

## Integration inventory (current state)

| Provider | Legacy location(s) | Protocol | Auth mechanism | Data touched (first-pass) | Retry / idempotency behavior | Sync vs async candidate | Secret location pattern |
|---|---|---|---|---|---|---|---|
| HPCI (payments) | `LandcorSystem/Landcor.Integration/Billing/HPCIPaymentProcessor.cs` | Provider API (service adapter) | API credentials/certs (legacy config-bound) | Order/cart/payment transaction tables and payment-related sprocs (confirm exact names) | Appears synchronous; idempotency controls need explicit keying in modernization | Keep synchronous in early phase; async only for non-customer-facing reconciliation | Legacy `Web.config` / `App.config` patterns |
| BC Online | `LandcorSystem/BCOnlineService/*`, `LandcorSystem/Landcor.Integration/*` | Service/API integration | Credential-based integration account | Integration staging/sync tables and BC mapping sprocs (confirm exact names) | Worker-style retry likely in service loop; verify backoff/dead-letter behavior | Async candidate (queue-backed worker) | Service config files |
| NetSuite | `LandcorSystem/NetsuiteService/*` | Extract/integration job | Service account/API credentials | Extract/audit/outbox-like tracking tables and reconciliation sprocs (confirm exact names) | Batch retry semantics need explicit outbox/inbox tracking in target state | **First async migration target** | Service config files / scheduled runtime context |
| Solidifi | `LandcorSystem/Landcor.Integration/*Solidifi*` | Provider API | Credential-based | Appraisal/report status tables + integration mappings (confirm exact names) | Retry/idempotency contract requires confirmation | Async candidate | Legacy config |
| ConstantContact | `LandcorSystem/Landcor.Integration/*ConstantContact*` | Provider API | API key/token | Contact/subscription sync tables (confirm exact names) | Typically eventual, low-criticality; still needs deterministic retry policy | Async candidate (low blast radius) | Legacy config |
| Email/notifications | `LandcorSystem/*` (mail sender components) | SMTP/provider API | SMTP/API credentials | Notification log/email queue tables (confirm exact names) | Usually best-effort in legacy stacks; target should use queued reliable delivery | Async candidate | App/service config |

## Integration module boundary definition (PR1 stance)

**Integration =** provider-facing adapter behavior (serialization/mapping, auth handshake, retries, rate-limits, callback verification, reconciliation) owned by the Integrations module.

**Not integration =** core domain workflow decisions (pricing, valuation rules, order state machine), which remain in domain modules and call Integrations via contracts.

### Required reliability patterns for Integrations module

1. Transactional outbox for publish reliability.
2. Idempotency keys per provider operation and business correlation key.
3. Bounded retries with jitter + provider-specific dead-letter queues.
4. Correlation IDs propagated across API → queue → worker → provider callback.
5. Provider-specific rate-limit policies and circuit-breakers.

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

#### Decision for first vertical slice

**PR1 decision:** implement an internal **compatibility auth façade first** for migrated paths (token boundary with legacy session bridge), then swap upstream identity provider to Entra ID in a later phase once compatibility risk is retired.

#### Tenant / org model
- **Platform tenant:** Landcor platform boundary.
- **Organization tenant:** customer organizations isolated by `OrganizationId`/`TenantId` claim.
- **Sub-organization support:** optional broker/team subgroup under a customer org (needed for delegated administration).
- **Internal tenant realm:** Landcor internal operations/support users with elevated operational scopes.

#### RBAC shape
- **Roles (coarse):** `OrgUser`, `OrgManager`, `Appraiser`, `BillingOperator`, `IntegrationOperator`, `SupportAdmin`, `PlatformAdmin`.
- **Permissions (fine):** verb/resource permissions (e.g., `orders.read`, `orders.submit`, `reports.generate`, `integrations.retry`, `users.manage`).
- **Scopes (token/API):** API scopes mapped per module (`property.read`, `order.write`, `report.read`, `admin.ops`).
- **Policy model:** role grants default permissions; explicit deny/elevation for admin operations.

#### Migration approach
1. **Phase 1 (coexistence):** legacy session auth remains for legacy UI; introduce token validation at new .NET 8 API boundary.
2. **Phase 2 (dual auth):** compatibility bridge issues modern tokens from validated legacy session for migrated paths.
3. **Phase 3 (provider swap):** integrate Entra ID as primary IdP for new clients while legacy bridge remains for compatibility.
4. **Phase 4 (cutover):** migrated modules require token-based auth; legacy session reduced to compatibility facade only.
5. **Phase 5 (retire):** remove session bridge after UI and SOAP clients are fully migrated.

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
