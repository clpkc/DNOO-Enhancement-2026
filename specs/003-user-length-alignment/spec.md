# Feature Specification: User Length Alignment Tool

**Feature Branch**: `003-user-length-tool`

**Created**: 2026-06-08

**Status**: Draft

**Input**: User description: "Create a specification for one DNOO GIS enhancement item only. Requirement ID: 02 - User length alignment tool. Business requirement: A new attribute rule on Electric Line sets User Length (LENGTHUSER) default value equal to the system-calculated shape length (SHAPE_Length) when a new feature is created. Users can still update the value afterward. Business purpose: Oracle WACS adopts user length and this attribute cannot be NULL. Please produce: business problem, current pain point, target users, desired future behaviour, in-scope / out-of-scope, assumptions, dependencies, constraints, acceptance criteria, edge cases / open questions. Scope notes: GIS only. Do not introduce ADMS scope unless it is an explicitly stated dependency. Keep the wording aligned with the business requirement above."

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

### User Story 1 - Default User Length On Create (Priority: P1)

As a GIS editor, when I create a new Electric Line feature, the User Length value is
defaulted to the system-calculated shape length so that the attribute is populated
without manual entry.

**Why this priority**: This is the core business requirement and directly addresses the
need for a non-null User Length value on creation.

**Independent Test**: Create a new Electric Line feature and verify that User Length is
automatically set to the same value as the system-calculated shape length at the time of
creation.

**Acceptance Scenarios**:

1. **Given** a user creates a new Electric Line feature, **When** the feature is saved,
  **Then** LENGTHUSER is defaulted to SHAPE_Length.
2. **Given** a new Electric Line feature is created, **When** no manual User Length is
  entered, **Then** the record is still saved with a non-null LENGTHUSER value.

---

### User Story 2 - Allow User Update After Defaulting (Priority: P2)

As a GIS editor, I can still update the User Length value after feature creation when a
business adjustment is required.

**Why this priority**: The requirement explicitly preserves user control after the
default is applied.

**Independent Test**: Create a new Electric Line feature, confirm the default value is
applied, then update User Length and verify the user-entered value is retained.

**Acceptance Scenarios**:

1. **Given** LENGTHUSER has been defaulted from SHAPE_Length, **When** the user edits
  LENGTHUSER afterward, **Then** the updated value is saved and not overwritten by the
  defaulting rule during that edit.

---

### User Story 3 - Protect Downstream GIS Data Usage (Priority: P3)

As a GIS data owner, I need newly created Electric Line records to have a populated User
Length value so downstream consumers such as Oracle WACS receive usable data.

**Why this priority**: Data completeness for downstream GIS usage is the business reason
for the enhancement and must remain reliable.

**Independent Test**: Create new Electric Line records through the standard workflow and
verify that the resulting records always contain a non-null User Length value.

**Acceptance Scenarios**:

1. **Given** a new Electric Line feature is created through the normal GIS workflow,
   **When** the record is available for downstream use, **Then** LENGTHUSER is populated
   and available for systems that adopt user length.

---

### Edge Cases

<!--
  ACTION REQUIRED: The content in this section represents placeholders.
  Fill them out with the right edge cases.
-->

- The created Electric Line has a zero shape length.
- The created Electric Line geometry is invalid or cannot calculate shape length at save
  time.
- A user supplies a User Length value during creation and expects it to remain unchanged.
- An existing Electric Line record is edited after creation.
- Bulk creation or import creates multiple Electric Line features in one operation.
- Open question: Should the defaulting rule apply only to interactive creation, or also
  to batch-created Electric Line features that use the same creation path?

## Requirements *(mandatory)*

<!--
  ACTION REQUIRED: The content in this section represents placeholders.
  Fill them out with the right functional requirements.
-->

### Functional Requirements

- **FR-001**: System MUST set User Length (LENGTHUSER) to the system-calculated shape
  length (SHAPE_Length) when a new Electric Line feature is created.
- **FR-002**: System MUST ensure LENGTHUSER is not null after creation of a new Electric
  Line feature through the supported GIS creation workflow.
- **FR-003**: System MUST allow users to update LENGTHUSER after the feature has been
  created.
- **FR-004**: System MUST NOT overwrite a user-updated LENGTHUSER value during later
  edits unless the feature is being created anew.
- **FR-005**: System MUST apply the defaulting behavior only to Electric Line features.
- **FR-006**: System MUST keep unrelated GIS editing behavior unchanged.
- **FR-007**: System MUST support downstream GIS data usage where Oracle WACS adopts
  user length, while keeping Oracle WACS as a dependency context rather than added scope.

