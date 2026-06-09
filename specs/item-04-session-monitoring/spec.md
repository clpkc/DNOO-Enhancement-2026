# 04 - Session Monitoring Mechanism

**Feature Branch**: `item-04-session-monitoring`

**Created**: 2026-06-08

**Status**: Draft

**Input**: User description: "Create a specification for one DNOO GIS enhancement item only. Requirement ID: 04 - Session monitoring mechanism. Business requirement: Add a History tab in ArcFM Session Manager (ArcGIS Pro) to let users track session lifecycle after approval. Display: Session ID, last updated time, status values: Pending for approval, Processing, Post Succeeded, Post Failed. Failed sessions should show error messages. Business purpose: Eliminate the need for manual checking or admin tool access. Please produce: business problem, current pain point, target users, desired future behaviour, in-scope / out-of-scope, assumptions, dependencies, constraints, acceptance criteria, error handling expectations, edge cases / open questions. Scope notes: GIS only. Do not introduce ADMS scope unless it is an explicitly stated dependency. Keep the wording aligned with the business requirement above."

## Clarifications

### Session 2026-06-09

- Q: What is the tool identity? → A: A new ArcGIS Pro add-in (plugin) under ArcFM XI Session Manager; the History view is part of this add-in, not a tab on the existing Session Manager.
- Q: Whose sessions are shown? → A: Historical GDBM sessions associated with the authenticated user only.
- Q: How does the lookback window work? → A: A date range filter (start date / end date) in the left panel lets users refine displayed sessions; there is no fixed default lookback window.
- Q: What is the UI layout? → A: Header toolbar (Sort By dropdown, Search with partial string matching and dynamic filtering); left panel (date range filter, scrollable session list); right panel top (Session ID, last updated, status); right panel lower (structured activity records with label, message, timestamp, user context).
- Q: When is the Error Message field shown? → A: Only when status = Post Failed; it is hidden for all other status values.

## Clarifications

### Session 2026-06-09

- Q: Is this a new tab or a new add-in? → A: New ArcGIS Pro add-in/plugin under ArcFM XI Session Manager.
- Q: Whose sessions are shown? → A: Historical GDBM sessions associated with the authenticated user only.
- Q: Is there a lookback window or a date range filter? → A: Date range filter (start date / end date) in the left panel; user selects the range.
- Q: How does search work? → A: Free-text input with partial string matching and dynamic filtering — no manual refresh needed.
- Q: Is Error Message always shown or conditionally shown? → A: Error Message field is shown only when status = Post Failed; for all other statuses it remains hidden.

## User Scenarios & Testing *(mandatory)*

<!--
  IMPORTANT: User stories should be PRIORITIZED as user journeys ordered by importance.
  Each user story/journey must be INDEPENDENTLY TESTABLE - meaning if you implement just ONE of them,
  you should still have a viable MVP (Minimum Viable Product) that delivers value.

  Assign priorities (P1, P2, P3, etc.) to each story, where P1 is the most critical.
  Think of each story as a standalone slice of functionality that can be:
  - Developed independently
  - Tested independently
  - Deployed independently
  - Demonstrated to users independently
-->

### User Story 1 - View Own GDBM Session Lifecycle in the Add-in (Priority: P1)

As a GIS user, I open the Historical Session Status add-in and see my own GDBM sessions
listed with Session ID, last updated time, and lifecycle status, without using admin
tools.

**Why this priority**: This is the direct business need and primary value of the
enhancement.

**Independent Test**: Open the add-in and verify that the authenticated user's GDBM
sessions are listed with Session ID, last updated time, and one of the required status
values. Confirm sessions belonging to other users are not shown.

**Acceptance Scenarios**:

1. **Given** a session associated with the authenticated user has entered post-approval
   processing, **When** the user opens the add-in, **Then** the session appears in the
   session list with Session ID, last updated time, and current status.
2. **Given** a session status changes during processing, **When** the user selects the
   session in the list, **Then** the updated status and last updated time are shown in
   the right panel.
3. **Given** the user applies a date range filter, **When** start and end dates are set,
   **Then** the session list is refined to show only sessions within that range.

---

### User Story 2 - Identify Failed Sessions and View Error Details (Priority: P2)

As a GIS user, I can see the Error Message field for Post Failed sessions so I can
understand the failure reason without manual investigation through admin-only tools.

**Why this priority**: Failure visibility removes manual checking delays and supports
operational recovery.

**Independent Test**: Select a known Post Failed session in the add-in and verify the
Error Message field is visible and populated. Select a non-failed session and verify the
Error Message field is hidden.

**Acceptance Scenarios**:

1. **Given** a session is in Post Failed status, **When** the user selects that session
   in the list, **Then** the Error Message field is shown in the right panel with a
   failure message.
2. **Given** a session is in any status other than Post Failed, **When** the user selects
   it, **Then** the Error Message field is hidden from the right panel.

