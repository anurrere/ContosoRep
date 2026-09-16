# Tasks: Document Upload and Management

**Input**: Design documents from `specs/001-document-upload-management/`
**Prerequisites**: [plan.md](plan.md), [spec.md](spec.md), [data-model.md](data-model.md), [contracts/document-management-contract.md](contracts/document-management-contract.md), [quickstart.md](quickstart.md)

## Phase 1: Setup

- [ ] T001 Create the document feature source folders and placeholder boundaries under `ContosoDashboard/Models`, `ContosoDashboard/Services`, `ContosoDashboard/Pages`, and `ContosoDashboard/App_Data/uploads` without placing uploads under `ContosoDashboard/wwwroot`
- [ ] T002 [P] Add the optional Azure scanner project structure under `ContosoDocumentScanner/Functions` and document its independent deployment boundary in `ContosoDocumentScanner/README.md`
- [ ] T003 [P] Add document storage, queue, and scan configuration sections to `ContosoDashboard/appsettings.json` and `ContosoDashboard/appsettings.Development.json` with local-safe defaults and no credentials

## Phase 2: Foundational

- [ ] T004 Add `Document`, `DocumentShare`, and `DocumentActivity` entities with integer keys, required fields, text categories, quarantine state, scan attempt state, and relationships in `ContosoDashboard/Models/Document.cs`, `ContosoDashboard/Models/DocumentShare.cs`, and `ContosoDashboard/Models/DocumentActivity.cs`
- [ ] T005 Configure document, share, activity, User, Project, and TaskItem relationships, indexes, field lengths, and active-status query support in `ContosoDashboard/Data/ApplicationDbContext.cs`
- [ ] T006 [P] Define storage and scan queue abstractions in `ContosoDashboard/Services/IFileStorageService.cs` and `ContosoDashboard/Services/IScanQueue.cs` using the contracts in `specs/001-document-upload-management/contracts/document-management-contract.md`
- [ ] T007 [P] Implement secure generated-path local file operations outside `wwwroot` in `ContosoDashboard/Services/LocalFileStorageService.cs`, rejecting path traversal and untrusted filenames
- [ ] T008 Implement the local scan queue/background worker in `ContosoDashboard/Services/LocalScanQueueWorker.cs` with the same scan-job schema, quarantine-only access, retry handling, and idempotent state transitions
- [ ] T009 Register the document service, local storage, scan queue, background worker, and configuration options in `ContosoDashboard/Program.cs` while preserving existing authentication and security middleware
- [ ] T010 Add shared document category, status, file-type whitelist, size-limit, and scan-result validation constants in `ContosoDashboard/Services/DocumentValidation.cs`

## Phase 3: User Story 1 - Upload and Organize a Document (Priority: P1)

**Goal**: Authenticated users can upload supported files with required metadata, receive per-file outcomes, and keep files quarantined until a clean scan.

**Independent Test**: Upload a supported file, verify it is stored outside `wwwroot` with `Quarantined` status and a queued scan job, then complete a clean scan and verify it becomes `Available` with searchable metadata.

- [ ] T011 [US1] Define upload, metadata, per-file result, and scan-job request models in `ContosoDashboard/Services/DocumentUploadModels.cs`
- [ ] T012 [US1] Implement validation for the exact 25 MB limit, supported PDF/Office/text/JPEG/PNG types, required title/category, allowed category text values, and optional description/project/task/tags in `ContosoDashboard/Services/DocumentValidation.cs`
- [ ] T013 [US1] Implement the durable upload sequence in `ContosoDashboard/Services/DocumentService.cs`: authorize the user, generate `{userId}/{projectId-or-personal}/{guid}.{extension}`, save the file, create quarantined metadata, enqueue one scan job, and clean up on failure
- [ ] T014 [US1] Implement independent multi-file upload outcomes and per-file progress state in `ContosoDashboard/Services/DocumentService.cs`, ensuring one invalid or unsafe file does not cancel valid files
- [ ] T015 [US1] Add the protected upload and metadata form with per-file progress, success/error results, quarantine messaging, and `@key` reset behavior in `ContosoDashboard/Pages/Documents.razor`
- [ ] T016 [US1] Register document navigation and upload styling states in `ContosoDashboard/Shared/NavMenu.razor` and `ContosoDashboard/wwwroot/css/site.css`
- [ ] T017 [US1] Add upload, quarantine, scan-result, and failure activity records through `ContosoDashboard/Services/DocumentService.cs` without storing file contents or scanner secrets

