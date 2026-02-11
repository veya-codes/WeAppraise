# Legacy Monolith Assessment and Azure Modularization Plan

## 1) Phase 0 — Quick triage

### Triage Summary
- **Application style is mixed**:
  - ASP.NET **Web Forms** web apps (`Landcor.UI.Realtor`, `Landcor.UI.Commercial`) indicated by Web Application project type GUID and `Global.asax`.【F:LandcorSystem/Landcor.UI.Realtor/Landcor.UI.Realtor.csproj†L11-L19】【F:LandcorSystem/Landcor.UI.Realtor/Global.asax.cs†L9-L20】
  - **ASMX SOAP services** in `Landcor.Webservice.Public` and `Landcor.Service.Internal` (`*.asmx` + WebService attributes).【F:LandcorSystem/Landcor.Webservice.Public/LandcorService.cs†L18-L23】【F:LandcorSystem/Landcor.Service.Internal/Landcor.Service.Internal.csproj†L169-L180】
  - **WCF endpoints** and net.tcp hosting configured in `Landcor.Service/App.config`; `.svc` endpoints also exist.【F:LandcorSystem/Landcor.Service/App.config†L157-L168】【F:LandcorSystem/Landcor.Service.Internal/Landcor.Service.Internal.csproj†L293-L307】
  - **Windows Services / WinExe workers** (`Landcor.Service`, `BCOnlineService`, `SalesOnDiskService`) and one scheduled-task style executable (`NetsuiteService`).【F:LandcorSystem/Landcor.Service/Landcor.Service.csproj†L9-L14】【F:LandcorSystem/BCOnlineService/BCOnlineSvc.csproj†L8-L12】【F:LandcorSystem/SalesOnDiskService/Program.cs†L27-L31】【F:LandcorSystem/NetsuiteService/Program.cs†L15-L26】
- **Framework versions**: predominant **.NET Framework 4.5.1**, with older 3.5 pockets in the repo (outside main stack).【F:LandcorSystem/Landcor.UI.Realtor/Landcor.UI.Realtor.csproj†L16-L18】【F:LandcorSystem/Landcor.Service/Landcor.Service.csproj†L13-L13】
- **Key libraries**: Entity Framework 6, Serilog packages, Telerik, Enterprise Library Logging.【F:LandcorSystem/Landcor.UI.Realtor/packages.config†L3-L12】【F:LandcorSystem/Landcor.UI.Realtor/Web.config†L12-L15】【F:LandcorSystem/Landcor.Webservice.Public/Web.config†L4-L15】
- **How it runs/deploys**:
  - IIS-hosted web apps/services (Web.config + auth/session/http handlers).【F:LandcorSystem/Landcor.UI.Realtor/Web.config†L131-L203】【F:LandcorSystem/Landcor.Webservice.Public/Web.config†L76-L112】
  - Windows services via `ServiceBase.Run(...)`.【F:LandcorSystem/Landcor.Service/Program.cs†L22-L35】【F:LandcorSystem/BCOnlineService/Program.cs†L51-L62】
  - NetSuite extract can run as scheduled task/manual IIS-hosted flow per code comment.【F:LandcorSystem/NetsuiteService/Program.cs†L13-L17】

### Solution Map
- `LandcorSystem.sln` is the primary monolith solution with many projects across UI, services, data access, integration, and tests.【F:LandcorSystem/LandcorSystem.sln†L25-L132】
- Core runtime projects (high-level responsibilities):
  - `Landcor.UI.Realtor`, `Landcor.UI.Commercial`, `Landcor.UI.Consumer` — Web UIs.【F:LandcorSystem/LandcorSystem.sln†L46-L50】
  - `Landcor.Webservice.Public` — external/public SOAP endpoints for search/reporting operations.【F:LandcorSystem/LandcorSystem.sln†L50-L50】【F:LandcorSystem/Landcor.Webservice.Public/LandcorService.cs†L22-L40】
  - `Landcor.Service` + `Landcor.Service.Internal` — internal service host and service implementations/report generation/transactions.【F:LandcorSystem/LandcorSystem.sln†L58-L66】【F:LandcorSystem/Landcor.Service/App.config†L160-L168】
  - `Landcor.Service.Business`, `Landcor.Service.Business.Realtor`, `Landcor.UI.Process` — business/application logic and orchestration layer.【F:LandcorSystem/LandcorSystem.sln†L73-L80】【F:LandcorSystem/Landcor.UI.Process/Landcor.UI.Process.csproj†L250-L273】
  - `Landcor.DataAccess`, `Landcor.Entity` — database access and entities.【F:LandcorSystem/LandcorSystem.sln†L69-L73】
  - `Landcor.Integration` — external adapters (BC Online, billing, ConstantContact, Solidifi).【F:LandcorSystem/LandcorSystem.sln†L83-L83】【F:LandcorSystem/Landcor.Integration/Billing/HPCIPaymentProcessor.cs†L12-L22】
  - `BCOnlineService`, `SalesOnDiskService`, `NetsuiteService` — integration/background workers.【F:LandcorSystem/LandcorSystem.sln†L30-L37】

