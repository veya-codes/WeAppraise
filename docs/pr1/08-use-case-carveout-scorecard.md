# PR1.08 — Core Use-Case Inventory and Carve-Out Scorecard

This scorecard lists the highest-priority legacy use-cases to unblock PR2/PR3 planning.

## Top use-cases (first-pass)

| # | Use-case | Current entry point(s) | Primary tables/sprocs (first-pass) | External calls | Carve-out candidate |
|---|---|---|---|---|---|
| 1 | Checkout / order submit | Realtor/Consumer UI + service business layer | Cart/order/payment sprocs (`Insert_Shopping_Cart`-style flows) | HPCI payment API | Later (high-risk) |
| 2 | Generate report | UI + internal service/reporting flow | Report/order/property sprocs | Report rendering and storage integrations | Medium |
| 3 | Retrieve report | UI + SOAP operations | Report retrieval sprocs/read models | None or storage API | Medium |
| 4 | Partner SOAP property search | `Landcor.Webservice.Public` ASMX methods | Property search sprocs | None | Medium |
| 5 | Partner SOAP order/report ops | `Landcor.Webservice.Public` ASMX methods | Order/report/account sprocs | Possible payment/report integrations | Medium/High |
| 6 | Realtor property search | `Landcor.UI.Realtor` + business layer | Property search sprocs/tables | None | Medium |
| 7 | User login/session validation | UI process/auth logic + web pipeline | Account user/session tables/sprocs | None | Medium |
| 8 | Admin user/account maintenance | Admin/internal UI flows | Account/admin config tables/sprocs | Email notifications | Medium |
| 9 | BC Online sync run | `BCOnlineService` worker | Integration staging/mapping tables | BC Online API | High carve-out candidate |
| 10 | NetSuite extract run | `NetsuiteService` scheduled job | Extract/audit/reconciliation tables | NetSuite API | **First carve-out** |
| 11 | SalesOnDisk processing | `SalesOnDiskService` worker | Sales/reporting tables and sprocs | External delivery endpoints (confirm) | Medium |
| 12 | ConstantContact sync | Integration layer calls | Contact sync tables/sprocs | ConstantContact API | High carve-out candidate |
| 13 | Solidifi status sync | Integration layer calls | Appraisal/report mapping tables | Solidifi API | Medium |
| 14 | Notification/email dispatch | UI/service flows and workers | Notification log tables | SMTP/provider API | High carve-out candidate |
| 15 | Retry/reconciliation operations | Service/worker maintenance paths | Integration state/retry tables | Provider APIs | High carve-out candidate |

## Carve-out scoring dimensions

Use this rubric to prioritize PR2/PR3 slices:
- **Blast radius:** user-facing risk and transactional criticality.
- **Coupling depth:** number of modules/tables touched.
- **Operational pain:** incidents/retries/manual intervention burden.
- **Contract clarity:** how well provider and payload contracts are understood.
- **Parity feasibility:** ability to dual-run and compare outcomes safely.

## Recommended order from scorecard

1. NetSuite extract run.
2. ConstantContact sync (or email dispatch queue).
3. BC Online sync.
4. Solidifi status sync.
5. Payment-adjacent integrations only after auth + order/report boundaries stabilize.
