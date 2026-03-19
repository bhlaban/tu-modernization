# API Contracts: AI Chapter Assistant

**Date**: 2026-03-18
**Feature**: [spec.md](spec.md)
**Base URL**: `/api/v1`
**Auth**: JWT Bearer token (all endpoints except `/auth/*`)

## Response Envelope

All API responses use a consistent envelope structure (Constitution Principle II):

```json
// Success
{
  "data": { ... },
  "meta": { "correlationId": "uuid", "timestamp": "ISO8601" }
}

// Error
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable message"
  },
  "meta": { "correlationId": "uuid", "timestamp": "ISO8601" }
}

// Paginated
{
  "data": [ ... ],
  "pagination": { "page": 1, "pageSize": 20, "totalCount": 42, "totalPages": 3 },
  "meta": { "correlationId": "uuid", "timestamp": "ISO8601" }
}
```

## Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| VALIDATION_ERROR | 400 | Input validation failed |
| UNAUTHORIZED | 401 | Missing or invalid auth token |
| FORBIDDEN | 403 | Insufficient role permissions |
| NOT_FOUND | 404 | Resource not found |
| CONFLICT | 409 | Resource already exists (e.g., duplicate email) |
| INPUT_TOO_LONG | 400 | Message exceeds 2,000 character limit |
| UNSUPPORTED_FORMAT | 400 | Document format not supported |
| SERVICE_UNAVAILABLE | 503 | AI service temporarily unavailable (FR-014) |
| INTERNAL_ERROR | 500 | Unexpected server error |

---

## Authentication Endpoints

### POST /auth/register

Register a new user account.

**Request**:
```json
{
  "email": "leader@example.com",
  "password": "SecureP@ss1",
  "name": "Jane Smith",
  "chapterName": "Rocky Mountain Chapter"
}
```

**Response** (201 Created):
```json
{
  "data": {
    "id": "uuid",
    "email": "leader@example.com",
    "name": "Jane Smith",
    "role": "ChapterLeader",
    "chapterName": "Rocky Mountain Chapter"
  },
  "meta": { "correlationId": "uuid", "timestamp": "2026-03-18T10:00:00Z" }
}
```

**Validation**:
- `email`: Required, valid email format, unique
- `password`: Required, min 8 chars, at least 1 uppercase, 1 lowercase, 1 digit
- `name`: Required, 1-200 chars
- `chapterName`: Optional, 0-200 chars

### POST /auth/login

Authenticate and receive a JWT token.

**Request**:
```json
{
  "email": "leader@example.com",
  "password": "SecureP@ss1"
}
```

**Response** (200 OK):
```json
{
  "data": {
    "token": "eyJhbGciOi...",
    "expiresAt": "2026-03-18T22:00:00Z",
    "user": {
      "id": "uuid",
      "email": "leader@example.com",
      "name": "Jane Smith",
      "role": "ChapterLeader",
      "chapterName": "Rocky Mountain Chapter"
    }
  },
  "meta": { "correlationId": "uuid", "timestamp": "2026-03-18T10:00:00Z" }
}
```

**Error** (401): Invalid email or password.

---

## Chat Endpoints

### POST /chat/conversations

Create a new conversation.

**Auth**: Required (any role)

**Request**: *(empty body)*

**Response** (201 Created):
```json
{
  "data": {
    "id": "uuid",
    "title": null,
    "status": "Active",
    "startedAt": "2026-03-18T10:00:00Z",
    "lastActivityAt": "2026-03-18T10:00:00Z"
  },
  "meta": { "correlationId": "uuid", "timestamp": "2026-03-18T10:00:00Z" }
}
```

### GET /chat/conversations

List the current user's conversations (most recent first).

**Auth**: Required (any role)
**Query params**: `page` (default 1), `pageSize` (default 20, max 100), `status` (optional: `Active`, `Archived`)

