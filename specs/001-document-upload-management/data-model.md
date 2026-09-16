# Data Model: Document Upload and Management

## Document

Represents an active or quarantined uploaded file and its searchable metadata.

| Field | Type/shape | Rules |
|---|---|---|
| DocumentId | integer | Primary key; consistent with existing entities |
| Title | text | Required; trimmed; non-empty |
| Description | text, optional | User-provided descriptive text |
| Category | text | Required; one of Project Documents, Team Resources, Personal Files, Reports, Presentations, Other |
| Tags | text, optional | User-provided searchable tags |
| FileType | text | Required MIME type; supports up to 255 characters |
| FileSize | integer | Required; greater than zero and no more than 25 MB |
| OriginalFileName | text | Display-only metadata; never used as a storage path |
| FilePath | text | Required relative storage name; GUID-based and outside `wwwroot` |
| Status | text | Quarantined, Available, Rejected, or Deleted; only Available documents are accessible |
| UploadedAt | datetime | Required |
| UploadedByUserId | integer | Required foreign key to User |
| ProjectId | integer, optional | Foreign key to Project; access follows project membership plus explicit sharing |
| TaskId | integer, optional | Foreign key to TaskItem; task association must be within the task's project |
| ScannedAt | datetime, optional | Set only after safety scan completes |
| ScanAttempts | integer | Number of scan jobs attempted; bounded by retry policy |
| LastScanError | text, optional | Non-sensitive diagnostic state for administrators |

### Relationships

- User 1-to-many Document through `UploadedByUserId`.
- Project 1-to-many Document through optional `ProjectId`.
- TaskItem 1-to-many Document through optional `TaskId`.
- Document 1-to-many DocumentShare.
- Document 1-to-many DocumentActivity, retained after active deletion.

## DocumentShare

Represents explicit read-only access for an individual user or the members of the associated project team.

| Field | Type/shape | Rules |
|---|---|---|
| DocumentShareId | integer | Primary key |
| DocumentId | integer | Required foreign key |
| SharedWithUserId | integer, optional | Individual recipient; may be outside the project |
| SharedWithProjectTeam | boolean | When true, resolves to members of the associated project |
| SharedAt | datetime | Required |
| SharedByUserId | integer | Required foreign key to User |
| RevokedAt | datetime, optional | Revocation removes access without deleting audit history |

Exactly one recipient mode is required: an individual user or the associated project team.
Sharing never grants edit, replace, or delete rights.

## DocumentActivity

Minimal administrator-visible audit record for document activity.

| Field | Type/shape | Rules |
|---|---|---|
| DocumentActivityId | integer | Primary key |
| DocumentId | integer | Retained identifier even after active deletion |
| TitleSnapshot | text | Captured at action time; no file contents |
| ActorUserId | integer | Required foreign key to User |
| ActionType | text | Upload, Download, Delete, Share, Replace, MetadataUpdate, or ScanResult |
| OccurredAt | datetime | Required |
| Details | text, optional | Non-sensitive action context |

Deletion removes the active Document row/file or marks it unavailable according to implementation, but retains the minimal activity record required by the specification.

## Authorization Rules

- Employees can create personal documents and project documents only where they are authorized project members.
- Team Leads can manage documents uploaded by their team members according to existing role semantics.
- Project Managers can manage documents associated with their projects.
- Administrators can access all active documents and audit reports.
- Owners and explicit recipients can read available documents; explicit recipients are read-only even without project membership.
- Quarantined, rejected, and deleted documents are not returned by active list, search, preview, or download operations.

## Scan Job

The scan job is an integration message, not a persisted business entity. It contains:

- `DocumentId`: integer document identifier.
- `FilePath`: generated relative storage path resolved through `IFileStorageService`.
- `ContentType`: captured MIME type used by the scanner.
- `Attempt`: delivery/processing attempt number.
- `RequestedAt`: UTC enqueue time.

The message is published only after the file and quarantined metadata are saved successfully. A queue-triggered worker MUST treat delivery as at-least-once, re-read current document state, and avoid changing documents that are already Available, Rejected, or Deleted.
