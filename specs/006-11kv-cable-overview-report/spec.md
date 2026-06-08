# Feature Specification: 11kV Cable Overview Report Generation

**Feature Branch**: `006-create-11kv-cable-report`

**Created**: 2026-06-08

**Status**: Draft

**Input**: User description: "Create a specification for one DNOO GIS enhancement item only. Requirement ID: 09 - 11kV cable overview report generation. Business requirement: Create a new monthly reporting interface exporting all 11kV cable sections with length, material, cable size, insulation, commissioning data, substation connectivity, joint locations (latitude / longitude), network topology. Business purpose: Support the 11kV cable joint replacement initiative with detailed distribution data for planning and government reporting. Please produce: business problem, current pain point, target users / report consumers, desired future behaviour, reporting frequency, in-scope / out-of-scope, assumptions, dependencies, constraints, acceptance criteria, data field expectations, edge cases / open questions. Scope notes: GIS only. Do not introduce ADMS scope unless it is an explicitly stated dependency. Keep the wording aligned with the business requirement above."

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

### User Story 1 - Generate Monthly 11kV Cable Export (Priority: P1)

As a GIS reporting user, I generate a monthly report export containing all 11kV cable
sections with the required planning and reporting fields.

**Why this priority**: This is the core delivery objective and the direct output needed
for the joint replacement initiative.

**Independent Test**: Run monthly export and verify all 11kV cable sections are present
with required fields populated in the report output.

**Acceptance Scenarios**:

1. **Given** the monthly reporting cycle is executed, **When** the report is generated,
  **Then** all 11kV cable sections are included with required data fields.
2. **Given** a cable section has joint location and topology data, **When** it appears in
  the export, **Then** latitude/longitude and topology details are included.

---

### User Story 2 - Support Planning and Government Reporting Consumers (Priority: P2)

As a planning or reporting consumer, I receive a monthly report with distribution-level
11kV cable details that can be used for initiative planning and government reporting.

**Why this priority**: Consumer usability is the business value that justifies the new
interface.

**Independent Test**: Provide generated report to representative consumers and verify all
required planning/reporting fields are present and interpretable.

**Acceptance Scenarios**:

1. **Given** the report is delivered monthly, **When** planning users review it,
   **Then** they can identify cable attributes, connectivity, and joint positions needed
   for replacement planning.

---

### User Story 3 - Improve Data Integrity and Traceability (Priority: P3)

As a GIS data owner, I need the report output to reflect consistent and traceable 11kV
distribution data to reduce manual reconciliation before publication.

**Why this priority**: Data quality and traceability are critical for recurring external
reporting.

**Independent Test**: Compare a sample set of exported rows with GIS source records and
verify key fields and links align.

**Acceptance Scenarios**:

1. **Given** a generated monthly report, **When** sampled records are reconciled to GIS
  source, **Then** required cable and location data match source records.

---

### Edge Cases

- A cable section has missing commissioning data in source GIS records.
- A cable section has one or more joint locations with missing latitude or longitude.
- A cable section has ambiguous or incomplete network topology linkage.
- Duplicate cable identifiers appear due to source data issues.
- Monthly run occurs when some source records are being updated concurrently.
- Open question: What output file format is required for monthly government reporting
  submission (for example CSV, XLSX, or both)?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST provide a monthly reporting interface for exporting all 11kV
  cable sections.
- **FR-002**: System MUST include the following data fields for each exported cable
  section: length, material, cable size, insulation, commissioning data, substation
  connectivity, joint locations (latitude/longitude), and network topology.
- **FR-003**: System MUST include all 11kV cable sections available in GIS scope at the
  time of monthly export.
- **FR-004**: System MUST generate report data suitable for planning and government
  reporting use.
- **FR-005**: System MUST represent joint location data in latitude and longitude values.
- **FR-006**: System MUST keep report generation within GIS scope and MUST NOT introduce
  ADMS workflow or design scope.
- **FR-007**: System MUST keep unrelated GIS workflows unchanged.

### Business Problem *(mandatory)*

The 11kV cable joint replacement initiative requires detailed and recurring distribution
data, but current reporting does not provide a dedicated monthly interface covering all
required cable, joint, and topology information.

### Current Pain Point *(mandatory)*

