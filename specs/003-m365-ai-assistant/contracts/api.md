# API Contracts: M365 AI Chapter Assistant

**Date**: 2026-03-18 | **Branch**: `003-m365-ai-assistant`

> These contracts define the interface between the React frontend and the .NET backend API.
> OpenAPI/Swagger definitions will be generated from these contracts during implementation.

## Base URL

```
/api/v1
```

## Response Envelope

All responses use a consistent envelope structure per Constitution Principle II (API-First Design).

### Success Response

```json
{
  "data": { ... },
  "meta": {
    "correlationId": "uuid",
    "timestamp": "2026-03-18T14:30:00Z"
  }
}
```

### Error Response

```json
{
  "error": {
    "code": "string",
    "message": "string",
    "details": [ ... ]
  },
  "meta": {
    "correlationId": "uuid",
    "timestamp": "2026-03-18T14:30:00Z"
  }
}
```

### Pagination (where applicable)

```json
{
  "data": [ ... ],
  "pagination": {
    "total": 42,
    "offset": 0,
    "limit": 20,
    "hasMore": true
  },
  "meta": { ... }
}
```

---

## Authentication

All endpoints require a valid Bearer token obtained via MSAL.js (Authorization Code Flow with PKCE). The token is issued by the chapter's Entra ID tenant and validated by the backend.

```
Authorization: Bearer {access_token}
```

The backend validates:
1. Token signature and expiry
2. Audience matches the app registration client ID
3. Tenant ID matches a registered chapter
4. User is a member of the chapter's designated security group

Unauthorized requests return `401`. Users not in the security group return `403`.

---

## Endpoints

### 1. POST /api/v1/chat

Send a user query and receive an AI-generated response.

**Request**:

