# Implementation Plan: Document Upload and Management

**Branch**: `001-document-upload-management` | **Date**: 2026-09-16 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/001-document-upload-management/spec.md`

## Summary

Add secure document upload and management to the existing offline Blazor Server dashboard. The implementation will introduce integer-keyed document metadata, explicit sharing and audit entities, a quarantine-first upload workflow, and a replaceable `IFileStorageService` with a local filesystem implementation. After durable local save, a scan-job message is published asynchronously; an optional Azure Function with an Azure Queue Storage trigger performs malware scanning and updates the document state idempotently. Existing project, task, user, notification, authentication, and dashboard services remain the integration boundaries. Protected delivery operations and service methods enforce authorization independently of route identifiers.

## Technical Context

**Language/Version**: C# / .NET 10.0  
**Primary Dependencies**: ASP.NET Core Blazor Server, Razor Pages, Entity Framework Core 10 SQLite, existing mock cookie authentication; optional Azure Storage Queues and Azure Functions Queue Storage trigger for asynchronous scanning  
**Storage**: SQLite metadata in `App_Data/contosodashboard.db`; local files outside `wwwroot` under application data; scan messages use a local queue adapter offline or Azure Queue Storage in the optional cloud deployment; abstraction supports future Azure Blob implementation  
**Testing**: Existing repository has no test project; focused manual verification in `quickstart.md`, with automated tests recommended when a harness is introduced  
**Target Platform**: Windows or other .NET 10 host; offline-capable local development  
**Project Type**: Single web application  
**Performance Goals**: Search and list views within 2 seconds for 500 documents; 25 MB upload within 30 seconds under typical conditions; preview within 3 seconds  
**Constraints**: Training-only mock authentication; local filesystem required; files outside `wwwroot`; 25 MB limit; whitelist file types; quarantine until asynchronous scan success; integer IDs; text categories; cloud services optional for training  
**Scale/Scope**: Existing seeded ContosoDashboard users/projects/tasks; one feature spanning data, services, protected file delivery, Blazor pages, dashboard, task, notification, and reporting surfaces

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **Training Purpose and Scope**: PASS. The design remains offline-first and does not add production identity or cloud dependencies.
- **Layered Design and Replaceable Infrastructure**: PASS. Storage is behind `IFileStorageService`; UI, services, entities, and persistence remain separate.
- **Authorization by Default**: PASS. Service operations and protected file delivery apply ownership, role, membership, and explicit-share rules; route IDs are not authorization.
- **Verifiable Behavior**: PASS. The clarified spec, contract, and repeatable quickstart cover acceptance, IDOR, quarantine, partial batches, sharing, deletion audit retention, and performance.
- **Simplicity and Maintainability**: PASS. The plan extends existing services and `ApplicationDbContext` without a second application or unnecessary dependency.
- **Security and Technology Constraints**: PASS. Files use generated relative paths outside `wwwroot`, scanning precedes access, and existing authentication/security middleware remains intact.

## Project Structure

### Documentation (this feature)

```text
specs/001-document-upload-management/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── document-management-contract.md
├── checklists/
│   └── requirements.md
└── spec.md
```

### Source Code (repository root)

```text
ContosoDashboard/
├── Data/
│   └── ApplicationDbContext.cs
├── Models/
│   ├── Document.cs                 # New active document metadata
│   ├── DocumentShare.cs            # New explicit sharing records
│   └── DocumentActivity.cs         # New retained audit records
├── Services/
│   ├── IFileStorageService.cs      # New replaceable storage boundary
│   ├── LocalFileStorageService.cs  # New offline filesystem implementation
│   ├── IDocumentService.cs         # New document business contract
│   ├── DocumentService.cs          # New validation/auth/workflow service
│   ├── IScanQueue.cs                # New scan-job publishing boundary
│   ├── LocalScanQueueWorker.cs      # New offline background scan worker
│   ├── AzureQueueScanPublisher.cs   # Optional Azure Queue Storage adapter
│   ├── NotificationService.cs      # Extend for document notifications
│   ├── DashboardService.cs         # Extend recent documents/count
│   └── TaskService.cs              # Extend authorized task attachment context
├── Pages/
│   ├── Documents.razor             # New list/search/filter/manage view
│   ├── ProjectDetails.razor        # Extend project document section
│   ├── Tasks.razor                 # Extend task attachment flow
│   ├── Index.razor                 # Extend recent documents/count
│   └── DocumentDownload.cshtml.cs  # Protected delivery handler or equivalent endpoint
├── Shared/
│   └── NavMenu.razor               # Add protected Documents navigation
├── wwwroot/
│   └── css/site.css                # Document upload/list states
└── Program.cs                      # Register storage/document services and delivery route

ContosoDocumentScanner/              # Optional Azure deployment component
└── Functions/
	└── DocumentScanFunction.cs      # Azure Queue Storage-triggered scan handler
```

**Structure Decision**: Keep the existing single ASP.NET Core web project for the offline training path. Add document entities under `Models`, persistence configuration in `Data/ApplicationDbContext.cs`, business rules and queue abstractions in `Services`, and protected Blazor/Razor delivery surfaces in `Pages`. The optional `ContosoDocumentScanner` Azure Functions component owns only queue-triggered scanning; it communicates through the documented scan-job and persistence boundaries. Do not serve uploaded files from `wwwroot`.

## Phase 0: Research Summary

Research decisions are recorded in [research.md](research.md): reuse the current Blazor/service architecture, isolate storage behind an interface, separate active documents from retained audit events, use integer/text constraints, process batches independently, enforce service-level authorization, process scans asynchronously through a Queue Storage-triggered Azure Function when deployed, and use manual verification because no test project exists.

## Phase 1: Design Summary

- [data-model.md](data-model.md) defines `Document`, `DocumentShare`, and `DocumentActivity`, relationships, states, fields, and authorization rules.
- [contracts/document-management-contract.md](contracts/document-management-contract.md) defines the user-facing, storage, authorization, error, and asynchronous scan-job contracts.
- [quickstart.md](quickstart.md) defines repeatable end-to-end and performance validation scenarios.

## Post-Design Constitution Re-check

All gates continue to pass. Azure Functions and Queue Storage are optional adapters for the production migration path; the offline implementation remains local and cloud-free. The design does not introduce a justified constitution violation, so the Complexity Tracking section remains empty.

## Complexity Tracking

No constitution violations requiring justification.
