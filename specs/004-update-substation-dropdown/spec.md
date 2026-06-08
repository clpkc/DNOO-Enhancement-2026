# Feature Specification: Update Substation Task Dropdown List

**Feature Branch**: `004-update-substation-dropdown`

**Created**: 2026-06-08

**Status**: Draft

**Input**: User description: "Create a specification for one DNOO GIS enhancement item only. Requirement ID: 03 - Update substation task drop down list. Business requirement: Enhance the Update Substation ArcFM Task to support SSNAME / SSNUM dropdown updates not only for Substations, but also for: HV PM TX Transformers, Reclosers, HV Switches on HV Poles. Include server-side validation for correct SSNAME / SSNUM pairing from SAP_SUBSTATION_TABLE. Business purpose: Prevent data integrity issues with partnering systems like EWMS. Please produce: business problem, current pain point, target users, desired future behaviour, in-scope / out-of-scope, assumptions, dependencies, constraints, acceptance criteria, validation rules, edge cases / open questions. Scope notes: GIS only. Do not introduce ADMS scope unless it is an explicitly stated dependency. Keep validation and data integrity requirements explicit. Keep the wording aligned with the business requirement above."

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

### User Story 1 - Update Supported Asset Types With Valid Substation Pairing (Priority: P1)

As a GIS editor, I use the Update Substation ArcFM Task to update SSNAME and SSNUM for
supported asset types and receive only valid pairings.

**Why this priority**: This is the direct business behavior requested and the main
control needed to prevent invalid GIS data updates.

**Independent Test**: Execute the Update Substation task on each supported asset type,
choose a substation dropdown value, and verify that the corresponding SSNAME and SSNUM
pair is accepted only when it matches the server-side reference table.

**Acceptance Scenarios**:

1. **Given** a supported asset type with an editable substation assignment, **When** the
  user updates the dropdown selection using a valid SSNAME and SSNUM pair, **Then** the
  change is saved successfully.
2. **Given** a supported asset type, **When** the user attempts to save an invalid
  SSNAME and SSNUM combination, **Then** the update is rejected by server-side
  validation.

---

### User Story 2 - Support Additional GIS Asset Types (Priority: P2)

As a GIS editor, I can use the same Update Substation task behavior for HV PM TX
Transformers, Reclosers, and HV Switches on HV Poles, not only for Substations.

**Why this priority**: Expanding supported asset coverage removes inconsistent workflows
and reduces manual workaround across related GIS asset classes.

**Independent Test**: Run the task separately on each newly supported asset type and
verify the dropdown update and validation behavior matches the substation workflow.

**Acceptance Scenarios**:

1. **Given** an HV PM TX Transformer, Recloser, or HV Switch on HV Pole, **When** the
  user performs the Update Substation task, **Then** the dropdown update process is
  available and follows the same validation rules as for Substations.

---

### User Story 3 - Protect Partnering System Data Integrity (Priority: P3)

As a GIS data owner, I need substation attribute updates to remain consistent so
partnering systems such as EWMS do not receive mismatched SSNAME and SSNUM values.

**Why this priority**: The business purpose is to prevent downstream data integrity
issues, so protection of partnered GIS data usage must be explicit.

**Independent Test**: Save valid and invalid substation pair updates and verify that
only valid records are persisted for downstream consumption.

**Acceptance Scenarios**:

1. **Given** a record updated through the task, **When** downstream data is reviewed,
   **Then** saved SSNAME and SSNUM values always match the approved pairing from
   SAP_SUBSTATION_TABLE.

---

### Edge Cases

- SAP_SUBSTATION_TABLE returns no matching record for a selected SSNAME or SSNUM.
- SAP_SUBSTATION_TABLE contains duplicate or conflicting SSNAME and SSNUM pairings.
- The selected asset already contains one value but not the other.
- The user opens the task for an unsupported asset type.
- The server-side reference table is temporarily unavailable during save.
- Open question: Should the dropdown display both SSNAME and SSNUM together to reduce
  user selection error, or is the current display format sufficient if validation blocks
  invalid combinations?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST allow the Update Substation ArcFM Task to update SSNAME and
  SSNUM for Substations, HV PM TX Transformers, Reclosers, and HV Switches on HV Poles.
- **FR-002**: System MUST validate SSNAME and SSNUM pairing on the server side against
  SAP_SUBSTATION_TABLE before saving an update.
- **FR-003**: System MUST reject updates where the SSNAME and SSNUM combination does not
  exist as an approved pair in SAP_SUBSTATION_TABLE.
- **FR-004**: System MUST save updates successfully when the submitted SSNAME and SSNUM
  combination matches SAP_SUBSTATION_TABLE.
- **FR-005**: System MUST apply the same validation behavior consistently across all
  supported asset types.
- **FR-006**: System MUST provide a clear validation failure response when pairing is
  invalid or when reference validation cannot be completed.
- **FR-007**: System MUST keep unsupported asset types outside the task scope.
- **FR-008**: System MUST keep unrelated GIS task behavior unchanged.
- **FR-009**: System MUST support partnering system data integrity needs such as EWMS
  without expanding scope into partnering system workflow or design.

### Business Problem *(mandatory)*

