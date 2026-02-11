# PR1.02 — Integrations, Auth Model, and Data Usage

## Integrations map

| Integration | Legacy location(s) | Pattern | Criticality | Notes |
|---|---|---|---|---|
| Payment (HPCI) | `LandcorSystem/Landcor.Integration/Billing/HPCIPaymentProcessor.cs` | Synchronous service adapter from app/service logic | High | Payment path risk + security controls needed |
| BC Online | `LandcorSystem/BCOnlineService/*`, `LandcorSystem/Landcor.Integration/*` | Windows Service job + adapter calls | Medium/High | Background workload likely batch-oriented |
| NetSuite | `LandcorSystem/NetsuiteService/*` | Scheduled extract executable | Medium | Candidate for queue-triggered worker first |
| Solidifi | `LandcorSystem/Landcor.Integration/*Solidifi*` | Integration adapter | Medium | Exact contracts unknown; confirm SLA/retry behavior |
| ConstantContact | `LandcorSystem/Landcor.Integration/*ConstantContact*` | Integration adapter | Low/Medium | Marketing workflow likely separable |
| Email/notifications | `LandcorSystem/*` (TBD) | SMTP/app-triggered notifications | Medium | Confirm templates + delivery provider |
| Reporting services | `LandcorSystem/Landcor.Service.Internal/*`, `LandcorSystem/Landcor.Service.Business/*` | Service-side generation | High | Coupled to property/order workflows |

## Internal integrations and coupling

| Producer | Consumer | Coupling type | Risk |
|---|---|---|---|
| UI/WebService | `Landcor.Service.Business*` | Direct library reference | Medium |
| Service.Business | `Landcor.DataAccess` | Shared mapper/stored proc access | High |
| UI.Process | Auth/account services in business/data layer | Shared library + shared DB model | High |
| Workers | `Landcor.Integration` + `Landcor.DataAccess` | Reused shared libraries | High |

## Authentication and authorization model (current)

- User auth appears to be custom account-based checks in service and UI process layers.
- Session and login stamp validation are request-pipeline concerns in Web Forms apps.
- No evidence in the baseline of modern OIDC/OAuth2 boundary enforcement.
- Authorization appears mostly role/claim checks coupled to account data in SQL.

### Auth unknowns to confirm

- Password hashing algorithm and rotation strategy.
- MFA availability and enforcement scope.
- API consumer authentication model for public SOAP endpoints.
- Admin privilege model and audit trail coverage.

## Database usage patterns

### Observed patterns

1. **Stored procedure-centric** workflows for key transactions (e.g., shopping cart/report/order style operations).
2. **Raw SQL + mapper classes** in shared `Landcor.DataAccess`.
3. **Shared schema access** by multiple apps/services/workers.
4. **Mixed EF + direct SQL** likely coexisting depending on module.

### DB hotspots (expected)

- Property search/read models.
- Shopping cart/order/report transactions.
- Account and session management.
- Integration staging/reconciliation tables.

## Data flows by app/service

| App / host | Inbound | Core processing | Outbound |
|---|---|---|---|
| Store (consumer purchase flow) | Web request | Search → cart/order → payment → report generation | DB writes, payment calls, email/report artifacts |
| Appraiser | Web/internal requests | Assignment/review/report workflows | DB state changes, integration callbacks |
| RealEstate (Realtor UI) | Web request + auth/session | Property search, order/report retrieval | DB reads/writes, public/internal service calls |
| Admin | Internal/admin web access | Account/support/config operations | DB admin updates, audit/log outputs |
| WebService (public SOAP) | Partner SOAP call | Authenticate → execute search/report operations | SOAP response + DB/service calls |
| WindowsService(s) | Timers/service start | Batch sync/extract/reconciliation | External integration APIs + DB updates |

> Unknown: some app labels (Store/Appraiser/Admin) may map to legacy project names differently. Confirm naming alignment during PR2 implementation planning.
