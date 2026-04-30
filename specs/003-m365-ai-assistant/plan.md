# Implementation Plan: M365 AI Chapter Assistant

**Branch**: `003-m365-ai-assistant` | **Date**: 2026-03-18 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/003-m365-ai-assistant/spec.md`

## Summary

Build an AI assistant for TU chapter leaders that connects to their independent M365 Business Standard tenant to answer natural-language questions, summarize communications, look up financial data, find documents, check calendars, and perform basic write actions (create events, send emails) — all sourced from SharePoint, Exchange Online, Teams, and OneDrive via Microsoft Graph API. The assistant authenticates via Entra ID multi-tenant app registration, restricts access to a designated security group per tenant, and requires explicit user confirmation before any write action.

## Technical Context

**Language/Version**: C# / .NET 8 (backend), TypeScript / React (frontend)
**Primary Dependencies**: Microsoft Graph SDK for .NET, Microsoft Identity (MSAL) for multi-tenant auth, Azure OpenAI Service (or OpenAI API) for LLM, ASP.NET Core for web API
**Storage**: Azure Cosmos DB for conversation sessions and audit logs (TTL policies for 30-min session expiry and 90-day audit log retention); M365 data accessed in-place via Graph API (no data replication)
**Testing**: xUnit (backend), Jest + React Testing Library (frontend), Playwright (E2E)
**Target Platform**: Azure App Service (web), accessible via modern web browsers
**Project Type**: Web application (React SPA frontend + .NET API backend)
**Performance Goals**: <30s end-to-end response time (SC-001), support concurrent users across multiple chapter tenants
**Constraints**: Must operate within delegated M365 permissions (no app-only permissions that exceed user access); 30-minute session timeout; 90-day audit log retention with auto-purge; multi-tenant Entra ID registration
**Scale/Scope**: Hundreds of independent chapter tenants, each with a handful to dozens of active leaders; hundreds to low thousands of documents per chapter

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### I. Test-First Development (NON-NEGOTIABLE)
- **Status**: PASS
- Plan: Acceptance tests for each user story derived from spec scenarios. xUnit for backend services (Graph integration, auth, session management). Jest for frontend components. Playwright for E2E flows. TDD cycle enforced: tests written before implementation.

### II. API-First Design
- **Status**: PASS
- Plan: OpenAPI contracts defined for all backend endpoints before implementation. React frontend and .NET backend share contracts. Consistent response envelope for query results, errors, and citations. API versioning from day one.

### III. Security by Default
- **Status**: PASS
- Plan: Multi-tenant Entra ID authentication with delegated permissions (on behalf of user). Security group membership check on every request. All Graph API calls scoped to user's permissions. OWASP Top 10 addressed: input validation on queries, no SQL/injection vectors (Graph API + LLM prompt), CSRF/XSS protections on frontend. HTTPS enforced. Sensitive data (financial info, member data) never cached — read from M365 on demand. Write actions require explicit user confirmation (FR-019).

### IV. Simplicity & Pragmatism
- **Status**: PASS
- Plan: Delegated Graph API permissions (simplest auth model — no daemon/app permissions). Session state in lightweight store (not a full RDBMS). LLM integration via a single service abstraction. No speculative features — write actions limited to calendar events and emails per spec. Two-project structure (backend + frontend) only.

### V. Observability
- **Status**: PASS
- Plan: Correlation IDs on all API requests. Structured logging with Serilog. Audit log capturing query text, user identity, timestamp, and write action details (FR-016). Health check endpoint exposed. Timing metrics on Graph API calls and LLM inference.

**Gate Result**: ALL PASS — proceed to Phase 0.

## Project Structure

### Documentation (this feature)

```text
specs/003-m365-ai-assistant/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/           # Phase 1 output
└── tasks.md             # Phase 2 output (/speckit.tasks command)
```

### Source Code (repository root)

```text
backend/
├── src/
│   ├── Models/          # Domain entities (Conversation, Query, SourceReference, AuditEntry)
│   ├── Services/        # Business logic (GraphService, LlmService, SessionService, AuditService)
│   ├── Auth/            # Multi-tenant Entra ID auth, security group validation
│   └── Api/             # ASP.NET Core controllers, middleware, OpenAPI config
└── tests/
    ├── Unit/            # Service-level xUnit tests
    ├── Integration/     # Graph API + LLM integration tests
    └── Contract/        # API contract verification tests

frontend/
├── src/
│   ├── components/      # Chat UI, citation cards, action confirmation dialogs
│   ├── pages/           # Main chat page, auth callback, error pages
│   └── services/        # API client, auth (MSAL.js), session management
└── tests/               # Jest + RTL unit tests, Playwright E2E
```

**Structure Decision**: Web application with separate backend (.NET/C#) and frontend (React/TypeScript) projects, matching the constitution's technology stack. Backend handles all M365 integration and LLM orchestration; frontend provides the chat interface.

## Complexity Tracking

> No violations. All five constitution principles pass both pre-research and post-design checks.

## Post-Design Constitution Re-Check

*Re-evaluated after Phase 1 design artifacts (data-model.md, contracts/api.md, quickstart.md).*

| Principle | Status | Evidence |
|-----------|--------|----------|
| I. Test-First Development | PASS | Testing strategy: xUnit (backend), Jest+RTL (frontend), Playwright (E2E). Data model entities testable. API contracts enable contract tests. |
| II. API-First Design | PASS | Full API contracts documented in contracts/api.md. Consistent envelope. Versioned at /api/v1. |
| III. Security by Default | PASS | Multi-tenant Entra ID, security group validation per request, delegated permissions, write confirmation, input validation, audit logging. |
| IV. Simplicity & Pragmatism | PASS | Two projects only. Key-value session store. In-memory RAG. No speculative features. |
| V. Observability | PASS | Correlation IDs, health check endpoint, structured audit logging, timing metrics. |
