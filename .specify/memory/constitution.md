<!--
Sync Impact Report
- Version change: unresolved scaffold -> 1.0.0
- Modified principles: five placeholder principles -> five project-specific principles
- Added sections: Security and Technology Constraints; Development Workflow and Quality Gates
- Removed sections: none
- Follow-up TODO: confirm the original ratification date
-->

# ContosoDashboard Constitution

## Core Principles

### I. Training Purpose and Scope
ContosoDashboard MUST remain a local, offline-first training application. New work MUST preserve
the documented training-only scope, avoid production claims, and keep external service dependencies
optional. Rationale: the project teaches Spec-Driven Development in a reproducible environment.

### II. Layered Design and Replaceable Infrastructure
Features MUST keep UI, business services, data models, and persistence concerns separated. Infrastructure
dependencies MUST use interfaces when a cloud or storage implementation may later replace the local
implementation. Business logic MUST NOT depend directly on SQLite or local filesystem details.
Rationale: students must be able to understand boundaries and migration paths without rewriting behavior.

### III. Authorization by Default
Protected pages MUST require authentication, and services MUST enforce authorization for every resource
lookup or mutation. Resource access MUST be scoped to the authenticated user and permitted role or
membership; URL identifiers MUST never be treated as authorization. New data paths MUST include an
IDOR-focused test or documented verification. Rationale: defense in depth is a central learning goal.

### IV. Verifiable Behavior
Every feature MUST define acceptance criteria before implementation. Changes to services, authorization,
data relationships, or shared contracts MUST include focused automated tests where the project supports
them; otherwise, the change MUST include a repeatable manual verification procedure. A change is not
complete while its stated acceptance criteria remain unverified.

### V. Simplicity and Maintainability
Implementations MUST use the simplest design that satisfies the approved requirements and existing
project conventions. New abstractions, dependencies, and configuration MUST have a documented reason.
Async database and service operations MUST remain non-blocking, and queries MUST avoid avoidable N+1
loading. Rationale: training code must expose useful patterns without hiding them behind needless complexity.

## Security and Technology Constraints

The application MUST target the repository's supported .NET and ASP.NET Core versions, use Blazor Server,
Entity Framework Core, and SQLite unless an approved feature changes that boundary. Authentication remains
mock authentication for training and MUST NOT be represented as production-ready. Security-sensitive
changes MUST preserve secure cookie settings, authorization attributes, service-level checks, and existing
security headers unless an explicit requirement replaces them. File features MUST generate unique storage
paths before persistence and MUST validate ownership before download, update, or deletion.

## Development Workflow and Quality Gates

Work MUST follow the Spec-Driven Development sequence: specify the behavior, plan the design, generate
actionable tasks, implement the smallest coherent change, and verify the result. Reviews MUST check
constitution compliance, acceptance criteria, authorization boundaries, data access behavior, and relevant
documentation. Build, test, and static-analysis failures introduced by a change MUST be resolved before
completion or explicitly recorded as a known limitation. Unrelated refactoring MUST remain out of scope.

## Governance

This constitution takes precedence over local conventions when they conflict. Amendments MUST state the
reason for the change, identify affected principles or sections, update the version and amendment date,
and include any migration or follow-up work needed by existing features. The version follows semantic
versioning: MAJOR for incompatible governance changes or removals, MINOR for new or materially expanded
principles, and PATCH for clarifications or non-semantic corrections.

Every feature review MUST verify the applicable principles and quality gates. The constitution MUST be
reviewed whenever the architecture, authentication model, persistence boundary, or development workflow
changes. Deliberate exceptions MUST be recorded in the relevant feature artifacts and approved before
implementation.

**Version**: 1.0.0 | **Ratified**: TODO(RATIFICATION_DATE): confirm original adoption date | **Last Amended**: 2026-09-16
