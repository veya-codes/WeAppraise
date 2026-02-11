# PR1.01 — Current State Solution Map

## Scope and evidence
This solution map is based on the existing legacy assessment artifact in this repository (`ModernizationAssessment.md`), which references source paths from the `LDC-Legacy` codebase.

> Constraint: direct network cloning of `https://github.com/veya-codes/LDC-Legacy` failed in this environment (HTTP 403). All findings below should be treated as a **documented baseline** and revalidated by checking the referenced legacy paths directly.

## Solution-level map

| Area | Project/App (legacy path) | Type | Runtime/Host assumption | Role in system |
|---|---|---|---|---|
| Web UI | `LandcorSystem/Landcor.UI.Realtor` | ASP.NET Web Forms | IIS (classic web app) | Realtor-facing web experience, auth/session aware request pipeline |
| Web UI | `LandcorSystem/Landcor.UI.Commercial` | ASP.NET Web Forms | IIS | Commercial-facing web experience |
| Web UI | `LandcorSystem/Landcor.UI.Consumer` | ASP.NET Web app | IIS | Consumer-facing web surface |
| Public API | `LandcorSystem/Landcor.Webservice.Public` | ASMX SOAP service | IIS | Public partner/service endpoints for search/reporting |
| Internal API | `LandcorSystem/Landcor.Service.Internal` | ASMX + WCF service implementation | IIS / service host | Internal operations and service contracts |
| Service host | `LandcorSystem/Landcor.Service` | Windows Service / WinExe | Windows Service Control Manager | Hosts internal net.tcp/webHttp endpoints |
| Business layer | `LandcorSystem/Landcor.Service.Business` | Class library | In-process with service/web hosts | Property search/report orchestration |
| Business layer | `LandcorSystem/Landcor.Service.Business.Realtor` | Class library | In-process | Realtor-specific business use cases |
| App orchestration | `LandcorSystem/Landcor.UI.Process` | Class library / MVC-like support | In-process with UI | Login/auth flow and process coordination |
| Data access | `LandcorSystem/Landcor.DataAccess` | Class library | In-process | Stored procedure/raw SQL + mapper layer |
| Domain/entity | `LandcorSystem/Landcor.Entity` | Class library | In-process | Data contracts/entities used across layers |
| Integrations | `LandcorSystem/Landcor.Integration` | Class library | In-process + background worker use | External providers (billing, BC Online, ConstantContact, Solidifi) |
| Background worker | `LandcorSystem/BCOnlineService` | Windows Service | Windows Service | BC Online sync/integration workloads |
| Background worker | `LandcorSystem/SalesOnDiskService` | Windows Service | Windows Service | Sales-on-disk processing |
| Scheduled/worker | `LandcorSystem/NetsuiteService` | Console/scheduled executable | Scheduled Task/manual | NetSuite extract jobs |
| Database scripts | `LandcorSystem/Landcor.SQL` | SQL schema/sprocs | SQL Server | Core persistence model + procedural logic |

## Major component inventory

- **Entry points:** Web Forms `Global.asax`, ASMX `.asmx` services, WCF `.svc` endpoints, and multiple Windows Service `Program.cs` entry points.
- **Architecture shape:** mixed-host monolith with shared business/data libraries consumed by web apps, services, and workers.
- **Tight coupling indicator:** shared `Landcor.DataAccess` and `Landcor.Entity` reused by multiple app types.

## Runtime hosting assumptions

1. IIS hosts user-facing web apps and ASMX endpoints.
2. A Windows service host runs net.tcp/internal service endpoints.
3. Additional domain/integration jobs run as Windows Services and scheduled console executables.
4. SQL Server is required for transactional workflows due to heavy stored procedure usage.

## TODO verification list (when legacy repo is directly accessible)

- Confirm `LandcorSystem.sln` project list and active/deprecated projects.
- Confirm which web apps/services are still deployed to production.
- Confirm all runtime hosts (IIS sites/app pools, Windows Services, scheduled tasks).
- Confirm environment-specific config transforms (`Web.*.config`, `App.*.config`) and deployment bindings.
