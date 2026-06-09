# Feature Specification: Load Taking Mobile App Enhancement

**Feature Branch**: `009-gis-ssnum-security`

**Created**: 2026-06-08

**Status**: Draft

**Input**: User description: "Create a specification for one DNOO GIS enhancement item only. Requirement ID: 15 - Load taking mobile app enhancement. Business requirement: Update GIS_SPS_INTF_001 to add SSNUM column to TX_CORE_LOAD table and TX_LOAD_READING table. The SSNUM should be populated from the HV substation containing the transformer. Also address SQL injection risk in GIS SPS INTF 004 search APIs. Business purpose: Ensure substation number data is carried through to Tx Load Reading records. Please produce: business problem, current pain point, target users / impacted systems, desired future behaviour, in-scope / out-of-scope, assumptions, dependencies, constraints, acceptance criteria, data propagation expectations, security considerations, edge cases / open questions. Scope notes: GIS only. Do not introduce ADMS scope unless it is an explicitly stated dependency. Keep both data handling and security requirements explicit. Keep the wording aligned with the business requirement above."

## Clarifications

### Session 2026-06-09

- Q: What is the correct field name and table scope for SSNUM population? → A: Existing SS_NO field (alias SUBSTATION_NO) in TX_LOAD_READING only; no new column; TX_CORE_LOAD is out of scope.
- Q: What is the asset-type-specific SSNUM source rule? → A: HV SS Transformer → containing HV Substation's SSNUM; HV PM Transformer → transformer's own SSNUM; all other types → SS_NO not populated.
- Q: What is the null-handling rule when SSNUM is empty? → A: If SSNUM is empty in the Transformer or Structure Boundary, SS_NO in TX_LOAD_READING is null.
- Q: Is SQL injection remediation (GIS_SPS_INTF_004) in scope for this item? → A: Deferred — flagged as out of scope for this item pending clarification; requires a separate GIS enhancement item or explicit re-scoping confirmation.

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

### User Story 1 - Populate SS_NO for HV SS Transformer from Containing HV Substation (Priority: P1)

As a GIS integration user, when load data is processed for an HV SS Transformer, the
SS_NO field in TX_LOAD_READING is populated using the SSNUM of the containing HV
Substation.

**Why this priority**: This is the primary asset-type case and directly addresses the
core business outcome of carrying substation number data into Tx load reading records.

**Independent Test**: Process load records for HV SS Transformer features with known
containing HV Substation SSNUM values and verify SS_NO in TX_LOAD_READING is populated
with the HV Substation's SSNUM.

**Acceptance Scenarios**:

1. **Given** an HV SS Transformer associated to an HV Substation with a non-null SSNUM,
   **When** TX_LOAD_READING is populated, **Then** SS_NO is set to the HV Substation's
   SSNUM.
2. **Given** an HV SS Transformer where the containing HV Substation's SSNUM is empty
   or null, **When** TX_LOAD_READING is populated, **Then** SS_NO in TX_LOAD_READING is
   null.

---

### User Story 2 - Populate SS_NO for HV PM Transformer and Exclude Other Asset Types (Priority: P2)

As a GIS integration user, when load data is processed for an HV PM Transformer, the
SS_NO field in TX_LOAD_READING is populated using the transformer's own SSNUM directly.
For all other Transformer asset types, SS_NO is not populated.

**Why this priority**: Correct asset-type differentiation prevents incorrect SSNUM
values being written and ensures only supported asset types receive SS_NO population.

**Independent Test**: Process load records for (a) an HV PM Transformer with a known
own SSNUM, and (b) a Transformer of an asset type other than HV SS or HV PM. Verify
SS_NO in TX_LOAD_READING reflects the transformer's own SSNUM for HV PM, and is not
populated for unsupported asset types.

**Acceptance Scenarios**:

1. **Given** an HV PM Transformer with a non-null own SSNUM, **When** TX_LOAD_READING
   is populated, **Then** SS_NO is set to the transformer's own SSNUM.
2. **Given** a Transformer that is neither HV SS nor HV PM asset type, **When**
   TX_LOAD_READING is populated, **Then** SS_NO is not populated for that record.

---

### User Story 3 - SQL Injection Remediation for GIS_SPS_INTF_004 Search APIs *(Deferred — scope pending confirmation)*

> **Note**: This user story is flagged as **out of scope for this item** pending explicit re-scoping confirmation. The original business requirement referenced this concern; however, the design extract limits this item's scope to SS_NO population in TX_LOAD_READING only. A separate GIS enhancement item or explicit approval is required before this story can be actioned.

As a GIS API consumer, search endpoints in GIS_SPS_INTF_004 need to handle inputs safely
to prevent SQL injection risk while preserving expected query behavior.

**Why this priority**: Security risk mitigation is required to protect data integrity and
service reliability.

**Independent Test**: Submit normal and malicious search inputs and verify malicious
patterns are safely handled without unsafe query execution.

**Acceptance Scenarios**:

1. **Given** a valid search request, **When** it is submitted to GIS_SPS_INTF_004,
   **Then** expected results are returned.