**Response** (200 OK):
```json
{
  "data": [
    {
      "id": "uuid",
      "title": "Stream cleanup planning",
      "status": "Active",
      "startedAt": "2026-03-18T10:00:00Z",
      "lastActivityAt": "2026-03-18T10:30:00Z",
      "messageCount": 4
    }
  ],
  "pagination": { "page": 1, "pageSize": 20, "totalCount": 5, "totalPages": 1 },
  "meta": { "correlationId": "uuid", "timestamp": "2026-03-18T10:00:00Z" }
}
```

### GET /chat/conversations/{conversationId}

Get a conversation with its messages.

**Auth**: Required (owner only — FR-010)
**Query params**: `page` (default 1), `pageSize` (default 50, max 200) — for messages

**Response** (200 OK):
```json
{
  "data": {
    "id": "uuid",
    "title": "Stream cleanup planning",
    "status": "Active",
    "startedAt": "2026-03-18T10:00:00Z",
    "lastActivityAt": "2026-03-18T10:30:00Z",
    "messages": [
      {
        "id": "uuid",
        "sender": "User",
        "content": "How do I plan a stream cleanup?",
        "timestamp": "2026-03-18T10:00:00Z",
        "sourceReferences": [],
        "feedback": null
      },
      {
        "id": "uuid",
        "sender": "Assistant",
        "content": "Here's a step-by-step guide for planning a stream cleanup...",
        "timestamp": "2026-03-18T10:00:03Z",
        "sourceReferences": [
          {
            "id": "uuid",
            "documentId": "uuid",
            "documentTitle": "Event Planning Guide 2026",
            "relevanceScore": 0.92
          }
        ],
        "feedback": { "rating": "Positive", "createdAt": "2026-03-18T10:01:00Z" }
      }
    ]
  },
  "pagination": { "page": 1, "pageSize": 50, "totalCount": 2, "totalPages": 1 },
  "meta": { "correlationId": "uuid", "timestamp": "2026-03-18T10:00:00Z" }
}
```

**Error** (404): Conversation not found or not owned by current user.

### POST /chat/conversations/{conversationId}/messages

Send a message and receive the assistant's response.

**Auth**: Required (owner only)

**Request**:
```json
{
  "content": "How do I plan a stream cleanup event?"
}
```

**Validation**:
- `content`: Required, 1-2,000 characters (FR-011)

**Response** (200 OK):
```json
{
  "data": {
    "userMessage": {
      "id": "uuid",
      "sender": "User",
      "content": "How do I plan a stream cleanup event?",
      "timestamp": "2026-03-18T10:05:00Z"
    },
    "assistantMessage": {
      "id": "uuid",
      "sender": "Assistant",
      "content": "Planning a stream cleanup involves several steps...",
      "timestamp": "2026-03-18T10:05:04Z",
      "sourceReferences": [
        {
          "id": "uuid",
          "documentId": "uuid",
          "documentTitle": "Event Planning Guide 2026",
          "relevanceScore": 0.92
        }
      ]
    }
  },
  "meta": { "correlationId": "uuid", "timestamp": "2026-03-18T10:05:04Z" }
}
```

**Error** (400 INPUT_TOO_LONG): Message exceeds 2,000 characters.
**Error** (503 SERVICE_UNAVAILABLE): AI service is temporarily unavailable (FR-014).

### POST /chat/messages/{messageId}/feedback

Submit thumbs-up/thumbs-down feedback on an assistant response.

**Auth**: Required (conversation owner only)

**Request**:
```json
{
  "rating": "Positive"
}
```

**Validation**:
- `rating`: Required, one of `Positive` or `Negative`
- Message must be an assistant message (Sender = Assistant)
- Only one feedback per message (overwrites previous)

**Response** (200 OK):
```json
{
  "data": {
    "id": "uuid",
    "messageId": "uuid",
    "rating": "Positive",
    "createdAt": "2026-03-18T10:06:00Z"
  },
  "meta": { "correlationId": "uuid", "timestamp": "2026-03-18T10:06:00Z" }
}
```

