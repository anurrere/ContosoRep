# Research: Document Upload and Management

## Decision: Use the existing Blazor Server and service-layer architecture

**Rationale**: The application already uses Razor Pages for authentication, Blazor Server for protected user workflows, EF Core SQLite for persistence, and scoped services for business operations. Extending those boundaries avoids a major rewrite and follows the constitution's simplicity and layered-design principles.

**Alternatives considered**: Adding a separate SPA/API project was rejected because it would introduce unnecessary deployment and authentication complexity for an offline training application.

## Decision: Add an infrastructure storage abstraction with a local implementation

**Rationale**: The feature requires local filesystem storage outside `wwwroot` and an Azure migration path. An `IFileStorageService` boundary lets `DocumentService` coordinate validation, authorization, persistence, and notifications without depending on `System.IO`. The local implementation can generate GUID-based relative paths and perform save, read, delete, and URL/response preparation operations.

**Alternatives considered**: Direct filesystem calls from the Blazor page were rejected because they would violate layered design, complicate authorization, and block future storage replacement.

## Decision: Store active documents and audit events separately

**Rationale**: Deletion must remove active file access and metadata while retaining a minimal administrator-visible audit record. Separate document and activity records allow active queries to exclude deleted documents without destroying evidence needed for reporting.

**Alternatives considered**: Soft-deleting the full document record was rejected because it could retain sensitive metadata longer than necessary and make active search/download filtering error-prone. Deleting audit rows was rejected by the clarification decision.

## Decision: Use integer document keys and text categories

**Rationale**: The stakeholder specification requires integer identifiers consistent with existing entities and human-readable category values. Categories should be validated against a fixed allowed list at the service boundary.

**Alternatives considered**: GUID document keys and enum-backed categories were rejected because they contradict the approved feature constraints.

## Decision: Process multi-file uploads independently with quarantine-first status

**Rationale**: Each file can succeed or fail without invalidating its neighbors. A quarantine/pending-scan state prevents access until malware scanning succeeds, including when scanning is unavailable. The UI can report per-file progress and outcome.

**Alternatives considered**: Atomic batch processing was rejected because one bad file would unnecessarily prevent valid files from being stored. Treating scan-unavailable files as safe was rejected by the security clarification.

## Decision: Process malware scans asynchronously with an Azure Functions Queue Storage trigger

**Rationale**: The upload request must save each valid file and metadata quickly while keeping the document quarantined. After the file and metadata are durably stored, the application publishes a scan message containing the document identifier, relative storage path, content type, and attempt metadata to Azure Queue Storage. An Azure Function with a Queue Storage trigger consumes the message, reads the file through the storage abstraction, runs the malware scanner, and updates the document to Available or Rejected. Retry and poison-queue behavior prevents transient scanner failures from exposing files or blocking unrelated uploads.

The offline training path remains cloud-free: it uses the same scan-job contract with a local queue/background worker or deterministic scanner adapter. Azure Queue Storage and the Azure Function are optional deployment adapters selected through configuration, not required dependencies of the Blazor application.

**Alternatives considered**: Scanning synchronously inside the Blazor upload request was rejected because it would make 25 MB uploads dependent on scanner latency and could exhaust request resources. A timer-only in-process job was rejected as the production integration because it is less durable across restarts and does not provide Queue Storage retry/poison-message semantics.

## Decision: Make scan messages idempotent and status transitions guarded

**Rationale**: Queue delivery is at-least-once, so the function may receive the same message more than once. The scan handler must re-read the document, ignore already Available, Rejected, or Deleted records, and apply a result only when the document is still Quarantined. Updates must be atomic and auditable through a ScanResult activity.

**Alternatives considered**: Assuming exactly-once queue delivery was rejected because retries and duplicate deliveries are normal failure behavior for queue-triggered functions.

## Decision: Enforce authorization in services and protected delivery endpoints

**Rationale**: Project membership, explicit read-only shares, ownership, project-manager rights, and administrator reporting are resource decisions. Service methods and the file delivery path must independently authorize access so a URL or stale client state cannot bypass IDOR protections.

**Alternatives considered**: Relying only on page visibility was rejected because direct requests and stale links would remain unsafe.

## Decision: Use focused manual verification until a test project exists

**Rationale**: The repository contains no automated test project. The quickstart will define repeatable scenarios covering authentication, IDOR protection, quarantine, partial batch success, sharing, deletion audit retention, and performance smoke checks.

**Alternatives considered**: Adding a full test framework as part of planning was rejected as out of scope for this feature plan; the implementation should add focused tests if the project establishes a test harness during delivery.
