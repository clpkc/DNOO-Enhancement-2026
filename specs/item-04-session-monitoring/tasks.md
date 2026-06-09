# Tasks: 04 - Session Monitoring Mechanism

**Branch**: `item-04-session-monitoring` | **Date**: 2026-06-09
**Spec**: [spec.md](spec.md) | **Plan**: [plan.md](plan.md)

---

## Overview

Deliver a new ArcGIS Pro add-in (Historical Session Status) hosted under ArcFM XI
Session Manager. The add-in displays the authenticated user's own GDBM post-approval
session lifecycle. Features: header Sort By / dynamic search, left panel date range
filter + scrollable session list, right panel session detail (Session ID, last updated,
status), conditional Error Message (Post Failed only), and lower activity record panel.
No ADMS scope. Existing Session Manager behavior must not regress.

---

## Group A — Analysis and Environment Readiness

Tasks in this group can run in parallel. Complete before build work begins.

**A-1** — Confirm ArcFM XI Session Manager supports ArcGIS Pro add-in (plugin)
registration in the target version. Document the supported plugin mechanism and entry
point requirements.
*Expected result*: Plugin support confirmed; DEP-002 risk resolved; any version
incompatibility escalated.

**A-2** — Confirm the GDBM session lifecycle data query interface:
- Fields available for authenticated user's sessions: Session ID, last updated timestamp,
  status, failure/error details, activity records.
- Confirm how the authenticated user's identity is resolved (ArcGIS Pro / ArcFM context).
- Confirm user-scoped query can exclude other users' sessions (CON-002).
*Expected result*: Query interface and available fields documented; DEP-001 resolved;
user isolation mechanism confirmed.

**A-3** — Confirm the four GDBM source status values map to the four required display
values:
- Pending for approval, Processing, Post Succeeded, Post Failed.
- Document any transformation needed between source status codes and display labels.
*Expected result*: Status mapping table documented; CON-003 confirmed.

**A-4** — Confirm GDBM activity record structure: fields available for lower panel
display (coloured label, descriptive message, timestamp, user context). Identify any
fields that are unavailable and agree on fallback behavior (omit and document gap).
*Expected result*: Activity record fields confirmed or gaps documented with agreed
fallback.

**A-5** — Obtain or generate test data set in the GDBM development environment covering:
- Sessions for the authenticated user in all four lifecycle statuses.
- At least one Post Failed session with error message, and one Post Failed without error
  message (to test EH-002 fallback).
- Sessions across multiple date ranges to support filter testing.
*Expected result*: Test data set confirmed available; test scenarios fully exercisable.

---

## Group B — Add-in Registration and Shell

*Depends on A-1 and A-2 completing first.*

**B-1** — Create and register a new ArcGIS Pro add-in targeting ArcFM XI Session Manager
(FR-001). The add-in is a new separate plugin — it must not modify or replace any
existing Session Manager tab or feature.
*Expected result*: Add-in appears under ArcFM XI Session Manager; existing Session
Manager behavior unchanged.

**B-2** — Implement the authenticated user session scope query on add-in load (FR-002 /
CON-002): retrieve GDBM session lifecycle data for the authenticated user only. Confirm
no sessions from other users are returned in any test scenario.
*Expected result*: Only authenticated user's sessions returned; other users' data not
accessible.

---

## Group C — Header Toolbar: Sort and Search

*Depends on B-1 and B-2. C-1 and C-2 can run in parallel.*

**C-1** — Implement Sort By dropdown in the header toolbar allowing session list to be
sorted by available sort criteria (FR-003).
*Expected result*: Dropdown present in header; selecting a sort criterion reorders the
session list.

**C-2** — Implement the search field with free-text partial string matching and dynamic
(real-time) filtering of the session list (FR-003 / SC-003). The list must update as
characters are typed — no manual refresh required.
*Expected result*: Session list filters dynamically on each keystroke; no refresh button
needed.

---

## Group D — Left Panel: Date Range Filter and Session List

*Depends on B-1 and B-2. D-1 and D-2 can run in parallel.*

