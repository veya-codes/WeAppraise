# PR1.01 — Current State Solution Map

## Scope and evidence
This solution map is derived from `ModernizationAssessment.md`, which already cites concrete legacy code paths under `LandcorSystem/*`.

> Constraint: direct network clone of `https://github.com/veya-codes/LDC-Legacy` is blocked in this runtime (HTTP 403), so this is a source-referenced baseline from the in-repo assessment artifact.

## Solution-level map (observed runtime inventory)

| Area | Project/App (legacy path) | Type | Runtime/Host | Deployment status (from available evidence) | Role in system |
|---|---|---|---|---|---|
| Web UI | `LandcorSystem/Landcor.UI.Realtor` | ASP.NET Web Forms | IIS web app | Active runtime entry point observed in assessment | Realtor-facing UI and request pipeline |
| Web UI | `LandcorSystem/Landcor.UI.Commercial` | ASP.NET Web Forms | IIS web app | Listed in primary solution; deployment confirmation pending ops export | Commercial-facing UI |
| Web UI | `LandcorSystem/Landcor.UI.Consumer` | ASP.NET web app | IIS web app | Listed in primary solution; deployment confirmation pending ops export | Consumer-facing UI |
| Public API | `LandcorSystem/Landcor.Webservice.Public` | ASMX SOAP service | IIS web service | Active service endpoint pattern observed | External/partner SOAP operations |
| Internal API | `LandcorSystem/Landcor.Service.Internal` | ASMX/WCF service implementation | IIS + service host | Active endpoint artifacts observed | Internal service operations |
| Service host | `LandcorSystem/Landcor.Service` | Windows Service / WinExe | Windows service host | Active host entry point observed | Hosts internal net.tcp/webHttp endpoints |
| Business layer | `LandcorSystem/Landcor.Service.Business` | Class library | In-process | Shared dependency in runtime graph | Property/report orchestration |
| Business layer | `LandcorSystem/Landcor.Service.Business.Realtor` | Class library | In-process | Shared dependency in runtime graph | Realtor-specific business flows |
| App orchestration | `LandcorSystem/Landcor.UI.Process` | Class library | In-process | Shared dependency in runtime graph | Login/auth and process orchestration |
| Data access | `LandcorSystem/Landcor.DataAccess` | Class library | In-process | Active DB hotspot in assessment | SQL/stored-procedure access |
| Domain/entity | `LandcorSystem/Landcor.Entity` | Class library | In-process | Shared dependency in runtime graph | Shared data contracts/entities |
| Integrations | `LandcorSystem/Landcor.Integration` | Class library | In-process + workers | Active provider adapter references observed | Billing/BC Online/ConstantContact/Solidifi adapters |
| Worker | `LandcorSystem/BCOnlineService` | Windows Service | Service Control Manager | Active worker entry point observed | BC Online synchronization |
| Worker | `LandcorSystem/SalesOnDiskService` | Windows Service | Service Control Manager | Active worker entry point observed | Sales-on-disk processing |
| Worker | `LandcorSystem/NetsuiteService` | Console/scheduled executable | Scheduled task/manual execution | Active scheduled-job style entry point observed | NetSuite extraction |
| Database | `LandcorSystem/Landcor.SQL` | SQL scripts/schema/sprocs | SQL Server | Active persistence dependency observed | Shared transactional persistence |

## Runtime composition

- **Entrypoints:** `Global.asax` (web), `*.asmx` and `*.svc` endpoints, and `Program.cs` service/worker entry points.
- **Hosting:** IIS for web and SOAP services; Windows Service host for internal services/workers; scheduled executable for NetSuite extraction.
- **Coupling:** multiple entry points share `Landcor.Service.Business*`, `Landcor.DataAccess`, and `Landcor.Entity`.

## Production deployment confirmation needed

To convert this from “evidence-observed” to “ops-confirmed deployed”, collect:
1. IIS site + app pool export from each environment.
2. Windows service list (service name, binary path, startup mode).
3. Scheduled task export for NetSuite and related jobs.