---

### User Story 3 - Search and Filter Sessions Without Admin Dependency (Priority: P3)

As a GIS operations lead, I need the add-in to support search and date range filtering
so users can locate specific sessions and determine lifecycle outcomes without admin tool
access.

**Why this priority**: This aligns directly to the business purpose and operational
efficiency objective; search and filtering reduce time spent locating sessions manually.

**Independent Test**: Use the search field to find a session by partial string and verify
the list filters dynamically. Apply a date range and verify only sessions within the
range are shown. Confirm all monitoring is possible without admin tool access.

**Acceptance Scenarios**:

1. **Given** a user types a partial string in the search field, **When** characters are
   entered, **Then** the session list filters dynamically without requiring a manual
   refresh.
2. **Given** users monitor sessions from the add-in, **When** routine tracking is
   performed, **Then** session state can be determined without admin tool access.

---

### Edge Cases

- Last updated time is missing or delayed for a session record.
- Status transitions skip expected order (for example, Pending for approval directly to
  Post Failed).
- Multiple sessions share similar timestamps; users identify them by Session ID.
- Error Message text is unavailable for a Post Failed record; fallback message shown.
- The authenticated user has no sessions within the selected date range; the session list
  shows empty with a clear message.
- Date range filter is set with end date earlier than start date.
- Search field input matches no sessions; list shows empty state without error.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST provide a new ArcGIS Pro add-in (Historical Session Status)
  under ArcFM XI Session Manager for post-approval GDBM session lifecycle monitoring.
- **FR-002**: The add-in MUST show only GDBM sessions associated with the authenticated
  user.
- **FR-003**: The add-in header toolbar MUST include a Sort By dropdown and a search
  field supporting free-text partial string matching with dynamic filtering (no manual
  refresh required).
- **FR-004**: The left panel MUST provide a date range filter (start date and end date)
  to refine the displayed session list.
- **FR-005**: The left panel MUST display a scrollable list of the authenticated user's
  historical sessions; selecting a session updates the right panel.
- **FR-006**: The right panel (top section) MUST display Session ID, last updated time,
  and current status for the selected session.
- **FR-007**: Session status MUST be limited to the following four values only:
  Pending for approval, Processing, Post Succeeded, Post Failed.
- **FR-008**: The right panel MUST show an Error Message field only when status = Post
  Failed; for all other statuses the field MUST be hidden.
- **FR-009**: The right panel lower area MUST display structured activity records for
  the selected session, each containing a coloured label, descriptive activity message,
  timestamp, and associated user context text.
- **FR-010**: System MUST update displayed status and last updated time to reflect the
  latest known lifecycle state for the selected session.
- **FR-011**: System MUST allow routine session lifecycle monitoring without requiring
  admin tool access.
- **FR-012**: System MUST keep unrelated GIS Session Manager behavior unchanged.
- **FR-013**: System MUST keep scope limited to GIS session monitoring and MUST NOT
  introduce ADMS workflow or design scope.

### Business Problem *(mandatory)*

After session approval, users lack direct visibility of their GDBM session lifecycle
progress and outcome within ArcFM Session Manager, leading to reliance on manual
checking or admin tools.

### Current Pain Point *(mandatory)*

Users must manually follow up or seek admin assistance to know whether an approved
session is still processing, succeeded, or failed, which delays response and increases
operational overhead.

### Target Users *(mandatory)*

- GIS editors and operational users tracking post-approval session progress.
- GIS operations leads monitoring routine session outcomes.

### Desired Future Behaviour *(mandatory)*

Users open the Historical Session Status add-in under ArcFM XI Session Manager
(ArcGIS Pro) and immediately see their own GDBM sessions. They can filter by date range
and search by partial string to locate specific sessions. The right panel shows Session
ID, last updated time, lifecycle status, and — only for Post Failed sessions — an error
message. A structured activity record area provides additional session event detail.

### In Scope *(mandatory)*

- New Historical Session Status add-in (ArcGIS Pro plugin) under ArcFM XI Session
  Manager.
- Display of authenticated user's own GDBM sessions only.
- Header toolbar with Sort By and search (partial string, dynamic filtering).
- Left panel date range filter and scrollable session list.
- Right panel top: Session ID, last updated time, status (four values only).
- Right panel Error Message field: shown only for Post Failed, hidden otherwise.
- Right panel lower: structured activity records (label, message, timestamp, user
  context).
- User monitoring without admin tool dependency.

### Out of Scope *(mandatory)*

- ADMS workflow, design, or implementation changes.
- Changes to approval workflow logic outside monitoring visibility.
- Redesign of admin tools.
- Changes to statuses beyond the defined four values.

### Business Requirements *(mandatory)*

- **BR-001**: Provide a new Historical Session Status add-in under ArcFM XI Session
  Manager to let users monitor their own GDBM session lifecycle after approval.