**D-1** — Implement the date range filter (start date / end date) in the left panel
(FR-004 / SC-004). Refine the session list to only sessions within the specified range.
Implement invalid date range guard (EH-005): if end date < start date, surface a clear
validation message and do not display results.
*Expected result*: Date filter applied correctly; invalid range blocked with clear
message.

**D-2** — Implement the scrollable session list in the left panel (FR-005): display
filtered sessions; selecting a session updates the right panel. Implement empty state:
if no sessions match filter/search, display a clear empty state message — not a silent
empty list.
*Expected result*: Session list updates on filter/search; empty state message shown when
no results; selecting a session populates the right panel.

---

## Group E — Right Panel Top: Session Detail

*Depends on D-2 (session selection wired). E-1 and E-2 can run in parallel.*

**E-1** — Implement Session ID, last updated time, and lifecycle status display for the
selected session in the right panel top section (FR-006 / FR-007). Status must be one
of exactly four values: Pending for approval, Processing, Post Succeeded, Post Failed
(CON-003). Apply the status mapping from A-3.
*Expected result*: All four status values displayed correctly for corresponding sessions;
no additional values appear.

**E-2** — Implement the conditional Error Message field (FR-008 / SC-002 / EH-001):
- When status = Post Failed: show Error Message field with failure message content from
  GDBM data.
- When status = Post Failed and error message text is unavailable: show defined fallback
  message (EH-002).
- For all other statuses: Error Message field is hidden.
*Expected result*: Error Message visible only for Post Failed sessions; hidden for all
others; fallback message shown when source data has no error text.

---

## Group F — Right Panel Lower: Activity Records

*Depends on E-1 (session selection and right panel wired).*

**F-1** — Implement the structured activity records panel in the right panel lower area
(FR-009): display one row per activity record, each containing coloured label,
descriptive activity message, timestamp, and user context text. Source from GDBM
activity record data confirmed in A-4.
*Expected result*: Activity records displayed for selected session; each row contains all
four fields (or agreed fallback if a field is unavailable per A-4).

---

## Group G — Resilience and Error Messages

*Depends on B-2. Can run in parallel with Groups C–F.*

**G-1** — Implement data retrieval resilience (EH-003): if GDBM lifecycle data cannot
be retrieved, display a clear message that monitoring data is currently unavailable.
The message must not expose admin-only details (EH-004).
*Expected result*: Clear unavailability message shown on data retrieval failure; no
admin-only detail in message text.

---

## Group H — Validation and Test Evidence

*Depends on E-2, F-1, and G-1 completing. H-1 through H-13 can run in parallel.*

**H-1** — **Evidence: Open add-in — user with sessions.**
Log in and open add-in. Verify authenticated user's sessions listed with Session ID,
last updated time, and valid status.
*Expected result*: Screenshot / record confirming SC-001 / FR-002.

**H-2** — **Evidence: Open add-in — user with no sessions.**
Open add-in with a user who has no sessions in the range. Verify empty state message
shown; no error.
*Expected result*: Screenshot confirming empty state behavior.

**H-3** — **Evidence: User isolation.**
Open add-in as User A; confirm sessions belonging to User B are not visible.
*Expected result*: Record confirming CON-002.

**H-4** — **Evidence: Post Failed — Error Message shown.**
Select a known Post Failed session. Verify Error Message field is visible with failure
content.
*Expected result*: Screenshot confirming SC-002 / EH-001.

**H-5** — **Evidence: Post Failed — Error Message fallback.**
Select a Post Failed session where error text is unavailable in source data. Verify
defined fallback message is shown.
*Expected result*: Screenshot confirming EH-002.

**H-6** — **Evidence: Non-Post-Failed — Error Message hidden.**
Select sessions in each of the three non-failed statuses. Verify Error Message field is
hidden for all.
*Expected result*: Screenshot confirming SC-002 / FR-008.

**H-7** — **Evidence: Dynamic search — partial string.**
Type a partial string in the search field. Verify session list filters in real time
without manual refresh.
*Expected result*: Screen recording / screenshot confirming SC-003 / FR-003.

