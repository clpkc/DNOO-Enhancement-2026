# Feature Specification: Update Substation Task Dropdown List

**Feature Branch**: `004-update-substation-dropdown`

**Created**: 2026-06-08

**Status**: Draft

**Input**: User description: "Create a specification for one DNOO GIS enhancement item only. Requirement ID: 03 - Update substation task drop down list. Business requirement: Enhance the Update Substation ArcFM Task to support SSNAME / SSNUM dropdown updates not only for Substations, but also for: HV PM TX Transformers, Reclosers, HV Switches on HV Poles. Include server-side validation for correct SSNAME / SSNUM pairing from SAP_SUBSTATION_TABLE. Business purpose: Prevent data integrity issues with partnering systems like EWMS. Please produce: business problem, current pain point, target users, desired future behaviour, in-scope / out-of-scope, assumptions, dependencies, constraints, acceptance criteria, validation rules, edge cases / open questions. Scope notes: GIS only. Do not introduce ADMS scope unless it is an explicitly stated dependency. Keep validation and data integrity requirements explicit. Keep the wording aligned with the business requirement above."

## Clarifications

### Session 2026-06-09

- Q: What is the actual staging table name? → A: SAP_SUBSTATION_DATA (not SAP_SUBSTATION_TABLE); populated from a network share file import by the EWMS integration.
- Q: Are all HV Switches in scope? → A: No — only `ElectricDevice.HV Switch.Switch` where `PLACEMENT = Pole-Mounted`.
- Q: How does dropdown auto-fill work? → A: Selecting a substation name auto-fills the number; selecting a number auto-fills the name; Update button commits the selection.
- Q: Is there a naming rule for SSNAME? → A: Yes — SSNAME must be suffixed with "S/S" if the value from EWMS does not already end with "S/S".
- Q: How are used values handled in the dropdown? → A: Previously used substation values are excluded from future dropdown selection, controlled by the IS_USED attribute in SAP_SUBSTATION_DATA.

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
supported asset types, selecting from dropdown values sourced from SAP_SUBSTATION_DATA,
and receive only valid pairings.

**Why this priority**: This is the direct business behavior requested and the main
control needed to prevent invalid GIS data updates.

**Independent Test**: Execute the Update Substation task on each supported asset type,
choose a substation name or number from the dropdown (and confirm the other auto-fills),
then save and verify the SSNAME and SSNUM pair is accepted only when it matches a record
in SAP_SUBSTATION_DATA.

**Acceptance Scenarios**:

1. **Given** a supported asset type, **When** the user selects an SSNAME from the
   dropdown, **Then** the SSNUM auto-fills with the paired value from SAP_SUBSTATION_DATA,
   and the update is saved successfully when the Update button is pressed.
2. **Given** a supported asset type, **When** the user selects an SSNUM from the
   dropdown, **Then** the SSNAME auto-fills with the paired value from SAP_SUBSTATION_DATA.
3. **Given** a supported asset type, **When** the user attempts to save a combination
   where SSNAME and SSNUM do not come from the same record in SAP_SUBSTATION_DATA, **Then**
   the update is rejected by server-side validation.

---

### User Story 2 - Support Additional GIS Asset Types (Priority: P2)

As a GIS editor, I can use the same Update Substation task behavior for HV PM TX
Transformers (`ElectricDevice.Transformer.HV PM TX`), Reclosers
(`ElectricDevice.HV Switch.Recloser`), and Pole-Mounted HV Switches
(`ElectricDevice.HV Switch.Switch` where `PLACEMENT = Pole-Mounted`), not only for
Substations.

**Why this priority**: Expanding supported asset coverage removes inconsistent workflows
and reduces manual workaround across related GIS asset classes.

**Independent Test**: Run the task separately on each newly supported asset type and
verify the dropdown update and validation behavior matches the substation workflow. Also
confirm that HV Switch assets with `PLACEMENT` other than Pole-Mounted are not supported.

**Acceptance Scenarios**:

1. **Given** an HV PM TX Transformer, Recloser, or Pole-Mounted HV Switch, **When** the
   user performs the Update Substation task, **Then** the dropdown update process is
   available and follows the same validation rules as for Substations.
2. **Given** an HV Switch asset with `PLACEMENT` other than Pole-Mounted, **When** the
   user attempts the Update Substation task, **Then** the asset is treated as unsupported
   and the task does not proceed.

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

- SAP_SUBSTATION_DATA returns no matching record for a selected SSNAME or SSNUM.
- SAP_SUBSTATION_DATA contains duplicate or conflicting SSNAME and SSNUM pairings.
- The selected asset already contains one value but not the other.
- The user opens the task for an unsupported asset type (e.g., an HV Switch with
  `PLACEMENT` other than Pole-Mounted).
