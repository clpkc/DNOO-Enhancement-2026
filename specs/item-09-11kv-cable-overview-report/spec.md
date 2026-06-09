# Feature Specification: 11kV Cable Overview Report Generation

**Feature Branch**: `006-create-11kv-cable-report`

**Created**: 2026-06-08

**Status**: Draft

**Input**: User description: "Create a specification for one DNOO GIS enhancement item only. Requirement ID: 09 - 11kV cable overview report generation. Business requirement: Create a new monthly reporting interface exporting all 11kV cable sections with length, material, cable size, insulation, commissioning data, substation connectivity, joint locations (latitude / longitude), network topology. Business purpose: Support the 11kV cable joint replacement initiative with detailed distribution data for planning and government reporting. Please produce: business problem, current pain point, target users / report consumers, desired future behaviour, reporting frequency, in-scope / out-of-scope, assumptions, dependencies, constraints, acceptance criteria, data field expectations, edge cases / open questions. Scope notes: GIS only. Do not introduce ADMS scope unless it is an explicitly stated dependency. Keep the wording aligned with the business requirement above."

## Clarifications

### Session 2026-06-09

- Q: What cable scope is in scope — all 11kV or in-service only? → A: In-service 11kV cable only; all other lifecycle states are excluded.
- Q: What is the required output format? → A: CSV.
- Q: Does the joint location record include commissioning data? → A: Yes — joint locations include latitude, longitude, and commissioning data.
- Q: Are OHL circuit elements included? → A: No — OHL circuit elements are explicitly excluded; coverage is HV underground cable and joints for closed ring circuits only.
- Q: How are terminated substations identified for tracing? → A: By tracing from Device features that have SOM_SS and SOM_CCT and are contained by a Substation; each unique terminated substation is identified and must be part of the underground circuit.

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

### User Story 1 - Generate Monthly 11kV In-Service Cable CSV Export (Priority: P1)

As a GIS reporting user, I generate a monthly CSV report export by month end, covering
all in-service 11kV cable sections and their joint locations for closed ring underground
circuits, with the required planning and reporting fields.

**Why this priority**: This is the core delivery objective and the direct output needed
for the joint replacement initiative.

**Independent Test**: Run monthly export and verify all in-service 11kV cable sections
are present (OHL excluded), all joint locations include commissioning data, and output
is in CSV format delivered by month end.

**Acceptance Scenarios**:

1. **Given** the monthly reporting cycle is executed, **When** the report is generated,
   **Then** all in-service 11kV cable sections for closed ring underground circuits are
   included in the CSV export with required data fields.
2. **Given** a cable section has joint locations, **When** it appears in the export,
   **Then** each joint location includes latitude, longitude, and commissioning data.
3. **Given** OHL circuit elements exist in the network, **When** the report is generated,
   **Then** OHL elements are excluded from the export.

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
- A cable section has one or more joint locations with missing latitude, longitude, or
  commissioning data.
- A cable section has ambiguous or incomplete network topology linkage.
- Duplicate cable identifiers appear due to source data issues.
- Monthly run occurs when some source records are being updated concurrently.
- Device feature has SOM_SS and SOM_CCT but is not contained by a Substation — tracing
  exclusion rule applies.
- A circuit contains both underground and OHL segments — only the underground segments
  are included; OHL elements are excluded.
- A circuit is not a closed ring — explicit scope constraint; behavior must be defined
  before Detailed Design.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST provide a monthly reporting interface (CLP GIS Report
  integration) exporting all in-service 11kV cable sections for closed ring underground
  circuits. OHL circuit elements MUST be excluded.
- **FR-002**: System MUST include the following data fields for each exported cable
  section: length, material, cable size, insulation, commissioning data, connected
  substation.
- **FR-003**: System MUST include all joint location records for each exported cable
  section with: latitude, longitude, and commissioning data.
- **FR-004**: System MUST include network topology data indicating connectivity between
  cable sections and joints.
