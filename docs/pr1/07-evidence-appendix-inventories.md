# PR1.07 — Evidence Appendix: Inventories (First-Pass)

This appendix consolidates first-pass inventories requested for PR1. Source evidence is `ModernizationAssessment.md` path references into `LandcorSystem/*`.

## A) Solution inventory (projects/apps/services/workers)

| System element | Legacy repo path | Type | Observed deployment/runtime state |
|---|---|---|---|
| Landcor.UI.Realtor | `LandcorSystem/Landcor.UI.Realtor` | Web Forms app | IIS-hosted runtime entry point observed |
| Landcor.UI.Commercial | `LandcorSystem/Landcor.UI.Commercial` | Web Forms app | In solution; deployment confirmation pending ops export |
| Landcor.UI.Consumer | `LandcorSystem/Landcor.UI.Consumer` | Web app | In solution; deployment confirmation pending ops export |
| Landcor.Webservice.Public | `LandcorSystem/Landcor.Webservice.Public` | ASMX SOAP service | IIS service endpoint pattern observed |
| Landcor.Service.Internal | `LandcorSystem/Landcor.Service.Internal` | ASMX/WCF service impl | Internal endpoint artifacts observed |
| Landcor.Service | `LandcorSystem/Landcor.Service` | Windows service host | Service host entry point observed |
| BCOnlineService | `LandcorSystem/BCOnlineService` | Windows service | Service executable entry observed |
| SalesOnDiskService | `LandcorSystem/SalesOnDiskService` | Windows service | Service executable entry observed |
| NetsuiteService | `LandcorSystem/NetsuiteService` | Scheduled executable | Scheduled/manual execution pattern observed |
| Landcor.Service.Business* | `LandcorSystem/Landcor.Service.Business*` | Business libraries | Shared runtime dependency observed |
| Landcor.UI.Process | `LandcorSystem/Landcor.UI.Process` | Process orchestration library | Shared runtime dependency observed |
| Landcor.DataAccess | `LandcorSystem/Landcor.DataAccess` | Data access library | Shared DB hotspot observed |
| Landcor.Entity | `LandcorSystem/Landcor.Entity` | Entity library | Shared contract dependency observed |
| Landcor.Integration | `LandcorSystem/Landcor.Integration` | Integration adapters | Shared provider adapter dependency observed |
| Landcor.SQL | `LandcorSystem/Landcor.SQL` | DB schema/sprocs | Shared persistence dependency observed |

## B) Endpoint inventory (first-pass)

### Web/UI entry surfaces

| Surface | Path/evidence pattern | Host |
|---|---|---|
| Realtor UI app pipeline | `Landcor.UI.Realtor/Global.asax(.cs)` | IIS |
| UI pages/controllers (first-pass) | `Landcor.UI.*/*.aspx`, `Landcor.UI.Process/Controllers/*` | IIS |

### SOAP/WCF surfaces

| Surface | Path/evidence pattern | Host |
|---|---|---|
| Public SOAP service methods | `Landcor.Webservice.Public/LandcorService.cs` (+ `.asmx`) | IIS |
| Internal service endpoints | `Landcor.Service.Internal/*.svc`, service config endpoints | IIS/service host |
| net.tcp/webHttp host endpoints | `Landcor.Service/App.config` service model section | Windows service host |

## C) Job inventory (Windows services / scheduled jobs)

| Job | Path | Runtime | What it does | Frequency trigger | Dependencies |
|---|---|---|---|---|---|
| Landcor Service Host | `Landcor.Service/Program.cs` | Windows service | Hosts internal service endpoints | Service startup / always-on | Internal service libs + DB |
| BC Online Service | `BCOnlineService/Program.cs` | Windows service | BC Online integration synchronization | Service timer loop (confirm exact interval) | BC Online provider + DB |
| Sales On Disk Service | `SalesOnDiskService/Program.cs` | Windows service | Sales-on-disk processing workflow | Service timer loop (confirm exact interval) | DB + integration libs |
| NetSuite Service | `NetsuiteService/Program.cs` | Scheduled executable | NetSuite extract job | Scheduled task/manual run | NetSuite integration + DB |

## D) Integration inventory (provider/protocol/auth/retry/secrets)

| Provider | Protocol (first-pass) | Auth mechanism (first-pass) | Retry/idempotency (first-pass) | Secrets location |
|---|---|---|---|---|
| HPCI | Provider API/service adapter | Credentials/certs from config | Appears synchronous; explicit idempotency keys needed in target | Web/App config |
| BC Online | Provider API | Integration service account | Worker retries likely implicit; formal policy TBD | Service config |
| NetSuite | Job/API integration | Service account/API keys | Batch retry behavior to be formalized with outbox + queue | Service config / task context |
| Solidifi | Provider API | Credential-based auth | Retry semantics TBD | Config |
| ConstantContact | Provider API | API token/key | Retry semantics TBD | Config |
| Email | SMTP/provider API | SMTP/API credentials | Best-effort likely; target should queue and track delivery | Config |

## Evidence quality and next confirmation step

This appendix is a first-pass inventory with source path evidence from the assessment artifact. For merge readiness, run an environment-backed extraction:
- IIS inventory export,
- Windows services export,
- Scheduled tasks export,
- endpoint crawl list,
- secret/config key inventory.
