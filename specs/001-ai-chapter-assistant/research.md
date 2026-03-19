# Research: AI Chapter Assistant

**Date**: 2026-03-18
**Feature**: [spec.md](spec.md)

## 1. RAG Pipeline Implementation in .NET

**Decision**: Use Microsoft Semantic Kernel for LLM orchestration and RAG pipeline.

**Rationale**: Semantic Kernel provides built-in abstractions for multiple LLM providers, memory management, and plugin extensibility. It is actively maintained by Microsoft and integrates naturally with .NET 8 and Azure services. It avoids building custom orchestration plumbing while remaining flexible enough to swap providers.

**Alternatives considered**:
- **Direct OpenAI SDK**: More control but requires building orchestration, memory, and retrieval logic manually. Violates Simplicity principle for a standard RAG use case.
- **LangChain.NET**: More flexible for complex agent workflows, but heavier than needed for grounded Q&A.

**Architecture pattern**: Service layer with clear boundaries:
- `DocumentService` — ingestion, chunking
- `EmbeddingService` — abstracts embedding provider
- `VectorStoreService` — abstracts vector database
- `RetrievalService` — RAG query logic (retrieve → augment → generate)
- `ChatService` — conversation management, context assembly

## 2. Vector Database

**Decision**: PostgreSQL + pgvector extension.

**Rationale**: The project already uses PostgreSQL for relational data (users, conversations, messages). Adding the pgvector extension avoids introducing a second data store, reducing operational complexity. At ~100 documents and ~500 users, pgvector handles the scale comfortably. EF Core integration is straightforward via `Npgsql.EntityFrameworkCore.PostgreSQL` with vector column types. Embedding dimension: 1536 (OpenAI text-embedding-3-small).

**Alternatives considered**:
- **Azure AI Search**: Excellent hybrid search but costs ~$50-100/mo vs ~$10-20/mo for pgvector; overkill for 100 documents.
- **Qdrant**: Good for vector-only workloads but adds operational infrastructure for a small-scale project.
- **Pinecone**: Managed service reduces ops burden but introduces vendor lock-in and higher cost.

## 3. Document Processing Pipeline

**Decision**: Modular pipeline with specialized open-source parsers per format.

**Rationale**: Each document format has a mature, free .NET library. A common `IDocumentParser` interface abstracts format differences. Fixed-size chunking with overlap (512 tokens, 128 overlap) is the starting strategy — simple, predictable, and sufficient for organizational documents.

**Libraries selected**:
| Format | Package | License |
|--------|---------|---------|
| PDF | PdfSharp | MIT |
| DOCX | DocumentFormat.OpenXml | MIT |
| Markdown | Markdig | BSD |
| Plain text | System.IO (built-in) | — |

**Chunking strategy**: Fixed-size with overlap (512 tokens, 128 token overlap). This is the simplest approach that prevents concept fragmentation. Can be upgraded to semantic chunking later if retrieval quality is insufficient.

**Alternatives considered**:
- **Aspose**: Commercial ($999/dev/yr); higher quality PDF parsing but unnecessary expense for organizational documents.
- **Semantic chunking**: Better retrieval quality but significantly more complex to implement. Deferred per Simplicity principle.

## 4. LLM Provider

**Decision**: Azure OpenAI with Semantic Kernel abstraction for provider swapability.

**Rationale**: Azure OpenAI offers enterprise compliance (SOC2), AAD integration, and predictable pricing with Azure commitment discounts. Semantic Kernel's `IChatCompletionService` abstraction allows switching to OpenAI API or other providers without code changes. For ~100 queries/day, estimated cost is ~$35-175/mo depending on model (GPT-3.5 vs GPT-4).

**Cost estimate** (100 queries/day = ~3,000/month):
- GPT-3.5 Turbo: ~$35/mo
- GPT-4: ~$175/mo
- Recommendation: Start with GPT-3.5 Turbo for development; evaluate GPT-4 for production based on answer quality.

**Alternatives considered**:
- **OpenAI API direct**: Similar capability but higher per-token costs; no Azure compliance benefits.
- **Open-source models (Llama, Mistral)**: Zero token cost but requires self-hosted GPU infrastructure; operational burden violates Simplicity principle for a small nonprofit project.

## 5. Authentication

**Decision**: ASP.NET Core Identity with JWT bearer tokens.

**Rationale**: ASP.NET Core Identity is the built-in authentication framework for .NET. It handles email/password registration, login, password hashing (PBKDF2), and role-based authorization out of the box. JWT tokens are stateless, cache-friendly, and work well with React SPAs. No additional infrastructure required.

**Alternatives considered**:
- **IdentityServer / Duende**: Full OAuth2/OIDC server; overkill for email/password auth with two roles.
- **Auth0 / Okta**: Managed auth services; adds external dependency and cost for a simple auth model.

## 6. Frontend State Management

**Decision**: React Context + useReducer for auth state; local component state + React Query for server data.

**Rationale**: The assistant UI has minimal client-side state (current conversation, auth token). React Query handles server state (conversation history, messages) with built-in caching and refetching. No need for Redux or Zustand given the narrow scope.

**Alternatives considered**:
- **Redux**: Too much boilerplate for a chat UI with 4 pages.
- **Zustand**: Lightweight but unnecessary when React Context + React Query cover all needs.