## 2) Phase A — Entry points, composition, runtime model

### Entry Points & Request Pipelines
- **Web Forms UI entry**: `Global.asax` initializes logging and invokes login stamp verification on each request-state acquisition.【F:LandcorSystem/Landcor.UI.Realtor/Global.asax.cs†L11-L30】
- **Public SOAP entry**: `LandcorService` web methods authenticate (`AccountUserService.AccountUserAuth`) then call property/report services.【F:LandcorSystem/Landcor.Webservice.Public/LandcorService.cs†L23-L41】
- **Business-to-data flow**: `PropertySearchService` delegates to `PropertySearchMapper` (data access) using stored procedure constants and SQL commands.【F:LandcorSystem/Landcor.Service.Business/PropertySearchService.cs†L23-L33】【F:LandcorSystem/Landcor.DataAccess/PropertySearchMapper.cs†L14-L37】
- **Service host entry**: internal service host process (`Landcor.Service`) starts `TcpWindowsService`; endpoints configured in App.config (net.tcp/webHttp).【F:LandcorSystem/Landcor.Service/Program.cs†L22-L35】【F:LandcorSystem/Landcor.Service/App.config†L160-L175】
- **Windows worker entries**: BC Online and SalesOnDisk run as `ServiceBase`; NetSuite extract starts task logic directly in `Main`.【F:LandcorSystem/BCOnlineService/Program.cs†L34-L62】【F:LandcorSystem/SalesOnDiskService/Program.cs†L15-L31】【F:LandcorSystem/NetsuiteService/Program.cs†L17-L33】

### Dependency Graph Sketch
Observed shape:
1. UI/ASMX/WCF host layer (`Landcor.UI.*`, `Landcor.Webservice.Public`, `Landcor.Service`)  
2. Application orchestration (`Landcor.UI.Process`, `Landcor.Service.Business*`)  
3. Data access (`Landcor.DataAccess` with mappers + EF context)  
4. SQL Server (`connLandcor`, `connRealtor`, `connPropertySearch`)  
5. External integrations (`Landcor.Integration`, generated service references).【F:LandcorSystem/Landcor.UI.Process/Landcor.UI.Process.csproj†L250-L273】【F:LandcorSystem/Landcor.DataAccess/LandcorCommercialModel.Context.cs†L16-L33】【F:LandcorSystem/Landcor.UI.Realtor/Web.config†L99-L104】【F:LandcorSystem/Landcor.Integration/BCOnlinePortal/VerifyUserService.cs†L16-L25】

**DI status:** mostly **manual `new` + static service calls** (no container evidence in inspected entry paths). Example: `new LoginHandler()`, static `AccountUserService`.【F:LandcorSystem/Landcor.UI.Process/Controllers/LoginController.cs†L16-L27】

## 3) Phase B — Domains and seams (bounded contexts)

### Capability Map
| Capability | Evidence in code | Likely DB objects (evidence-based) |
|---|---|---|
| Identity & Account | `AccountUserService`, login stamp logic in `LoginController`; FormsAuth in Web.config. | `ACCOUNT_USER`, `ACCOUNT`, `LoginSecurityStamp`, `LoginHistory`; auth/account sprocs in constants. |
| Property Search & Data | `FindPropertyBy*` methods, `PropertySearchService`, `PropertySearchMapper` using `SP_SEARCH_PROPERTY`/GPS. | `usp_Search_Service_Property` family, property-related product sprocs, `v_Property`/property functions. |
| Shopping Cart & Checkout | `usp_Insert_Shopping_Cart` script handles item insertion, coupons, multi-property items. | `SHOPPING_CART`, `SHOPPING_CART_DETAILS`, `SHOPPING_CART_TRANSACTION`, pricing fn and cart sprocs. |
| Reporting/Product Generation | Many report generators in internal service project; `ReportGenerationMode` config. | Product/report sprocs such as `usp_Product_Profiler`, `usp_Product_*` constants. |
| Billing/Payments | HPCI payment processor and transaction request/response models. | Transaction/cart tables and billing views/functions/sprocs (from constants and SQL folder). |
| External Integrations | BC Online VerifyUser, LTSA references, ConstantContact, Solidifi, NetSuite task. | Integration state likely mixed in `Landcor_Commercial` (UNKNOWN exact table ownership). |