2. **Given** a malicious search pattern intended for injection, **When** it is submitted,
   **Then** the request is safely handled and no unsafe query behavior occurs.

---

### Edge Cases

- HV SS Transformer where the containing HV Substation's SSNUM is null or empty:
  SS_NO in TX_LOAD_READING is null (per explicit null-handling rule).
- HV PM Transformer where the transformer's own SSNUM is null or empty: SS_NO in
  TX_LOAD_READING is null.
- Transformer asset type that is neither HV SS nor HV PM: SS_NO is not populated in
  TX_LOAD_READING.
- Transformer with no resolvable containing HV Substation at processing time (HV SS
  type): SS_NO is null.
- Search API receives encoded payloads or unusual Unicode patterns.
- Same transformer reassigned to another HV Substation between load cycles.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST populate the existing SS_NO field (alias: SUBSTATION_NO) in
  TX_LOAD_READING based on the Transformer asset type. No new column is added.
- **FR-002**: For HV SS Transformer asset type, system MUST derive SS_NO from the SSNUM
  of the containing HV Substation.
- **FR-003**: For HV PM Transformer asset type, system MUST populate SS_NO directly
  from the transformer's own SSNUM attribute.
- **FR-004**: For all Transformer asset types other than HV SS and HV PM, system MUST
  NOT populate SS_NO in TX_LOAD_READING.
- **FR-005**: If the SSNUM source (HV Substation or transformer, as applicable) is
  empty or null, system MUST write SS_NO as null in TX_LOAD_READING.
- **FR-006**: *(Deferred — scope pending confirmation)* System MUST address SQL injection
  risk in GIS_SPS_INTF_004 search APIs. This requirement is flagged out of scope for
  this item per the design extract; confirm or assign to a separate GIS enhancement item.
- **FR-007**: *(Deferred — scope pending confirmation)* System MUST preserve intended
  search functionality for valid inputs while preventing unsafe query behavior.
- **FR-008**: System MUST keep scope limited to GIS and MUST NOT introduce ADMS workflow
  or design scope.

### Business Problem *(mandatory)*

The existing SS_NO field in TX_LOAD_READING is not populated with the correct SSNUM
value for transformer features. Additionally, SQL injection risk in GIS_SPS_INTF_004
search APIs is a separately deferred concern pending scope confirmation.

### Current Pain Point *(mandatory)*

The SS_NO field in TX_LOAD_READING is not correctly populated from the SSNUM of the
associated transformer or containing substation, meaning substation number data is
absent or incorrect for Tx load reading records.

### Target Users / Impacted Systems *(mandatory)*

- GIS integration and support users managing load-taking interface outputs.
- Impacted systems and consumers relying on Tx load records produced through GIS SPS
  interfaces.

### Desired Future Behaviour *(mandatory)*

The existing SS_NO field in TX_LOAD_READING is reliably populated based on the
transformer asset type: for HV SS Transformers, SS_NO is derived from the containing HV
Substation's SSNUM; for HV PM Transformers, SS_NO is taken from the transformer's own
SSNUM; for all other Transformer asset types, SS_NO is not populated. Where the SSNUM
source is null or empty, SS_NO is null.

### In Scope *(mandatory)*

- Populate the existing SS_NO field (alias: SUBSTATION_NO) in TX_LOAD_READING via
  GIS_SPS_INTF_001, using asset-type-specific SSNUM source rules.
- *(Deferred — pending confirmation)* Address SQL injection risk in GIS_SPS_INTF_004
  search APIs; not actioned under this item until explicitly re-scoped.

### Out of Scope *(mandatory)*

- TX_CORE_LOAD changes of any kind.
- Adding a new column to TX_LOAD_READING; this enhancement uses the existing SS_NO
  field.
- Changes to other logic in the load-taking interface beyond SS_NO population.
- ADMS workflow, design, or implementation changes.
- Redesign of mobile app functionality beyond required interface data/security updates.
- Unrelated GIS interfaces outside GIS_SPS_INTF_001 and GIS_SPS_INTF_004.

### Business Requirements *(mandatory)*

- **BR-001**: Populate the existing SS_NO field in TX_LOAD_READING with the SSNUM of
  the associated transformer feature, based on transformer asset type.
- **BR-002**: For HV SS Transformer: derive SS_NO from the containing HV Substation's
  SSNUM. For HV PM Transformer: use the transformer's own SSNUM. For all other
  Transformer asset types: SS_NO is not populated.
- **BR-003**: Ensure substation number data is carried through to Tx Load Reading
  records for supported transformer asset types.
- **BR-004**: *(Deferred — scope pending confirmation)* Address SQL injection risk in
  GIS SPS INTF 004 search APIs. Flagged out of scope per design extract; requires
  separate GIS enhancement item or explicit re-scoping approval.

### Dependencies *(mandatory)*

- **DEP-001**: GIS mapping between HV SS Transformer and its containing HV Substation
  is available at load processing time.
- **DEP-002**: SSNUM is maintained on the HV Substation (for HV SS Transformer) and on
  the transformer feature itself (for HV PM Transformer).
