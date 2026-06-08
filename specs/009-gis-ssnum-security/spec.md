# Feature Specification: Load Taking Mobile App Enhancement

**Feature Branch**: `009-gis-ssnum-security`

**Created**: 2026-06-08

**Status**: Draft

**Input**: User description: "Create a specification for one DNOO GIS enhancement item only. Requirement ID: 15 - Load taking mobile app enhancement. Business requirement: Update GIS_SPS_INTF_001 to add SSNUM column to TX_CORE_LOAD table and TX_LOAD_READING table. The SSNUM should be populated from the HV substation containing the transformer. Also address SQL injection risk in GIS SPS INTF 004 search APIs. Business purpose: Ensure substation number data is carried through to Tx Load Reading records. Please produce: business problem, current pain point, target users / impacted systems, desired future behaviour, in-scope / out-of-scope, assumptions, dependencies, constraints, acceptance criteria, data propagation expectations, security considerations, edge cases / open questions. Scope notes: GIS only. Do not introduce ADMS scope unless it is an explicitly stated dependency. Keep both data handling and security requirements explicit. Keep the wording aligned with the business requirement above."

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

### User Story 1 - Propagate SSNUM Through Load Interfaces (Priority: P1)

As a GIS integration user, when load data is processed, SSNUM is populated from the HV
substation containing the transformer and carried into both TX_CORE_LOAD and
TX_LOAD_READING.

**Why this priority**: This is the primary business outcome and directly addresses data
completeness for Tx load reading records.

**Independent Test**: Process transformer load records with known HV substation mapping
and verify SSNUM appears in both target tables.

**Acceptance Scenarios**:

1. **Given** a transformer is associated to an HV substation with SSNUM, **When** data
  is written through GIS_SPS_INTF_001, **Then** SSNUM is populated in TX_CORE_LOAD.
2. **Given** TX_CORE_LOAD receives SSNUM, **When** corresponding TX_LOAD_READING records
  are generated, **Then** SSNUM is carried through to TX_LOAD_READING.

---

### User Story 2 - Improve Data Consistency for Impacted Systems (Priority: P2)

As an impacted system stakeholder, I need SSNUM consistency between load core and load
reading data so downstream consumers receive aligned substation number values.

**Why this priority**: Consistency across related records reduces reconciliation effort
and prevents data mismatch issues.

**Independent Test**: Compare related rows across TX_CORE_LOAD and TX_LOAD_READING for
the same transformer load cycle and verify SSNUM alignment.

**Acceptance Scenarios**:

1. **Given** load records for a transformer are processed in a cycle, **When**
  TX_CORE_LOAD and TX_LOAD_READING are compared, **Then** SSNUM values match.

---

### User Story 3 - Protect Search APIs from SQL Injection (Priority: P3)

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

- Transformer has no resolvable HV substation mapping at processing time.
- HV substation SSNUM is null or stale for an otherwise valid transformer association.
- TX_CORE_LOAD is updated but TX_LOAD_READING insert/update is delayed or partial.
- Same transformer is reassigned to another HV substation between load cycles.
- Search API receives encoded payloads or unusual Unicode patterns.
- Open question: If SSNUM cannot be resolved for a record, should the record be blocked
  from TX_LOAD_READING or saved with an explicit unresolved-status marker?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST add SSNUM column to TX_CORE_LOAD in GIS_SPS_INTF_001 scope.
- **FR-002**: System MUST add SSNUM column to TX_LOAD_READING in GIS_SPS_INTF_001 scope.
- **FR-003**: System MUST populate SSNUM from the HV substation containing the
  transformer.
- **FR-004**: System MUST carry SSNUM through from TX_CORE_LOAD to related
  TX_LOAD_READING records.
- **FR-005**: System MUST keep SSNUM values consistent across related core and reading
  records for the same transformer load cycle.
- **FR-006**: System MUST address SQL injection risk in GIS_SPS_INTF_004 search APIs.
- **FR-007**: System MUST preserve intended search functionality for valid inputs while
  preventing unsafe query behavior.
- **FR-008**: System MUST keep scope limited to GIS and MUST NOT introduce ADMS workflow
  or design scope.

### Business Problem *(mandatory)*

Substation number data is not consistently propagated into Tx load reading records, and
search API security risk creates potential exposure for unsafe query execution.

### Current Pain Point *(mandatory)*

Users and impacted systems must handle mismatched or missing SSNUM values between core
and reading records, while search interfaces require remediation for SQL injection risk.

### Target Users / Impacted Systems *(mandatory)*

- GIS integration and support users managing load-taking interface outputs.
- Impacted systems and consumers relying on Tx load records produced through GIS SPS
  interfaces.