Evidence: identity/account/auth.【F:LandcorSystem/Landcor.UI.Process/Controllers/LoginController.cs†L21-L46】【F:LandcorSystem/Landcor.SQL/Tables/Landcor_Commercial.dbo.ACCOUNT_USER.sql†L9-L71】  
Evidence: property search flow.【F:LandcorSystem/Landcor.Webservice.Public/LandcorService.cs†L22-L41】【F:LandcorSystem/Landcor.Service.Business/PropertySearchService.cs†L23-L33】【F:LandcorSystem/Landcor.DataAccess/PropertySearchMapper.cs†L14-L37】  
Evidence: shopping cart flow/table model.【F:LandcorSystem/Landcor.SQL/Sprocs/Insert_Shopping_Cart.SQL†L36-L80】【F:LandcorSystem/Landcor.SQL/Tables/Landcor_Commercial.dbo.SHOPPING_CART.sql†L7-L21】【F:LandcorSystem/Landcor.SQL/Tables/Landcor_Commercial.dbo.SHOPPING_CART_TRANSACTION.sql†L9-L36】  
Evidence: integration endpoints/types.【F:LandcorSystem/Landcor.Integration/Billing/HPCIPaymentProcessor.cs†L14-L22】【F:LandcorSystem/Landcor.Integration/BCOnlinePortal/VerifyUserService.cs†L16-L25】【F:LandcorSystem/Landcor.Integration/ConstantContact/ConstantContactService.cs†L24-L37】

### Coupling Hotspots
- **Shared DB + many stored procedures** across capabilities; `StoredProcedureConstants` mixes accounts, products, shopping cart, NetSuite, coupons, reports in one class (cross-context coupling).【F:LandcorSystem/Landcor.DataAccess/StoredProcedureConstants.cs†L17-L37】【F:LandcorSystem/Landcor.DataAccess/StoredProcedureConstants.cs†L40-L75】【F:LandcorSystem/Landcor.DataAccess/StoredProcedureConstants.cs†L173-L214】
- **Static services + direct mapper instantiation** reduce testability and hide boundaries (`PropertySearchService`, `LoginController`).【F:LandcorSystem/Landcor.Service.Business/PropertySearchService.cs†L23-L33】【F:LandcorSystem/Landcor.UI.Process/Controllers/LoginController.cs†L16-L27】
- **Configuration-based environment split** (`Debug/Production/Staging` plus many plaintext secrets) raises operational coupling/security risk.【F:LandcorSystem/LandcorSystem.sln†L134-L151】【F:LandcorSystem/Landcor.UI.Realtor/Web.config†L99-L104】
- **Integration mixed into core business projects** via direct project refs (`Landcor.Service.Business` -> `Landcor.Integration`).【F:LandcorSystem/Landcor.Service.Internal/Landcor.Service.Internal.csproj†L258-L269】

### Integration Inventory
- **Payments (HostedPCI/HPCI)**: HTTP POST to hosted PCI endpoints with appSettings credentials/token fields.【F:LandcorSystem/Landcor.Integration/Billing/HPCIPaymentProcessor.cs†L14-L22】【F:LandcorSystem/Landcor.Integration/Billing/HPCIPaymentProcessor.cs†L90-L115】
- **BC Online**: generated WCF SOAP contracts (`VerifyUser`).【F:LandcorSystem/Landcor.Integration/BCOnlinePortal/VerifyUserService.cs†L16-L25】
- **LTSA TitleDirect**: WCF metadata/service references in internal service project and title mode config key in web app/service configs.【F:LandcorSystem/Landcor.Service.Internal/Landcor.Service.Internal.csproj†L169-L177】【F:LandcorSystem/Landcor.UI.Realtor/Web.config†L48-L49】
- **Constant Contact**: external HTTP API calls using login/password query params from config facade.【F:LandcorSystem/Landcor.Integration/ConstantContact/ConstantContactService.cs†L24-L47】
- **SMTP/email**: SMTP host values in configs and logging email listeners.【F:LandcorSystem/Landcor.UI.Realtor/Web.config†L45-L47】【F:LandcorSystem/Landcor.Webservice.Public/Web.config†L10-L12】
- **NetSuite extract**: dedicated executable task path and associated transaction sprocs constants (`SP_GET_NETSUITE_*`).【F:LandcorSystem/NetsuiteService/Program.cs†L17-L33】【F:LandcorSystem/Landcor.DataAccess/StoredProcedureConstants.cs†L173-L180】

