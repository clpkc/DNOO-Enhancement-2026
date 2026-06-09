# Implementation Plan: 04 - Session Monitoring Mechanism

**Branch**: `item-04-session-monitoring` | **Date**: 2026-06-09 | **Spec**: [spec.md](spec.md)

---

## Summary

Deliver a new ArcGIS Pro add-in (Historical Session Status) under ArcFM XI Session
Manager that lets authenticated users view and monitor their own post-approval GDBM
session lifecycle without admin tool access. The add-in provides search, date range
filtering, session detail (Session ID, last updated time, status), conditional Error
Message display (Post Failed only), and a structured activity record panel.

No ADMS scope. GDBM session data is the sole data source.

---

## Constitution Check

| Gate | Status | Notes |
|------|--------|-------|
| Scope gate — GIS only | ✅ Pass | ArcFM XI Session Manager add-in only; no ADMS scope |
| Clarity gate — requirements documented | ✅ Pass | All BRs, FRs, EH, assumptions, dependencies, constraints explicit in spec |
| Data integrity gate — integrity controls defined | ✅ Pass | CON-002 (user isolation); EH-002 (fallback message); EH-003 (data unavailable); EH-005 (invalid date range) |
| Regression gate — non-regression activities defined | ✅ Pass | FR-012 / CON-005; non-regression test activities listed in Test Approach |
| Layering gate — requirement/design/build/test separated | ✅ Pass | Delivery Layer Mapping in spec; sections separated in this plan |

---

## Technical Context