- **FR-005**: System MUST identify terminated substations by tracing from Device features
  that have SOM_SS and SOM_CCT and are contained by a Substation. Only Device features
  meeting both criteria and contained by a Substation qualify as trace start points.
- **FR-006**: System MUST produce output in CSV format, delivered by month end.
- **FR-007**: System MUST generate report data suitable for planning and government
  reporting use.
- **FR-008**: System MUST keep report generation within GIS scope and MUST NOT introduce
  ADMS workflow or design scope.
- **FR-009**: System MUST keep unrelated GIS workflows unchanged.

### Business Problem *(mandatory)*

The 11kV cable joint replacement initiative requires detailed and recurring distribution
data for in-service underground closed ring circuits, but current reporting does not
provide a dedicated monthly CSV interface covering all required cable section, joint,
and topology information.

### Current Pain Point *(mandatory)*

Users spend time collecting and reconciling 11kV cable data manually from multiple views,
which delays planning cycles and increases risk of incomplete reporting submissions.

### Target Users / Report Consumers *(mandatory)*

- GIS reporting users generating monthly exports.
- Joint replacement initiative planners using cable distribution detail.
- Government reporting consumers requiring monthly submission data.

### Desired Future Behaviour *(mandatory)*

Users run one monthly report interface (CLP GIS Report integration) that exports
in-service 11kV cable sections for closed ring underground circuits in CSV format,
delivered by month end. The export includes cable attributes, connected substation,
joint locations with commissioning data, and network topology (cable-to-joint
connectivity). OHL elements are excluded.

### Reporting Frequency *(mandatory)*

- Monthly recurring report generation; output delivered by month end.

### In Scope *(mandatory)*

- New monthly GIS reporting interface (CLP GIS Report integration) for 11kV in-service
  cable overview export in CSV format.
- Coverage: in-service HV underground cable and joints for closed ring circuits only.
- Substation identification via tracing from Device features with SOM_SS and SOM_CCT,
  contained by a Substation.
- Inclusion of all required cable section, joint location (with commissioning data), and
  topology data fields.
- Report content suitable for planning and government reporting use, delivered by month
  end.

### Out of Scope *(mandatory)*

- ADMS workflow, design, or implementation changes.
- OHL (overhead line) circuit elements.
- Non-in-service 11kV cable sections (decommissioned, planned, etc.).
- Non-11kV cable reporting scope.
- Redesign of external government reporting systems.
- Changes to unrelated GIS operational workflows.

### Business Requirements *(mandatory)*

- **BR-001**: Create a monthly reporting interface exporting all in-service 11kV cable
  sections for closed ring underground circuits in CSV format, delivered by month end.
- **BR-002**: Include required cable section fields: length, material, cable size,
  insulation, commissioning data, connected substation.
- **BR-003**: Include all joint location fields: latitude, longitude, commissioning data.
- **BR-004**: Include network topology data showing cable-to-joint connectivity.
- **BR-005**: Support 11kV cable joint replacement planning and government reporting.

### Dependencies *(mandatory)*

- **DEP-001**: GIS source records contain in-service 11kV cable sections and required
  attribute data for underground closed ring circuits.
- **DEP-002**: Joint location coordinates and commissioning data are available or
  derivable from GIS data.
- **DEP-003**: Device features have SOM_SS and SOM_CCT attributes populated and are
  contained by Substation features in GIS for trace-based substation identification.
- **DEP-004**: Network topology links between cable sections and joints are maintained
  in GIS.
- **DEP-005**: Detailed Design for this item (Batch 2) is completed before implementation
  planning begins; current status is Pre-Design.

### Constraints *(mandatory)*

- **CON-001**: Scope is GIS only; CLP GIS Report integration is the affected interface.
- **CON-002**: Coverage is in-service HV underground closed ring circuits only; OHL
  elements and non-in-service cable are excluded.