## 4) Phase C — Data model & SQL Server reality check

### Data Model Snapshot
- **Databases referenced directly in config**: `Landcor_Commercial_Dev`, `Landcor_Realtor_Dev`, `Landcor_PropertySearch` (plus prod/staging variants in comments/configs).【F:LandcorSystem/Landcor.UI.Realtor/Web.config†L99-L113】
- **Account domain core tables**: `ACCOUNT_USER` (user profile, password fields, language/role/account relations), `ACCOUNT`, role/language/state tables by FK.【F:LandcorSystem/Landcor.SQL/Tables/Landcor_Commercial.dbo.ACCOUNT_USER.sql†L9-L71】
- **Commerce/transaction tables**: `SHOPPING_CART`, `SHOPPING_CART_TRANSACTION` with FK from transaction to cart and cart to account user.【F:LandcorSystem/Landcor.SQL/Tables/Landcor_Commercial.dbo.SHOPPING_CART.sql†L7-L21】【F:LandcorSystem/Landcor.SQL/Tables/Landcor_Commercial.dbo.SHOPPING_CART_TRANSACTION.sql†L9-L36】
- **Hybrid ORM + SP access**: EF context has only a small subset (`ACCOUNT`, `ACCOUNT_USER`, etc.), while most features use stored procedures/ADO.NET mappers.【F:LandcorSystem/Landcor.DataAccess/LandcorCommercialModel.Context.cs†L28-L32】【F:LandcorSystem/Landcor.DataAccess/PropertySearchMapper.cs†L14-L37】

### DB Coupling Report
- **High coupling via transaction script style sprocs**: `usp_Insert_Shopping_Cart` includes product pricing, coupon calculation, multi-property splitting and item creation in one procedure.【F:LandcorSystem/Landcor.SQL/Sprocs/Insert_Shopping_Cart.SQL†L89-L137】【F:LandcorSystem/Landcor.SQL/Sprocs/Insert_Shopping_Cart.SQL†L165-L185】
- **Read model complexity**: `usp_Product_Profiler` builds large temp table and composes from functions/prod DB objects, indicating reporting/query coupling and likely expensive cross-entity reads.【F:LandcorSystem/Landcor.SQL/Sprocs/Product_Profiler.sql†L48-L57】【F:LandcorSystem/Landcor.SQL/Sprocs/Product_Profiler.sql†L174-L220】
- **UNKNOWN**: full set of triggers, SQL Agent jobs, and runtime dependencies between `Landcor_Commercial`, `Landcor_Realtor`, and `Landcor_PropertySearch` cannot be confirmed from inspected files alone.
  - **Inspect next**: DBA scripts for jobs/triggers, live DB metadata (`sys.triggers`, `msdb..sysjobs`), and production query plans/slow proc stats.

## 5) Phase D — Target modular architecture (modular monolith first)

### Proposed Module List
| Module | Responsibility | Owned data (target) | Public contracts |
|---|---|---|---|
| IdentityAccess | Users, auth, session/security stamp, password policy | `ACCOUNT_USER`, `LoginSecurityStamp`, `LoginHistory`, role tables | `IIdentityFacade` (`Authenticate`, `GetUser`, `IssueSessionStamp`) |
| PropertySearch | Search by address/legal/PID/GPS and property lookup | Search-related reads from `PropertySearch` DB and property views/sprocs | `IPropertySearchFacade` (query DTOs) |
| ProductCatalogPricing | Product metadata + price calc + entitlement | `PRODUCT`, pricing/coupon rule tables/functions | `IProductPricingFacade` |
| CartCheckout | Cart, cart items, checkout state, transaction creation | `SHOPPING_CART*`, `SHOPPING_CART_TRANSACTION` | `ICartFacade`, `ICheckoutFacade` |
| ReportingDocuments | Report generation orchestration (server/local, PDF docs) | report request/metadata tables (if any), not identity/cart tables | `IReportFacade` |
| BillingPayments | Payment authorization/capture/credit/void and reconciliation | payment attempt/outbox tables | `IPaymentFacade`, payment events |
| Integrations | Adapters for BC Online, LTSA, ConstantContact, Solidifi, NetSuite | integration config + integration state tables | Adapter interfaces per provider |
| AdminBackoffice | Admin workflows/news/coupons/support tooling | NEWS, PRESS, COUPON and admin audit tables | internal admin APIs |

