# Research: M365 AI Chapter Assistant

**Date**: 2026-03-18 | **Branch**: `003-m365-ai-assistant`

## 1. Microsoft Graph API — Unified Search & Content Retrieval

### Decision
Use Microsoft Search API (`/search/query`) as the unified entry point for querying across M365 workloads (SharePoint, OneDrive, Exchange, Teams). Parallel queries by workload type where the API does not support cross-workload combinations in a single call.

### Rationale
- The `/search/query` endpoint supports multiple entity types (`driveItem`, `listItem`, `message`, `event`, `chatMessage`) but cannot combine Exchange entities (message/event) with SharePoint/Teams in a single request.
- Parallelizing per-workload queries minimizes latency while staying within API constraints.
- Reduces client-side orchestration complexity compared to calling individual resource endpoints directly.

### Alternatives Considered
- **Direct individual endpoint calls** (`/me/messages`, `/me/drive/items`, etc.) — simpler initially but requires more orchestration code and lacks relevance ranking.
- **Azure Cognitive Search with Graph indexer** — powerful but overkill for chapter-scale data (hundreds to low thousands of documents); adds operational complexity.

### Key Details
- **Delegated permission scopes** (all delegated, no app-only):
  - `Files.Read.All` — SharePoint and OneDrive document search
  - `Sites.Read.All` — SharePoint site-level search
  - `Mail.Read` — Exchange email search
  - `Calendars.Read` (read) / `Calendars.ReadWrite` (write) — Calendar events
  - `Chat.Read` — Teams messages
  - `Mail.Send` — Send emails
  - `Group.Read.All` — Security group membership check (requires admin consent)
- **Rate limits**: ~4 search queries/sec/user; 1,000 requests/60s per user-app combination. Mitigate with parallel batching, exponential backoff on 429 responses, and short-lived session-level result caching (5–15 minutes).
- **Content extraction**: Graph API provides metadata and discovery; document content (Word, Excel, PowerPoint, PDF) accessed via `/drive/items/{id}/content` download endpoint, then parsed locally with libraries (e.g., iTextSharp for PDF, Open XML SDK for Office formats).

---

## 2. Multi-Tenant Entra ID Authentication

### Decision
Register a single multi-tenant app in the platform's Entra ID tenant (`signInAudience: AzureADMultipleOrgs`). Each chapter's tenant admin grants admin consent to create a service principal in their tenant.

### Rationale
- One app registration scales to hundreds of chapter tenants without code or configuration changes.
- Each tenant independently controls its service principal and consent.
- Admin consent model ensures enterprise compliance and eliminates per-user consent dialogs.

### Alternatives Considered
- **Per-tenant app registration** — one app per chapter. Rejected due to massive operational overhead for onboarding and maintenance.
- **Single-tenant with custom multi-tenancy layer** — rejected; harder to audit and comply with M365 policies.

### Key Details
- **Auth flow**: Authorization Code Flow with PKCE (frontend SPA initiates) + On-Behalf-Of flow (backend exchanges user token for Graph API token).
- **Admin consent URL**: `https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/authorize?prompt=admin_consent&client_id={app-id}&...`
- **Security group check**: On every request, backend calls `GET /me/memberOf?$filter=id eq '{group-id}'` to verify user is in the designated "Chapter Leaders" security group. Cache result for 5 minutes per session to reduce Graph calls.
- **Token management**: Backend stores refresh tokens securely (encrypted); mints fresh access tokens for Graph API calls. User identity preserved in M365 audit logs.

---

## 3. Write Actions via Microsoft Graph

### Decision
Support two write actions: create calendar events (`POST /me/events`) and send emails (`POST /me/sendMail`). Both use delegated permissions executing under the authenticated user's identity.

### Rationale
- `Calendars.ReadWrite` replaces `Calendars.Read` (superset) — single scope upgrade enables both read and write for calendar.
- `Mail.Send` is a separate, granular scope — can send without granting read access (least privilege).
- Both operations are well-documented, single-endpoint Graph calls with predictable request/response structures.
- All write actions require explicit user confirmation before execution (FR-019).

### Alternatives Considered
- **Draft-then-send pattern** (create draft via `Mail.ReadWrite`, then send) — more steps with no functional benefit for this use case.
- **App-only permissions for writes** — rejected per spec requirement that all actions execute under user identity.

