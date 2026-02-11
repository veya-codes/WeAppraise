# PR1 — Legacy Assessment + Target Architecture (Docs)

This folder contains the PR1 documentation set for assessing legacy current state and defining a target architecture direction.

## Start here (PR2 gate)

- **Open Questions / Verification Checklist:** [09-open-questions-verification-checklist.md](./09-open-questions-verification-checklist.md)

## Open Questions / Verification Checklist (review quick list)

- [ ] Export production IIS sites/app pools + bindings.
- [ ] Export Windows Services list + service accounts + start modes.
- [ ] Export Scheduled Tasks list + triggers + run-as identities.
- [ ] Inventory SQL Agent jobs (if any).
- [ ] Inventory secret locations + rotation owners.
- [ ] Capture traffic volume and top endpoints (baseline telemetry).
- [ ] Confirm identity/tenant model with business (internal vs external orgs).

## Decision defaults for PR2

- API hosting: Azure App Service.
- Background processing: .NET Worker Service (Container Apps later if needed).
- Messaging: Azure Service Bus.
- DB migrations: EF Core migrations for new module-owned schemas.
- Secrets: Managed Identity + Key Vault.
- Observability: Application Insights + OpenTelemetry.

## Document index

1. [00-exec-summary.md](./00-exec-summary.md)
2. [01-current-state-solution-map.md](./01-current-state-solution-map.md)
3. [02-integrations-auth-data.md](./02-integrations-auth-data.md)
4. [03-dependency-diagrams.md](./03-dependency-diagrams.md)
5. [04-bounded-contexts-target-architecture.md](./04-bounded-contexts-target-architecture.md)
6. [05-strangler-migration-strategy.md](./05-strangler-migration-strategy.md)
7. [06-risks-assumptions.md](./06-risks-assumptions.md)
8. [07-evidence-appendix-inventories.md](./07-evidence-appendix-inventories.md)
9. [08-use-case-carveout-scorecard.md](./08-use-case-carveout-scorecard.md)
10. [09-open-questions-verification-checklist.md](./09-open-questions-verification-checklist.md)

## How to review this PR1 pack

1. Read `00` and `01` for scope and current-state framing.
2. Validate inventories and endpoint/job/provider details in `07`.
3. Review use-case scorecard and carve-out prioritization in `08`.
4. Review auth model + RBAC + coexistence decision in `02`.
5. Review bounded-context ownership/hosting/DB migration defaults in `04`.
6. Review first vertical strangler slice in `05`.
7. Treat `09` as the PR2 entry gate checklist.

## Notes

- This PR1 pass is evidence-driven from `ModernizationAssessment.md` path references.
- All repo path citations in PR1 map back to this artifact; PR1 does not yet include direct source scan evidence.
- PR1 is now evidence-complete at the source-code level; runtime confirmation remains.
- An environment-backed verification pass is still required (IIS/services/tasks exports) before implementation planning is finalized.
