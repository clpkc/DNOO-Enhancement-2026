# Feature Specification: Electric Line Splitting Tool

**Feature Branch**: `002-electric-line-split`

**Created**: 2026-06-08

**Status**: Draft

**Input**: User description: "Create a specification for one DNOO GIS enhancement item only. Requirement ID: 01 - Electric line splitting tool. Business requirement: The Split Line ArcFM task allows users to select a line feature, optionally input User Length, and split it at a chosen point. It automatically calculates User Length for child features based on the parent’s length ratio. Segment length is also updated proportionally. Example: parent line A = 100; split into B = 50 and C = 50; if segment length = 120, each child becomes 60. Business purpose: Address the need for automatic segment length updates without manual input for split or merged circuits. Please produce: business problem, current pain point, target users, desired future behaviour, in-scope / out-of-scope, assumptions, dependencies, constraints, acceptance criteria, edge cases / open questions. Scope notes: GIS only. Do not introduce ADMS scope unless it is an explicitly stated dependency. Keep the wording aligned with the existing business requirement above."

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

### User Story 1 - Split Electric Line With Automatic Length Redistribution (Priority: P1)

As a GIS editor, I split an electric line at a chosen point and the system
automatically recalculates User Length and Segment Length for the child features in
proportion to the parent line.

**Why this priority**: This is the core business outcome and directly removes manual
recalculation effort from the current split workflow.

**Independent Test**: Select a parent electric line with known User Length and Segment
Length, split it into two child features, and verify that both child values are updated
proportionally based on the split ratio.

**Acceptance Scenarios**:

1. **Given** a parent electric line with User Length 100 and Segment Length 120,
  **When** the user splits the line into two equal child lines, **Then** each child
  receives User Length 50 and Segment Length 60.
2. **Given** a parent electric line with stored length attributes, **When** the user
  splits it at a non-equal point, **Then** the child features inherit User Length and
  Segment Length values in the same proportion as their resulting line lengths.

---

### User Story 2 - Support Optional User Length Input During Split (Priority: P2)

As a GIS editor, I can optionally input User Length when performing the split and have
the child features recalculate from that parent value instead of requiring manual child
entry.

**Why this priority**: This preserves the existing task pattern while ensuring the new
automation still works for lines where the parent User Length is entered at split time.

**Independent Test**: Start a split on a line where the user provides a parent User
Length value during the task and verify the child attributes are calculated from that
value.

**Acceptance Scenarios**:

1. **Given** a line selected for split and no stored User Length is available,
   **When** the user enters a parent User Length in the task, **Then** the child
   features are assigned recalculated User Length values based on the parent length
   ratio.

---

### User Story 3 - Preserve Existing GIS Editing Workflow (Priority: P3)

As a GIS operations lead, I need the electric line splitting tool to fit the existing
GIS editing workflow without introducing regression to line editing outcomes.

**Why this priority**: The enhancement is only valuable if it improves efficiency
without disrupting current GIS editing operations.

**Independent Test**: Execute the existing split-line editing workflow with and without
optional User Length input and verify the workflow remains usable while length values
are updated automatically.

**Acceptance Scenarios**:

1. **Given** an editor uses the existing split-line task flow, **When** the split is
  completed, **Then** the edit succeeds without requiring manual child length entry and
  without changing unrelated GIS editing behavior.

---

### Edge Cases

<!--
  ACTION REQUIRED: The content in this section represents placeholders.
  Fill them out with the right edge cases.
-->

- The parent line has User Length but no Segment Length.
- The parent line has Segment Length but no stored User Length and the user does not
  provide one during the task.
- The split creates child lengths that result in decimal ratios and rounding is needed.
- The selected line is too short or the split point is too close to an endpoint.
- The parent line has zero or null length attributes.
- Open question: If merged circuit behavior is still required, should it be handled as a
  separate GIS enhancement item because the stated business requirement describes
  split-line behavior only?

## Requirements *(mandatory)*

<!--
  ACTION REQUIRED: The content in this section represents placeholders.
  Fill them out with the right functional requirements.
-->

### Functional Requirements

- **FR-001**: System MUST allow the user to select an electric line feature and split it
  at a chosen point within the existing Split Line ArcFM task.
- **FR-002**: System MUST support optional parent User Length input during the split
  task.
- **FR-003**: System MUST calculate User Length for each child feature according to the
  child line length ratio compared with the parent line.
- **FR-004**: System MUST update Segment Length for each child feature according to the
  same proportional ratio used for the split.
- **FR-005**: System MUST preserve the total parent User Length and total parent Segment
  Length across the created child features, subject to approved rounding rules.
- **FR-006**: System MUST complete the split without requiring manual entry of child
  User Length or child Segment Length values.
