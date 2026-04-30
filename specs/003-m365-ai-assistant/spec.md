# Feature Specification: M365 AI Chapter Assistant

**Feature Branch**: `003-m365-ai-assistant`  
**Created**: 2026-03-18  
**Status**: Draft  
**Input**: User description: "Build an AI assistant for Trout Unlimited chapter leaders that leverages data stored in the products/services included in their Microsoft 365 Business Standard subscription"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Ask Questions About Chapter Documents (Priority: P1)

A chapter leader asks the AI assistant a natural-language question about information stored across their chapter's Microsoft 365 environment. The assistant searches relevant files, emails, and conversations to provide an accurate, sourced answer. For example, a chapter president preparing for a board meeting asks "What were the action items from our last board meeting?" and the assistant locates the meeting minutes in SharePoint, extracts the action items, and presents them with a link back to the source document.

**Why this priority**: This is the most foundational capability — retrieving and synthesizing information from scattered M365 data sources. Every other feature builds on top of this ability to find and surface relevant content. Chapter leaders currently waste significant time manually searching across email, SharePoint, and Teams for answers.

**Independent Test**: Can be fully tested by asking the assistant various factual questions whose answers exist in the chapter's M365 data (SharePoint documents, emails, Teams messages) and verifying the assistant returns correct, sourced answers.

**Acceptance Scenarios**:

1. **Given** a chapter leader is signed in and their chapter has meeting minutes stored in SharePoint, **When** they ask "What were the action items from our last board meeting?", **Then** the assistant returns the action items from the most recent meeting minutes with a reference to the source document.
2. **Given** a chapter leader asks a question whose answer spans multiple documents, **When** the assistant finds relevant content in both a SharePoint file and an email thread, **Then** it synthesizes the information and cites both sources.
3. **Given** a chapter leader asks a question for which no relevant information exists in their M365 data, **When** the assistant cannot find matching content, **Then** it clearly states that no relevant information was found rather than fabricating an answer.
4. **Given** a chapter leader asks a question, **When** the assistant retrieves content, **Then** every factual claim in the response includes a reference to the specific source (document name, email subject, or Teams message).

---

### User Story 2 - Summarize Chapter Communications (Priority: P2)

A chapter leader asks the assistant to summarize recent communications on a specific topic or across a time period. The assistant gathers relevant emails, Teams channel messages, and shared documents to produce a concise summary. For example, a newly elected chapter treasurer asks "Summarize all communications about the spring banquet fundraiser from the past month" and receives a digest of key decisions, open questions, and who said what.

**Why this priority**: Chapter leaders are volunteers with limited time. They often fall behind on email threads and Teams conversations. Summarization helps them stay informed without reading every message, which directly reduces volunteer burnout and improves decision-making.

**Independent Test**: Can be tested by requesting summaries of communications on topics that have known email and Teams activity, then verifying the summary captures key points accurately.

**Acceptance Scenarios**:

1. **Given** a chapter leader wants to catch up on a topic, **When** they ask "Summarize the discussion about our conservation project on Elk Creek," **Then** the assistant gathers related emails and Teams messages and returns a chronological summary highlighting key decisions, open questions, and participants.
2. **Given** a chapter leader asks for a time-bounded summary, **When** they specify "from the past two weeks," **Then** the assistant limits its search to content from that date range.
3. **Given** a chapter leader asks for a summary on a topic with no related communications, **When** the assistant searches and finds nothing, **Then** it reports that no communications were found for that topic and time period.

---

### User Story 3 - Look Up Chapter Financial Information (Priority: P3)

A chapter leader asks the assistant about financial data stored in spreadsheets or documents within the chapter's M365 environment. For example, a chapter president asks "What is our current checking account balance?" or "How much did we spend on conservation projects last year?" and the assistant locates the relevant budget spreadsheet or treasurer's report and extracts the answer.

**Why this priority**: Financial visibility is critical for chapter governance. Treasurers and presidents frequently need quick access to budget data for decision-making, grant applications, and reporting to the national organization. This data is typically locked in spreadsheets that require manual navigation.

**Independent Test**: Can be tested by asking financial questions whose answers exist in known budget spreadsheets or financial reports in SharePoint/OneDrive, and verifying the assistant returns the correct figures with source references.

**Acceptance Scenarios**:

1. **Given** a chapter has a budget spreadsheet in SharePoint, **When** a chapter leader asks "What was our total revenue last year?", **Then** the assistant locates the budget spreadsheet, extracts the total revenue figure, and returns it with a reference to the source file.
2. **Given** financial data is spread across multiple files (e.g., quarterly reports), **When** the leader asks a question that requires aggregation, **Then** the assistant identifies the relevant files and presents the combined information, noting which figures came from which sources.
3. **Given** a chapter leader asks about financial data that doesn't exist in their M365 environment, **When** no relevant financial documents are found, **Then** the assistant states it could not find the requested financial information.

---

### User Story 4 - Find and Retrieve Chapter Documents (Priority: P4)

A chapter leader asks the assistant to find a specific document or type of document. The assistant searches across SharePoint and OneDrive to locate matching files and provides direct links. For example, a volunteer coordinator asks "Find the liability waiver form we use for stream cleanup events" and the assistant locates and links to the relevant document.

**Why this priority**: Document discovery across SharePoint sites and OneDrive folders is a common pain point, especially for newer leaders unfamiliar with the chapter's folder structure. This capability reduces time spent navigating file hierarchies and ensures important documents are accessible.

**Independent Test**: Can be tested by asking the assistant to find documents that exist in known locations within the chapter's M365 environment, and verifying correct documents are returned with working links.

**Acceptance Scenarios**:

1. **Given** a chapter leader needs a specific document, **When** they ask "Find the chapter bylaws," **Then** the assistant locates the bylaws document in SharePoint or OneDrive and provides a direct link to it.
2. **Given** multiple documents match a request, **When** the leader asks "Find all documents related to our annual banquet," **Then** the assistant returns a list of matching documents with brief descriptions and links.
3. **Given** the assistant finds documents with similar names in different locations, **When** presenting results, **Then** it includes the folder path or site context to disambiguate.

---

### User Story 5 - Get Upcoming Events and Calendar Information (Priority: P5)

A chapter leader asks the assistant about upcoming chapter events, meetings, or deadlines from shared calendars. For example, a chapter president asks "What chapter events do we have in the next 30 days?" and the assistant retrieves and lists upcoming events from the chapter's shared calendar with dates, times, and locations.

**Why this priority**: Calendar awareness helps leaders plan and coordinate. While less complex than document search, it rounds out the assistant's ability to answer common operational questions and reduces context-switching between applications.

**Independent Test**: Can be tested by asking about upcoming events that exist on the chapter's shared calendar and verifying the assistant returns accurate event details.

**Acceptance Scenarios**:

1. **Given** a chapter has events on their shared calendar, **When** a leader asks "What meetings do we have this month?", **Then** the assistant returns a list of upcoming meetings with dates, times, and locations.
2. **Given** no events exist in the requested time period, **When** the leader asks about upcoming events, **Then** the assistant reports that no events were found for the specified period.
3. **Given** an event has attached agenda documents or notes, **When** the assistant lists the event, **Then** it mentions and links to any associated documents.

---

### User Story 6 - Perform Write Actions (Create Events, Send Emails) (Priority: P6)

A chapter leader asks the assistant to perform a write action — creating a calendar event or drafting and sending an email — on their behalf. The assistant generates a preview of the proposed action and requires explicit confirmation before executing it. For example, a chapter president says "Schedule a board meeting for next Tuesday at 7pm at the community center" and the assistant drafts a calendar event with the specified details, presents it for review, and creates it only after the leader confirms.

**Why this priority**: Once chapter leaders trust the assistant for information retrieval (US1–US5), enabling basic write actions eliminates additional context-switching into Outlook/Calendar. However, write actions carry higher risk than read-only queries, hence the confirmation requirement and lower priority.

**Independent Test**: Can be tested by requesting write actions (create event, send email), verifying the confirmation preview is accurate, and confirming that the action executes correctly under the user's identity and permissions.

**Acceptance Scenarios**:

