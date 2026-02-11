# PR1.04 — Bounded Contexts and Target Architecture (.NET 8 Modular Monolith)

## Proposed bounded contexts

| Bounded context | Responsibilities | Primary data ownership | External dependencies |
|---|---|---|---|
| Identity & Access | Authentication, authorization, session/token lifecycle, account policy | Users, roles, credentials metadata, auth audit | Identity provider (future), email/SMS for verification |
| Property & Valuation | Property search, valuation inputs, appraiser domain rules | Property entities, valuation artifacts, search indexes | Geodata/registry sources, appraisal partners |
| Orders & Cart | Cart, checkout, order lifecycle, payment intents | Cart/order tables, transaction state | Payment gateway (HPCI), reporting trigger |
| Reporting & Documents | Report generation, formatting, retrieval, document storage metadata | Report records, generation queue state | Storage provider, templating engine |
| Integrations | Outbound/inbound provider adapters, retries, reconciliation | Integration outbox/inbox, provider mapping tables | BC Online, NetSuite, Solidifi, ConstantContact |
| Admin & Operations | Internal tools, feature flags, support actions, operational controls | Admin config, audit trails, operational overrides | Monitoring/alerting, job orchestration |

## Module dependency rules

1. API/host layer can call application services for each module.
2. Modules may not directly reference another module's persistence layer.
3. Cross-module reads use published queries/read models.
4. Cross-module commands use explicit contracts and internal mediator/event bus patterns.
5. Integration adapter code lives only in Integrations module.

## Reference target architecture

- **Runtime:** .NET 8 single deployable (initially), ASP.NET Core host.
- **Persistence:** Azure SQL with schema segmentation by module.
- **Messaging:** Service Bus for async jobs/integration workflows.
- **Observability:** OpenTelemetry + Application Insights.
- **Security:** Managed Identity + Key Vault, policy-based auth.
- **Background work:** hosted services for lightweight jobs; external worker processes only where isolation is required.

## Transitional architecture decisions (PR1)

- Preserve existing business behaviors first; refactor boundaries before behavior.
- Introduce anti-corruption adapters around legacy stored procedures and provider SDKs.
- Keep SOAP compatibility via facade endpoints during transition where partner contracts require it.

## Initial ADR backlog

1. ADR-001: Modular monolith package/project structure.
2. ADR-002: Auth modernization path (custom auth to OIDC-compatible boundary).
3. ADR-003: SQL strategy (sproc compatibility vs staged EF/Dapper modernization).
4. ADR-004: Integration reliability patterns (outbox/retry/idempotency).
5. ADR-005: Reporting pipeline architecture and storage choices.
