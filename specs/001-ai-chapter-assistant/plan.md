# Implementation Plan: AI Chapter Assistant

**Branch**: `001-ai-chapter-assistant` | **Date**: 2026-03-18 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/001-ai-chapter-assistant/spec.md`

## Summary

Build a RAG-powered AI assistant for Trout Unlimited chapter leaders. The assistant provides a conversational interface where leaders ask natural-language questions about running their chapters and receive answers grounded in TU organizational documents. The backend is a .NET/C# Web API with a document ingestion pipeline (chunking, embedding, vector search). The frontend is a standalone React SPA with a chat UI. Conversations, feedback, and knowledge base documents are persisted server-side. Authentication is email/password with role-based access (chapter-leader, admin).

## Technical Context

**Language/Version**: C# / .NET 8, TypeScript / React 18
**Primary Dependencies**: ASP.NET Core Web API, Entity Framework Core, Azure OpenAI (or OpenAI-compatible), a vector database (e.g., PostgreSQL + pgvector or Azure AI Search), React, Axios
**Storage**: PostgreSQL (relational data: users, conversations, messages, feedback, document metadata) + vector store (document embeddings for RAG retrieval)
**Testing**: xUnit + FluentAssertions (.NET), Jest + React Testing Library (frontend)
**Target Platform**: Linux server (containerized), modern browsers (Chrome, Edge, Firefox, Safari)
**Project Type**: Web application (API + SPA)
**Performance Goals**: <10s end-to-end response time (SC-001, 95th percentile); document ingestion <5 min (SC-005)
**Constraints**: 100 concurrent users (SC-006); 2,000 char input limit; 12-month conversation retention
**Scale/Scope**: ~400 TU chapters, initial user base ~500 chapter leaders, knowledge base ~100 documents

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Gate | Pre-Design | Post-Design |
|-----------|------|------------|-------------|
| I. Test-First (NON-NEGOTIABLE) | TDD workflow defined; acceptance tests mapped to user story scenarios | ✅ PASS | ✅ PASS — xUnit + Jest selected; test directories in project structure; 9 acceptance scenarios map to contract/integration tests |
| II. API-First | API contracts documented before implementation | ✅ PASS | ✅ PASS — contracts/api.md defines 12 endpoints with request/response schemas, error codes, and consistent envelope |
| III. Security by Default | Auth on all endpoints; input validation; encryption at rest/in transit | ✅ PASS | ✅ PASS — JWT auth on all endpoints (except /health); RBAC enforced (admin-only on /admin/*); password hashing via PBKDF2; input validation documented per endpoint |
| IV. Simplicity | No speculative features; justified dependencies | ✅ PASS | ✅ PASS — pgvector avoids second data store; Semantic Kernel is single orchestration dependency; fixed-size chunking over semantic chunking; GPT-3.5 default over GPT-4 |
| V. Observability | Correlation IDs, structured logging, health checks | ✅ PASS | ✅ PASS — correlationId in every response envelope and Message entity; GET /health endpoint with db + AI service checks |

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
backend/
├── src/
│   └── TUAssistant.Api/
│       ├── Controllers/        # API controllers (Chat, Auth, Documents, Health)
│       ├── Models/             # Domain entities and DTOs
│       ├── Services/           # Business logic (RAG pipeline, auth, document processing)
│       ├── Data/               # EF Core DbContext, migrations, repositories
│       ├── Middleware/         # Correlation ID, error handling, auth middleware
│       └── Program.cs
├── tests/
│   ├── TUAssistant.Api.Unit/
│   ├── TUAssistant.Api.Integration/
│   └── TUAssistant.Api.Contract/
└── TUAssistant.sln

frontend/
├── src/
│   ├── components/            # Chat UI, message bubbles, feedback controls, source citations
│   ├── pages/                 # ChatPage, LoginPage, HistoryPage, AdminPage
│   ├── services/              # API client, auth service
│   ├── hooks/                 # Custom React hooks
│   ├── types/                 # TypeScript interfaces
│   └── App.tsx
├── tests/
│   ├── components/
│   └── pages/
├── package.json
└── tsconfig.json
```

**Structure Decision**: Web application (Option 2) — .NET backend API + React frontend SPA. This aligns with the constitution's technology stack (.NET/C# backend, React frontend) and the spec's requirement for a standalone SPA (FR-015) with a server-side database (FR-009). The backend owns all data persistence, RAG pipeline, and auth; the frontend is a stateless SPA that communicates via the documented API contracts.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th project] | [current need] | [why 3 projects insufficient] |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |
