# PR1.04 — Bounded Contexts and Target Architecture (.NET 8 Modular Monolith)

## Proposed bounded contexts

| Bounded context | Responsibilities | Primary data ownership | Owned schema (provisional) | External dependencies |
|---|---|---|---|---|
| Identity & Access | Authentication, authorization, token/session boundary, account policy | Users, roles, permission grants, auth audit | `identity.*` | Identity provider integration, email/SMS verification |
| Property & Valuation | Property search, valuation rules, appraisal workflows | Property entities, valuation artifacts, search projections | `property.*`, `valuation.*` | Registry/geodata sources, appraisal partners |
| Orders & Cart | Cart, checkout, order lifecycle, payment orchestration | Cart/order/payment-intent tables | `orders.*`, `cart.*` | Payment providers, reporting trigger contracts |
| Reporting & Documents | Report generation and retrieval | Report metadata, generation state, document references | `reporting.*`, `documents.*` | Template engine, storage provider |
| Integrations | Provider adapters, retries, reconciliation | Outbox/inbox, provider mapping and delivery state | `integrations.*` | BC Online, NetSuite, Solidifi, ConstantContact |
| Admin & Operations | Support/admin tooling, feature flags, operational controls | Admin config, operational audit trail | `admin.*`, `ops.*` | Monitoring/alerting systems |

## Explicit data ownership and access rules

1. **Table ownership:** each module owns its tables/schema; non-owners cannot query/write those tables directly.
2. **Cross-module reads:** only through published query contracts/read models/API endpoints.
3. **Cross-module writes:** only through explicit commands/events; no direct SQL writes into another module’s schema.
4. **Stored procedures:** allowed only behind the owning module’s DAL boundary; no shared cross-domain “kitchen sink” sprocs.
5. **Migration compatibility:** legacy sprocs remain temporarily but must be wrapped by module-scoped adapters.
6. **Contract versioning:** inter-module contracts are versioned and backward-compatible during migration windows.

## Module dependency rules

1. Host/API can call application services for each module.
2. Modules cannot reference another module’s persistence implementation.
3. Integration provider SDK/API clients exist only in Integrations module.
4. Reporting, orders, and admin invoke integrations via module contracts (sync or async).

## Hosting recommendation (PR1 default)

- **API host default:** Azure App Service (fastest operational path for .NET 8 modular monolith).
- **Background workers default:** .NET Worker Service, with Container Apps as a later deployment target when needed.
- **When to choose AKS instead:** only if requirements exceed App Service + Container Apps (custom networking mesh, sidecars, high-scale microservice estate, or strict platform controls).

## Database migration stance (PR1 default)

- **Default choice now:** EF Migrations for **new module-owned schemas** (`identity.*`, `orders.*`, etc.).
- **Legacy schema/sproc changes:** managed separately as SQL-first scripts during strangler period.
- **Future shift trigger to Flyway:** adopt Flyway when multiple independently deployed services require strict SQL-first, repeatable, centrally governed migration pipelines.


## PR2 defaults (decision baseline)

- **API hosting:** Azure App Service.
- **Background processing:** .NET Worker Service first (Container Apps deployment target can be introduced later if scale/ops requires it).
- **Messaging:** Azure Service Bus.
- **DB migrations:** EF Core migrations for new module-owned schemas.
- **Secrets:** Managed Identity + Key Vault.
- **Observability:** Application Insights + OpenTelemetry.
- **Legacy DB evolution during strangler:** SQL-first scripts for legacy schemas/sprocs.
- **Promotion rule:** any deviation from these defaults requires an ADR before PR2 implementation starts.

### Why these defaults for PR2

These defaults optimize PR2 for operational simplicity and delivery speed: App Service aligns with the team’s IIS-style hosting familiarity and reduces platform overhead for a single API deployable, while .NET Worker Service keeps background processing explicit in code before introducing additional container platform complexity. Service Bus is selected as the common async backbone to standardize retries and dead-letter handling across integration flows.

### Decision boundary: when to deviate

Move API or workers to Container Apps when throughput/auto-scaling pressure, background-job isolation needs, or independent deployment cadence materially exceed App Service + Worker Service operational limits. Move to AKS only for advanced platform requirements (service mesh/sidecars, complex networking controls, or large-scale microservice topology).

For database migrations, PR2 defaults to EF Core migrations for new module-owned schemas to maximize developer velocity. Flyway can be introduced later if DBA governance requires SQL-first, centrally managed, repeatable migration pipelines across multiple independently deployed services.

## Reference target architecture

- **Runtime:** .NET 8 modular monolith (single deployable initially).
- **Persistence:** Azure SQL with schema-per-module segmentation and controlled read models.
- **Messaging:** Service Bus for async workflows and integration delivery.
- **Observability:** OpenTelemetry + Application Insights.
- **Security:** Managed Identity + Key Vault + policy-based authorization.
- **Background processing:** hosted services for light jobs, dedicated worker processes where isolation is needed.

## Transitional architecture decisions (PR1)

- Preserve behavior first; move boundaries before rewriting business logic.
- Place anti-corruption adapters around legacy DAL/sprocs and external provider connectors.
- Keep compatibility facades for legacy SOAP consumers during phased migration.

## Initial ADR backlog

1. ADR-001: Modular monolith package/project structure.
2. ADR-002: Tenant and RBAC model implementation.
3. ADR-003: Sproc compatibility and module-scoped DAL strategy.
4. ADR-004: Integration reliability (outbox, retries, idempotency keys, dead-letter policy).
5. ADR-005: Auth coexistence plan (legacy session + token boundary with Entra transition).
6. ADR-006: Hosting baseline (App Service + Container Apps workers) and AKS trigger conditions.
7. ADR-007: DB migration governance (EF for new schemas + SQL-first for legacy).