## Phase 4: User Story 2 - Find and Use Authorized Documents (Priority: P1)

**Goal**: Users can list, sort, filter, search, preview, and download only available documents they are authorized to access.

**Independent Test**: Verify owner, project-member, explicit-share, administrator, non-member, quarantined, rejected, and deleted access outcomes through both UI actions and direct delivery requests.

- [ ] T018 [US2] Implement authorized document queries with active-status filtering, ownership/project/share/admin rules, title/category/date/size sorting, category/project/date filters, and title/description/tag/uploader/project search in `ContosoDashboard/Services/DocumentService.cs`
- [ ] T019 [US2] Add the protected document list, search, sort, and filter UI with empty, quarantined, rejected, and unavailable states in `ContosoDashboard/Pages/Documents.razor`
- [ ] T020 [US2] Implement authorization-checked download and preview delivery outside `wwwroot` in `ContosoDashboard/Pages/DocumentDownload.cshtml.cs` or the equivalent protected endpoint
- [ ] T021 [US2] Add project document loading and member-only display/download behavior in `ContosoDashboard/Pages/ProjectDetails.razor`
- [ ] T022 [US2] Add download and preview activity recording and ensure stale links cannot access replaced, deleted, rejected, or quarantined files in `ContosoDashboard/Services/DocumentService.cs`

## Phase 5: User Story 3 - Maintain Owned and Project Documents (Priority: P2)

**Goal**: Owners and authorized project managers can update metadata, replace files, and permanently delete documents while preserving minimal audit evidence.

**Independent Test**: As owner and project manager, edit, replace, and delete permitted documents; as an unauthorized user, repeat each operation and verify denial plus unchanged data.

- [ ] T023 [US3] Implement owner/project-manager/admin authorization for metadata updates, replacement, deletion, and audit-report access in `ContosoDashboard/Services/DocumentService.cs`
- [ ] T024 [US3] Implement metadata edit and replacement workflows with validation, new generated paths, old-file cleanup, quarantine rescan, and no stale-download access in `ContosoDashboard/Services/DocumentService.cs`
- [ ] T025 [US3] Implement confirmed deletion that removes active file access and metadata while retaining `DocumentActivity` title snapshot, document identifier, actor, action, and timestamp in `ContosoDashboard/Services/DocumentService.cs`
- [ ] T026 [US3] Add edit, replace, confirmation-delete, and failure states to `ContosoDashboard/Pages/Documents.razor`
- [ ] T027 [US3] Add administrator activity and summary reporting for document types, active uploaders, access patterns, and retained delete events in `ContosoDashboard/Pages/DocumentReports.razor` and `ContosoDashboard/Services/DocumentService.cs`

## Phase 6: User Story 4 - Share Documents and Connect Them to Work (Priority: P2)

**Goal**: Owners can grant read-only individual or project-team access, and users can attach documents to authorized tasks with notifications.

**Independent Test**: Share with an individual outside the project and with the associated project team; verify read-only access, notification, Shared with Me visibility, and no management rights. Attach a document from an authorized task and verify project inheritance.

- [ ] T028 [US4] Implement individual and associated-project-team share creation, revocation, read-only authorization, duplicate-share prevention, and share audit events in `ContosoDashboard/Services/DocumentService.cs`
- [ ] T029 [US4] Add Shared with Me queries and sharing controls to `ContosoDashboard/Pages/Documents.razor`, resolving team recipients from current project membership
- [ ] T030 [US4] Extend `ContosoDashboard/Services/NotificationService.cs` with document-share and project-document notification creation for eligible recipients
- [ ] T031 [US4] Add task document attachment and upload context with task-project consistency checks in `ContosoDashboard/Pages/Tasks.razor` and `ContosoDashboard/Services/TaskService.cs`
- [ ] T032 [US4] Add project notification and task attachment audit events in `ContosoDashboard/Services/DocumentService.cs`

## Phase 7: User Story 5 - Monitor Document Activity (Priority: P3)

**Goal**: Administrators can review document activity and generate reports while non-administrators remain denied.

**Independent Test**: Perform upload, scan, download, share, replace, metadata-update, and delete operations, then verify administrator-only activity/report output and retained delete evidence.

- [ ] T033 [US5] Implement administrator-only activity queries and report aggregation for document types, active uploaders, access patterns, and scan outcomes in `ContosoDashboard/Services/DocumentService.cs`
- [ ] T034 [US5] Complete administrator report filtering, empty states, and authorization denial handling in `ContosoDashboard/Pages/DocumentReports.razor`

