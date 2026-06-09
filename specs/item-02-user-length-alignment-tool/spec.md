# 02 - User Length Alignment Tool

**Feature Branch**: `item-02-user-length-alignment-tool`

**Created**: 2026-06-08

**Status**: Draft

**Input**: User description: "Create a specification for one DNOO GIS enhancement item only. Requirement ID: 02 - User length alignment tool. Business requirement: A new attribute rule on Electric Line sets User Length (LENGTHUSER) default value equal to the system-calculated shape length (SHAPE_Length) when a new feature is created. Users can still update the value afterward. Business purpose: Oracle WACS adopts user length and this attribute cannot be NULL. Please produce: business problem, current pain point, target users, desired future behaviour, in-scope / out-of-scope, assumptions, dependencies, constraints, acceptance criteria, edge cases / open questions. Scope notes: GIS only. Do not introduce ADMS scope unless it is an explicitly stated dependency. Keep the wording aligned with the business requirement above."

## Clarifications

### Session 2026-06-09

- Q: When exactly does the rule fire? → A: Only when LENGTHUSER is null or ≤ 0 at creation time; if the user supplies a valid value (> 0), the rule does not overwrite it.
- Q: What is the asset scope? → A: ElectricLine feature class, all Asset Groups, all Asset Types.
- Q: Does the rule apply to batch/import creation paths? → A: Yes — the attribute rule fires on all ElectricLine creation events regardless of method.

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

### User Story 1 - Default User Length On Create When Not Already Set (Priority: P1)

As a GIS editor, when I create a new Electric Line feature without supplying a User
Length value (or with a zero/negative value), the User Length is automatically defaulted
to the system-calculated shape length so the attribute is populated without manual entry.

**Why this priority**: This is the core business requirement and directly addresses the
need for a non-null User Length value on creation.

**Independent Test**: Create a new Electric Line feature without entering a User Length
value and verify that LENGTHUSER is automatically set to SHAPE_Length at save. Also
create a feature with LENGTHUSER explicitly set to a value > 0 and verify the
user-supplied value is preserved without being overwritten.

**Acceptance Scenarios**:

1. **Given** a user creates a new Electric Line feature with no LENGTHUSER supplied,
   **When** the feature is saved, **Then** LENGTHUSER is defaulted to SHAPE_Length.
2. **Given** a new Electric Line feature is created with LENGTHUSER = 0 or null,
   **When** the feature is saved, **Then** the record is saved with a non-null
   LENGTHUSER equal to SHAPE_Length.
3. **Given** a new Electric Line feature is created with a user-supplied LENGTHUSER > 0,
   **When** the feature is saved, **Then** the user-supplied LENGTHUSER value is
   preserved and the rule does not overwrite it.

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

- The created Electric Line has a zero or negative shape length — LENGTHUSER would be
  defaulted to the same value; downstream NULL prevention is satisfied but zero-length
  records should be validated separately.
- The created Electric Line geometry is invalid or cannot calculate shape length at save
  time — behavior when SHAPE_Length is unavailable must be defined.
- A user supplies a LENGTHUSER value > 0 during creation — the rule MUST NOT overwrite
  it; the user-supplied value is preserved.
- A user supplies a LENGTHUSER value of 0 or null during creation — the rule fires and
  defaults to SHAPE_Length.
- An existing Electric Line record is edited after creation — the rule does not apply;
  LENGTHUSER retains its current value.
- Bulk creation or import creates multiple Electric Line features in one operation — the
  attribute rule fires for each created feature regardless of creation method.

## Requirements *(mandatory)*

<!--
  ACTION REQUIRED: The content in this section represents placeholders.
  Fill them out with the right functional requirements.
-->

### Functional Requirements

- **FR-001**: When a new ElectricLine feature is created, if LENGTHUSER is null or ≤ 0,
  the system MUST automatically set LENGTHUSER to SHAPE_Length.
- **FR-002**: When a new ElectricLine feature is created with a user-supplied LENGTHUSER
  value > 0, the system MUST preserve that value and MUST NOT overwrite it.
- **FR-003**: System MUST ensure LENGTHUSER is not null after creation of any new
  ElectricLine feature through any creation method.
- **FR-004**: System MUST allow users to update LENGTHUSER after the feature has been
  created.
- **FR-005**: System MUST NOT apply the defaulting rule during edits to existing
  ElectricLine features.
- **FR-006**: System MUST apply the defaulting behavior to ElectricLine, all Asset
  Groups, all Asset Types.
- **FR-007**: System MUST apply the rule consistently regardless of creation method
  (interactive, batch, or import).
- **FR-008**: System MUST keep unrelated GIS editing behavior unchanged.
- **FR-009**: System MUST support downstream GIS data usage where Oracle WACS adopts
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