### Key Details
- **Create event**: `POST /me/events` with `Calendars.ReadWrite` scope. Returns `201 Created` with event ID and web link.
- **Send email**: `POST /me/sendMail` with `Mail.Send` scope. Returns `202 Accepted` (async delivery).
- **Confirmation flow**: Assistant presents a preview (event details or email draft) → user confirms or cancels → only then does backend execute the Graph API call.

---

## 4. LLM Integration — RAG Architecture

### Decision
Use Azure OpenAI Service with Semantic Kernel (.NET) for LLM orchestration. Implement in-memory RAG (Retrieval-Augmented Generation) per conversation session — no persistent vector store.

### Rationale
- **Azure OpenAI** provides enterprise-grade compliance (SOC 2, data not used for training, Azure Monitor integration for audit logging per FR-016).
- **Semantic Kernel** aligns with the .NET/C# tech stack, provides built-in plugin abstraction for Graph API connectors, and is mature for production use.
- **In-memory RAG** avoids data residency concerns (no persistent copy of M365 data). Search results are embedded and ranked within the session, then discarded on session expiry.
- RAG pattern eliminates hallucination risk by grounding LLM responses in retrieved M365 content (per SC-007: 95% accuracy on "no data" scenarios).

### Alternatives Considered
- **Direct OpenAI API** — acceptable for MVP/testing but lacks enterprise compliance guarantees and Azure Monitor integration.
- **Persistent vector database** (Pinecone, Weaviate, Azure AI Search) — overkill for chapter-scale data; adds data residency complexity.
- **Microsoft 365 Copilot SDK** — newer platform; Semantic Kernel is more mature. Can revisit if TU adopts Copilot infrastructure.
- **Self-hosted open-source LLM** (Llama 2, etc.) — rejected for MVP due to operational overhead of hosting/scaling.

### Key Details
- **RAG pipeline**: (1) Graph API search → (2) content download/extraction → (3) chunk + embed → (4) vector similarity ranking → (5) prompt assembly with `[Source X]` markers → (6) LLM generation → (7) citation post-processing to hydrate source URLs.
- **Citation approach**: Inline `[Source X]` markers in the LLM prompt; post-process output to replace markers with clickable links to original M365 content. Satisfies FR-007 (citations) and FR-008 (direct links).
- **Conversation context**: Maintain chat history within the 30-minute session for follow-up questions (FR-012). Truncate older exchanges when approaching token limits.

---

## 5. Session & Audit Storage

### Decision
Use Azure Cosmos DB (or Azure Table Storage for simpler MVP) for conversation session state and audit logs.

### Rationale
- Conversation sessions need key-value storage with TTL support (30-minute expiry).
- Audit logs need append-only writes with 90-day retention and automatic purge.
- Cosmos DB supports both with TTL policies natively; Azure Table Storage is a simpler/cheaper alternative with programmatic purge.
- Neither requires a full relational database — aligns with Simplicity & Pragmatism principle.

### Alternatives Considered
- **SQL Server / PostgreSQL** — heavier than needed for key-value session data and append-only audit logs.
- **Redis** — good for sessions but not ideal for 90-day audit retention.
- **In-memory only** — sessions lost on app restart; audit logs lost entirely.

### Key Details
- **Session document**: `{ tenantId, userId, conversationId, messages[], createdAt, lastActivityAt }` with 30-minute TTL.
- **Audit log document**: `{ tenantId, userId, queryText, actionType?, targetResource?, timestamp }` with 90-day TTL.
- **Partition key**: `tenantId` for tenant isolation in multi-tenant scenarios.

---

## Permission Scope Summary

| Operation | Scope | Admin Consent Required |
|-----------|-------|----------------------|
| Search documents (SharePoint/OneDrive) | `Files.Read.All` | No |
| Search SharePoint sites | `Sites.Read.All` | No |
| Search Teams messages | `Chat.Read` | No |
| Search emails | `Mail.Read` | No |
| Read calendar events | `Calendars.ReadWrite`* | No |
| Create calendar events | `Calendars.ReadWrite` | No |
| Send emails | `Mail.Send` | No |
| Check security group membership | `Group.Read.All` | Yes |

*Using `Calendars.ReadWrite` instead of `Calendars.Read` since write capability is required.