- The server-side reference table SAP_SUBSTATION_DATA is temporarily unavailable during
  save.
- An SSNAME value from EWMS does not end with "S/S" — the "S/S" suffix must be appended
  before storing.
- All available substation values are marked IS_USED = true; the dropdown presents no
  selectable options.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST allow the Update Substation ArcFM Task to update SSNAME and
  SSNUM for Substations, HV PM TX Transformers (`ElectricDevice.Transformer.HV PM TX`),
  Reclosers (`ElectricDevice.HV Switch.Recloser`), and Pole-Mounted HV Switches
  (`ElectricDevice.HV Switch.Switch` where `PLACEMENT = Pole-Mounted`).
- **FR-002**: System MUST source dropdown values for SSNAME and SSNUM from
  SAP_SUBSTATION_DATA.
- **FR-003**: System MUST auto-fill SSNUM when the user selects an SSNAME from the
  dropdown, and auto-fill SSNAME when the user selects an SSNUM from the dropdown, using
  the paired value from the same SAP_SUBSTATION_DATA record.
- **FR-004**: System MUST exclude substation values where IS_USED = true from the
  dropdown selection list.
- **FR-005**: System MUST validate SSNAME and SSNUM pairing on the server side against
  SAP_SUBSTATION_DATA before saving an update.
- **FR-006**: System MUST reject updates where the SSNAME and SSNUM combination does not
  originate from the same record in SAP_SUBSTATION_DATA.
- **FR-007**: System MUST save updates successfully when the submitted SSNAME and SSNUM
  combination matches a single record in SAP_SUBSTATION_DATA.
- **FR-008**: System MUST apply the same validation behavior consistently across all
  supported asset types.
- **FR-009**: System MUST provide a clear validation failure response when pairing is
  invalid or when reference validation cannot be completed.
- **FR-010**: System MUST keep HV Switch assets where `PLACEMENT` is not Pole-Mounted
  outside the task scope.
- **FR-011**: System MUST keep unrelated GIS task behavior unchanged.
- **FR-012**: System MUST support partnering system data integrity needs such as EWMS
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
Reclosers, and Pole-Mounted HV Switches (`PLACEMENT = Pole-Mounted`) with
dropdown-based SSNAME and SSNUM updates sourced from SAP_SUBSTATION_DATA. Selecting
either field auto-fills the other. Previously used values (IS_USED = true) are excluded
from the dropdown. The server only accepts combinations that originate from the same
SAP_SUBSTATION_DATA record. SSNAME values are stored with an "S/S" suffix appended
where not already present.

### In Scope *(mandatory)*

- Extending Update Substation task support to: Substations, HV PM TX Transformers,
  Reclosers, and Pole-Mounted HV Switches (`PLACEMENT = Pole-Mounted`).
- Dropdown values sourced from SAP_SUBSTATION_DATA with auto-fill behavior (name ↔ number).
- Exclusion of IS_USED = true values from dropdown selection.
- Server-side validation of SSNAME and SSNUM pairings against SAP_SUBSTATION_DATA.
- SSNAME "S/S" suffix rule applied on save.
- Explicit validation feedback for invalid or unverified pairings.
- GIS data integrity protections for downstream usage.

### Out of Scope *(mandatory)*

- ADMS workflow, design, or implementation changes.
- Changes to asset types not listed in the business requirement.
- Changes to partnering system behavior such as EWMS processing logic.
- Broader substation master data governance redesign.

### Business Requirements *(mandatory)*

- **BR-001**: Enhance the Update Substation ArcFM Task to support SSNAME and SSNUM
  dropdown updates for Substations, HV PM TX Transformers, Reclosers, and
  Pole-Mounted HV Switches (`PLACEMENT = Pole-Mounted`).
- **BR-002**: Source dropdown values from SAP_SUBSTATION_DATA with bidirectional
  auto-fill (name selects number; number selects name) and exclude IS_USED = true values.
- **BR-003**: Include server-side validation that SSNAME and SSNUM originate from the
  same SAP_SUBSTATION_DATA record, with SSNAME "S/S" suffix applied where missing.
- **BR-004**: Prevent data integrity issues with partnering systems like EWMS.

### Dependencies *(mandatory)*

- **DEP-001**: SAP_SUBSTATION_DATA staging table is available as the authoritative
  reference for SSNAME and SSNUM pairings; it is populated from a file delivered to a
  well-known network share by the EWMS integration.
- **DEP-002**: The Update Substation ArcFM Task remains the GIS workflow entry point.
- **DEP-003**: Supported GIS asset types expose the required SSNAME and SSNUM fields.
- **DEP-004**: IS_USED attribute in SAP_SUBSTATION_DATA is maintained by the import
  alignment process to reflect used values.
- **DEP-005**: EWMS remains a downstream dependency context only; the staging import
  process is owned by the EWMS integration and is not modified by this enhancement.