The Update Substation ArcFM Task currently does not provide the same controlled dropdown
update behavior across all required GIS asset types, and incorrect SSNAME and SSNUM
pairings can create invalid data for downstream use.

### Current Pain Point *(mandatory)*

Users must manage substation attribute updates across multiple asset types without a
consistent validated workflow, increasing the risk of mismatched SSNAME and SSNUM values
and subsequent data integrity issues.

### Target Users *(mandatory)*

- GIS editors updating substation-related attributes.
- GIS data owners responsible for substation reference data quality.
- Operational stakeholders relying on downstream systems such as EWMS.

### Desired Future Behaviour *(mandatory)*

The Update Substation ArcFM Task supports Substations, HV PM TX Transformers,
Reclosers, and HV Switches on HV Poles with dropdown-based SSNAME and SSNUM updates,
and the server only accepts combinations that are valid in SAP_SUBSTATION_TABLE.

### In Scope *(mandatory)*

- Extending Update Substation task support to the listed GIS asset types.
- Server-side validation of SSNAME and SSNUM pairings against SAP_SUBSTATION_TABLE.
- Explicit validation feedback for invalid or unverified pairings.
- GIS data integrity protections for downstream usage.

### Out of Scope *(mandatory)*

- ADMS workflow, design, or implementation changes.
- Changes to asset types not listed in the business requirement.
- Changes to partnering system behavior such as EWMS processing logic.
- Broader substation master data governance redesign.

### Business Requirements *(mandatory)*

- **BR-001**: Enhance the Update Substation ArcFM Task to support SSNAME and SSNUM
  dropdown updates for Substations, HV PM TX Transformers, Reclosers, and HV Switches
  on HV Poles.
- **BR-002**: Include server-side validation for correct SSNAME and SSNUM pairing from
  SAP_SUBSTATION_TABLE.
- **BR-003**: Prevent data integrity issues with partnering systems like EWMS.

### Dependencies *(mandatory)*

- **DEP-001**: SAP_SUBSTATION_TABLE is available as the authoritative reference for
  SSNAME and SSNUM pairings.
- **DEP-002**: The Update Substation ArcFM Task remains the GIS workflow entry point.
- **DEP-003**: Supported GIS asset types expose the required SSNAME and SSNUM fields.
- **DEP-004**: EWMS remains a downstream dependency context only.

### Constraints *(mandatory)*

- **CON-001**: Scope is GIS only.
- **CON-002**: Validation and data integrity requirements must remain explicit.
- **CON-003**: No ADMS scope may be introduced unless explicitly stated as a dependency.
- **CON-004**: The enhancement must not regress existing GIS task behavior for current
  supported use.

### Validation Rules *(mandatory)*

- **VR-001**: A submitted SSNAME and SSNUM pair is valid only if the exact pairing exists
  in SAP_SUBSTATION_TABLE.
- **VR-002**: A record MUST NOT be saved when SSNAME is provided without a matching
  SSNUM pair in SAP_SUBSTATION_TABLE.
- **VR-003**: A record MUST NOT be saved when SSNUM is provided without a matching
  SSNAME pair in SAP_SUBSTATION_TABLE.
- **VR-004**: Validation failures MUST be returned from the server side before the task
  update is committed.
- **VR-005**: When reference validation cannot be completed, the update MUST fail safe
  rather than saving an unverified pairing.

### Key Entities *(include if feature involves data)*

- **Supported GIS Asset**: A Substation, HV PM TX Transformer, Recloser, or HV Switch
  on HV Pole that can be updated through the Update Substation task.
- **Substation Pairing**: A valid SSNAME and SSNUM combination defined in
  SAP_SUBSTATION_TABLE.
- **Validation Result**: The server-side outcome indicating whether the submitted
  pairing can be saved.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In 100% of tested valid update cases across all supported asset types,
  SSNAME and SSNUM updates are saved successfully.
- **SC-002**: In 100% of tested invalid pairing cases, the update is rejected before
  save.
- **SC-003**: In 100% of tested saved records updated through this task, SSNAME and
  SSNUM match a valid pairing from SAP_SUBSTATION_TABLE.
- **SC-004**: Manual correction of invalid SSNAME and SSNUM pairing errors for supported
  task updates is reduced by at least 80% in tested operational scenarios compared with
  the current workflow.

## Assumptions

- SAP_SUBSTATION_TABLE is maintained as the authoritative source for valid SSNAME and
  SSNUM pairings.
- Supported asset types already participate in the GIS workflow where the task can be
  applied.
- Existing user access and permissions for the Update Substation task remain unchanged.
- EWMS consumes the GIS output as a downstream dependency and does not require scope
  expansion in this enhancement.

## Delivery Layer Mapping *(mandatory)*

Document each major item with an explicit layer label so reviewers can separate intent,
design, build work, and verification:

- **Business Requirement**: Supported GIS asset types must allow validated SSNAME and
  SSNUM dropdown updates through the Update Substation task.
- **Technical Design**: Extend task support for the listed asset types and validate
  submitted SSNAME and SSNUM pairs on the server side using SAP_SUBSTATION_TABLE.
- **Implementation Task**: Add asset coverage and fail-safe validation behavior without
  changing unrelated GIS workflows.
- **Test Consideration**: Verify valid saves, invalid rejections, reference-table
  unavailability handling, and non-regression of existing task behavior.