## Phase 8: Optional Azure Asynchronous Scan Deployment

**Goal**: Provide the production migration adapter using Azure Queue Storage and an Azure Function without making Azure required for offline training.

**Independent Test**: Upload a document with the Azure adapter enabled, verify the exact queue message, Function-triggered scan, guarded state transition, retries, poison queue behavior, and idempotent duplicate delivery.

- [ ] T035 [P] Implement the Azure Queue Storage publisher adapter in `ContosoDashboard/Services/AzureQueueScanPublisher.cs` using configuration-based queue and connection settings without hard-coded secrets
- [ ] T036 [P] Create the optional Azure Functions project and Queue Storage-triggered handler in `ContosoDocumentScanner/Functions/DocumentScanFunction.cs` with the documented scan-job schema
- [ ] T037 Implement scanner processing in `ContosoDocumentScanner/Functions/DocumentScanFunction.cs`: resolve the document, read through the storage boundary, run malware scanning, update only `Quarantined` records, record `ScanResult`, and handle duplicate delivery idempotently
- [ ] T038 Configure retry limits, poison queue routing, structured failure logging, and non-sensitive diagnostics for the Azure Function in `ContosoDocumentScanner/host.json` and `ContosoDocumentScanner/local.settings.example.json`
- [ ] T039 Document local versus Azure deployment configuration and queue-trigger verification in `ContosoDocumentScanner/README.md` and `specs/001-document-upload-management/quickstart.md`

## Phase 9: Dashboard Integration and Polish

- [ ] T040 [P] Extend `ContosoDashboard/Services/DashboardService.cs` with the user’s five most recent available documents and document count, excluding quarantined/rejected/deleted records
- [ ] T041 [P] Add Recent Documents and document-count presentation states to `ContosoDashboard/Pages/Index.razor`
- [ ] T042 [P] Add document list, upload, quarantine, preview, sharing, notification, and report styling refinements in `ContosoDashboard/wwwroot/css/site.css`
- [ ] T043 Update `README.md` with local storage location, quarantine/scan behavior, optional Azure Queue Storage and Function configuration, and security verification steps
- [ ] T044 Execute all repeatable scenarios in `specs/001-document-upload-management/quickstart.md`, including IDOR denial, partial batch success, scan retries, poison queue handling, duplicate delivery, sharing permissions, deletion audit retention, and performance checks

## Dependencies and Execution Order

```text
Setup (T001-T003)
  -> Foundational (T004-T010)
  -> US1 Upload (T011-T017)
  -> US2 Retrieval (T018-T022)
  -> US3 Maintenance (T023-T027)
  -> US4 Sharing/Integration (T028-T032)
  -> US5 Reporting (T033-T034)
  -> Optional Azure Scan (T035-T039; requires T006, T008, T013, T017)
  -> Dashboard/Polish (T040-T044)
```

User Story 1 is the MVP and must be independently demonstrable before expanding retrieval and management. User Story 2 depends on documents and authorization from User Story 1. User Stories 3 and 4 build on available document retrieval and metadata. User Story 5 depends on activity events from the preceding stories. The Azure deployment phase is optional for offline training but depends on the shared scan contract and quarantine workflow.

## Parallel Execution Examples

### Setup and foundation

```text
T002 (Azure project docs) || T003 (configuration)
T006 (interfaces) || T007 (local storage) || T010 (validation constants)
```

### User Story 1

```text
T011 (upload models) || T012 (validation rules)
T015 (upload UI) || T016 (navigation/styles)
```

### User Story 2

```text
T019 (document list UI) || T021 (project view)
T020 (protected delivery) || T022 (download audit)
```

### User Story 4 and optional Azure scan

```text
T029 (sharing UI) || T030 (notifications) || T031 (task attachments)
T035 (queue publisher) || T036 (Function scaffold)
```

### Polish

```text
T040 (dashboard service) || T041 (dashboard UI) || T042 (styles)
```

## Implementation Strategy

1. **MVP**: Complete Setup, Foundational, User Story 1, and the minimum authorized retrieval slice from User Story 2. This delivers secure local upload, quarantine, scan processing, and access-controlled document listing.
2. **Incremental delivery**: Complete the remainder of User Story 2, then User Stories 3 and 4, followed by administrator reporting in User Story 5.
3. **Migration adapter**: Add the optional Azure Queue Storage publisher and Queue-triggered Function only after the local scan contract and state transitions are stable.
4. **Verification**: Run the quickstart scenarios after each story and perform the full cross-cutting verification before completion.
