# PR1 Diagram Notes

## Reading guidance

- **Current-state graph** emphasizes coupling and runtime heterogeneity.
- **Target-state graph** emphasizes module boundaries and isolation of external provider concerns.

## Constraints captured

- Keep a single deployable in early modernization phases.
- Avoid direct module-to-module table access; use contracts/read models.
- Prefer asynchronous integration operations where feasible.
