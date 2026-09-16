# Quickstart: Document Upload and Management

## Prerequisites

- .NET 10 SDK
- PowerShell
- Repository checkout at `C:\trainingProjects\ContosoRep`
- No cloud services required

## Start the application

```powershell
Set-Location .\ContosoDashboard
dotnet run
```

Open the HTTPS URL printed by the application and sign in through the mock login page. Use the seeded users listed in `README.md` to verify role differences.

## Core validation scenarios

1. **Single upload**: As an Employee, upload a supported file at or below 25 MB with a title and category. Confirm per-file progress, an Available result after scanning, metadata in My Documents, and a recent-document dashboard entry.
2. **Validation rejection**: Attempt an unsupported file and a file larger than 25 MB. Confirm each is rejected with a clear reason and no active document or downloadable file is created.
3. **Quarantine behavior**: Simulate an unavailable or unsuccessful malware scan. Confirm the file remains quarantined and is absent from lists, search, preview, and download until a scan succeeds.
4. **Partial batch success**: Upload a valid file together with an invalid file. Confirm the valid file completes and the invalid file has an independent failure result.
5. **Project access**: Sign in as a project member and confirm project documents are visible and downloadable. Sign in as a non-member and confirm they are absent and inaccessible by direct URL.
6. **Explicit sharing**: Share a document with an individual outside its project. Confirm read-only access and notification. Confirm the recipient cannot edit, replace, or delete it.
7. **Team sharing**: Share a project document with its project team. Confirm current project members see it in Shared with Me and department-only users do not receive access.
8. **Lifecycle management**: As owner or project manager, edit metadata, replace a file, and delete after confirmation. Confirm old file references cannot download the removed file.
9. **Audit retention**: As Administrator, confirm upload, download, share, replace, and delete actions appear in activity reporting. After deletion, confirm the minimal title/identifier/actor/action/time record remains without file contents.
10. **Task and dashboard integration**: Attach or upload from an authorized task. Confirm project inheritance, notification behavior, recent five documents, and document count.
11. **Role isolation**: As Employee, Team Lead, Project Manager, and Administrator, confirm each role can perform only the actions defined in the authorization contract.

## Asynchronous scan validation

### Offline training path

- Configure the local scan queue/background worker adapter.
- Upload a supported file and confirm the request completes with the document in Quarantined status.
- Confirm one scan message is created only after the file and metadata are saved.
- Complete a clean scan and confirm the worker changes the document to Available and records ScanResult activity.
- Simulate a scanner error and confirm retry behavior leaves the document Quarantined; simulate a positive detection and confirm Rejected status.
- Deliver the same message twice and confirm the second delivery does not duplicate state transitions or expose a deleted document.

### Azure integration path (optional deployment)

- Configure an Azure Storage Queue connection and queue name through deployment settings; do not hard-code credentials.
- Deploy the scanner Function with a Queue Storage trigger bound to the scan queue and a poison queue for exhausted retries.
- Upload a file through the Blazor application and confirm the queue message contains only the documented scan-job fields.
- Confirm the Function updates the shared document store through the application/service boundary and records a ScanResult activity.
- Confirm transient Function failures retry and an exhausted message is visible for administrator review without making the file available.

## Performance smoke checks

- Seed or create up to 500 authorized documents and confirm list load completes within 2 seconds.
- Search the same collection by title, description, tag, uploader, and project and confirm results return within 2 seconds.
- Upload a 25 MB supported file under typical local/network conditions and confirm completion within 30 seconds, excluding scan time when the scan is intentionally unavailable.
- Preview an authorized PDF and image and confirm each loads within 3 seconds.

## Verification references

- Data and state rules: [data-model.md](data-model.md)
- User/storage/authorization behavior: [contracts/document-management-contract.md](contracts/document-management-contract.md)
- Acceptance criteria: [spec.md](spec.md)
