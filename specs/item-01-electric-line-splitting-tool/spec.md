# 01 - Electric Line Splitting Tool

**Feature Branch**: `item-01-electric-line-splitting-tool`

**Created**: 2026-06-08

**Status**: Draft

**Input**: User description: "Create a specification for one DNOO GIS enhancement item only. Requirement ID: 01 - Electric line splitting tool. Business requirement: The Split Line ArcFM task allows users to select a line feature, optionally input User Length, and split it at a chosen point. It automatically calculates User Length for child features based on the parent’s length ratio. Segment length is also updated proportionally. Example: parent line A = 100; split into B = 50 and C = 50; if segment length = 120, each child becomes 60. Business purpose: Address the need for automatic segment length updates without manual input for split or merged circuits. Please produce: business problem, current pain point, target users, desired future behaviour, in-scope / out-of-scope, assumptions, dependencies, constraints, acceptance criteria, edge cases / open questions. Scope notes: GIS only. Do not introduce ADMS scope unless it is an explicitly stated dependency. Keep the wording aligned with the existing business requirement above."

## Clarifications

### Session 2026-06-09

- Q: Does the dialog show GlobalId or a separate Electric Line ID? → A: Electric Line ID is populated from the selected line's GlobalId.
- Q: Is the User Length field editable after pre-population? → A: Yes — the field is explicitly editable; an editor-supplied override takes precedence for child redistribution.
- Q: Does the user input the split ratio? → A: No — the system derives the ratio from the actual geometry split point; user does not input it.
- Q: What happens at the degenerate split edge case? → A: The task must prevent or handle splits that would produce a zero-length child (split point at or near an existing endpoint); the task must not silently create zero-length features.

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

### User Story 1 - Split Electric Line With Automatic User Length Redistribution (Priority: P1)

As a GIS editor, I split an electric line at a chosen point and the system
automatically recalculates LENGTHUSER for the child features in proportion to the
geometry split.

**Why this priority**: This is the core business outcome and directly removes manual
LENGTHUSER recalculation from the split workflow.

**Independent Test**: Select a parent ElectricLine with a known LENGTHUSER value,
split it into two child features, and verify that both child LENGTHUSER values are
updated proportionally based on the geometry split ratio.

**Acceptance Scenarios**:

1. **Given** a parent ElectricLine where LENGTHUSER = 100, **When** the user splits the
   line so that the geometry splits in a 1:1 ratio, **Then** each child receives
   LENGTHUSER = 50.
2. **Given** a parent ElectricLine where LENGTHUSER = 100, **When** the user splits the
   line so that the geometry splits in a 2:3 ratio, **Then** one child receives
   LENGTHUSER = 40 and the other receives LENGTHUSER = 60.

---

### User Story 2 - Dialog Pre-Populates User Length and Accepts Override (Priority: P2)

As a GIS editor, when I open the split task dialog, the User Length field is
pre-populated with the line's stored LENGTHUSER (if > 0) or Shape_Length (as fallback),
and I can edit it before confirming the split.

**Why this priority**: This preserves editor control over the parent length value while
ensuring automation works for lines regardless of whether LENGTHUSER is already stored.

**Independent Test**: Start the split task on (a) a line where LENGTHUSER > 0 and
(b) a line where LENGTHUSER = 0 or null. Verify pre-population source and that an
edited value is used for child redistribution.

**Acceptance Scenarios**:

1. **Given** an ElectricLine where LENGTHUSER = 80, **When** the editor opens the task
   dialog, **Then** the User Length field is pre-populated with 80.
2. **Given** an ElectricLine where LENGTHUSER = 0 or null, **When** the editor opens
   the task dialog, **Then** the User Length field is pre-populated with Shape_Length.
3. **Given** a pre-populated User Length of 80, **When** the editor overrides it with
   100 and then selects the split point, **Then** child LENGTHUSER values are
   redistributed based on 100, not 80.

---

### User Story 3 - Task Is Executable Only in an Active Edit Session (Priority: P3)

As a GIS operations lead, I need the electric line split task to enforce that it runs
only when GIS editing is active, and that it leaves unrelated GIS editing behavior
unchanged.

**Why this priority**: Scope and edit-session gating prevent data integrity failures and
user confusion outside a valid edit context.

**Independent Test**: Attempt the split task outside an edit session and verify it
cannot be executed; then run it within an edit session and verify unrelated GIS editing
behavior is unaffected.

**Acceptance Scenarios**:

1. **Given** no active edit session, **When** the user attempts to invoke the split
   task, **Then** the task does not execute and the user receives an appropriate
   message.
