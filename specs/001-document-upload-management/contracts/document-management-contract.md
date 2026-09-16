# Document Management Contracts

## User-Facing Contract

The protected document-management experience exposes these capabilities:

- Upload one or more supported files with title, category, and optional description, tags, project, and task.
- Show per-file progress and a success or failure result for every file.
- List, sort, filter, and search only documents the current user may access.
- Preview authorized PDFs and images; download any authorized available file.
- Edit metadata, replace files, delete after confirmation, and share read-only access.
- Show shared documents in `Shared with Me`, recent documents on the dashboard, and document counts.
- Show in-app notifications for shares and eligible project additions.
- Restrict audit reports and activity details to administrators.

## Storage Boundary Contract

The business layer depends on an abstraction equivalent to:

```text
UploadAsync(stream, relativeName, contentType) -> stored relative path
DeleteAsync(relativePath) -> completion
DownloadAsync(relativePath) -> readable stream
GetUrlAsync(relativePath, expiration) -> delivery reference
```

Implementations MUST:

- Store files outside publicly served web content.
- Accept only generated relative paths; never use an untrusted filename as a path.
- Preserve the GUID-based `{userId}/{projectId-or-personal}/{uniqueId}.{extension}` organization.
- Report storage failures without leaving an accessible metadata record.
- Permit a local filesystem implementation without requiring cloud SDK dependencies.

## Asynchronous Scan Contract

After a valid file is saved and its Document record is created in `Quarantined` status, the application MUST publish one scan message to the configured scan queue. The message schema is:

```json
{
	"documentId": 123,
	"filePath": "42/17/8c0f...pdf",
	"contentType": "application/pdf",
	"attempt": 1,
	"requestedAt": "2026-09-16T15:00:00Z"
}
```

The production adapter uses Azure Queue Storage and an Azure Function with a Queue Storage trigger. The function MUST:

1. Resolve the document by integer identifier and verify it is still Quarantined.
2. Read the file through the storage boundary rather than constructing a public URL.
3. Run the configured malware scanner.
4. Set the document to Available only on a successful clean result, or Rejected on a positive detection.
5. Record a ScanResult activity without storing file contents or scanner secrets.
6. Treat delivery as at-least-once and make repeated messages idempotent.
7. Allow transient failures to retry and route exhausted messages to a poison queue for administrator review.

The offline adapter MUST implement the same message and state-transition contract without requiring Azure services. Scan-unavailable files remain Quarantined and inaccessible.

## Authorization Contract

Every operation receives the authenticated user context and applies these rules:

| Operation | Allowed actors |
|---|---|
| Upload personal document | Authenticated employee or higher role |
| Upload project document | Authorized project member or project manager |
| View/download/preview | Owner, project member, explicit share recipient, or administrator; document must be Available |
| Edit metadata/replace | Owner or authorized project manager |
| Delete | Owner, authorized project manager, or administrator |
| Share | Owner or authorized project manager; recipient access is read-only |
| Audit report | Administrator only |

Explicit individual shares grant read-only access independent of project membership. Team shares resolve to members of the associated project team. No operation may authorize solely from a route identifier.

## Error Contract

User-visible failures MUST identify the affected file and reason without exposing filesystem paths, internal exception details, or unauthorized metadata. Batch uploads return independent outcomes; one rejected or quarantined file does not invalidate other valid files.
