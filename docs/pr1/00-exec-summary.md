# PR1.00 — Executive Summary

This PR establishes the **current-state baseline** for modernizing LDC-Legacy into WeAppraise.

## What the current state appears to be

- A mixed .NET Framework monolith (primarily .NET 4.5.1) composed of Web Forms apps, ASMX SOAP services, WCF endpoints, and Windows/Scheduled background processes.
- A shared business and data-access core reused across UI, services, and workers.
- Significant reliance on SQL stored procedures and shared schema patterns.
- Integration-heavy workflows (payments, BC Online, NetSuite, Solidifi, ConstantContact, email/reporting) embedded into app/service layers.

## Why this matters for modernization

- Hosting model heterogeneity (IIS + Windows Services + scheduled jobs) increases migration complexity.
- Cross-project shared libraries and direct data access patterns indicate high coupling.
- Auth/session and configuration/secrets practices are legacy-oriented and need hardening for cloud deployment.
- A modular monolith on .NET 8 provides a lower-risk first target than immediate microservice decomposition.

## Recommended direction in PR1

1. Establish bounded contexts and explicit module boundaries in a .NET 8 modular monolith.
2. Keep one deployable initially, with strong internal interfaces and anti-corruption adapters around legacy integrations.
3. Apply strangler routing/use-case migration incrementally, starting with lower-coupled integration slices.
4. Track unknowns as explicit risks/assumptions and convert them into discovery tasks.

## Evidence quality note

- Findings are derived from the in-repo assessment artifact (`ModernizationAssessment.md`) which includes path-level references into the legacy codebase.
- Direct clone access to `LDC-Legacy` was blocked in this environment; PR1 therefore includes explicit TODOs where direct verification is required.


## PR1 refinements added

- Added a first-pass evidence appendix with solution/endpoint/job/integration inventories (`07`).
- Added explicit AuthN/AuthZ target model (tenant model, RBAC shape, migration coexistence).
- Added explicit data ownership and stored-procedure boundary rules per bounded context.
- Added one concrete first vertical strangler slice: NetSuite extract via outbox + Service Bus worker.
