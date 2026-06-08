# 04 - Session monitoring mechanism

## What the enhancement is
This is a new Historical Session Status add-in tool under ArcFM XI Session Manager.
Its purpose is to let users monitor historical GDBM sessions associated with the authenticated user.

## Business purpose / problem statement
The design says users need visibility into whether a session is:
- still in progress
- successfully posted
- failed

## Detailed logic / behaviour
UI / behavior explicitly described:

Header toolbar:
- Sort By dropdown
- Search field with:
  - free-text input
  - partial string matching
  - dynamic filtering without manual refresh

Left panel:
- Date range filter:
  - start date
  - end date
  - used to refine displayed sessions
- Session list:
  - scrollable list of the user's historical sessions
  - selecting a session updates the detail panel

Right panel - Session Information (top section displays):
- Session ID
- Last updated time
- Current status

Lifecycle statuses explicitly listed (only these four):
- Pending for approval
- Processing
- Post Succeeded
- Post Failed

Explicit status definitions:
- Pending for Approval:
  - Session submitted for approval but not yet approved
- Processing:
  - Session approved and currently being processed by GDBM
- Post Succeeded:
  - Session posted successfully to Default
  - Reconcile Events completed
  - PrePost Reconcile Events completed
  - Post Events completed
  - all required action handlers executed without error
- Post Failed:
  - An error occurred during a stage of the GDBM posting workflow

Error handling in UI:
- Error Message field is shown only when status = Post Failed
- For other statuses, the field remains hidden

Additional session detail area (lower panel) contains structured activity/metadata records, including:
- coloured label
- descriptive activity message
- timestamp
- associated user context text

Examples explicitly mentioned:
- Assignment events
- Session creation events

## Assumptions / constraints
- Monitoring scope is historical sessions associated with the authenticated user.
- Status display is limited to the four explicitly listed values.

## Notes from review / clarification
- Item described as a new ArcGIS Pro plugin/add-in under ArcFM XI Session Manager.