- **CON-003**: Output format is CSV; report must be delivered by month end.
- **CON-004**: No ADMS scope may be introduced unless explicitly stated as a dependency.
- **CON-005**: Monthly generation must not regress existing GIS reporting workflows.
- **CON-006**: This item is Batch 2 / Pre-Design status; Detailed Design must be
  completed before implementation planning can proceed.

### Data Field Expectations *(mandatory)*

**Cable Section fields (one row per in-service 11kV cable section):**
- **DF-001**: Length.
- **DF-002**: Material.
- **DF-003**: Cable size.
- **DF-004**: Insulation.
- **DF-005**: Commissioning data.
- **DF-006**: Connected substation identifier.

**Joint Location fields (one row per joint on each cable section):**
- **DF-007**: Latitude.
- **DF-008**: Longitude.
- **DF-009**: Commissioning data.

**Network Topology fields:**
- **DF-010**: Connectivity reference between cable sections and joints.

### Key Entities *(include if feature involves data)*

- **In-Service 11kV Cable Section**: Export row representing one in-service underground
  HV cable section in a closed ring circuit, with cable attributes and connected
  substation.
- **Cable Joint Location**: Record linked to a cable section including latitude,
  longitude, and commissioning data.
- **Topology Link**: Connectivity reference describing cable-to-joint relationships
  within the underground circuit.
- **Terminated Substation**: Substation identified by tracing from Device features with
  SOM_SS and SOM_CCT that are contained by a Substation and are part of the underground
  circuit.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In 100% of monthly CSV report runs, all in-service 11kV cable sections for
  closed ring underground circuits are included; OHL elements are absent from the export.
- **SC-002**: In at least 98% of exported cable section rows, all required cable fields
  are populated from source records; exceptions are explicitly flagged.
- **SC-003**: In at least 98% of exported joint location rows, latitude, longitude, and
  commissioning data are populated; exceptions are explicitly flagged.
- **SC-004**: Planning and reporting users can use the generated monthly CSV report for
  routine initiative planning and reporting in at least 90% of tested scenarios without
  manual multi-source reconciliation.
- **SC-005**: Monthly CSV report is delivered by month end in 100% of tested runs.
- **SC-006**: Manual data preparation effort for monthly 11kV reporting is reduced by at
  least 70% compared with the current workflow.

## Assumptions

- GIS data model includes identifiable in-service 11kV cable sections and associated
  attributes for underground closed ring circuits.
- Device features in GIS have SOM_SS and SOM_CCT populated and are spatially contained
  by Substation features sufficient for trace-based substation identification.
- OHL circuit elements are identifiable and separable from underground cable data at
  export time.
- Monthly reporting schedule and month-end deadline are agreed with planning and
  reporting stakeholders.
- Government reporting consumers accept CSV as the submission format.
- Existing access controls for GIS reporting users remain unchanged.
- Detailed Design for this item is delivered before implementation planning begins
  (current status: Batch 2 / Pre-Design).

## Delivery Layer Mapping *(mandatory)*

Document each major item with an explicit layer label so reviewers can separate intent,
design, build work, and verification:

- **Business Requirement**: Provide monthly in-service 11kV cable overview CSV export
  with cable section attributes, joint locations (including commissioning data), and
  cable-to-joint topology for closed ring underground circuits; delivered by month end.
- **Technical Design**: CLP GIS Report integration traces from Device features with
  SOM_SS and SOM_CCT contained by a Substation to identify terminated substations and
  their in-service underground cable/joint data. Produces CSV output. OHL excluded.
  Requires Detailed Design before implementation.
- **Implementation Task**: Implement trace logic, field mapping for cable sections, joint
  locations, and topology, CSV generation, and month-end scheduling.
- **Test Consideration**: Validate in-service filter (OHL excluded), joint commissioning
  data population, trace logic (SOM_SS + SOM_CCT + containment), month-end delivery,
  flagged exceptions, and non-regression of existing GIS reporting workflows.