### Desired Future Behaviour *(mandatory)*

SSNUM is reliably populated from the HV substation containing the transformer and
propagated into both TX_CORE_LOAD and TX_LOAD_READING, while GIS_SPS_INTF_004 search
APIs safely process input without SQL injection exposure.

### In Scope *(mandatory)*

- Add and populate SSNUM in TX_CORE_LOAD and TX_LOAD_READING via GIS_SPS_INTF_001.
- Maintain SSNUM propagation consistency from core load to load reading records.
- Address SQL injection risk in GIS_SPS_INTF_004 search APIs.

### Out of Scope *(mandatory)*

- ADMS workflow, design, or implementation changes.
- Redesign of mobile app functionality beyond required interface data/security updates.
- Unrelated GIS interfaces outside GIS_SPS_INTF_001 and GIS_SPS_INTF_004.

### Business Requirements *(mandatory)*

- **BR-001**: Update GIS_SPS_INTF_001 to add SSNUM to TX_CORE_LOAD and
  TX_LOAD_READING.
- **BR-002**: Populate SSNUM from the HV substation containing the transformer.
- **BR-003**: Ensure substation number data is carried through to Tx Load Reading
  records.
- **BR-004**: Address SQL injection risk in GIS SPS INTF 004 search APIs.

### Dependencies *(mandatory)*

- **DEP-001**: GIS mapping between transformer and containing HV substation is available.
- **DEP-002**: SSNUM is maintained in HV substation source data.
- **DEP-003**: Interface paths for GIS_SPS_INTF_001 and GIS_SPS_INTF_004 are available.

### Constraints *(mandatory)*

- **CON-001**: Scope is GIS only.
- **CON-002**: Data handling and security requirements must remain explicit.
- **CON-003**: No ADMS scope may be introduced unless explicitly stated as dependency.
- **CON-004**: Existing GIS integration behavior outside this requirement must not
  regress.

### Data Propagation Expectations *(mandatory)*

- **DP-001**: SSNUM value in TX_CORE_LOAD reflects the HV substation containing the
  transformer at processing time.
- **DP-002**: SSNUM value in TX_LOAD_READING matches the related TX_CORE_LOAD SSNUM for
  the same load context.
- **DP-003**: Missing or unresolved SSNUM conditions are detectable for operational
  follow-up.

### Security Considerations *(mandatory)*

- **SEC-001**: Search inputs to GIS_SPS_INTF_004 are handled in a way that prevents SQL
  injection execution paths.
- **SEC-002**: Security controls must not break valid search behavior for normal user
  inputs.
- **SEC-003**: Suspicious search input patterns are safely handled and traceable for
  investigation.

### Key Entities *(include if feature involves data)*

- **TX_CORE_LOAD Record**: Core load interface row requiring SSNUM population.
- **TX_LOAD_READING Record**: Load reading interface row requiring propagated SSNUM.
- **Transformer-HV Substation Association**: Source relationship used to determine SSNUM.
- **Search API Request**: Input payload for GIS_SPS_INTF_004 search operations.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In 100% of tested records with valid transformer-to-substation mapping,
  SSNUM is populated in TX_CORE_LOAD.
- **SC-002**: In at least 99% of tested related record pairs,
  TX_LOAD_READING.SSNUM matches TX_CORE_LOAD.SSNUM for the same load context.
- **SC-003**: SQL injection test payloads against GIS_SPS_INTF_004 search APIs do not
  produce unsafe query execution behavior in 100% of test cases.
- **SC-004**: Manual reconciliation effort for missing or mismatched SSNUM values in Tx
  load reading data is reduced by at least 80% compared with current workflow.

## Assumptions

- Transformer to HV substation association is available at load processing time.
- SSNUM is a stable identifier for the containing HV substation in GIS data.
- Security testing can exercise representative malicious input patterns for search APIs.
- Operational teams can review flagged unresolved SSNUM cases.

## Delivery Layer Mapping *(mandatory)*

Document each major item with an explicit layer label so reviewers can separate intent,
design, build work, and verification:

- **Business Requirement**: Carry SSNUM from HV substation mapping through Tx load data
  and reduce security risk in search APIs.
- **Technical Design**: Populate SSNUM in core load flow, propagate to load reading
  records, and enforce safe input handling in GIS_SPS_INTF_004 search endpoints.
- **Implementation Task**: Add SSNUM fields and propagation behavior, then remediate
  search API SQL injection exposure while preserving valid search outcomes.
- **Test Consideration**: Validate SSNUM mapping/propagation integrity,
  unresolved-mapping handling, malicious input protection, and non-regression of valid
  searches.
