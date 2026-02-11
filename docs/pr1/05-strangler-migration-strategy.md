# PR1.05 — Strangler Migration Strategy

## Carve-out priority

### First carve-out: Integrations module

**Why first:**
- High operational risk and change frequency (provider credentials, formats, retries).
- Often lower business-rule density than core valuation/order logic.
- Good candidate for async decoupling with clear contracts.
- Enables immediate reliability improvements (idempotency, retries, dead-letter handling).

## Migration waves

| Wave | Scope | Deliverables | Exit criteria |
|---|---|---|---|
| Wave 0 | Baseline + observability | Endpoint inventory, telemetry, health checks, error budgets | Current-state behavior measurable |
| Wave 1 | Modularize in place | Module skeletons, dependency rules, adapter seams | New code follows module boundaries |
| Wave 2 | Integrations strangler | Outbox + queue worker + provider adapters | At least one provider path no longer direct-coupled |
| Wave 3 | Reporting pipeline isolation | Async generation and document storage abstraction | Report jobs resilient and replayable |
| Wave 4 | Auth and API surface modernization | Token-based boundary + facade for legacy SOAP clients | Legacy auth/session coupling reduced |
| Wave 5 | Core domain refactor (property/order) | Context-owned repositories + command/query contracts | Cross-context DB writes removed from hot paths |

## Tactical approach

1. **Route new integration workloads through new module endpoints first** while legacy paths remain as fallback.
2. **Mirror-write or dual-run** for selected non-destructive workflows to validate parity.
3. **Use feature flags** to switch path-by-path with rapid rollback.
4. **Apply contract tests** at module and external provider boundaries.

## What not to carve out first

- Do **not** extract Property & Valuation first; high coupling + high business criticality.
- Do **not** split Orders/Cart into a separate deployable before payment/report contracts stabilize.

## Success metrics

- Mean time to recover (MTTR) for integration failures improves.
- Reduction in direct calls from UI/business layers to provider-specific code.
- Increased percentage of async, replayable external operations.
- No regression in checkout/search/report core business SLAs.
