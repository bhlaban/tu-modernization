<!--
  Sync Impact Report
  ==================
  Version change: 1.0.0 → 1.1.0 (MINOR — material expansion)
  Modified principles: None
  Added sections: None
  Removed sections: None
  Modified sections:
    - Technology Stack: added User Data Storage line
      (Microsoft 365 SharePoint, OneDrive, Azure Storage)
  Templates requiring updates:
    - plan-template.md ✅ no changes needed (Constitution Check is dynamic)
    - spec-template.md ✅ no changes needed (generic structure)
    - tasks-template.md ✅ no changes needed (generic structure)
    - checklist-template.md ✅ no changes needed (generic structure)
    - agent-file-template.md ✅ no changes needed (generic structure)
  Follow-up TODOs:
    - specs/001-ai-chapter-assistant/research.md may need
      updated storage decisions if M365 integration is in scope
      for the AI assistant feature.
-->

# TU Modernization Platform Constitution

## Core Principles

### I. Test-First Development (NON-NEGOTIABLE)

TDD is mandatory for all production code. The Red-Green-Refactor
cycle MUST be strictly followed:

- Tests MUST be written before implementation code.
- Tests MUST fail before implementation begins (Red).
- Implementation MUST target passing the failing tests (Green).
- Refactoring MUST NOT change external behavior (Refactor).
- Every user story MUST have acceptance tests derived from its
  acceptance scenarios.
- Integration tests MUST cover cross-boundary interactions
  (API ↔ frontend, service ↔ database).

**Rationale**: TDD produces higher confidence in correctness,
drives cleaner API design, and catches regressions immediately.

### II. API-First Design

All backend functionality MUST be exposed through well-defined
API contracts before frontend implementation begins:

- API contracts MUST be documented (OpenAPI/Swagger) before
  endpoint implementation.
- Frontend and backend teams MUST agree on contracts before
  parallel development proceeds.
- Breaking API changes MUST be versioned and communicated with
  a migration path.
- All API responses MUST use consistent envelope structures
  for data, errors, and pagination.

**Rationale**: A .NET backend serving a React frontend requires
stable, well-documented contracts to enable parallel development
and reduce integration friction.

### III. Security by Default

Security MUST be built into every feature from the start, not
added retroactively:

- Authentication and authorization MUST be enforced on every
  API endpoint (deny by default).
- User input MUST be validated and sanitized at system boundaries.
- Sensitive data (member info, financial records) MUST be
  encrypted at rest and in transit.
- Dependencies MUST be regularly audited for known
  vulnerabilities.
- OWASP Top 10 risks MUST be addressed in design reviews.

**Rationale**: Chapter leadership tools handle member data and
potentially financial information. Security failures erode trust
and may violate data protection obligations.

### IV. Simplicity & Pragmatism

Every design decision MUST favor the simplest solution that
meets current requirements:

- YAGNI: Features MUST NOT be built speculatively for future
  needs that are not yet confirmed.
- New abstractions MUST justify their existence against inline
  alternatives.
- Third-party dependencies MUST be evaluated for maintenance
  burden before adoption.
- Code MUST be readable by a contributor unfamiliar with the
  codebase within reasonable effort.

**Rationale**: As a tool for nonprofit chapter leadership,
long-term maintainability by a small team is more valuable
than architectural sophistication.

### V. Observability

Production systems MUST provide sufficient telemetry to
diagnose issues without reproducing them locally:

- All API requests MUST be logged with correlation IDs.
- Errors MUST produce structured log entries with sufficient
  context for diagnosis.
- Health check endpoints MUST be exposed for monitoring.
- Performance-critical paths MUST emit timing metrics.

**Rationale**: Chapter tools must be reliable. When issues
occur, operators need to diagnose and resolve them quickly
without disrupting chapter activities.

## Technology Stack

- **Backend**: .NET / C#
- **Frontend**: React (JavaScript/TypeScript)
- **User Data Storage**: Microsoft 365 (SharePoint, OneDrive)
  and/or Azure Storage for end-user documents, forms, and
  data files
- **Documentation**: Markdown
- **API Specification**: OpenAPI / Swagger
- **Testing**: xUnit (.NET), Jest/React Testing Library (frontend)
- **Source Control**: Git (GitHub)

All technology choices MUST be validated against the Simplicity
& Pragmatism principle before adoption. New frameworks or
libraries require justification in the relevant plan document.

## Development Workflow

- All changes MUST be made on feature branches and merged via
  pull request.
- Pull requests MUST pass all automated tests before merge.
- Pull requests MUST be reviewed by at least one other
  contributor.
- Commits MUST follow conventional commit format
  (e.g., `feat:`, `fix:`, `docs:`, `test:`).
- The `main` branch MUST always be in a deployable state.

## Governance

This constitution supersedes all other development practices
and conventions in the project. Compliance is mandatory.

- **Amendment process**: Proposed changes MUST be documented in
  a pull request with rationale. Changes require review and
  approval before merge.
- **Versioning**: The constitution follows semantic versioning
  (MAJOR.MINOR.PATCH). MAJOR for principle removals or
  redefinitions, MINOR for additions or material expansions,
  PATCH for clarifications and wording fixes.
- **Compliance review**: All pull requests and code reviews
  MUST verify adherence to these principles. Violations MUST
  be resolved before merge.
- **Complexity justification**: Any deviation from the
  Simplicity principle MUST be documented in the Complexity
  Tracking section of the relevant plan document.

**Version**: 1.1.0 | **Ratified**: 2026-03-18 | **Last Amended**: 2026-03-18
