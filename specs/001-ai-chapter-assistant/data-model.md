# Data Model: AI Chapter Assistant

**Date**: 2026-03-18
**Feature**: [spec.md](spec.md)
**Storage**: PostgreSQL + pgvector

## Entity Relationship Diagram

```
┌──────────────┐       ┌──────────────────┐
│     User     │       │  KBDocument      │
│──────────────│       │──────────────────│
│ Id (PK)      │       │ Id (PK)          │
│ Email        │       │ Title            │
│ PasswordHash │       │ FileName         │
│ Name         │       │ Format           │
│ Role         │       │ UploadedById(FK) │
│ ChapterName  │       │ UploadedAt       │
│ CreatedAt    │       │ ProcessingStatus │
│ UpdatedAt    │       │ ContentHash      │
└──────┬───────┘       │ CreatedAt        │
       │               │ UpdatedAt        │
       │ 1:N           └────────┬─────────┘
       │                        │ 1:N
       ▼                        ▼
┌──────────────────┐   ┌──────────────────┐
│  Conversation    │   │  DocumentChunk   │
│──────────────────│   │──────────────────│
│ Id (PK)          │   │ Id (PK)          │
│ UserId (FK)      │   │ DocumentId (FK)  │
│ Title            │   │ Content          │
│ StartedAt        │   │ ChunkIndex       │
│ LastActivityAt   │   │ Embedding(vec)   │
│ Status           │   │ TokenCount       │
│ CreatedAt        │   │ CreatedAt        │
└──────┬───────────┘   └──────────────────┘
       │ 1:N
       ▼
┌──────────────────┐
│    Message       │
│──────────────────│
│ Id (PK)          │
│ ConversationId   │
│   (FK)           │
│ Sender           │
│ Content          │
│ CorrelationId    │
│ Timestamp        │
│ CreatedAt        │
└──────┬───────────┘
       │ 1:N              1:N
       ├──────────────────────┐
       ▼                      ▼
┌──────────────────┐   ┌──────────────────┐
│ SourceReference  │   │ ResponseFeedback │
│──────────────────│   │──────────────────│
│ Id (PK)          │   │ Id (PK)          │
│ MessageId (FK)   │   │ MessageId (FK)   │
│ DocumentId (FK)  │   │ Rating           │
│ ChunkId (FK)     │   │ CreatedAt        │
│ DocumentTitle    │   └──────────────────┘
│ RelevanceScore   │
│ CreatedAt        │
└──────────────────┘
```

## Entities

### User

Represents an authenticated user of the system (chapter leader or administrator).

| Field | Type | Constraints | Notes |
|-------|------|-------------|-------|
| Id | GUID | PK | Generated server-side |
| Email | string(256) | UNIQUE, NOT NULL | Used for login |
| PasswordHash | string(512) | NOT NULL | Hashed via ASP.NET Core Identity (PBKDF2) |
| Name | string(200) | NOT NULL | Display name |
| Role | enum | NOT NULL | `ChapterLeader` or `Admin` |
| ChapterName | string(200) | NULLABLE | Chapter affiliation; null for admin-only users |
| CreatedAt | DateTimeOffset | NOT NULL | Auto-set on creation |
| UpdatedAt | DateTimeOffset | NOT NULL | Auto-set on update |

**Validation rules**:
- Email must be a valid email format
- Name must be 1-200 characters
- Role must be one of the defined enum values

### Conversation

A series of messages between a user and the assistant.

| Field | Type | Constraints | Notes |
|-------|------|-------------|-------|
| Id | GUID | PK | Generated server-side |
| UserId | GUID | FK → User.Id, NOT NULL | Owner of the conversation |
| Title | string(200) | NULLABLE | Auto-generated from first message; editable |
| StartedAt | DateTimeOffset | NOT NULL | When first message was sent |
| LastActivityAt | DateTimeOffset | NOT NULL | Updated on each new message |
| Status | enum | NOT NULL, DEFAULT Active | `Active` or `Archived` |
| CreatedAt | DateTimeOffset | NOT NULL | Auto-set on creation |

**Validation rules**:
- A conversation MUST belong to exactly one user (FR-010)
- Status transitions: Active → Archived (one-way)
- Conversations inactive >24 hours are auto-archived on next access

**State transitions**:
```
[New] → Active → Archived
```

### Message

A single exchange (user question or assistant response) within a conversation.

| Field | Type | Constraints | Notes |
|-------|------|-------------|-------|
| Id | GUID | PK | Generated server-side |
| ConversationId | GUID | FK → Conversation.Id, NOT NULL | Parent conversation |
| Sender | enum | NOT NULL | `User` or `Assistant` |
| Content | text | NOT NULL | Message text; max 2,000 chars for user messages |
| CorrelationId | string(64) | NOT NULL | For observability (FR-012) |
| Timestamp | DateTimeOffset | NOT NULL | When message was sent/received |
| CreatedAt | DateTimeOffset | NOT NULL | Auto-set on creation |

