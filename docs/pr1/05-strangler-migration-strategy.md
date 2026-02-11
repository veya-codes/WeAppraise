# PR1.05 — Strangler Migration Strategy

## Carve-out priority

### First carve-out: Integrations module

**Why first**
- High operational risk and frequent provider-side change.
- Lower core-domain blast radius than valuation/order internals.
- Immediate reliability gains via async outbox/retry/dead-letter patterns.

### First provider target decision

**PR1 decision:** NetSuite is the first provider carve-out target (lower blast radius than HPCI payments, clearer batch boundary than appraisal/payment hot paths).

## First concrete vertical slice (PR1 target)

### Slice: NetSuite extract modernization

**Legacy flow (current):**
`NetsuiteService` scheduled executable reads DB and pushes extracts directly to NetSuite integration path.

**Target strangler flow:**
1. Existing trigger writes an `NetSuiteExtractRequested` record to Integrations outbox table.
2. Outbox publisher emits message to Service Bus queue `integrations.netsuite.extract`.
3. Integrations worker consumes message, performs extraction transform, calls NetSuite API.
4. Worker writes delivery result (`Succeeded/Failed/Retrying`) with idempotency key.
5. Reporting/read model exposes parity metrics and status.

### Inputs / outputs
- **Inputs:** tenant/org context, extract window, batch identifier, correlation id.
- **Outputs:** provider delivery status, external reference id, retry count, failure reason, completion timestamp.

### Failure modes and retry behavior
- **Transient provider/network error:** exponential backoff retry (bounded attempts) with jitter.
- **Permanent contract/validation error:** send to dead-letter queue and mark failed with actionable reason.
- **Duplicate delivery risk:** enforce idempotency key per extract batch + provider endpoint.
- **DB/outbox publish failure:** transactional outbox pattern ensures eventual publish.

### Parity proof plan
1. **Dual-run mode:** run legacy NetSuite path and new queue-based path for same batch (read-only compare first).
2. **Mirrored write compare:** compare payload hashes, external response codes, and completion outcomes.
3. **Cutover gate:** require parity threshold (e.g., >= 99.5% match) over agreed sample window.
4. **Rollback:** feature flag routes traffic back to legacy path immediately if parity/SLI regresses.

## Migration waves

| Wave | Scope | Deliverables | Exit criteria |
|---|---|---|---|
| Wave 0 | Baseline + observability | Endpoint/job inventory, telemetry, error budgets | Current-state behavior measurable |
| Wave 1 | Modularize in place | Module skeletons, dependency rules, adapter seams | New code follows module boundaries |
| Wave 2 | Integrations strangler | NetSuite vertical slice + outbox + queue worker + parity dashboard | >= 60% of NetSuite integration calls executed through outbox+worker path for 2 consecutive release cycles |
| Wave 3 | Reporting isolation | Async generation + document abstraction | Report jobs resilient and replayable |
| Wave 4 | Auth modernization | Token boundary + compatibility bridge for legacy clients | Legacy session coupling reduced |
| Wave 5 | Core domain refactor | Context-owned repos/contracts | Cross-context DB writes removed from hot paths |

## What not to carve out first

- Do not extract Property & Valuation first (high coupling/high business criticality).
- Do not split Orders/Cart into a separate deployable before payment/report contracts stabilize.