```json
{
  "conversationId": "uuid | null",
  "query": "What were the action items from our last board meeting?"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| conversationId | string (GUID) | No | Existing conversation ID for follow-ups. Null/omitted starts a new conversation. |
| query | string | Yes | Natural-language question or request. Max 2,000 characters. |

**Response** (200 OK):

```json
{
  "data": {
    "conversationId": "uuid",
    "response": "Based on the meeting minutes from March 5th [1], the action items were: ...",
    "citations": [
      {
        "index": 1,
        "sourceType": "Document",
        "title": "Board Meeting Minutes - March 2026.docx",
        "location": "Chapter Documents > Meeting Minutes",
        "webUrl": "https://chapter123.sharepoint.com/sites/docs/...",
        "snippet": "Action items: 1) Review conservation budget..."
      }
    ],
    "actionRequest": null,
    "sessionExpiresAt": "2026-03-18T15:00:00Z"
  },
  "meta": {
    "correlationId": "uuid",
    "timestamp": "2026-03-18T14:30:00Z"
  }
}
```

**Response with Action Request** (200 OK):

When the assistant proposes a write action, the response includes an `actionRequest`:

```json
{
  "data": {
    "conversationId": "uuid",
    "response": "I'll create a calendar event for the board meeting. Please review the details below and confirm.",
    "citations": [],
    "actionRequest": {
      "actionId": "uuid",
      "actionType": "CreateCalendarEvent",
      "status": "PendingConfirmation",
      "preview": {
        "subject": "Monthly Board Meeting",
        "startDateTime": "2026-04-15T19:00:00",
        "endDateTime": "2026-04-15T20:30:00",
        "timeZone": "America/Denver",
        "location": "Elk Creek Community Center",
        "body": "Monthly board meeting to discuss Q2 conservation projects.",
        "attendees": ["president@chapter123.org", "treasurer@chapter123.org"],
        "isOnlineMeeting": true
      }
    },
    "sessionExpiresAt": "2026-03-18T15:00:00Z"
  },
  "meta": { ... }
}
```

**Error Responses**:

| Code | Error Code | Description |
|------|-----------|-------------|
| 400 | `INVALID_QUERY` | Query is empty or exceeds 2,000 characters |
| 401 | `UNAUTHORIZED` | Missing or invalid Bearer token |
| 403 | `ACCESS_DENIED` | User is not a member of the chapter's security group |
| 404 | `SESSION_EXPIRED` | Provided conversationId has expired (30-min timeout) |
| 429 | `RATE_LIMITED` | Too many requests; includes `Retry-After` header |
| 502 | `UPSTREAM_ERROR` | M365 Graph API or LLM service unavailable |

---

### 2. POST /api/v1/chat/{conversationId}/confirm

Confirm or cancel a pending write action.

**Request**:

```json
{
  "actionId": "uuid",
  "decision": "confirm"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| actionId | string (GUID) | Yes | ID of the pending action from the chat response |
| decision | string | Yes | `"confirm"` or `"cancel"` |

**Response — Confirmed + Executed** (200 OK):

```json
{
  "data": {
    "actionId": "uuid",
    "status": "Executed",
    "result": {
      "message": "Calendar event 'Monthly Board Meeting' created successfully.",
      "webUrl": "https://outlook.office365.com/calendar/item/..."
    }
  },
  "meta": { ... }
}
```

**Response — Cancelled** (200 OK):

```json
{
  "data": {
    "actionId": "uuid",
    "status": "Cancelled",
    "result": {
      "message": "Action cancelled. No changes were made."
    }
  },
  "meta": { ... }
}
```

**Response — Failed** (200 OK):

```json
{
  "data": {
    "actionId": "uuid",
    "status": "Failed",
    "result": {
      "message": "Failed to create calendar event. The calendar service returned an error.",
      "errorDetail": "The mailbox is full or unavailable."
    }
  },
  "meta": { ... }
}
```

**Error Responses**:

| Code | Error Code | Description |
|------|-----------|-------------|
| 400 | `INVALID_DECISION` | Decision is not `"confirm"` or `"cancel"` |
| 401 | `UNAUTHORIZED` | Missing or invalid Bearer token |
| 404 | `ACTION_NOT_FOUND` | ActionId not found or session expired |
| 409 | `ACTION_ALREADY_RESOLVED` | Action was already confirmed, cancelled, or executed |

---

### 3. GET /api/v1/health

Health check endpoint for monitoring (Constitution Principle V — Observability).

**Response** (200 OK):

```json
{
  "data": {
    "status": "healthy",
    "checks": {
      "graphApi": "reachable",
      "llmService": "reachable",
      "sessionStore": "reachable"
    },
    "version": "1.0.0"
  },
  "meta": {
    "correlationId": "uuid",
    "timestamp": "2026-03-18T14:30:00Z"
  }
}
```

**Response** (503 Service Unavailable):

```json
{
  "data": {
    "status": "degraded",
    "checks": {
      "graphApi": "unreachable",
      "llmService": "reachable",
      "sessionStore": "reachable"
    },
    "version": "1.0.0"
  },
  "meta": { ... }
}
```

No authentication required.

---

### 4. GET /api/v1/admin/consent

Initiates the Entra ID admin consent flow for a new chapter's tenant. A chapter tenant admin navigates to this URL to grant the application the required Microsoft Graph permissions.

**No authentication required** (the admin will authenticate via Entra ID redirect).

**Response** (302 Found):

Redirects to `https://login.microsoftonline.com/common/adminconsent` with the application's `client_id`, `redirect_uri` (pointing to the callback below), and `state` (CSRF token).

---

### 5. GET /api/v1/admin/consent/callback

Handles the Entra ID admin consent callback. On success, creates a `Chapter` entity in `PendingConfiguration` status (consent granted but chapter details not yet provided). Redirects the admin to the registration form.

**Query Parameters** (set by Entra ID):

| Parameter | Type | Description |
|-----------|------|-------------|
| admin_consent | boolean | Whether consent was granted |
| tenant | string (UUID) | The consenting tenant's ID |
| state | string | CSRF token for validation |
| error | string? | Error code if consent was denied |
| error_description | string? | Human-readable error message |

**Response** (302 Found — consent granted):

Redirects to `/admin/register?tenantId={tenant}` (the registration form where the admin provides chapter details and security group).

**Error Responses**:

| Code | Error Code | Description |
|------|-----------|-------------|
| 400 | `INVALID_STATE` | CSRF state token mismatch |
| 400 | `CONSENT_DENIED` | Admin denied consent (`error` param present) |
| 409 | `CHAPTER_ALREADY_REGISTERED` | Tenant already has an active registration |

---

### 6. POST /api/v1/admin/register

Completes chapter onboarding after admin consent. The tenant admin provides the chapter name, chapter number, and the Object ID of the M365 security group that controls assistant access.

**Authentication**: Requires a valid Bearer token from the consenting tenant's admin.

**Request**:

```json
{
  "tenantId": "uuid",
  "chapterName": "Elk Creek Chapter",
  "chapterNumber": "123",
  "securityGroupId": "uuid"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| tenantId | string (GUID) | Yes | The tenant ID from the consent callback |
| chapterName | string | Yes | Display name of the chapter |
| chapterNumber | string | Yes | National TU chapter number |
| securityGroupId | string (GUID) | Yes | Object ID of the M365 security group controlling assistant access |

**Response** (201 Created):

```json
{
  "data": {
    "tenantId": "uuid",
    "chapterName": "Elk Creek Chapter",
    "chapterNumber": "123",
    "securityGroupId": "uuid",
    "status": "Active",
    "registeredAt": "ISO 8601 datetime"
  },
  "meta": {
    "correlationId": "uuid",
    "timestamp": "ISO 8601 datetime"
  }
}
```

**Error Responses**:

| Code | Error Code | Description |
|------|-----------|-------------|
| 400 | `INVALID_REQUEST` | Missing or invalid fields |
| 401 | `UNAUTHORIZED` | Missing or invalid Bearer token |
| 403 | `NOT_TENANT_ADMIN` | Authenticated user is not an admin of the specified tenant |
| 404 | `CONSENT_NOT_FOUND` | No consent record found for this tenantId (admin must consent first) |
| 409 | `CHAPTER_ALREADY_REGISTERED` | Tenant already has an active registration |

---

## Data Types Reference

### Citation

```json
{
  "index": 1,
  "sourceType": "Document | Email | TeamsMessage | CalendarEvent",
  "title": "string",
  "location": "string",
  "webUrl": "string (URL)",
  "snippet": "string"
}
```

### ActionRequest

```json
{
  "actionId": "uuid",
  "actionType": "CreateCalendarEvent | SendEmail",
  "status": "PendingConfirmation | Confirmed | Cancelled | Executed | Failed",
  "preview": "CalendarEventPreview | EmailPreview"
}
```

### CalendarEventPreview

```json
{
  "subject": "string",
  "startDateTime": "ISO 8601 datetime",
  "endDateTime": "ISO 8601 datetime",
  "timeZone": "string (IANA)",
  "location": "string | null",
  "body": "string | null",
  "attendees": ["email@example.com"],
  "isOnlineMeeting": true
}
```

### EmailPreview

```json
{
  "subject": "string",
  "body": "string (HTML)",
  "toRecipients": ["email@example.com"],
  "ccRecipients": ["email@example.com"],
  "saveToSentItems": true
}
```