- **BR-002**: Display Session ID, last updated time, and statuses (Pending for approval,
  Processing, Post Succeeded, Post Failed) with search and date range filtering.
- **BR-003**: Show error messages for Post Failed sessions; hide the field for all other
  statuses.
- **BR-004**: Eliminate the need for manual checking or admin tool access for routine
  session monitoring.

### Dependencies *(mandatory)*

- **DEP-001**: GDBM session lifecycle source data provides Session ID, timestamps,
  status, failure details, and activity records for the authenticated user.
- **DEP-002**: ArcFM XI Session Manager (ArcGIS Pro) supports the add-in for GIS users.

### Constraints *(mandatory)*

- **CON-001**: Scope is GIS only; add-in is under ArcFM XI Session Manager.
- **CON-002**: Session display is restricted to the authenticated user's own GDBM
  sessions; other users' sessions must not be visible.
- **CON-003**: Status display is limited to exactly four values; no additional statuses
  may be introduced.
- **CON-004**: No ADMS scope may be introduced unless explicitly stated as dependency.
- **CON-005**: Existing session approval and posting behaviors must not regress.

### Error Handling Expectations *(mandatory)*

- **EH-001**: If a session is Post Failed, History MUST show an error message for that
  session.
- **EH-002**: If failure details are unavailable, History MUST show a clear fallback
  message indicating details are not available.
- **EH-003**: If lifecycle data cannot be retrieved temporarily, users MUST receive a
  clear message that monitoring data is currently unavailable.
- **EH-004**: Error handling messages MUST not expose admin-only details or require admin
  tool access for interpretation.

### Key Entities *(include if feature involves data)*

- **GDBM Session**: The session entity whose lifecycle is monitored; scoped to the
  authenticated user's own sessions only.
- **Session History Record**: Display entry in the add-in containing Session ID, last
  updated time, current status, and optional Error Message (Post Failed only).
- **Session Lifecycle Status**: Controlled status value restricted to exactly four
  values — Pending for approval, Processing, Post Succeeded, Post Failed — with the
  following meanings:
  - *Pending for approval*: Session submitted for approval but not yet approved.
  - *Processing*: Session approved and currently being processed by GDBM.
  - *Post Succeeded*: Session posted to Default; Reconcile Events, PrePost Reconcile
    Events, and Post Events all completed without error.
  - *Post Failed*: An error occurred during a stage of the GDBM posting workflow.
- **Activity Record**: Structured lower-panel entry for a session containing a coloured
  label, descriptive activity message, timestamp, and user context text (e.g.,
  assignment events, session creation events).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In 100% of tested records, the add-in displays Session ID, last updated
  time, and one valid lifecycle status for the authenticated user's GDBM sessions only.
- **SC-002**: In 100% of tested Post Failed sessions, the Error Message field is shown
  with a failure message or the defined fallback message. In 100% of non-Post-Failed
  sessions, the Error Message field is hidden.
- **SC-003**: In 100% of tested cases, the session list filters dynamically when partial
  search string is entered, without manual refresh.
- **SC-004**: In 100% of tested cases, applying a date range filter shows only sessions
  within the specified range.
- **SC-005**: In at least 90% of routine monitoring scenarios, users determine session
  lifecycle outcome using the add-in without admin tool access.
- **SC-006**: Manual follow-up checks for approved session lifecycle status are reduced
  by at least 80% in tested operational scenarios compared to current workflow.

## Assumptions

- GDBM session lifecycle events after approval are accessible to the add-in for the
  authenticated user's sessions.
- Users performing monitoring have standard access to ArcFM XI Session Manager (ArcGIS
  Pro).
- Error message content needed for user action is available in GDBM source data for
  Post Failed sessions under normal operation.
- Routine monitoring does not require expanding into admin workflow features.
- Activity records (assignment events, session creation events) are available in
  GDBM session data for display in the lower detail panel.

## Delivery Layer Mapping *(mandatory)*

Document each major item with an explicit layer label so reviewers can separate intent,
design, build work, and verification:

- **Business Requirement**: Users must track their own approved GDBM session lifecycle
  and failures in the Historical Session Status add-in without manual checking or admin
  tool access.
- **Technical Design**: New ArcGIS Pro add-in under ArcFM XI Session Manager with header
  search/sort, left panel date range filter and session list, right panel status/detail
  (Error Message conditionally shown), and lower activity detail panel.
- **Implementation Task**: Deliver add-in with authenticated-user session scope, search
  and filter behaviors, status definitions, conditional Error Message field, and activity
  record display while preserving existing Session Manager behavior.
- **Test Consideration**: Validate user session isolation, date range filter, dynamic
  search filtering, status value coverage (all four), conditional Error Message display
  (shown/hidden), activity record display, fallback messages, and non-regression of
  existing GIS session operations.