**H-8** — **Evidence: Search with no match.**
Enter a string that matches no sessions. Verify empty state message shown; no error.
*Expected result*: Screenshot confirming edge case behavior.

**H-9** — **Evidence: Date range filter — valid range.**
Apply a date range containing known sessions. Verify only sessions within range shown.
*Expected result*: Record confirming SC-004 / FR-004.

**H-10** — **Evidence: Date range filter — end before start.**
Set end date earlier than start date. Verify clear validation message appears; no
results displayed.
*Expected result*: Screenshot confirming EH-005.

**H-11** — **Evidence: All four status values displayed.**
Cycle through sessions in each of the four lifecycle statuses. Verify each status maps
to the correct display label.
*Expected result*: Test matrix confirming FR-007 / CON-003.

**H-12** — **Evidence: Activity records displayed.**
Select a session with activity record data. Verify each row shows label, message,
timestamp, and user context.
*Expected result*: Screenshot confirming FR-009.

**H-13** — **Evidence: GDBM data temporarily unavailable.**
Simulate retrieval failure. Verify clear unavailability message displayed; no admin
detail exposed.
*Expected result*: Screenshot confirming EH-003 / EH-004.

**H-14** — **Evidence: Non-regression — existing Session Manager.**
Run regression pass on ArcFM XI Session Manager approval and posting workflows. Verify
no change in behavior.
*Expected result*: Regression pass record confirming FR-012 / CON-005.

---

## Group I — Deployment Preparation

*Depends on all Group H evidence gathered and approved.*

**I-1** — Package the add-in for deployment to the UAT environment. Confirm it loads
under ArcFM XI Session Manager, user isolation is active, and all UI panels are present.
Run smoke test.
*Expected result*: Add-in deployed to UAT; smoke test passed; no configuration delta.

**I-2** — Prepare release notes covering: add-in name (Historical Session Status), host
(ArcFM XI Session Manager), session scope (authenticated user only), search/filter
features, four status values, conditional Error Message, activity record panel, and what
is unchanged (existing Session Manager behavior).
*Expected result*: Release notes reviewed and approved by GIS lead.

**I-3** — Confirm production deployment checklist covers: add-in registration, GDBM
data interface access, authentication identity resolution, and rollback steps.
*Expected result*: Deployment checklist signed off.

---

## Group J — Documentation Update

*Can run in parallel with Group I.*

**J-1** — Update spec.md Status field from "Draft" to "Ready for Implementation" once
all validation evidence is gathered.
*Expected result*: Spec status reflects implementation readiness.

**J-2** — Record final test evidence file paths and test pass/fail outcomes in the spec
or linked test artifact for BA / QA review.
*Expected result*: Evidence traceable from spec to test records.

---

## Dependency Summary

```
A-1, A-2, A-3, A-4, A-5     (parallel — no dependencies)
          ↓
       B-1 → B-2             (sequential)
          ↓
C-1, C-2 / D-1, D-2 / G-1   (parallel after B-2)
          ↓
       E-1, E-2              (parallel after D-2)
          ↓
          F-1                (after E-1)
          ↓
H-1 through H-14             (parallel after E-2, F-1, G-1)
          ↓
I-1, I-2, I-3 / J-1, J-2    (I and J parallel after H approved)
```

---

## Open Items Carried from Plan

| Item | Status | Action Required |
|------|--------|----------------|
| GDBM query interface scope | Risk (High) | Confirmed in A-2 before build; escalate if fields missing |
| Four status values → GDBM mapping | Risk (Medium) | Confirmed in A-3; transformation documented before E-1 |
| Activity record fields availability | Risk (Low) | Confirmed in A-4; fallback agreed before F-1 |
| ArcFM XI add-in registration support | Risk (High) | Confirmed in A-1; escalate if version incompatibility |
| User identity resolution + isolation | Risk (High) | Confirmed in A-2; multi-user isolation tested explicitly in H-3 |