### Module Boundary Rules (do/don’t)
- **Do** keep module internals private; expose only module facades + contracts project.
- **Do** map current static calls to injected interface facades (starting with anti-corruption adapters).
- **Do** isolate SQL access per module namespace/project (`*.Infrastructure.Sql`).
- **Don’t** allow one module to call another module’s repositories/mappers directly.
- **Don’t** share entity classes across modules except versioned contract DTOs/events.
- **Don’t** keep all stored procedure constants in one global class; split by module.

### Inter-Module Contracts (initial sketches)
- `IdentityAuthenticated` event: `{UserId, SessionId, SourceSite, Timestamp}`
- `CartCheckedOut` event: `{CartId, UserId, TotalBeforeCoupon, TotalAfterCoupon, Items[]}`
- `PaymentAuthorized|PaymentFailed` events: `{TransactionId, GatewayRef, ReasonCode}`
- `ReportRequested|ReportGenerated` events: `{RequestId, ProductId, PropertyId, OutputUri}`
- `ExternalOrderSubmitted` event (BC Online/LTSA/Solidifi): `{Provider, CorrelationId, Status}`

### Dependencies (allowed/forbidden)
- Allowed: `Host -> Module.Application -> Module.Domain + Module.Infrastructure`
- Allowed: `Module.Application -> OtherModule.Contracts` only
- Forbidden: `ModuleA.Infrastructure -> ModuleB.Infrastructure`
- Forbidden: direct SQL reads to tables owned by another module (use facade/read model)

### Test approach per module
- Unit tests for domain/application logic
- Contract tests for facades/events
- Integration tests for SQL mappers and external adapters
- Smoke tests for host composition and critical workflows

## 6) Phase E — Azure-native deployment mapping (PaaS-first)

### Near-term (modular monolith deployed together)
- **Compute**: Azure App Service (Windows) for IIS-hosted .NET Framework apps/services.
  - Rationale: current stack is .NET Framework 4.5.1 + Web Forms/ASMX/WCF/IIS assumptions.【F:LandcorSystem/Landcor.UI.Realtor/Landcor.UI.Realtor.csproj†L11-L19】【F:LandcorSystem/Landcor.Webservice.Public/Web.config†L76-L93】
- **Background workers**: keep Windows-service workloads initially as WebJobs or separate Windows VM/Container fallback (**UNKNOWN** feasibility for each service without install/runtime validation).
- **Database**: Azure SQL Database preferred; if unsupported SQL features/procedural dependencies block migration, use Azure SQL Managed Instance.
- **Secrets**: Azure Key Vault + Managed Identity; remove plaintext config secrets.
- **Observability**: Application Insights + Log Analytics (replace/adapt existing Serilog + EntLib sinks).【F:LandcorSystem/Landcor.UI.Realtor/packages.config†L4-L12】【F:LandcorSystem/Landcor.Webservice.Public/Web.config†L8-L15】
- **Edge**: Azure Front Door + WAF (global entry, TLS termination, path routing for strangler rollout).
- **CI/CD**: GitHub Actions/Azure DevOps with deployment slots and config transformations.

### Longer-term (optional extraction)
- Identity/PropertySearch/ProductCatalog/Cart/Reporting may remain modular monolith unless scale/team autonomy pressures demand extraction.
- Extraction candidates:
  - **Integrations module** -> Azure Functions or Container Apps (event/schedule driven).
  - **Reporting module** -> Container Apps/App Service for CPU-heavy generation.
  - **Billing module** -> App Service + Service Bus command queue for resilient payment pipeline.
- Async backbone:
  - Service Bus for command/work queues (`CheckoutRequested`, `GenerateReport`, `SubmitExternalOrder`)
  - Event Grid for coarse-grained business events where appropriate.

