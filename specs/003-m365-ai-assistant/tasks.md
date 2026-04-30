# Tasks: M365 AI Chapter Assistant

**Input**: Design documents from `/specs/003-m365-ai-assistant/`
**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: Included per Constitution Principle I (Test-First Development — NON-NEGOTIABLE). Tests are written before implementation within each phase.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Web app**: `backend/src/`, `backend/tests/`, `frontend/src/`, `frontend/tests/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization, solution structure, and dependency configuration

- [ ] T001 Create .NET 8 solution and backend project structure per plan (`backend/src/`, `backend/tests/Unit/`, `backend/tests/Integration/`, `backend/tests/Contract/`)
- [ ] T002 Initialize React/TypeScript frontend project with Vite in `frontend/`
- [ ] T003 [P] Add backend NuGet dependencies: Microsoft.Graph, Microsoft.Identity.Web, Microsoft.SemanticKernel, Azure.AI.OpenAI, Serilog, Swashbuckle in `backend/src/Api/Api.csproj`
- [ ] T004 [P] Add frontend npm dependencies: @azure/msal-browser, @azure/msal-react, axios in `frontend/package.json`
- [ ] T005 [P] Configure backend appsettings structure with sections for AzureAd, AzureOpenAI, SessionStore, and Logging in `backend/src/Api/appsettings.json` and `backend/src/Api/appsettings.Development.json`
- [ ] T006 [P] Configure ESLint, Prettier for frontend and .editorconfig for backend

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

### Tests for Foundational Phase

- [ ] T007 [P] Write xUnit tests for multi-tenant Entra ID token validation (valid token, expired token, wrong audience, wrong tenant) in `backend/tests/Unit/Auth/TokenValidationTests.cs`
- [ ] T008 [P] Write xUnit tests for security group membership check (member, non-member, group not found) in `backend/tests/Unit/Auth/SecurityGroupAuthorizationTests.cs`
- [ ] T009 [P] Write xUnit tests for Conversation entity creation, TTL expiry, and Turn appending in `backend/tests/Unit/Models/ConversationTests.cs`
- [ ] T010 [P] Write xUnit tests for AuditEntry creation and validation (query logging, write action logging, no response content) in `backend/tests/Unit/Models/AuditEntryTests.cs`
- [ ] T011 [P] Write xUnit tests for SessionService (create session, retrieve session, expire after 30 min inactivity, update LastActivityAt) in `backend/tests/Unit/Services/SessionServiceTests.cs`
- [ ] T012 [P] Write xUnit tests for AuditService (log query, log write action, verify 90-day TTL) in `backend/tests/Unit/Services/AuditServiceTests.cs`
- [ ] T013 [P] Write contract tests for POST /api/v1/chat request/response envelope per contracts/api.md in `backend/tests/Contract/ChatEndpointContractTests.cs`
- [ ] T014 [P] Write contract tests for POST /api/v1/chat/{conversationId}/confirm request/response envelope in `backend/tests/Contract/ConfirmEndpointContractTests.cs`
- [ ] T015 [P] Write contract tests for GET /api/v1/health response structure in `backend/tests/Contract/HealthEndpointContractTests.cs`
- [ ] T016 [P] Write xUnit tests for ChapterRegistrationService — create chapter in PendingConfiguration on consent, complete registration with chapter details + SecurityGroupId, retrieve chapter by TenantId, deactivate chapter in `backend/tests/Unit/Services/ChapterRegistrationServiceTests.cs`
- [ ] T017 [P] Write contract tests for GET /api/v1/admin/consent (redirect), GET /api/v1/admin/consent/callback (redirect to registration form), and POST /api/v1/admin/register (complete onboarding) per contracts/api.md in `backend/tests/Contract/AdminConsentContractTests.cs`

### Implementation for Foundational Phase

- [ ] T018 [P] Create domain model classes: Conversation, Turn, SourceReference, ActionRequest, CalendarEventPreview, EmailPreview in `backend/src/Models/`
- [ ] T019 [P] Create domain model classes: Chapter (registration), AuditEntry in `backend/src/Models/`
- [ ] T020 [P] Create enum types: SourceType, ActionType, ActionStatus, ChapterStatus in `backend/src/Models/Enums.cs`
- [ ] T021 Implement multi-tenant Entra ID authentication middleware using Microsoft.Identity.Web in `backend/src/Auth/AuthConfiguration.cs`
- [ ] T022 Implement security group membership authorization handler — check user membership in chapter's designated security group via Graph API with 5-min session cache in `backend/src/Auth/SecurityGroupAuthorizationHandler.cs`
- [ ] T023 Implement SessionService — create, retrieve, update, and expire conversation sessions in Cosmos DB with 30-min TTL in `backend/src/Services/SessionService.cs`
- [ ] T024 Implement AuditService — log queries and write actions to Cosmos DB with 90-day TTL in `backend/src/Services/AuditService.cs`
- [ ] T025 Configure Cosmos DB client and connection in `backend/src/Services/StorageConfiguration.cs`
- [ ] T026 Implement ChapterRegistrationService — create Chapter entity in PendingConfiguration on consent callback, complete registration (set chapterName, chapterNumber, securityGroupId, transition to Active) on POST /api/v1/admin/register, look up chapter by TenantId for authorization checks in `backend/src/Services/ChapterRegistrationService.cs`
- [ ] T027 Implement admin consent controller — GET /api/v1/admin/consent redirects to Entra ID admin consent URL, GET /api/v1/admin/consent/callback stores consent and redirects to registration form, POST /api/v1/admin/register accepts chapter details + SecurityGroupId and activates chapter in `backend/src/Api/Controllers/AdminConsentController.cs`
- [ ] T028 Implement API response envelope (data, error, meta with correlationId) and global exception handler middleware in `backend/src/Api/Middleware/ResponseEnvelopeMiddleware.cs`
- [ ] T029 Implement correlation ID middleware — generate and propagate correlation IDs across requests in `backend/src/Api/Middleware/CorrelationIdMiddleware.cs`
- [ ] T030 Configure Serilog structured logging with correlation ID enrichment in `backend/src/Api/Program.cs`
- [ ] T031 Implement health check endpoint (GET /api/v1/health) checking Graph API, LLM service, and session store reachability in `backend/src/Api/Controllers/HealthController.cs`
- [ ] T032 Configure ASP.NET Core pipeline: auth, CORS, Swagger/OpenAPI, middleware registration in `backend/src/Api/Program.cs`
- [ ] T033 [P] Implement MSAL.js authentication provider with multi-tenant config in `frontend/src/services/authService.ts`
- [ ] T034 [P] Implement API client service with Bearer token attachment and error handling per response envelope in `frontend/src/services/apiClient.ts`
- [ ] T035 Implement AuthCallback page handling Entra ID redirect in `frontend/src/pages/AuthCallback.tsx`
- [ ] T036 Implement protected route wrapper that checks authentication and displays sign-in prompt in `frontend/src/components/ProtectedRoute.tsx`

**Checkpoint**: Foundation ready — multi-tenant auth, session management, audit logging, API envelope, and frontend auth all operational. User story implementation can now begin.

---

## Phase 3: User Story 1 — Ask Questions About Chapter Documents (Priority: P1) 🎯 MVP

**Goal**: Chapter leaders can ask natural-language questions and get accurate, sourced answers from their M365 data (SharePoint, Exchange, Teams, OneDrive)

**Independent Test**: Ask factual questions whose answers exist in the chapter's M365 data and verify correct, cited responses

### Tests for User Story 1

> **Write these tests FIRST, ensure they FAIL before implementation**

- [ ] T037 [P] [US1] Write xUnit tests for GraphSearchService — search across SharePoint, Exchange, Teams, OneDrive with parallel workload queries; include FR-014 date-relative query tests ("last month", "past 30 days", "next week" resolved to correct date ranges); include test for item-level 403 (Graph returns permission denied on individual item — service gracefully skips item and continues) in `backend/tests/Unit/Services/GraphSearchServiceTests.cs`
- [ ] T038 [P] [US1] Write xUnit tests for ContentExtractionService — extract text from Word, Excel, PowerPoint, PDF via download + local parsing in `backend/tests/Unit/Services/ContentExtractionServiceTests.cs`
- [ ] T039 [P] [US1] Write xUnit tests for LlmOrchestratorService — RAG pipeline: build prompt with [Source X] markers, generate response, post-process citations; also test FR-013 clarifying-question behavior (ambiguous query triggers clarification instead of answer) in `backend/tests/Unit/Services/LlmOrchestratorServiceTests.cs`
- [ ] T040 [P] [US1] Write xUnit tests for CitationService — map [Source X] markers in LLM output to SourceReference objects with web URLs in `backend/tests/Unit/Services/CitationServiceTests.cs`
- [ ] T041 [P] [US1] Write xUnit tests for ChatController — POST /api/v1/chat with new conversation, follow-up query, expired session, invalid input, Graph API unavailability returning 502 UPSTREAM_ERROR envelope in `backend/tests/Unit/Api/ChatControllerTests.cs`
- [ ] T042 [P] [US1] Write integration test for full RAG pipeline: Graph search → content extraction → LLM prompt → cited response; include performance assertion that end-to-end response completes within 30 seconds (SC-001) in `backend/tests/Integration/RagPipelineIntegrationTests.cs`
- [ ] T043 [P] [US1] Write Jest tests for ChatPage component — render chat interface, submit query, display response with citations in `frontend/tests/ChatPage.test.tsx`
- [ ] T044 [P] [US1] Write Jest tests for CitationCard component — render source reference with title, snippet, link, source type icon in `frontend/tests/CitationCard.test.tsx`

### Implementation for User Story 1

- [ ] T045 [US1] Implement GraphSearchService — parallel queries to Microsoft Search API (`/search/query`) across driveItem, listItem, message, chatMessage entity types using delegated permissions; gracefully skip items returning 403 (FR-009); truncate results exceeding 20 items per workload and signal to LLM to suggest narrowing scope in `backend/src/Services/GraphSearchService.cs`
- [ ] T046 [US1] Implement ContentExtractionService — download document content via `/drive/items/{id}/content` and parse Word (Open XML SDK), Excel (Open XML SDK), PowerPoint (Open XML SDK), PDF (iTextSharp) in `backend/src/Services/ContentExtractionService.cs`
- [ ] T047 [US1] Implement LlmOrchestratorService using Semantic Kernel — build system prompt with grounding instructions, assemble retrieved chunks with [Source X] markers, call Azure OpenAI, handle conversation context for follow-ups, detect ambiguous queries and trigger clarification (FR-013) in `backend/src/Services/LlmOrchestratorService.cs`
- [ ] T048 [US1] Implement CitationService — post-process LLM response to replace [Source X] markers with SourceReference objects containing web URLs and metadata; handle stale/deleted content by catching 404 on URL validation and annotating citation as potentially unavailable in `backend/src/Services/CitationService.cs`
- [ ] T049 [US1] Implement ChatController with POST /api/v1/chat endpoint — validate input, check session, orchestrate search → extract → LLM → cite pipeline, log audit entry, return response envelope in `backend/src/Api/Controllers/ChatController.cs`
- [ ] T050 [US1] Implement ChatPage with chat message list, query input, and submit button in `frontend/src/pages/ChatPage.tsx`
- [ ] T051 [P] [US1] Implement CitationCard component displaying source type, title, snippet, and clickable link to M365 content in `frontend/src/components/CitationCard.tsx`
- [ ] T052 [P] [US1] Implement ChatMessage component rendering assistant responses with inline citation references and expandable citation cards in `frontend/src/components/ChatMessage.tsx`
- [ ] T053 [US1] Implement conversation state management — track conversationId, display session expiry indicator, handle session timeout gracefully in `frontend/src/services/sessionManager.ts`

**Checkpoint**: User Story 1 complete. Chapter leaders can sign in, ask questions about their M365 data, and receive cited answers. This is a fully functional MVP.

---

## Phase 4: User Story 2 — Summarize Chapter Communications (Priority: P2)

**Goal**: Chapter leaders can request summaries of email threads and Teams conversations by topic or time period

**Independent Test**: Request summaries on topics with known communications activity and verify accurate, chronological digests

### Tests for User Story 2

- [ ] T054 [P] [US2] Write xUnit tests for SummarizationService — topic-based search, date range filtering, chronological summary generation with participant attribution in `backend/tests/Unit/Services/SummarizationServiceTests.cs`
- [ ] T055 [P] [US2] Write Jest tests for SummaryView component — render chronological summary with participant names, key decisions, and source links in `frontend/tests/SummaryView.test.tsx`

### Implementation for User Story 2

- [ ] T056 [US2] Implement SummarizationService — compose Graph search queries filtered by date range, aggregate results from email + Teams, build summarization prompt with chronological ordering and participant attribution in `backend/src/Services/SummarizationService.cs`
- [ ] T057 [US2] Extend LlmOrchestratorService to detect summarization intent and delegate to SummarizationService with appropriate system prompt in `backend/src/Services/LlmOrchestratorService.cs`
- [ ] T058 [US2] Implement SummaryView component — render formatted summary with timeline, participant highlights, decision points, and source citations in `frontend/src/components/SummaryView.tsx`

**Checkpoint**: User Stories 1 and 2 both functional. Leaders can ask questions and request communication summaries.

---

## Phase 5: User Story 3 — Look Up Chapter Financial Information (Priority: P3)

**Goal**: Chapter leaders can ask about financial data in spreadsheets/documents and get accurate figures with source references

**Independent Test**: Ask financial questions whose answers exist in known budget spreadsheets and verify correct figures returned

### Tests for User Story 3

- [ ] T059 [P] [US3] Write xUnit tests for SpreadsheetExtractionService — parse Excel workbooks, extract cell values by sheet/range, handle formulas and formatted numbers in `backend/tests/Unit/Services/SpreadsheetExtractionServiceTests.cs`
- [ ] T060 [P] [US3] Write Jest tests for financial data display — render currency values, table excerpts, and source spreadsheet links in `frontend/tests/FinancialDataDisplay.test.tsx`

### Implementation for User Story 3

- [ ] T061 [US3] Implement SpreadsheetExtractionService — deep Excel parsing using Open XML SDK to extract structured data (named ranges, tables, specific cells), handle currency formatting and formulas in `backend/src/Services/SpreadsheetExtractionService.cs`
- [ ] T062 [US3] Extend LlmOrchestratorService to detect financial queries and invoke SpreadsheetExtractionService for targeted data extraction before LLM prompt assembly in `backend/src/Services/LlmOrchestratorService.cs`
- [ ] T063 [P] [US3] Implement FinancialDataDisplay component — render extracted financial figures with currency formatting, optional table view, and source spreadsheet link in `frontend/src/components/FinancialDataDisplay.tsx`

**Checkpoint**: User Stories 1–3 functional. Leaders can ask questions, summarize communications, and look up financial data.

---

## Phase 6: User Story 4 — Find and Retrieve Chapter Documents (Priority: P4)

**Goal**: Chapter leaders can ask the assistant to find documents and receive a list of matching files with direct links

**Independent Test**: Ask to find documents that exist in known SharePoint/OneDrive locations and verify correct documents returned with working links

### Tests for User Story 4

- [ ] T064 [P] [US4] Write xUnit tests for DocumentDiscoveryService — search by filename, content, file type; return ranked results with metadata and web URLs; disambiguate similar names in `backend/tests/Unit/Services/DocumentDiscoveryServiceTests.cs`
- [ ] T065 [P] [US4] Write Jest tests for DocumentList component — render document results with file type icons, paths, descriptions, and clickable links in `frontend/tests/DocumentList.test.tsx`

### Implementation for User Story 4

- [ ] T066 [US4] Implement DocumentDiscoveryService — targeted Graph search for driveItem/listItem entities, extract file metadata (name, path, modified date, size, type), generate web URLs, rank by relevance in `backend/src/Services/DocumentDiscoveryService.cs`
- [ ] T067 [US4] Extend LlmOrchestratorService to detect document search intent and delegate to DocumentDiscoveryService, format results as a document list rather than a narrative answer in `backend/src/Services/LlmOrchestratorService.cs`
- [ ] T068 [P] [US4] Implement DocumentList component — render search results as a list with file type icon, document name, folder path, last modified date, and clickable link to M365 in `frontend/src/components/DocumentList.tsx`

**Checkpoint**: User Stories 1–4 functional. Full read-only assistant capabilities operational.

---

## Phase 7: User Story 5 — Get Upcoming Events and Calendar Information (Priority: P5)

**Goal**: Chapter leaders can ask about upcoming events and meetings from shared calendars and get structured event listings

**Independent Test**: Ask about upcoming events that exist on the chapter's shared calendar and verify accurate event details returned

### Tests for User Story 5

- [ ] T069 [P] [US5] Write xUnit tests for CalendarService — query Exchange Online calendar events by date range, retrieve shared calendar events, extract event details (subject, time, location, attendees, attachments) in `backend/tests/Unit/Services/CalendarServiceTests.cs`
- [ ] T070 [P] [US5] Write Jest tests for EventList component — render upcoming events with date/time, location, attendees, and links to attached documents in `frontend/tests/EventList.test.tsx`

### Implementation for User Story 5

- [ ] T071 [US5] Implement CalendarService — query `/me/events` and `/me/calendars/{id}/events` via Graph API for upcoming events by date range, extract subject, start/end, location, attendees, and linked attachments in `backend/src/Services/CalendarService.cs`
- [ ] T072 [US5] Extend LlmOrchestratorService to detect calendar queries and delegate to CalendarService, format results as structured event listings in `backend/src/Services/LlmOrchestratorService.cs`
- [ ] T073 [P] [US5] Implement EventList component — render events with date/time formatting, location, attendee list, and links to attached agenda documents in `frontend/src/components/EventList.tsx`

**Checkpoint**: All five read-oriented user stories functional. Full question-answering, summarization, financial lookup, document discovery, and calendar query capabilities operational.

---

## Phase 8: User Story 6 — Perform Write Actions (Create Events, Send Emails) (Priority: P6)

**Goal**: Chapter leaders can request write actions (create calendar events, send emails) with a confirmation safeguard before execution

**Independent Test**: Request write actions, verify accurate confirmation preview, confirm execution under user's identity and permissions

### Tests for User Story 6

- [ ] T074 [P] [US6] Write xUnit tests for WriteActionService — create calendar event via Graph API, send email via Graph API, handle confirmation flow (pending → confirmed → executed, pending → cancelled, confirmed → failed) in `backend/tests/Unit/Services/WriteActionServiceTests.cs`
- [ ] T075 [P] [US6] Write xUnit tests for ConfirmController — POST /api/v1/chat/{conversationId}/confirm with confirm, cancel, already-resolved, not-found scenarios in `backend/tests/Unit/Api/ConfirmControllerTests.cs`
- [ ] T076 [P] [US6] Write Jest tests for ActionConfirmationDialog — render calendar event preview or email draft preview, confirm and cancel buttons, loading and error states in `frontend/tests/ActionConfirmationDialog.test.tsx`

### Implementation for User Story 6

- [ ] T077 [US6] Implement WriteActionService — create calendar events via `POST /me/events` (Calendars.ReadWrite) and send emails via `POST /me/sendMail` (Mail.Send) using delegated permissions, with ActionRequest state machine (PendingConfirmation → Confirmed → Executed | Failed | Cancelled) in `backend/src/Services/WriteActionService.cs`
- [ ] T078 [US6] Implement ConfirmController with POST /api/v1/chat/{conversationId}/confirm endpoint — validate actionId, apply decision (confirm/cancel), execute Graph write via WriteActionService, log audit entry with action type and target resource in `backend/src/Api/Controllers/ConfirmController.cs`
- [ ] T079 [US6] Extend LlmOrchestratorService to detect write action intent (create event, send email), generate ActionRequest with preview data (CalendarEventPreview or EmailPreview), and attach to response in `backend/src/Services/LlmOrchestratorService.cs`
- [ ] T080 [US6] Implement ActionConfirmationDialog — show structured preview of proposed action (event details or email draft), confirm/cancel buttons, loading state during execution, success/failure result display in `frontend/src/components/ActionConfirmationDialog.tsx`
- [ ] T081 [US6] Extend ChatMessage component to detect actionRequest in response and render ActionConfirmationDialog inline, handle confirm/cancel API calls in `frontend/src/components/ChatMessage.tsx`

**Checkpoint**: Full write action flow operational. Users can create calendar events and send emails with confirmation safeguard. All 6 user stories complete.

---

## Phase 9: Polish & Cross-Cutting Concerns

**Purpose**: Quality improvements, error handling hardening, and documentation

- [ ] T082 [P] Implement global error page and user-friendly error messages for 401, 403 (include "contact your chapter admin for access" guidance), 502 errors in `frontend/src/pages/ErrorPage.tsx`
- [ ] T083 [P] Implement rate limiting middleware with Retry-After header support for 429 responses in `backend/src/Api/Middleware/RateLimitingMiddleware.cs`
- [ ] T084 [P] Add request input validation middleware — query length (max 2,000 chars), conversationId format validation in `backend/src/Api/Middleware/InputValidationMiddleware.cs`
- [ ] T085 [P] Add timing metrics for Graph API calls and LLM inference in `backend/src/Services/MetricsService.cs`
- [ ] T086 Write Playwright E2E test for full flow: sign in → ask question → receive cited answer → ask follow-up in `frontend/tests/e2e/chat-flow.spec.ts`
- [ ] T087 Write Playwright E2E test for write action flow: request event creation → review preview → confirm → verify result in `frontend/tests/e2e/write-action-flow.spec.ts`
- [ ] T088 [P] Add loading indicators and skeleton screens for chat responses in `frontend/src/components/LoadingIndicator.tsx`
- [ ] T089 Run quickstart.md validation — verify setup steps, test commands, and sample flow work end-to-end
- [ ] T090 Generate OpenAPI/Swagger specification from implemented controllers in `backend/src/Api/`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion — BLOCKS all user stories
- **User Stories (Phases 3–8)**: All depend on Foundational phase completion
  - US1 (Phase 3) must complete first (other stories extend LlmOrchestratorService)
  - US2–US5 (Phases 4–7) can proceed sequentially after US1 or in parallel with care around LlmOrchestratorService extensions
  - US6 (Phase 8): Depends on Phase 2 (Foundational) and Phase 3 (US1 — ChatController, LlmOrchestratorService must exist)
- **Polish (Phase 9)**: Depends on all functional phases being complete

### User Story Dependencies

- **US1 (P1)**: Depends on Foundational (Phase 2) — establishes core RAG pipeline, ChatController, and frontend chat UI
- **US2 (P2)**: Depends on US1 — extends LlmOrchestratorService with summarization delegation
- **US3 (P3)**: Depends on US1 — extends LlmOrchestratorService with financial query delegation
- **US4 (P4)**: Depends on US1 — extends LlmOrchestratorService with document discovery delegation
- **US5 (P5)**: Depends on US1 — extends LlmOrchestratorService with calendar query delegation
- **US6 (P6)**: Depends on US1 — extends LlmOrchestratorService with write action detection + new controller

### Within Each User Story

- Tests MUST be written and FAIL before implementation (Constitution Principle I)
- Backend services before controllers/endpoints
- Controllers/endpoints before frontend components
- Core implementation before LlmOrchestratorService integration

### Parallel Opportunities

- **Phase 1**: T003, T004, T005, T006 can all run in parallel
- **Phase 2 Tests**: T007–T017 can all run in parallel (independent test files)
- **Phase 2 Impl**: T018, T019, T020 in parallel; T033, T034 in parallel
- **Phase 3 Tests**: T037–T044 can all run in parallel
- **Phase 3 Impl**: T051, T052 in parallel (independent frontend components)
- **Phases 4–7**: US2, US3, US4, US5 tests within each phase can run in parallel; frontend components across stories can be built in parallel
- **Phase 8 Tests**: T074, T075, T076 can all run in parallel
- **Phase 9**: T082, T083, T084, T085, T088 can all run in parallel

---

## Parallel Example: User Story 1

```
Step 1 (parallel):  T037, T038, T039, T040, T041, T042, T043, T044  ← Write all tests (all fail)
Step 2 (parallel):  T045, T046                                       ← GraphSearchService + ContentExtractionService
Step 3 (sequential): T047                                             ← LlmOrchestratorService (depends on T045, T046)
Step 4 (sequential): T048                                             ← CitationService
Step 5 (sequential): T049                                             ← ChatController (depends on T045–T048)
Step 6 (parallel):  T050, T051, T052                                  ← Frontend components
Step 7 (sequential): T053                                             ← Session manager (depends on T050)
```

---

## Implementation Strategy

### MVP Scope
- **Phase 1** (Setup) + **Phase 2** (Foundational) + **Phase 3** (US1: Document Q&A)
- Delivers a fully functional AI assistant that can answer questions about chapter M365 data with citations
- Independently testable and demonstrable to stakeholders

### Incremental Delivery
1. **MVP**: Phases 1–3 (Setup + Foundation + Document Q&A)
2. **Increment 2**: Phase 4 (Communication Summarization)
3. **Increment 3**: Phase 5 (Financial Lookups)
4. **Increment 4**: Phase 6 (Document Discovery)
5. **Increment 5**: Phase 7 (Calendar Queries)
6. **Increment 6**: Phase 8 (Write Actions)
7. **Final**: Phase 9 (Polish)
