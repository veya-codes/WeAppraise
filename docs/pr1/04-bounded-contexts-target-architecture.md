# PR1.04 — Bounded Contexts and Target Architecture (.NET 8 Modular Monolith)

## Proposed bounded contexts

| Bounded context | Responsibilities | Primary data ownership | External dependencies |
|---|---|---|---|
| Identity & Access | Authentication, authorization, token/session boundary, account policy | Users, roles, permission grants, auth audit | Identity provider integration, email/SMS verification |
| Property & Valuation | Property search, valuation rules, appraisal workflows | Property entities, valuation artifacts, search projections | Registry/geodata sources, appraisal partners |
| Orders & Cart | Cart, checkout, order lifecycle, payment orchestration | Cart/order/payment-intent tables | Payment providers, reporting trigger contracts |
| Reporting & Documents | Report generation and retrieval | Report metadata, generation state, document references | Template engine, storage provider |
| Integrations | Provider adapters, retries, reconciliation | Outbox/inbox, provider mapping and delivery state | BC Online, NetSuite, Solidifi, ConstantContact |
| Admin & Operations | Support/admin tooling, feature flags, operational controls | Admin config, operational audit trail | Monitoring/alerting systems |

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
5. ADR-005: Auth coexistence plan (legacy session + token boundary).