- **FR-007**: System MUST leave unrelated GIS editing behavior unchanged.
- **FR-008**: System MUST use geometry length to determine the split ratio and then
  apply that ratio to recalculate User Length and Segment Length for the child
  features.
- **FR-009**: System MUST round child User Length and child Segment Length values to 2
  decimal places and assign any rounding residual to the longer child feature so that
  the child totals match the parent total.

### Business Requirements *(mandatory)*

- **BR-001**: Address the need for automatic Segment Length updates without manual input
  when an electric line is split.
- **BR-002**: Reduce manual effort and data-entry risk in the Split Line ArcFM task.
- **BR-003**: Keep the business wording and user behavior aligned to the existing split
  line requirement.

### Business Problem *(mandatory)*

When electric lines are split, child Segment Length values currently require manual
input or correction, which creates extra effort and increases the chance of incorrect
GIS attribute values.

### Current Pain Point *(mandatory)*

Users must manually determine or correct child length-related values after a split,
especially where Segment Length must be redistributed from the parent line. This slows
editing work and can introduce inconsistent data.

### Target Users *(mandatory)*

- GIS editors performing line split operations in ArcFM.
- GIS operations staff responsible for reviewing edit outcomes and data quality.

### Desired Future Behaviour *(mandatory)*

The Split Line ArcFM task recalculates child User Length and child Segment Length
automatically from the parent line and split ratio, so the user completes the split
without manually entering child values.

### In Scope *(mandatory)*

- Electric line split behavior in the existing Split Line ArcFM task.
- Proportional redistribution of User Length and Segment Length from parent to child
  features.
- Optional parent User Length entry when performing the split.
- GIS workflow outcomes only.

### Out of Scope *(mandatory)*

- ADMS workflow, design, or implementation changes.
- Broader electric circuit merge automation unless explicitly approved as a separate GIS
  item.
- Changes to unrelated GIS editing tools.

### Dependencies *(mandatory)*

- **DEP-001**: Existing Split Line ArcFM task behavior and GIS editing environment.
- **DEP-002**: Availability of parent line User Length and Segment Length attributes in
  the GIS data model or task input.
- **DEP-003**: Existing GIS data rules for electric line feature editing.
- **DEP-004**: Any ADMS reference remains dependency-only and must not alter ADMS scope.

### Constraints *(mandatory)*

- **CON-001**: Scope is GIS only.
- **CON-002**: Wording must remain aligned with the existing business requirement for
  split-line behavior.
- **CON-003**: The enhancement must not create regression in the current GIS split-line
  editing workflow.
- **CON-004**: No ADMS scope may be introduced unless explicitly stated as a dependency.

### Key Entities *(include if feature involves data)*

- **Parent Electric Line**: The original line feature selected for splitting, including
  its existing User Length and Segment Length values.
- **Child Electric Line**: Each resulting line feature created by the split, with
  recalculated User Length and Segment Length values.
- **Split Ratio**: The proportional relationship between each child line length and the
  parent line length used to redistribute attribute values.

## Success Criteria *(mandatory)*

<!--
  ACTION REQUIRED: Define measurable success criteria.
  These must be technology-agnostic and measurable.
-->

### Measurable Outcomes

- **SC-001**: Users can complete a standard electric line split without manual child
  User Length or Segment Length entry in 95% of tested cases.
- **SC-002**: In 100% of tested split cases, the total of child User Length values
  equals the parent User Length, subject to the approved rounding rule.
- **SC-003**: In 100% of tested split cases, the total of child Segment Length values
  equals the parent Segment Length, subject to the approved rounding rule.
- **SC-004**: Manual post-split correction effort for length-related attributes is
  reduced by at least 80% in tested split scenarios compared with the current workflow.

## Assumptions

<!--
  ACTION REQUIRED: The content in this section represents placeholders.
  Fill them out with the right assumptions based on reasonable defaults
  chosen when the feature description did not specify certain details.
-->

- The existing Split Line ArcFM task remains the user entry point for this enhancement.
- Parent electric lines already contain, or can accept, the required length-related
  attributes used for redistribution.
- The enhancement applies to split behavior only unless merge behavior is separately
  approved.
- Existing user access and GIS editing permissions remain unchanged.

## Delivery Layer Mapping *(mandatory)*

Document each major item with an explicit layer label so reviewers can separate intent,
design, build work, and verification:

- **Business Requirement**: When an electric line is split, child User Length and child
  Segment Length must update automatically without manual child input.
- **Technical Design**: Use the parent line values and the resulting split ratio to
  redistribute User Length and Segment Length to child features.
- **Implementation Task**: Update the split-line behavior so length-related child values
  are calculated and applied during the existing ArcFM task flow.
- **Test Consideration**: Verify equal and non-equal splits, optional parent User Length
  entry, rounding outcomes, and non-regression of the current GIS editing workflow.