2. **Given** an active edit session, **When** the editor completes the split, **Then**
   the edit succeeds without changing unrelated GIS editing behavior.

---

### Edge Cases

- LENGTHUSER = 0 or null on the parent line: the task falls back to Shape_Length for
  pre-population; child LENGTHUSER values are redistributed from that fallback value.
- The editor overrides the pre-populated User Length with a value greater or less than
  the geometry-derived length: child redistribution must use the editor-supplied value.
- The split creates child lengths that result in decimal ratios and rounding is needed.
- The selected split point is at or very close to an existing endpoint: the task must
  handle degenerate splits without producing zero-length child features silently.
- The parent ElectricLine belongs to an AssetGroup or AssetType combination not
  previously tested: the task applies to all AssetGroups and AssetTypes for the
  ElectricLine feature class.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST provide a new ArcFM task under the Line folder in the Tasks
  Pane that allows the user to select an ElectricLine feature and split it at a chosen
  point.
- **FR-002**: The task dialog MUST display the selected line's Electric Line ID
  (populated from GlobalId) and a User Length field.
- **FR-003**: The User Length field MUST be pre-populated as follows: if LENGTHUSER > 0
  then use LENGTHUSER; otherwise use Shape_Length. The field MUST remain editable.
- **FR-004**: System MUST derive the split ratio from the geometry of the selected split
  point; the user MUST NOT be required to input the ratio.
- **FR-005**: System MUST calculate LENGTHUSER for each child ElectricLine feature
  according to the split ratio applied to the parent User Length (whether stored or
  editor-supplied).
- **FR-006**: System MUST complete the split without requiring manual entry of child
  LENGTHUSER values.
- **FR-007**: System MUST leave unrelated GIS editing behavior unchanged.
- **FR-008**: System MUST round child LENGTHUSER values to 2 decimal places and assign
  any rounding residual to the longer child feature so that child totals match the
  parent total.
- **FR-009**: The task MUST only be executable within an active GIS edit session.
- **FR-010**: The task MUST apply to the ElectricLine feature class across all
  AssetGroups and AssetTypes.

### Business Requirements *(mandatory)*

- **BR-001**: Address the need for automatic LENGTHUSER redistribution without manual
  child entry when an ElectricLine is split.
- **BR-002**: Reduce manual effort and data-entry risk in the split-line task.
- **BR-003**: Segment Length (onsite installed length) remains a manual user input and
  is not in scope for automatic redistribution.

### Business Problem *(mandatory)*

When ElectricLine features are split, child LENGTHUSER values currently require manual
input or correction, which creates extra effort and increases the chance of incorrect
GIS attribute values.

### Current Pain Point *(mandatory)*

Users must manually determine or correct child LENGTHUSER values after a split.
This slows editing work and can introduce inconsistent GIS data. Segment Length
(onsite installed length) remains a separate manual field and is not affected by this
enhancement.

### Target Users *(mandatory)*

- GIS editors performing ElectricLine split operations in ArcGIS Pro (ArcFM task).
- GIS operations staff responsible for reviewing edit outcomes and data quality.

### Desired Future Behaviour *(mandatory)*

A new ArcFM task (under the Line folder in the Tasks Pane) presents the editor with the
ElectricLine's GlobalId and a pre-populated, editable User Length value. After the
editor selects the split point, the task splits the parent line into two child
ElectricLine features and automatically assigns child LENGTHUSER values in proportion
to the geometry split. No manual child LENGTHUSER entry is required. Segment Length
(onsite installed length) continues to be entered manually by business users.

### In Scope *(mandatory)*

- A new ArcFM task for splitting ElectricLine features, accessible under the Line
  folder in the Tasks Pane.
- Proportional redistribution of LENGTHUSER from parent to child ElectricLine features.
- Dialog display of Electric Line ID (GlobalId) and editable User Length field with
  pre-population logic (LENGTHUSER > 0 → LENGTHUSER; else → Shape_Length).
- System-derived split ratio from the chosen geometry split point.
- Applies to the ElectricLine feature class across all AssetGroups and AssetTypes.
- GIS workflow outcomes only; task executable within active edit sessions only.

### Out of Scope *(mandatory)*

- Automatic redistribution or calculation of Segment Length (onsite installed length);
  this field remains manually entered by business users.
- ADMS workflow, design, or implementation changes.
- Electric circuit merge automation.
- Changes to unrelated GIS editing tools.
- Any feature class other than ElectricLine.

### Dependencies *(mandatory)*

- **DEP-001**: ArcGIS Pro ArcFM task framework and the Line folder location in the
  Tasks Pane.