- **DEP-003**: Interface paths for GIS_SPS_INTF_001 and GIS_SPS_INTF_004 are available.

### Constraints *(mandatory)*

- **CON-001**: Scope is GIS only.
- **CON-002**: Data handling and security requirements must remain explicit.
- **CON-003**: No ADMS scope may be introduced unless explicitly stated as dependency.
- **CON-004**: Enhancement uses the existing SS_NO field in TX_LOAD_READING; no new
  column is added and no other logic is changed.
- **CON-005**: Existing GIS integration behavior outside this requirement must not
  regress.

### Data Propagation Expectations *(mandatory)*

- **DP-001**: For HV SS Transformer: SS_NO in TX_LOAD_READING reflects the SSNUM of the
  containing HV Substation at processing time.
- **DP-002**: For HV PM Transformer: SS_NO in TX_LOAD_READING reflects the transformer's
  own SSNUM at processing time.
- **DP-003**: For all other Transformer asset types: SS_NO in TX_LOAD_READING is not
  populated by this enhancement.
- **DP-004**: If the applicable SSNUM source is null or empty, SS_NO in TX_LOAD_READING
  is null.

### Security Considerations *(mandatory)*

> **Note**: The following items are flagged as **deferred — out of scope for this item**
> pending explicit re-scoping confirmation. A separate GIS enhancement item covering
> GIS_SPS_INTF_004 SQL injection remediation must be raised or this scope must be
> formally approved for inclusion here before planning.

- **SEC-001**: *(Deferred)* Search inputs to GIS_SPS_INTF_004 are handled in a way that
  prevents SQL injection execution paths.
- **SEC-002**: *(Deferred)* Security controls must not break valid search behavior for
  normal user inputs.
- **SEC-003**: *(Deferred)* Suspicious search input patterns are safely handled and
  traceable for investigation.

### Key Entities *(include if feature involves data)*

- **TX_LOAD_READING Record**: Load reading interface row; contains existing SS_NO field
  (alias: SUBSTATION_NO) populated by this enhancement.
- **HV SS Transformer**: Transformer asset type whose SS_NO is derived from its
  containing HV Substation's SSNUM.
- **HV PM Transformer**: Transformer asset type whose SS_NO is taken from the
  transformer's own SSNUM attribute.
- **HV Substation**: Spatial or relationship source of SSNUM for HV SS Transformer
  population.
- **Search API Request**: Input payload for GIS_SPS_INTF_004 search operations.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In 100% of tested records where the Transformer is HV SS type and the
  containing HV Substation has a non-null SSNUM, SS_NO in TX_LOAD_READING is populated
  with that SSNUM.
- **SC-002**: In 100% of tested records where the Transformer is HV PM type and the
  transformer's own SSNUM is non-null, SS_NO in TX_LOAD_READING is populated with that
  SSNUM.
- **SC-003**: In 100% of tested records where the applicable SSNUM source is null or
  empty, SS_NO in TX_LOAD_READING is null.
- **SC-004**: In 100% of tested records where the Transformer is neither HV SS nor HV
  PM type, SS_NO in TX_LOAD_READING is not populated by this enhancement.
- **SC-005**: *(Deferred)* SQL injection test payloads against GIS_SPS_INTF_004 search
  APIs do not produce unsafe query execution behavior in 100% of test cases. Deferred
  pending scope confirmation.
- **SC-006**: Manual reconciliation effort for missing or incorrect SS_NO values in Tx
  load reading data is reduced by at least 80% compared with current workflow.

## Assumptions

- The existing SS_NO field in TX_LOAD_READING is available for population; no schema
  change to add a new column is needed.
- Transformer asset type (HV SS, HV PM, or other) is determinable at load processing
  time via the GIS data model.
- The transformer-to-containing-HV-Substation association is resolvable at load
  processing time for HV SS Transformer features.
- SSNUM is a stable identifier on both HV Substation features and HV PM Transformer
  features in GIS data.
- Security testing can exercise representative malicious input patterns for search APIs.
- Operational teams can review records with null SS_NO for follow-up where applicable.

## Delivery Layer Mapping *(mandatory)*

Document each major item with an explicit layer label so reviewers can separate intent,
design, build work, and verification:

- **Business Requirement**: Populate the existing SS_NO field in TX_LOAD_READING with
  the correct SSNUM value based on transformer asset type. SQL injection remediation for
  GIS_SPS_INTF_004 is deferred pending scope confirmation.
- **Technical Design**: Apply asset-type branching to derive SS_NO: HV SS Transformer
  → containing HV Substation SSNUM; HV PM Transformer → transformer's own SSNUM; other
  types → not populated. Null source → null SS_NO.
- **Implementation Task**: Implement asset-type detection and SSNUM source lookup for
  SS_NO population in TX_LOAD_READING via GIS_SPS_INTF_001; implement null handling.
- **Test Consideration**: Verify SS_NO population for HV SS and HV PM asset types,
  null-source handling, non-population for unsupported asset types, and non-regression
  of existing GIS integration behavior. SQL injection testing deferred.
