# Data Model: M365 AI Chapter Assistant

**Date**: 2026-03-18 | **Branch**: `003-m365-ai-assistant`

## Entities

### Chapter (Registration)

Represents a TU chapter that has onboarded the assistant via admin consent.

| Field | Type | Description |
|-------|------|-------------|
| TenantId | string (GUID) | Entra ID tenant identifier for the chapter's M365 tenant |
| ChapterName | string? | Display name of the chapter (null until registration step) |
| ChapterNumber | string? | National TU chapter number (null until registration step) |
| SecurityGroupId | string (GUID)? | Object ID of the M365 security group controlling assistant access (null until registration step) |
| ConsentedAt | datetime | When the tenant admin granted admin consent |
| RegisteredAt | datetime? | When the admin completed registration (null if PendingConfiguration) |
| Status | enum | `PendingConfiguration`, `Active`, `Suspended`, `Revoked` |

**Relationships**: One Chapter → many Chapter Leaders (via M365 tenant membership + security group).

**Validation Rules**:
- TenantId must be a valid GUID and unique across registrations.
- SecurityGroupId must be a valid GUID referencing an existing group in the chapter's tenant (validated via Graph API on registration).
- Status defaults to `PendingConfiguration` on consent callback; transitions to `Active` when admin completes registration (POST /api/v1/admin/register).
- Chapter is not usable for authentication checks until Status is `Active`.

---

### Conversation

A chat session between a chapter leader and the assistant.

| Field | Type | Description |
|-------|------|-------------|
| ConversationId | string (GUID) | Unique session identifier |
| TenantId | string (GUID) | Chapter tenant for data isolation |
| UserId | string (GUID) | Entra ID object ID of the authenticated user |
| Turns | list | Ordered list of Turn objects (see below) |
| CreatedAt | datetime | When the conversation started |
| LastActivityAt | datetime | Timestamp of the most recent exchange |

**Relationships**: One Conversation → many Turns. Belongs to one Chapter (via TenantId).

**Validation Rules**:
- ConversationId generated server-side on first query.
- Session expires when `now - LastActivityAt > 30 minutes`.
- TTL: 30 minutes from LastActivityAt (auto-deleted from storage).

---

### Turn

A single user query + assistant response pair within a conversation. (Named "Turn" rather than "Exchange" to avoid confusion with Microsoft Exchange Online.)

| Field | Type | Description |
|-------|------|-------------|
| SequenceNumber | int | Order within the conversation (0-indexed) |
| UserQuery | string | The natural-language question or request |
| AssistantResponse | string | The assistant's generated response text |
| Citations | list | List of SourceReference objects included in the response |
| ActionRequest | ActionRequest? | Optional — populated if the assistant proposes a write action |
| Timestamp | datetime | When the exchange occurred |

**Validation Rules**:
- UserQuery must not be empty or exceed 2,000 characters.
- Citations list may be empty (e.g., for "no information found" responses).

---

### SourceReference

A citation pointing to the original M365 content from which the assistant derived information.

| Field | Type | Description |
|-------|------|-------------|
| SourceType | enum | `Document`, `Email`, `TeamsMessage`, `CalendarEvent` |
| Title | string | Document name, email subject, or message preview |
| Location | string | Site path, mailbox folder, channel name |
| WebUrl | string (URL) | Direct link to the source content in M365 |
| Snippet | string | Brief excerpt of the relevant content |
| RetrievedAt | datetime | When the content was fetched from Graph API |

**Validation Rules**:
- WebUrl must be a valid HTTPS URL pointing to a microsoft.com or sharepoint.com domain.
- SourceType must be one of the defined enum values.

---

### ActionRequest

A proposed write action that requires user confirmation before execution.

| Field | Type | Description |
|-------|------|-------------|
| ActionId | string (GUID) | Unique identifier for this action request |
| ActionType | enum | `CreateCalendarEvent`, `SendEmail` |
| Status | enum | `PendingConfirmation`, `Confirmed`, `Cancelled`, `Executed`, `Failed` |
| Preview | object | Structured preview of the action (event details or email draft) |
| ConfirmedAt | datetime? | When the user confirmed (null if pending/cancelled) |
| ExecutedAt | datetime? | When the Graph API call was made (null if not yet executed) |
| ErrorMessage | string? | Error details if execution failed |

**State Transitions**:
```
PendingConfirmation → Confirmed → Executed
PendingConfirmation → Cancelled
Confirmed → Failed
```

**Validation Rules**:
- Status starts as `PendingConfirmation` when assistant proposes an action.
- Transition to `Confirmed` only on explicit user approval.
- Transition to `Executed` only after successful Graph API call.
- `Cancelled` is a terminal state — no further transitions.

---

### CalendarEventPreview

Preview data for a proposed calendar event creation (embedded in ActionRequest.Preview).

| Field | Type | Description |
|-------|------|-------------|
| Subject | string | Event title |
| StartDateTime | datetime | Event start (with timezone) |
| EndDateTime | datetime | Event end (with timezone) |
| TimeZone | string | IANA or Windows timezone identifier |
| Location | string? | Physical or virtual location |
| Body | string? | Event description/notes |
| Attendees | list of string | Email addresses of attendees |
| IsOnlineMeeting | bool | Whether to create a Teams meeting link |

---

### EmailPreview

Preview data for a proposed email send (embedded in ActionRequest.Preview).

| Field | Type | Description |
|-------|------|-------------|
| Subject | string | Email subject line |
| Body | string | Email body (HTML or plain text) |
| ToRecipients | list of string | Primary recipient email addresses |
| CcRecipients | list of string | CC recipient email addresses |
| SaveToSentItems | bool | Whether to save in the user's Sent Items |

---

### AuditEntry

An immutable log record for every user interaction.

| Field | Type | Description |
|-------|------|-------------|
| EntryId | string (GUID) | Unique audit entry identifier |
| TenantId | string (GUID) | Chapter tenant for data isolation |
| UserId | string (GUID) | Entra ID object ID of the user |
| QueryText | string | Full text of the user's query |
| ActionType | string? | Write action type if applicable (e.g., `CreateCalendarEvent`, `SendEmail`) |
| TargetResource | string? | Target of the write action (e.g., calendar ID, recipient email) |
| Timestamp | datetime | When the interaction occurred |

**Validation Rules**:
- EntryId generated server-side.
- Append-only — entries are never modified or deleted manually.
- TTL: 90 days from Timestamp (auto-purged from storage).
- Response content is NOT logged (per FR-016).

---

## Entity Relationship Summary

```
Chapter (1) ──── (many) Conversation
                           │
                           ├── (many) Turn
                           │              │
                           │              ├── (many) SourceReference
                           │              └── (0..1) ActionRequest
                           │                           │
                           │                           └── (1) Preview
                           │                                (CalendarEventPreview | EmailPreview)
                           │
Chapter (1) ──── (many) AuditEntry
```

- **Chapter → Conversation**: Linked via TenantId. One chapter can have many concurrent conversations from different leaders.
- **Conversation → Turn**: One conversation contains an ordered sequence of turns.
- **Turn → SourceReference**: Each turn may cite zero or more sources.
- **Turn → ActionRequest**: At most one write action per turn. Requires confirmation cycle.
- **Chapter → AuditEntry**: One audit entry per user query, linked via TenantId for tenant-scoped log queries.