- **DEP-002**: Availability of LENGTHUSER and Shape_Length attributes on the
  ElectricLine feature class in the GIS data model.
- **DEP-003**: An active GIS edit session; the task cannot run outside one.
- **DEP-004**: Any ADMS reference remains dependency-only and must not alter ADMS
  scope.

### Constraints *(mandatory)*

- **CON-001**: Scope is GIS only; applies to ElectricLine feature class only.
- **CON-002**: Segment Length (onsite installed length) is not automatically
  redistributed; it remains a manual business-user input.
- **CON-003**: GIS captures only 2-dimensional circuit length; LENGTHUSER and
  Shape_Length reflect 2D geometry.
- **CON-004**: The task must only execute within an active GIS edit session.
- **CON-005**: The enhancement must not create regression in existing GIS editing
  behavior.
- **CON-006**: No ADMS scope may be introduced unless explicitly stated as a
  dependency.

### Error Handling Expectations

- **EH-001**: If the selected split point is at or very close to an existing endpoint
  such that a zero-length child would result, the task MUST prevent the split and present
  a clear message to the user. Zero-length child ElectricLine features MUST NOT be
  silently created.
- **EH-002**: If the task is invoked outside an active edit session, it MUST not execute
  and MUST present a clear message to the user (covered by FR-009 and CON-004).

### Key Entities *(include if feature involves data)*

- **Parent ElectricLine**: The original ElectricLine feature selected for splitting,
  including its LENGTHUSER (if > 0) or Shape_Length (fallback) and GlobalId.
- **Child ElectricLine**: Each resulting ElectricLine feature created by the split,
  with LENGTHUSER recalculated proportionally from the parent.
- **Split Ratio**: The proportional relationship between each child line's geometry
  length and the parent line's geometry length, derived from the chosen split point.
- **LENGTHUSER**: GIS-stored user-facing length attribute on ElectricLine; subject to
  automatic redistribution by this task.
- **Shape_Length**: GIS geometry-derived length used as fallback when LENGTHUSER = 0
  or null.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can complete a standard ElectricLine split without manual child
  LENGTHUSER entry in 95% of tested cases.
- **SC-002**: In 100% of tested split cases, the total of child LENGTHUSER values
  equals the parent LENGTHUSER (or the editor-supplied value), subject to the approved
  rounding rule.
- **SC-003**: In 100% of tested cases where LENGTHUSER = 0 or null, child LENGTHUSER
  values are correctly redistributed from Shape_Length.
- **SC-004**: Manual post-split correction effort for LENGTHUSER is reduced by at least
  80% in tested scenarios compared with the current workflow.
- **SC-005**: The task cannot be invoked outside an active edit session.

## Assumptions

- The new ArcFM task is the user entry point; it does not reuse or replace an existing
  split tool. The implementation calls an out-of-the-box split-at-point function
  internally.
- GIS captures only 2-dimensional circuit length; LENGTHUSER and Shape_Length are
  both 2D values.
- Segment Length (onsite installed length) is a separate field that business users
  continue to input manually; it is not in scope for automatic redistribution.
- The enhancement applies to the ElectricLine feature class across all AssetGroups
  and AssetTypes.
- Existing user access and GIS editing permissions remain unchanged.
- The task applies to split behavior only; merge behavior is out of scope.

## Delivery Layer Mapping *(mandatory)*

Document each major item with an explicit layer label so reviewers can separate intent,
design, build work, and verification:

- **Business Requirement**: When an ElectricLine is split, child LENGTHUSER values must
  update automatically from the geometry split ratio; Segment Length remains a manual
  user input.
- **Technical Design**: A new ArcFM task under the Line folder in the Tasks Pane
  presents the GlobalId and a pre-populated, editable User Length field (LENGTHUSER > 0
  → LENGTHUSER; else → Shape_Length). After the editor selects the split point, the
  task derives the ratio from geometry and applies it to LENGTHUSER for each child.
- **Implementation Task**: Build the new split task, wire the dialog fields, implement
  the LENGTHUSER pre-population logic, compute the geometry-based split ratio, and
  apply child LENGTHUSER values. Applies to ElectricLine feature class, all
  AssetGroups and AssetTypes. Task must only execute within an active edit session.
- **Test Consideration**: Verify equal and non-equal geometry splits, LENGTHUSER > 0
  pre-population, Shape_Length fallback when LENGTHUSER = 0 or null, editor-supplied
  override, rounding outcomes (2dp, residual to longer child), all AssetGroups and
  AssetTypes, and non-regression of existing GIS editing behavior.
