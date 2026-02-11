# PR1.09 — Open Questions and PR2 Verification Entry Gate

This is the **single source of truth** for unresolved questions and verification tasks from PR1.

## PR2 entry gate checklist

PR2 planning starts only when all **P0** items are closed and at least 80% of **P1** items are closed.

| Priority | Item | Owner | Verification artifact | Status |
|---|---|---|---|---|
| P0 | Confirm deployed app/service inventory (Y/N per project) | Platform/Ops | IIS/app-pool export + service inventory sheet | Open |
| P0 | Confirm scheduled jobs and frequencies | Platform/Ops | Scheduled Tasks export with owners and schedules | Open |
| P0 | Confirm top 15 use-cases to table/sproc mapping | App/Data | Use-case-to-data matrix checked into docs | Open |
| P0 | Confirm integration contracts and auth mechanisms per provider | Integrations | Contract register + credential owner list | Open |
| P0 | Confirm NetSuite vertical-slice acceptance metrics | Architecture + Product | Signed acceptance criteria (parity, latency, error budget) | Open |
| P0 | Confirm auth coexistence constraints (legacy sessions + token facade) | Identity | Auth ADR + compatibility test matrix | Open |
| P1 | Confirm external SLAs/rate limits for BC Online, NetSuite, Solidifi, ConstantContact | Integrations | SLA registry document | Open |
| P1 | Confirm data retention/compliance boundaries (PII/report artifacts) | Security/Compliance | Data classification and retention policy mapping | Open |
| P1 | Confirm peak traffic windows and performance baselines | Platform | Baseline dashboard snapshots (search/checkout/report) | Open |
| P1 | Confirm ownership map (team per bounded context) | Engineering leadership | RACI matrix | Open |
| P1 | Confirm migration runbooks (rollback/playbook) | Platform + App | Cutover/rollback runbook docs | Open |

## Consolidated unknowns from PR1

- Unknown authoritative list of production IIS sites/app pools/windows services/tasks.
- Unknown full external dependency register with credential ownership and rotation cycle.
- Unknown complete table/sproc mapping for all top use-cases.
- Unknown full provider retry/rate-limit/error semantics for each integration.
- Unknown precise compliance retention rules across user, order, and report artifacts.

## Cross-links

- See [PR1 index](./README.md).
- See [use-case scorecard](./08-use-case-carveout-scorecard.md).
- See [risk register and assumptions](./06-risks-assumptions.md).
