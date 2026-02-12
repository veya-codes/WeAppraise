# PR1.07 — Evidence Appendix: Inventories (First-Pass)

This appendix consolidates first-pass inventories requested for PR1. Source evidence is `ModernizationAssessment.md` path references into `LandcorSystem/*`.

Confidence legend:
- **High** = directly cited in assessment with concrete path/type evidence.
- **Medium** = strong inference from cited runtime patterns but not yet ops-confirmed.
- **Low** = placeholder needing runtime export confirmation.

## A) Solution inventory (projects/apps/services/workers)

| System element | Legacy repo path | Type | Observed deployment/runtime state | Confidence |
|---|---|---|---|---|
| Landcor.UI.Realtor | `LandcorSystem/Landcor.UI.Realtor` | Web Forms app | IIS-hosted runtime entry point observed | High |
| Landcor.UI.Commercial | `LandcorSystem/Landcor.UI.Commercial` | Web Forms app | In solution; deployment confirmation pending ops export | Medium |
| Landcor.UI.Consumer | `LandcorSystem/Landcor.UI.Consumer` | Web app | In solution; deployment confirmation pending ops export | Medium |
| Landcor.Webservice.Public | `LandcorSystem/Landcor.Webservice.Public` | ASMX SOAP service | IIS service endpoint pattern observed | High |
| Landcor.Service.Internal | `LandcorSystem/Landcor.Service.Internal` | ASMX/WCF service impl | Internal endpoint artifacts observed | High |
| Landcor.Service | `LandcorSystem/Landcor.Service` | Windows service host | Service host entry point observed | High |
| BCOnlineService | `LandcorSystem/BCOnlineService` | Windows service | Service executable entry observed | High |
| SalesOnDiskService | `LandcorSystem/SalesOnDiskService` | Windows service | Service executable entry observed | High |
| NetsuiteService | `LandcorSystem/NetsuiteService` | Scheduled executable | Scheduled/manual execution pattern observed | High |
| Landcor.Service.Business* | `LandcorSystem/Landcor.Service.Business*` | Business libraries | Shared runtime dependency observed | Medium |
| Landcor.UI.Process | `LandcorSystem/Landcor.UI.Process` | Process orchestration library | Shared runtime dependency observed | Medium |
| Landcor.DataAccess | `LandcorSystem/Landcor.DataAccess` | Data access library | Shared DB hotspot observed | High |
| Landcor.Entity | `LandcorSystem/Landcor.Entity` | Entity library | Shared contract dependency observed | Medium |
| Landcor.Integration | `LandcorSystem/Landcor.Integration` | Integration adapters | Shared provider adapter dependency observed | High |
| Landcor.SQL | `LandcorSystem/Landcor.SQL` | DB schema/sprocs | Shared persistence dependency observed | High |

## B) Endpoint inventory (first-pass)

### Web/UI entry surfaces

| Surface | Path/evidence pattern | Host | Confidence |
|---|---|---|---|
| Realtor UI app pipeline | `Landcor.UI.Realtor/Global.asax(.cs)` | IIS | High |
| UI pages/controllers (first-pass) | `Landcor.UI.*/*.aspx`, `Landcor.UI.Process/Controllers/*` | IIS | Medium |

### SOAP/WCF surfaces

| Surface | Path/evidence pattern | Host | Confidence |
|---|---|---|---|
| Public SOAP service methods | `Landcor.Webservice.Public/LandcorService.cs` (+ `.asmx`) | IIS | High |
| Internal service endpoints | `Landcor.Service.Internal/*.svc`, service config endpoints | IIS/service host | Medium |
| net.tcp/webHttp host endpoints | `Landcor.Service/App.config` service model section | Windows service host | High |

## C) Job inventory (Windows services / scheduled jobs)

| Job | Path | Runtime | What it does | Frequency trigger | Dependencies | Confidence |
|---|---|---|---|---|---|---|
| Landcor Service Host | `Landcor.Service/Program.cs` | Windows service | Hosts internal service endpoints | Service startup / always-on | Internal service libs + DB | High |
| BC Online Service | `BCOnlineService/Program.cs` | Windows service | BC Online integration synchronization | Service timer loop (confirm exact interval) | BC Online provider + DB | Medium |
| Sales On Disk Service | `SalesOnDiskService/Program.cs` | Windows service | Sales-on-disk processing workflow | Service timer loop (confirm exact interval) | DB + integration libs | Medium |
| NetSuite Service | `NetsuiteService/Program.cs` | Scheduled executable | NetSuite extract job | Scheduled task/manual run | NetSuite integration + DB | High |

## D) Integration inventory (provider/protocol/auth/retry/secrets)

| Provider | Protocol (first-pass) | Auth mechanism (first-pass) | Retry/idempotency (first-pass) | Secrets location | Confidence |
|---|---|---|---|---|---|
| HPCI | Provider API/service adapter | Credentials/certs from config | Appears synchronous; explicit idempotency keys needed in target | Web/App config | Medium |
| BC Online | Provider API | Integration service account | Worker retries likely implicit; formal policy TBD | Service config | Medium |
| NetSuite | Job/API integration | Service account/API keys | Batch retry behavior to be formalized with outbox + queue | Service config / task context | High |
| Solidifi | Provider API | Credential-based auth | Retry semantics TBD | Config | Medium |
| ConstantContact | Provider API | API token/key | Retry semantics TBD | Config | Medium |
| Email | SMTP/provider API | SMTP/API credentials | Best-effort likely; target should queue and track delivery | Config | Low |

## How we will confirm (runtime artifact checklist)

Collect and attach these artifacts before PR2 implementation kickoff:

1. **IIS inventory export** per environment:
   - site name, bindings, app pools, physical path, auth mode, and rewrite rules.
2. **Windows Services export**:
   - service name, display name, binary path, startup type, run-as account.
3. **Scheduled Tasks export**:
   - task name, trigger, action path/args, run-as account, retry settings.
4. **SQL Agent jobs export** (if present):
   - job name, schedule, step commands, owner.
5. **Endpoint traffic baseline**:
   - top endpoints, request volume, error rate, p95 latency.
6. **Secrets/dependency map**:
   - secret key name, storage location, consuming app/service, owner, rotation cadence.

## Evidence quality and next confirmation step

This appendix is a first-pass inventory with source path evidence from the assessment artifact. It should be treated as source-referenced until runtime exports above are attached and signed off by Ops/Platform.