Users spend time collecting and reconciling 11kV cable data manually from multiple views,
which delays planning cycles and increases risk of incomplete reporting submissions.

### Target Users / Report Consumers *(mandatory)*

- GIS reporting users generating monthly exports.
- Joint replacement initiative planners using cable distribution detail.
- Government reporting consumers requiring monthly submission data.

### Desired Future Behaviour *(mandatory)*

Users run one monthly report interface that exports complete 11kV cable section data,
including attribute, connectivity, joint location, and topology details needed for
planning and government reporting.

### Reporting Frequency *(mandatory)*

- Monthly recurring report generation.

### In Scope *(mandatory)*

- New monthly GIS reporting interface for 11kV cable overview export.
- Inclusion of all stated cable section data fields.
- Report content suitable for planning and government reporting use.

### Out of Scope *(mandatory)*

- ADMS workflow, design, or implementation changes.
- Non-11kV cable reporting scope.
- Redesign of external government reporting systems.
- Changes to unrelated GIS operational workflows.

### Business Requirements *(mandatory)*

- **BR-001**: Create a monthly reporting interface exporting all 11kV cable sections.
- **BR-002**: Include required data fields: length, material, cable size, insulation,
  commissioning data, substation connectivity, joint locations (latitude/longitude),
  network topology.
- **BR-003**: Support 11kV cable joint replacement planning and government reporting.

### Dependencies *(mandatory)*

- **DEP-001**: GIS source records contain the required 11kV cable attribute and network
  data.
- **DEP-002**: Joint location coordinates are available or derivable from GIS data.
- **DEP-003**: Substation connectivity and network topology links are maintained in GIS.

### Constraints *(mandatory)*

- **CON-001**: Scope is GIS only.
- **CON-002**: Wording and behavior remain aligned with the stated business requirement.
- **CON-003**: No ADMS scope may be introduced unless explicitly stated as a dependency.
- **CON-004**: Monthly generation must not regress existing GIS reporting workflows.

### Data Field Expectations *(mandatory)*

- **DF-001**: Length is included for each 11kV cable section.
- **DF-002**: Material is included for each 11kV cable section.
- **DF-003**: Cable size is included for each 11kV cable section.
- **DF-004**: Insulation is included for each 11kV cable section.
- **DF-005**: Commissioning data is included for each 11kV cable section.
- **DF-006**: Substation connectivity reference is included for each 11kV cable section.
- **DF-007**: Joint location latitude and longitude are included where applicable.
- **DF-008**: Network topology reference data is included for each 11kV cable section.

### Key Entities *(include if feature involves data)*

- **11kV Cable Section Record**: Export row representing one cable section and core
  attributes.
- **Joint Location Record**: Coordinate details (latitude/longitude) linked to cable
  joints.
- **Topology Link Record**: Connectivity references describing how cable sections relate
  within network topology and substation linkage.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In 100% of monthly report runs, all GIS-available 11kV cable sections are
  included in the export.
- **SC-002**: In at least 98% of exported rows, required data fields are populated from
  source records; exceptions are explicitly flagged.
- **SC-003**: Planning and reporting users can use the generated monthly report for
  routine initiative planning and reporting in at least 90% of tested scenarios without
  manual multi-source reconciliation.
- **SC-004**: Manual data preparation effort for monthly 11kV reporting is reduced by at
  least 70% compared with the current workflow.

## Assumptions

- GIS data model includes identifiable 11kV cable sections and associated attributes.
- Monthly reporting schedule is agreed with planning and reporting stakeholders.
- Government reporting consumers accept the generated report structure once agreed.
- Existing access controls for GIS reporting users remain unchanged.

## Delivery Layer Mapping *(mandatory)*

Document each major item with an explicit layer label so reviewers can separate intent,
design, build work, and verification:

- **Business Requirement**: Provide monthly 11kV cable overview export with detailed
  distribution data for planning and government reporting.
- **Technical Design**: Create GIS reporting interface that aggregates required cable,
  joint, connectivity, and topology fields into monthly export output.
- **Implementation Task**: Add report generation capability and field mapping for all
  required data elements within GIS scope.
- **Test Consideration**: Validate monthly completeness, field-level population, flagged
  exceptions, and non-regression of existing reporting workflows.