### Networking & Security Baseline
- Private Endpoints for Azure SQL, Key Vault, Storage
- Managed Identity for app-to-resource auth
- RBAC per module deployment identity
- WAF policy for public ingress; restrict admin endpoints
- Data protection baseline: encrypt-at-rest, TLS, key rotation, secret scanning in CI

## 7) Phase F — Incremental migration roadmap (Strangler, low-risk)

### Roadmap
1. **Stabilize and observe**
   - Deliverables: health checks, App Insights, centralized structured logs, baseline smoke tests.
   - Acceptance: p95 latency/error baseline; deploy + rollback rehearsed.
   - Rollback: keep current deployment artifacts/config swap.
   - Risks: hidden runtime dependencies; mitigate with canary + synthetic checks.

2. **Modularize in place**
   - Deliverables: module folders/projects, facade interfaces, module-specific SQL access layers.
   - Acceptance: no direct cross-module data access in new code; dependency checks in CI.
   - Rollback: keep adapter layer delegating to legacy implementations.
   - Risks: regression from static call refactor; mitigate with contract tests.

3. **Data ownership hardening**
   - Deliverables: owned-table registry, read models/views, cross-module access policy.
   - Acceptance: top workflows no longer need cross-module table writes.
   - Rollback: fallback to legacy sprocs while preserving compatibility wrappers.
   - Risks: performance changes; mitigate with query telemetry and index review.

4. **Extract first low-coupled modules**
   - Preferred candidates: Integrations (NetSuite/ConstantContact), Notifications/email pipeline.
   - Deliverables: independently deployable service(s), queue/event contracts, outbox pattern.
   - Acceptance: zero functional change in core checkout/search paths.
   - Rollback: feature flag route back to in-process adapter.

5. **Evaluate further extraction vs. stay modular monolith**
   - Deliverables: ADR with cost/benefit for each candidate module.
   - Acceptance: extraction only when justified by scale/ownership/SLA.
   - Rollback: keep module as in-process component.

### Risk Register (top)
- Plaintext secrets and hard-coded credentials in configs (security breach risk).【F:LandcorSystem/Landcor.UI.Realtor/Web.config†L99-L104】【F:LandcorSystem/Landcor.Service/App.config†L98-L109】
- Legacy auth/session patterns and machine key handling may complicate cloud ingress/session strategy.【F:LandcorSystem/Landcor.UI.Realtor/Web.config†L170-L203】【F:LandcorSystem/Landcor.UI.Process/Controllers/LoginController.cs†L48-L85】
- Stored-procedure-heavy transactional logic tightly couples domains and DB shape.【F:LandcorSystem/Landcor.SQL/Sprocs/Insert_Shopping_Cart.SQL†L36-L57】【F:LandcorSystem/Landcor.DataAccess/StoredProcedureConstants.cs†L17-L37】
- Mixed hosting model (IIS + Windows services + scheduled tasks) raises migration sequencing complexity.【F:LandcorSystem/Landcor.Service/Program.cs†L22-L35】【F:LandcorSystem/NetsuiteService/Program.cs†L15-L33】

### Open Questions / Required Discovery
- UNKNOWN: production traffic profile by endpoint/workflow and peak windows.
- UNKNOWN: exact SQL Agent jobs, triggers, linked servers, and CLR dependencies in production.
- UNKNOWN: external SLA/contracts and credential rotation process for BC Online/LTSA/HPCI/Solidifi.
- UNKNOWN: PCI segmentation and compensating controls currently in place.
- UNKNOWN: which apps are actively used vs. dormant projects in solution.

## Security-first observations
- Multiple plaintext secrets/API keys/DB passwords in repository config should be prioritized for immediate remediation (Key Vault + secret rotation).【F:LandcorSystem/Landcor.UI.Realtor/Web.config†L50-L59】【F:LandcorSystem/Landcor.UI.Realtor/Web.config†L99-L104】【F:LandcorSystem/Landcor.Service/App.config†L98-L113】
- PII exists in account tables (`name`, `email`, password-related fields, secret Q/A) and requires stricter access control, auditing, and data minimization during modularization.【F:LandcorSystem/Landcor.SQL/Tables/Landcor_Commercial.dbo.ACCOUNT_USER.sql†L11-L21】【F:LandcorSystem/Landcor.SQL/Tables/Landcor_Commercial.dbo.ACCOUNT_USER.sql†L14-L18】
