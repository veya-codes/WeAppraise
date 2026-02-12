# PR1.06 — Risk Register and Assumptions

## Risk register

| ID | Risk | Impact | Likelihood | Mitigation | Owner |
|---|---|---|---|---|---|
| R1 | Secrets/credentials present in legacy config patterns | High | High | Rotate secrets, move to Key Vault + managed identity, secret scanning in CI | Platform/Security |
| R2 | Stored-procedure-heavy workflows block rapid refactor | High | High | Introduce compatibility adapters, prioritize high-value sproc inventory | Data/App |
| R3 | Mixed hosting model causes migration sequencing failures | High | Medium | Host inventory, per-app runbook, staged cutover with rollback | Platform |
| R4 | Partner integration contracts undocumented or brittle | High | Medium | Contract tests, provider sandbox validation, change-notice process | Integrations |
| R5 | Legacy auth/session behavior not portable to cloud edge model | High | Medium | Auth ADR, token boundary design, phased compatibility mode | Identity |
| R6 | Unknown production usage of legacy apps/projects | Medium | High | Usage telemetry + stakeholder validation, deprecate dormant apps safely | Product/Eng |
| R7 | Data ownership boundaries unclear across modules | High | High | Context data ownership matrix, cross-context access policy | Architecture |
| R8 | Report generation SLAs degrade during transition | Medium | Medium | Async queue + replay and performance testing | Reporting |

## Assumptions list

| ID | Assumption | Confidence | Validation action |
|---|---|---|---|
| A1 | Primary legacy stack is .NET Framework 4.5.1 era | Medium | Confirm via direct csproj scan in `LDC-Legacy` |
| A2 | IIS hosts web and SOAP entry points | Medium | Confirm deployment topology with Ops |
| A3 | SQL Server stored procedures are core transaction mechanism | High | Produce sproc call inventory from DataAccess layer |
| A4 | Integrations can be decoupled before core domain rewrite | Medium | Pilot one provider migration (NetSuite or ConstantContact) |
| A5 | Modular monolith is acceptable interim target for roadmap | Medium | Confirm with engineering leadership and delivery constraints |
| A6 | Legacy repo scan in this PR is based on existing assessment artifact | High | Re-run end-to-end source scan once network access is available |

## Unknowns / TODOs

Unknowns are centralized in `09-open-questions-verification-checklist.md` and treated as PR2 entry-gate items.

- Unknown: authoritative list of production IIS sites, app pools, Windows services, and schedules.
- Unknown: full external dependency register including credentials owners and expiry cycles.
- Unknown: exact data retention/compliance requirements per workflow (PII/report artifacts).
- Unknown: performance baselines for search, checkout, and report generation peaks.

## Recommended immediate actions (first 2 weeks)

1. Run secret rotation and repository secret scanning.
2. Build endpoint + scheduled job inventory from production runtime.
3. Produce stored procedure usage matrix (caller → sproc → table group).
4. Define module ownership and publish initial architecture decision records.