### Constraints *(mandatory)*

- **CON-001**: Scope is GIS only.
- **CON-002**: Validation and data integrity requirements must remain explicit.
- **CON-003**: No ADMS scope may be introduced unless explicitly stated as a dependency.
- **CON-004**: The enhancement must not regress existing GIS task behavior for current
  supported use.

### Validation Rules *(mandatory)*

- **VR-001**: A submitted SSNAME and SSNUM pair is valid only if they originate from the
  same record in SAP_SUBSTATION_DATA (SUBSTATION → SSNUM, FL_EN_DESC → SSNAME mapping).
- **VR-002**: A record MUST NOT be saved when SSNAME is provided without a matching
  SSNUM from the same SAP_SUBSTATION_DATA record.
- **VR-003**: A record MUST NOT be saved when SSNUM is provided without a matching
  SSNAME from the same SAP_SUBSTATION_DATA record.
- **VR-004**: SSNAME MUST be stored with an "S/S" suffix; if the EWMS-sourced value does
  not already end with "S/S", the suffix must be appended before saving.
- **VR-005**: Dropdown values where IS_USED = true in SAP_SUBSTATION_DATA MUST NOT be
  presented to the user as selectable options.
- **VR-006**: Validation MUST be performed server-side before the task update is
  committed.
- **VR-007**: When reference validation cannot be completed (e.g., SAP_SUBSTATION_DATA
  unavailable), the update MUST fail safe rather than saving an unverified pairing.

### Key Entities *(include if feature involves data)*

- **Supported GIS Asset**: A Substation, HV PM TX Transformer
  (`ElectricDevice.Transformer.HV PM TX`), Recloser (`ElectricDevice.HV Switch.Recloser`),
  or Pole-Mounted HV Switch (`ElectricDevice.HV Switch.Switch` where
  `PLACEMENT = Pole-Mounted`) that can be updated through the Update Substation task.
- **SAP_SUBSTATION_DATA**: Staging table that holds valid substation records imported
  from the EWMS file feed. Key fields: SUBSTATION (→ SSNUM), FL_EN_DESC (→ SSNAME),
  IS_USED.
- **Substation Pairing**: A valid SSNAME and SSNUM combination derived from a single
  record in SAP_SUBSTATION_DATA.
- **IS_USED**: Attribute on SAP_SUBSTATION_DATA records that marks values already applied
  to the GIS model; IS_USED = true records are excluded from future dropdown selection.
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
- **SC-004**: In 100% of tested cases, dropdown values marked IS_USED = true in
  SAP_SUBSTATION_DATA are not presented as selectable options.
- **SC-005**: In 100% of tested saves, SSNAME values are stored with the "S/S" suffix
  appended where it was not already present.
- **SC-006**: Manual correction of invalid SSNAME and SSNUM pairing errors for supported
  task updates is reduced by at least 80% in tested operational scenarios compared with
  the current workflow.

## Assumptions

- SAP_SUBSTATION_DATA is maintained as the authoritative source for valid SSNAME and
  SSNUM pairings and is kept current by the EWMS file import process.
- The IS_USED attribute in SAP_SUBSTATION_DATA is correctly maintained by the import
  alignment process and accurately reflects which values are already in use.
- Supported asset types already participate in the GIS workflow where the task can be
  applied and expose SSNAME and SSNUM fields.
- Existing user access and permissions for the Update Substation task remain unchanged.
- EWMS consumes the GIS output as a downstream dependency and does not require scope
  expansion in this enhancement.
- The EWMS file feed and network share import process are outside this enhancement's
  scope and remain unchanged.

## Delivery Layer Mapping *(mandatory)*

Document each major item with an explicit layer label so reviewers can separate intent,
design, build work, and verification:

- **Business Requirement**: Supported GIS asset types must allow validated SSNAME and
  SSNUM dropdown updates through the Update Substation task, with IS_USED exclusion and
  SSNAME "S/S" suffix enforcement.
- **Technical Design**: Extend task support for the listed asset types (including
  PLACEMENT = Pole-Mounted constraint for HV Switch); source dropdowns from
  SAP_SUBSTATION_DATA with auto-fill; validate submitted pairs server-side using
  SAP_SUBSTATION_DATA same-record rule; apply "S/S" suffix rule on SSNAME save.
- **Implementation Task**: Add asset coverage, IS_USED filter, auto-fill behavior,
  fail-safe validation, and "S/S" suffix handling without changing unrelated GIS
  workflows.
- **Test Consideration**: Verify valid saves, invalid rejections, IS_USED exclusion,
  SSNAME "S/S" suffix, PLACEMENT filter for HV Switch, auto-fill behavior,
  SAP_SUBSTATION_DATA unavailability handling, and non-regression of existing task
  behavior.