| Item | Value |
|------|-------|
| Platform | ArcGIS Pro add-in (plugin) |
| Host | ArcFM XI Session Manager |
| Add-in name | Historical Session Status |
| Data source | GDBM session lifecycle data (authenticated user's sessions only) |
| Session scope | Authenticated user's own GDBM sessions only |
| Status values | Exactly four: Pending for approval, Processing, Post Succeeded, Post Failed |
| Error Message field | Visible only when status = Post Failed; hidden for all other statuses |
| ADMS | Not in scope |

---

## UI Layout

```
┌─────────────────────────────────────────────────────────────────┐
│  HEADER TOOLBAR                                                  │
│  [Sort By ▾]   [Search: partial string / dynamic filter      ]  │
├──────────────────────┬──────────────────────────────────────────┤
│  LEFT PANEL          │  RIGHT PANEL — TOP                       │
│                      │  Session ID:       [value]               │
│  Date Range Filter   │  Last Updated:     [value]               │
│  Start: [date]       │  Status:           [one of 4 values]     │
│  End:   [date]       │  Error Message:    [shown if Post Failed] │
│                      ├──────────────────────────────────────────┤
│  Session List        │  RIGHT PANEL — LOWER (Activity Records)  │
│  ─────────────────   │  [label] [message]  [timestamp] [user]   │
│  > Session #1        │  [label] [message]  [timestamp] [user]   │
│    Session #2        │  ...                                     │
│    ...               │                                          │
└──────────────────────┴──────────────────────────────────────────┘
```

---

## Implementation Approach

### Step 1 — Add-in Registration
- Create and register a new ArcGIS Pro add-in targeting ArcFM XI Session Manager.
- The add-in is a separate plugin; it does not modify or replace any existing Session
  Manager tab or feature.

### Step 2 — Authenticated User Session Scope
- On load, the add-in queries GDBM session lifecycle data scoped strictly to the
  authenticated user.
- Sessions belonging to other users MUST NOT be returned or displayed (CON-002 / FR-002).

### Step 3 — Header Toolbar: Sort and Search
- Sort By dropdown: allows sorting of the session list by available sort criteria.
- Search field: free-text partial string matching with dynamic (real-time) filtering
  of the session list — no manual refresh required (FR-003 / SC-003).

### Step 4 — Left Panel: Date Range Filter and Session List
- Date range filter (start date / end date): refines session list to sessions within the
  specified range (FR-004).
- Validation: if end date < start date, surface a clear validation message and do not
  display results (EH-005).
- Scrollable session list displays the filtered result set; selecting a session updates
  the right panel (FR-005).
- Empty state: if no sessions match the filter/search criteria, display a clear empty
  state message (not a silent empty list).

### Step 5 — Right Panel Top: Session Detail
- Display for the selected session: Session ID, last updated time, and current status
  (FR-006).
- Status is limited to exactly four values: Pending for approval, Processing, Post
  Succeeded, Post Failed (FR-007 / CON-003).

### Step 6 — Conditional Error Message
- When status = Post Failed: show Error Message field with failure message content
  from GDBM data (FR-008 / EH-001).
- If Post Failed but error message text is unavailable in source data: show a defined
  fallback message (EH-002).
- For all other status values: Error Message field is hidden (FR-008 / SC-002).

### Step 7 — Right Panel Lower: Activity Records
- Display structured activity records for the selected session (FR-009).
- Each record contains: coloured label, descriptive activity message, timestamp, and
  user context text (e.g., assignment events, session creation events).

### Step 8 — Data Retrieval Resilience
- If lifecycle data cannot be retrieved temporarily, display a clear message that
  monitoring data is currently unavailable (EH-003).
- Error messages must not expose admin-only detail (EH-004).

### Step 9 — Non-Regression
- Existing ArcFM XI Session Manager behavior, session approval, and posting workflows
  must remain unchanged (FR-012 / CON-005).

---

## Impacted Data and Interfaces

| Item | Impact |
|------|--------|
| ArcFM XI Session Manager | New add-in registered; existing behavior unchanged |
| GDBM session lifecycle data | Read-only; scoped to authenticated user |
| Session ID | Read and displayed |
| Last updated timestamp | Read and displayed |
| Session lifecycle status | Read; mapped to one of four defined values |
| Error Message / failure detail | Read and conditionally displayed (Post Failed only) |
| Activity records (label, message, timestamp, user context) | Read and displayed in lower panel |
| Other users' sessions | Never retrieved or displayed (CON-002) |
| Existing session approval / posting workflows | Not modified (FR-012 / CON-005) |

---

## Dependencies

| ID | Dependency | Risk if Unavailable |
|----|-----------|-------------------|
| DEP-001 | GDBM session lifecycle source data provides Session ID, timestamps, status, failure details, and activity records for authenticated user | Add-in cannot display session information; data retrieval fail-safe (EH-003) activates |
| DEP-002 | ArcFM XI Session Manager (ArcGIS Pro) supports add-in registration for GIS users | Add-in cannot be hosted; build blocked |

---

## Assumptions

- GDBM session lifecycle events after approval are accessible to the add-in for the
  authenticated user's sessions via an available data/query interface.
- The four defined status values map to states available in GDBM source data.
- Error message content for Post Failed sessions is available in GDBM source data under
  normal operation; EH-002 covers the fallback case.
- Activity record data (assignment events, session creation events) is available in
  GDBM session data.
- Users have standard access to ArcFM XI Session Manager (ArcGIS Pro).
- Routine monitoring does not require admin workflow features.

---

## Constraints

| ID | Constraint |
|----|-----------|
| CON-001 | Scope is GIS only; add-in is under ArcFM XI Session Manager |
| CON-002 | Only the authenticated user's own GDBM sessions are displayed |
| CON-003 | Status display is limited to exactly four values |
| CON-004 | No ADMS scope |
| CON-005 | Existing session approval and posting behaviors must not regress |

---

## Environments

| Environment | Purpose |
|-------------|---------|
| Development | ArcGIS Pro add-in development; ArcFM XI Session Manager; GDBM test session data |
| Test / UAT | All four status values; Post Failed sessions with error messages; sessions outside date range; dynamic search; multi-user isolation verification |
| Production | Live GIS environment; add-in deployed via standard GIS release process |

---

## Test Approach and Validation Evidence

| Test Case | Expected Evidence |
|-----------|------------------|
| Open add-in — user with sessions | Authenticated user's sessions listed with Session ID, last updated time, status (SC-001) |
| Open add-in — user with no sessions | Empty state message displayed; no error |
| Sessions of other users | Not listed (CON-002) |
| Select Post Failed session | Error Message field shown with failure message or fallback (SC-002 / EH-001/002) |
| Select non-Post-Failed session | Error Message field hidden (SC-002 / FR-008) |
| Dynamic search — partial string entry | Session list filters in real time without manual refresh (SC-003 / FR-003) |
| Search with no match | Empty list with clear message (edge case) |
| Date range filter — valid range | Only sessions within range shown (SC-004 / FR-004) |
| Date range filter — end before start | Clear validation message; no results displayed (EH-005) |
| No sessions in date range | Empty state message displayed; no error (edge case) |
| GDBM data temporarily unavailable | Clear unavailability message shown (EH-003) |
| Post Failed with no error detail in source | Defined fallback message shown (EH-002) |
| All four status values displayed correctly | Each status maps to correct value and label (FR-007 / CON-003) |
| Non-regression — existing Session Manager | No change in approval/posting behavior; regression test pass (FR-012 / CON-005) |

---

## Open Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| GDBM session data query interface scope not fully confirmed | Medium | High | Confirm authenticated-user query mechanism and available fields (Session ID, status, error message, activity records) before build starts (DEP-001) |
| Four status values do not fully map to GDBM source status model | Medium | Medium | Confirm GDBM status field values and mapping to the four required display values during design; document any transformation needed |
| Activity record fields (coloured label, user context) not available in GDBM data | Low | Medium | Verify activity record structure in GDBM source data; fallback: omit fields that are unavailable and document gap |
| Add-in registration not supported in target ArcFM XI version | Low | High | Verify ArcFM XI Session Manager plugin support in target version before build |
| User isolation — authentication identity resolution in GIS environment | Medium | High | Confirm how authenticated user identity is resolved in the ArcGIS Pro / ArcFM context; test multi-user isolation explicitly in UAT |