**Error** (400): Cannot give feedback on user messages.
**Error** (404): Message not found or not owned by current user.

---

## Knowledge Base Endpoints

### GET /admin/documents

List all knowledge base documents.

**Auth**: Required (Admin role only — FR-006)
**Query params**: `page` (default 1), `pageSize` (default 20), `status` (optional: `Pending`, `Processing`, `Completed`, `Failed`)

**Response** (200 OK):
```json
{
  "data": [
    {
      "id": "uuid",
      "title": "Event Planning Guide 2026",
      "fileName": "event-planning-guide-2026.pdf",
      "format": "PDF",
      "processingStatus": "Completed",
      "uploadedBy": { "id": "uuid", "name": "Admin User" },
      "uploadedAt": "2026-03-18T09:00:00Z",
      "chunkCount": 15
    }
  ],
  "pagination": { "page": 1, "pageSize": 20, "totalCount": 3, "totalPages": 1 },
  "meta": { "correlationId": "uuid", "timestamp": "2026-03-18T10:00:00Z" }
}
```

### POST /admin/documents

Upload a new document to the knowledge base.

**Auth**: Required (Admin role only)
**Content-Type**: `multipart/form-data`

**Request**:
- `file`: The document file (PDF, DOCX, MD, TXT)
- `title`: string (1-200 chars, required)

**Response** (202 Accepted):
```json
{
  "data": {
    "id": "uuid",
    "title": "Chapter Reporting Guide 2026",
    "fileName": "chapter-reporting-guide-2026.pdf",
    "format": "PDF",
    "processingStatus": "Pending",
    "uploadedAt": "2026-03-18T10:00:00Z"
  },
  "meta": { "correlationId": "uuid", "timestamp": "2026-03-18T10:00:00Z" }
}
```

**Error** (400 UNSUPPORTED_FORMAT): File format not supported. Supported: PDF, DOCX, MD, TXT.
**Error** (400 VALIDATION_ERROR): Title missing or exceeds 200 characters.

### PUT /admin/documents/{documentId}

Replace an existing document (re-uploads and re-processes).

**Auth**: Required (Admin role only)
**Content-Type**: `multipart/form-data`

**Request**: Same as POST.

**Response** (202 Accepted): Same shape as POST. ProcessingStatus resets to `Pending`.

### DELETE /admin/documents/{documentId}

Remove a document and all its chunks from the knowledge base.

**Auth**: Required (Admin role only)

**Response** (204 No Content)

**Error** (404): Document not found.

### GET /admin/documents/{documentId}

Get document details including processing status.

**Auth**: Required (Admin role only)

**Response** (200 OK):
```json
{
  "data": {
    "id": "uuid",
    "title": "Event Planning Guide 2026",
    "fileName": "event-planning-guide-2026.pdf",
    "format": "PDF",
    "processingStatus": "Completed",
    "uploadedBy": { "id": "uuid", "name": "Admin User" },
    "uploadedAt": "2026-03-18T09:00:00Z",
    "chunkCount": 15,
    "contentHash": "sha256:abcdef..."
  },
  "meta": { "correlationId": "uuid", "timestamp": "2026-03-18T10:00:00Z" }
}
```

---

## Health Endpoint

### GET /health

Health check for monitoring (Constitution Principle V).

**Auth**: None required

**Response** (200 OK):
```json
{
  "status": "Healthy",
  "checks": {
    "database": "Healthy",
    "aiService": "Healthy"
  },
  "timestamp": "2026-03-18T10:00:00Z"
}
```

**Response** (503 Service Unavailable):
```json
{
  "status": "Unhealthy",
  "checks": {
    "database": "Healthy",
    "aiService": "Unhealthy"
  },
  "timestamp": "2026-03-18T10:00:00Z"
}
```