When a new ElectricLine feature (any Asset Group, any Asset Type) is created with a null
or zero/negative LENGTHUSER, the attribute rule automatically sets LENGTHUSER to
SHAPE_Length. If the user supplies a valid LENGTHUSER > 0 at creation, it is preserved
unchanged. Users can still update LENGTHUSER after creation for business reasons. The
rule does not apply during edits to existing features.

### In Scope *(mandatory)*

- Attribute rule on ElectricLine feature class (all Asset Groups, all Asset Types) that
  sets LENGTHUSER = SHAPE_Length when LENGTHUSER is null or ≤ 0 at creation time.
- Preservation of user-supplied LENGTHUSER > 0 at creation (rule does not fire).
- Applies to all creation methods (interactive, batch, import).
- Preserving user ability to update LENGTHUSER after creation.
- GIS data completeness for the ElectricLine creation workflow.

### Out of Scope *(mandatory)*

- ADMS workflow, design, or implementation changes.
- Recalculation of LENGTHUSER for existing Electric Line records unless they are newly
  created.
- Changes to non-Electric Line feature classes.
- Broader downstream system redesign.

### Business Requirements *(mandatory)*

- **BR-001**: Apply an attribute rule to ElectricLine (all Asset Groups, all Asset Types)
  that sets LENGTHUSER = SHAPE_Length when LENGTHUSER is null or ≤ 0 at creation time.
- **BR-002**: Preserve a user-supplied LENGTHUSER > 0 at creation; rule must not
  overwrite it.
- **BR-003**: Ensure LENGTHUSER is not null after creation of any new ElectricLine
  feature.
- **BR-004**: Preserve the user's ability to update LENGTHUSER after creation.

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

- **ElectricLine**: GIS feature class (all Asset Groups, all Asset Types) subject to the
  creation-time attribute rule. Contains SHAPE_Length and LENGTHUSER attributes.
- **LENGTHUSER**: User Length attribute required to be non-null on newly created
  ElectricLine features; downstream systems such as Oracle WACS adopt this value.
- **SHAPE_Length**: System-calculated shape length used as the default source value when
  LENGTHUSER is null or ≤ 0 at creation time.

## Success Criteria *(mandatory)*

<!--
  ACTION REQUIRED: Define measurable success criteria.
  These must be technology-agnostic and measurable.
-->

### Measurable Outcomes

- **SC-001**: In 100% of tested new ElectricLine creation cases where no LENGTHUSER is
  supplied (or LENGTHUSER is null or ≤ 0), LENGTHUSER is populated with SHAPE_Length
  when the record is saved.
- **SC-002**: In 100% of tested new ElectricLine creation cases where LENGTHUSER > 0 is
  supplied by the user, the user-supplied value is preserved unchanged.
- **SC-003**: In 100% of tested new ElectricLine creation cases across all Asset Groups
  and Asset Types, LENGTHUSER is non-null after save.
- **SC-004**: In 100% of tested post-creation edits, a user-updated LENGTHUSER value is
  retained without being reset.
- **SC-005**: Manual correction of missing LENGTHUSER values for newly created
  ElectricLine features is reduced by at least 90% compared with the current workflow.

## Assumptions

- SHAPE_Length is available and system-calculated during new ElectricLine feature
  creation before the attribute rule fires.
- A LENGTHUSER value of 0 or negative is treated equivalently to null for the purpose of
  the defaulting rule.
- Users may legitimately adjust LENGTHUSER after creation for business reasons; the rule
  does not re-fire on subsequent edits.
- Existing permissions for creating and editing ElectricLine features remain unchanged.
- Oracle WACS consumes User Length as a downstream dependency and does not require scope
  expansion in this enhancement.
- Updating LENGTHUSER on existing ElectricLine records is explicitly out of scope.

## Delivery Layer Mapping *(mandatory)*

Document each major item with an explicit layer label so reviewers can separate intent,
design, build work, and verification:

- **Business Requirement**: New ElectricLine features (all Asset Groups, all Asset Types)
  must have LENGTHUSER populated as non-null; if not supplied or invalid at creation, it
  must default to SHAPE_Length; user-supplied values > 0 must be preserved.
- **Technical Design**: Attribute rule on ElectricLine creation: if LENGTHUSER is null
  or ≤ 0, set LENGTHUSER = SHAPE_Length; otherwise preserve existing value. Rule does
  not fire on edits.
- **Implementation Task**: Add ElectricLine creation attribute rule with conditional
  logic; verify it does not fire on edits or overwrite valid user-supplied values.
- **Test Consideration**: Validate null/zero defaulting, valid user-value preservation,
  no-edit-trigger behavior, all Asset Groups/Types coverage, batch/import path coverage,
  and non-regression of existing GIS edit flow.