**Validation rules**:
- User messages: max 2,000 characters (FR-011)
- Assistant messages: no length limit
- Content must not be empty or whitespace-only
- CorrelationId is set automatically by middleware

### SourceReference

Links an assistant message to the knowledge base document chunk(s) used to generate the answer.

| Field | Type | Constraints | Notes |
|-------|------|-------------|-------|
| Id | GUID | PK | Generated server-side |
| MessageId | GUID | FK → Message.Id, NOT NULL | The assistant message |
| DocumentId | GUID | FK → KBDocument.Id, NOT NULL | Source document |
| ChunkId | GUID | FK → DocumentChunk.Id, NULLABLE | Specific chunk (null if doc-level) |
| DocumentTitle | string(200) | NOT NULL | Denormalized for display (FR-013) |
| RelevanceScore | float | NOT NULL | Cosine similarity score from RAG retrieval |
| CreatedAt | DateTimeOffset | NOT NULL | Auto-set on creation |

**Validation rules**:
- RelevanceScore must be between 0.0 and 1.0
- Only attached to messages where Sender = Assistant

### ResponseFeedback

Binary quality rating on an assistant response.

| Field | Type | Constraints | Notes |
|-------|------|-------------|-------|
| Id | GUID | PK | Generated server-side |
| MessageId | GUID | FK → Message.Id, UNIQUE, NOT NULL | One feedback per message |
| Rating | enum | NOT NULL | `Positive` or `Negative` |
| CreatedAt | DateTimeOffset | NOT NULL | Auto-set on creation |

**Validation rules**:
- Only one feedback record per message (UNIQUE constraint on MessageId)
- Only allowed on messages where Sender = Assistant
- Only the conversation owner can submit feedback

### KBDocument

An organizational document uploaded to the knowledge base.

| Field | Type | Constraints | Notes |
|-------|------|-------------|-------|
| Id | GUID | PK | Generated server-side |
| Title | string(200) | NOT NULL | Display title |
| FileName | string(500) | NOT NULL | Original file name |
| Format | enum | NOT NULL | `PDF`, `DOCX`, `Markdown`, `PlainText` |
| UploadedById | GUID | FK → User.Id, NOT NULL | Admin who uploaded |
| UploadedAt | DateTimeOffset | NOT NULL | Upload timestamp |
| ProcessingStatus | enum | NOT NULL, DEFAULT Pending | `Pending`, `Processing`, `Completed`, `Failed` |
| ContentHash | string(128) | NOT NULL | SHA-256 hash for deduplication and update detection |
| CreatedAt | DateTimeOffset | NOT NULL | Auto-set on creation |
| UpdatedAt | DateTimeOffset | NOT NULL | Auto-set on update |

**Validation rules**:
- Format must match file extension
- UploadedById must reference a user with Admin role (FR-006)
- ContentHash is recomputed on update to detect changes

**State transitions**:
```
[Upload] → Pending → Processing → Completed
                  └→ Failed
[Re-upload] → Pending (old chunks deleted, re-processed)
```

### DocumentChunk

A chunk of a knowledge base document, stored with its vector embedding for RAG retrieval.

| Field | Type | Constraints | Notes |
|-------|------|-------------|-------|
| Id | GUID | PK | Generated server-side |
| DocumentId | GUID | FK → KBDocument.Id, NOT NULL, CASCADE DELETE | Parent document |
| Content | text | NOT NULL | The text content of this chunk |
| ChunkIndex | int | NOT NULL | Order within the document (0-based) |
| Embedding | vector(1536) | NOT NULL | pgvector embedding (text-embedding-3-small) |
| TokenCount | int | NOT NULL | Token count for context window budgeting |
| CreatedAt | DateTimeOffset | NOT NULL | Auto-set on creation |

**Validation rules**:
- ChunkIndex must be unique per document
- Cascade delete: when a KBDocument is deleted, all its chunks are deleted
- Embedding dimension: 1536 (OpenAI text-embedding-3-small)

## Indexes

| Table | Index | Type | Purpose |
|-------|-------|------|---------|
| User | IX_User_Email | UNIQUE | Login lookup |
| Conversation | IX_Conversation_UserId_LastActivityAt | COMPOSITE | User's conversation list, sorted by recency |
| Message | IX_Message_ConversationId_Timestamp | COMPOSITE | Message ordering within conversation |
| DocumentChunk | IX_DocumentChunk_Embedding | IVFFLAT (pgvector) | Vector similarity search |
| DocumentChunk | IX_DocumentChunk_DocumentId | BTREE | Chunk lookup by document |
| KBDocument | IX_KBDocument_ProcessingStatus | BTREE | Admin queue filtering |
| ResponseFeedback | IX_ResponseFeedback_MessageId | UNIQUE | One feedback per message |

## Retention Policy

- Conversations and messages: 12 months from `LastActivityAt`, then eligible for deletion
- Knowledge base documents and chunks: retained until explicitly deleted by admin
- Response feedback: retained as long as the parent message exists