1. **Given** a chapter leader asks the assistant to create a calendar event, **When** the assistant interprets the request, **Then** it presents a structured preview (subject, date/time, location, attendees) and waits for explicit confirmation before creating the event.
2. **Given** a chapter leader confirms a proposed calendar event, **When** the assistant executes the action, **Then** the event is created in the user's Exchange Online calendar via their delegated permissions, and the assistant confirms success with a link to the new event.
3. **Given** a chapter leader asks the assistant to send an email, **When** the assistant drafts the email, **Then** it presents a preview (recipients, subject, body) and waits for confirmation before sending.
4. **Given** a chapter leader reviews a proposed write action and decides not to proceed, **When** they cancel, **Then** no changes are made to their M365 environment and the assistant acknowledges the cancellation.
5. **Given** a chapter leader confirms a write action but the M365 service returns an error, **When** execution fails, **Then** the assistant informs the user of the failure with the error reason and suggests retrying or performing the action manually.
6. **Given** the assistant misinterprets a query as a write action (e.g., "draft an email" vs. "find the email I drafted"), **When** the confirmation preview is shown, **Then** the user can cancel without any write occurring, serving as a safeguard against misinterpretation.

---

### Edge Cases

- What happens when a chapter leader asks about data they don't have permission to access within M365? The assistant must respect existing M365 permissions and inform the user that the content is restricted rather than surfacing unauthorized data.
- How does the system handle ambiguous questions where multiple interpretations are possible? The assistant should ask a clarifying follow-up question rather than guessing.
- What happens when M365 services are temporarily unavailable? The assistant should inform the user that it cannot currently access the requested data and suggest trying again later.
- How does the system handle questions about data that has been deleted or moved? The assistant should indicate when referenced content may no longer be available at its last known location.
- What happens when documents contain sensitive information (e.g., member personal details, financial account numbers)? The assistant must only surface information to users who have permission to view the original source and must not cache or expose sensitive data beyond what was in the source.
- How does the assistant handle very large result sets (e.g., "Show me all emails from the past year")? The assistant should summarize results and offer to narrow the scope rather than attempting to dump all content.
- What happens when an authenticated M365 user who is not a member of the designated security group attempts to use the assistant? The assistant must deny access and display a message directing them to contact their chapter's tenant admin for access.
- What happens when the user confirms a write action (e.g., send email) but the action fails due to an M365 service error? The assistant must inform the user that the action failed, provide the error reason if available, and suggest retrying or performing the action manually.
- What happens if the assistant misinterprets a user's intent as a write action (e.g., "draft an email" vs. "find the email I drafted")? The confirmation step (FR-019) serves as a safeguard — the user can cancel before any write occurs.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The assistant MUST accept natural-language questions from chapter leaders and return relevant answers sourced from the chapter's Microsoft 365 data.
- **FR-002**: The assistant MUST search across SharePoint Online document libraries and sites accessible to the authenticated user.
- **FR-003**: The assistant MUST search across the user's Exchange Online mailbox (emails, calendar events).
- **FR-004**: The assistant MUST search across Teams messages in channels and chats accessible to the authenticated user.
- **FR-005**: The assistant MUST search across OneDrive for Business files accessible to the authenticated user.
- **FR-006**: The assistant MUST extract and interpret content from common document formats (Word documents, Excel spreadsheets, PowerPoint presentations, PDFs).
- **FR-007**: Every factual statement in the assistant's response MUST include a citation referencing the specific source document, email, or message.
- **FR-008**: The assistant MUST provide direct links to source documents and messages so the user can navigate to the original content.
- **FR-009**: The assistant MUST respect the authenticated user's existing Microsoft 365 permissions — it MUST NOT surface content or perform write actions that the user does not have permission for.
- **FR-010**: The assistant MUST clearly state when it cannot find relevant information rather than generating unsupported responses.
- **FR-011**: The assistant MUST authenticate users through their existing Microsoft 365 organizational identity (single sign-on with their chapter's independent M365 tenant account). The assistant MUST support multi-tenant registration so each chapter's tenant admin can authorize the assistant for their tenant. Access MUST be restricted to members of a designated M365 security group (e.g., "Chapter Leaders") configured by each chapter's tenant admin.
- **FR-012**: The assistant MUST support follow-up questions within a conversation, maintaining context from previous exchanges in the same session. A conversation session MUST expire after 30 minutes of inactivity, at which point the context is reset and a new session begins.
- **FR-013**: The assistant MUST ask clarifying questions when a user's query is ambiguous rather than making assumptions.
- **FR-014**: The assistant MUST handle date and time-relative queries (e.g., "last month," "next week," "past 30 days") correctly based on the current date.
- **FR-015**: The assistant MUST be accessible through a web browser without requiring installation of additional software on the user's device.
- **FR-016**: The assistant MUST log all user interactions for auditing purposes, including the full query text, the authenticated user identity, and the timestamp. Write actions MUST additionally log the action type and target resource. Query logs MUST be retained for 90 days and then automatically purged. Response content MUST NOT be logged.
- **FR-017**: The assistant MUST be able to create calendar events in the user's Exchange Online calendar or a shared chapter calendar when requested by the user.
- **FR-018**: The assistant MUST be able to draft and send emails via the user's Exchange Online mailbox when requested by the user.
- **FR-019**: The assistant MUST require explicit user confirmation before executing any write action (creating events, sending emails). The assistant MUST present a preview of the action (e.g., event details, email draft) and wait for the user to confirm or cancel.
- **FR-020**: Write actions MUST execute under the authenticated user's identity and M365 permissions. The assistant MUST NOT perform write actions that the user could not perform directly in M365.

### Key Entities

- **Chapter**: A local Trout Unlimited chapter with its own independent Microsoft 365 tenant, administered separately. Has a name, chapter number, tenant identity, and associated M365 data. Chapters operate independently with their own M365 admin and follow national TU guidelines. The assistant must be onboarded per-tenant.
- **Chapter Leader**: A volunteer who holds a leadership role (president, vice president, treasurer, secretary, conservation chair, etc.) within a chapter. Authenticated via their M365 organizational account and authorized through membership in a designated M365 security group configured by the chapter's tenant admin. Each leader has specific M365 permissions governing what data they can access.
- **Conversation**: A session between a chapter leader and the assistant, consisting of one or more turns (a turn is a single user query + assistant response pair). Maintains context for follow-up questions within the same session. Expires after 30 minutes of inactivity.
- **Source Reference**: A pointer to the original M365 content (document, email, message, calendar event) from which the assistant derived part of its response. Includes the content type, name/subject, location, and a direct link.
- **Query**: A natural-language question or request submitted by a chapter leader to the assistant.

## Assumptions

- Each chapter has its own independent Microsoft 365 Business Standard tenant with separate administration. The assistant must support multi-tenant onboarding.
- Chapter leaders have individual Microsoft 365 accounts within their chapter's tenant with appropriate licenses assigned.
- Each chapter's M365 tenant has standard security defaults enabled; no custom conditional access policies will block the assistant's access patterns.
- The assistant operates within the user's permission scope — it does not use elevated or application-level permissions that exceed what the user can already access directly.
- The assistant targets English-language content and queries for the initial release.
- Chapter sizes vary from a handful of volunteers to several hundred members, but the volume of M365 data per chapter is manageable (not enterprise-scale — typically hundreds to low thousands of documents, not millions).

## Clarifications

### Session 2026-03-18

- Q: How are TU chapters organized in Microsoft 365? → A: Each chapter has its own independent M365 tenant with separate admin.
- Q: Should audit logs include the full text of user queries, or only metadata? → A: Log full query text with a 90-day retention period; auto-purge after 90 days.
- Q: Who within a chapter's M365 tenant should be allowed to use the AI assistant? → A: Only members of a designated M365 security group configured by the tenant admin.
- Q: How long should a conversation session persist before context is reset? → A: 30 minutes of inactivity.
- Q: Should the assistant be strictly read-only or also perform write actions in M365? → A: Include basic write actions from the start (create calendar events, draft/send emails) with mandatory user confirmation before execution.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Chapter leaders can get an accurate, sourced answer to a factual question about their chapter data within 30 seconds of asking.
- **SC-002**: 80% of answers provided by the assistant are rated as accurate and helpful by chapter leaders in post-interaction feedback.
- **SC-003**: Chapter leaders reduce time spent searching for chapter information by at least 50% compared to manually navigating M365 applications.
- **SC-004**: 90% of chapter leaders can successfully ask a question and receive a useful answer on their first interaction without training or documentation.

> **Note**: SC-002, SC-003, and SC-004 are measured post-launch via usage analytics and user surveys. No in-app feedback mechanism is included in the initial release scope.
- **SC-005**: 100% of assistant responses that include factual claims contain at least one source citation linking to the original M365 content.
- **SC-006**: The assistant never surfaces content that the authenticated user does not have M365 permission to view (zero unauthorized data exposure incidents).
- **SC-007**: The assistant correctly declines to answer (rather than fabricating information) in at least 95% of cases where the requested information does not exist in the user's M365 data.