### Business Problem *(mandatory)*

New Electric Line features can be created without a populated User Length value even
though downstream GIS data usage requires that attribute to be available.

### Current Pain Point *(mandatory)*

Users must notice and correct missing User Length values manually after creation, which
creates extra effort and risks incomplete GIS data reaching downstream use.

### Target Users *(mandatory)*

- GIS editors creating new Electric Line features.
- GIS data owners responsible for completeness of Electric Line attributes.
- Operational users who rely on downstream systems adopting user length.

### Desired Future Behaviour *(mandatory)*

When a new Electric Line is created, LENGTHUSER is automatically defaulted to
SHAPE_Length so the field is populated immediately, while still allowing users to update
the value later if needed.

### In Scope *(mandatory)*

- Defaulting LENGTHUSER from SHAPE_Length during creation of new Electric Line features.
- Preserving user ability to update LENGTHUSER after creation.
- GIS data completeness for the Electric Line creation workflow.

### Out of Scope *(mandatory)*

- ADMS workflow, design, or implementation changes.
- Recalculation of LENGTHUSER for existing Electric Line records unless they are newly
  created.
- Changes to non-Electric Line feature classes.
- Broader downstream system redesign.

### Business Requirements *(mandatory)*

- **BR-001**: Set User Length default value equal to SHAPE_Length when a new Electric
  Line feature is created.
- **BR-002**: Ensure the User Length attribute is populated so it is not null for new
  Electric Line records.
- **BR-003**: Preserve the user's ability to update the value afterward.

### Dependencies *(mandatory)*

- **DEP-001**: Electric Line feature class supports LENGTHUSER and SHAPE_Length fields.
- **DEP-002**: Standard GIS feature creation workflow where the attribute rule can run.
- **DEP-003**: Oracle WACS dependency on user length as downstream context only.

### Constraints *(mandatory)*

- **CON-001**: Scope is GIS only.
- **CON-002**: Wording must remain aligned with the stated business requirement.
- **CON-003**: No ADMS scope may be introduced unless explicitly stated as a dependency.
- **CON-004**: The enhancement must not regress current GIS creation and edit workflows.

### Key Entities *(include if feature involves data)*

- **Electric Line**: GIS feature created by users, containing SHAPE_Length and
  LENGTHUSER attributes.
- **LENGTHUSER**: User Length attribute used downstream and required to be non-null on
  newly created Electric Line features.
- **SHAPE_Length**: System-calculated shape length used as the default source value at
  feature creation.

## Success Criteria *(mandatory)*

<!--
  ACTION REQUIRED: Define measurable success criteria.
  These must be technology-agnostic and measurable.
-->

### Measurable Outcomes

- **SC-001**: In 100% of tested new Electric Line creation cases, LENGTHUSER is
  populated when the record is saved.
- **SC-002**: In 100% of tested new Electric Line creation cases, the initial
  LENGTHUSER value matches SHAPE_Length at the time of creation.
- **SC-003**: In 100% of tested post-creation edits, a user-updated LENGTHUSER value is
  retained.
- **SC-004**: Manual correction of missing LENGTHUSER values for newly created Electric
  Line features is reduced by at least 90% compared with the current workflow.

## Assumptions

<!--
  ACTION REQUIRED: The content in this section represents placeholders.
  Fill them out with the right assumptions based on reasonable defaults
  chosen when the feature description did not specify certain details.
-->

- SHAPE_Length is available and system-calculated during new Electric Line feature
  creation.
- Users may legitimately adjust LENGTHUSER after creation for business reasons.
- Existing permissions for creating and editing Electric Line features remain unchanged.
- Oracle WACS consumes User Length as a downstream dependency and does not require scope
  expansion in this enhancement.

## Delivery Layer Mapping *(mandatory)*

Document each major item with an explicit layer label so reviewers can separate intent,
design, build work, and verification:

- **Business Requirement**: New Electric Line features must default LENGTHUSER to
  SHAPE_Length so the field is populated and not null.
- **Technical Design**: Apply a creation-time defaulting rule for Electric Line that
  copies SHAPE_Length into LENGTHUSER while allowing later user updates.
- **Implementation Task**: Add the Electric Line creation rule and verify it does not
  overwrite user changes after creation.
- **Test Consideration**: Validate new feature creation, null prevention, later user
  updates, and non-regression of the existing GIS edit flow.
