# Feature Specification: Meter SPSID Attribute Automatic Population

**Feature Branch**: `007-meter-spsid-population`

**Created**: 2026-06-08

**Status**: Draft

**Input**: User description: "Create a specification for one DNOO GIS enhancement item only. Requirement ID: 12 - Meter SPSID attribute automatic population. Business requirement: Update multiple integration interfaces: CCMS, CCS, FSS, OFS. When a supply point is updated on a meter, the SPSID attribute should also be automatically populated. Remove the outdated SUPPLY_POINT relation. Business purpose: Ensure the meter is correctly connected to the new supply point through both utility network connectivity and SPSID attribute. Please produce: business problem, current pain point, target users / impacted systems, desired future behaviour, in-scope / out-of-scope, assumptions, dependencies, constraints, acceptance criteria, interface / data consistency expectations, edge cases / open questions. Scope notes: GIS only. Do not introduce ADMS scope unless it is an explicitly stated dependency. Keep integration and data consistency requirements explicit. Keep the wording aligned with the business requirement above."

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

### User Story 1 - Auto-Populate SPSID on Meter Supply Point Update (Priority: P1)

As a GIS user updating meter supply points, I need SPSID to be automatically populated
when the meter supply point changes so meter connectivity data remains complete.

**Why this priority**: This is the core business requirement and prevents incomplete
meter-to-supply-point mapping.

**Independent Test**: Update a meter's supply point and verify SPSID is automatically
updated in the same operation.

**Acceptance Scenarios**:

1. **Given** a meter with an existing supply point association, **When** the user
  updates the supply point, **Then** SPSID is automatically populated to the matching
  new supply point identifier.
2. **Given** a successful supply point update, **When** the meter record is reviewed,
  **Then** utility network connectivity and SPSID reflect the same supply point.

---

### User Story 2 - Keep Integration Interfaces Consistent (Priority: P2)

As an integration stakeholder, I need CCMS, CCS, FSS, and OFS interfaces to receive
consistent SPSID-aligned meter updates.

**Why this priority**: Interface consistency is required to avoid mismatched downstream
meter/supply-point states.

**Independent Test**: Perform meter supply point updates and verify each listed
integration interface receives consistent SPSID data.

**Acceptance Scenarios**:

1. **Given** a meter supply point update is completed, **When** data is transmitted to
  CCMS, CCS, FSS, and OFS, **Then** each interface reflects the updated SPSID and
  aligned meter connectivity state.

---

### User Story 3 - Retire Outdated SUPPLY_POINT Relation (Priority: P3)

As a GIS data owner, I need the outdated SUPPLY_POINT relation removed so the meter
connection model relies on current connectivity and SPSID alignment only.

**Why this priority**: Removing legacy relation paths reduces ambiguity and prevents
conflicting data interpretations.

**Independent Test**: Validate that meter updates operate with SPSID population and
utility network connectivity without requiring SUPPLY_POINT relation.

**Acceptance Scenarios**:

1. **Given** the enhancement is active, **When** meter supply point updates are
  performed, **Then** no dependency on outdated SUPPLY_POINT relation remains.

---

### Edge Cases

- Meter supply point is updated to a value with no valid SPSID mapping.
- Existing meter has stale SPSID that conflicts with current utility network
  connectivity.
- Interface delivery succeeds for some systems (for example CCMS, CCS) and fails for
  others (for example FSS, OFS).
- Multiple updates are applied to the same meter in a short interval.
- Removal of SUPPLY_POINT relation affects legacy reporting references.
- Open question: Is there a required sequencing rule for interface updates across CCMS,
  CCS, FSS, and OFS, or is eventual consistency across all interfaces acceptable?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST automatically populate SPSID when a meter supply point is
  updated.
- **FR-002**: System MUST ensure populated SPSID corresponds to the new supply point
  connection in utility network connectivity.
- **FR-003**: System MUST update integration interfaces CCMS, CCS, FSS, and OFS with the
  updated meter and SPSID state.
- **FR-004**: System MUST remove dependency on the outdated SUPPLY_POINT relation for
  this workflow.
- **FR-005**: System MUST reject or flag updates where supply point to SPSID mapping is
  invalid or unavailable.
- **FR-006**: System MUST preserve consistent meter-to-supply-point representation across
  utility network connectivity and SPSID attribute.
- **FR-007**: System MUST keep scope limited to GIS integration/data consistency and
  MUST NOT introduce ADMS workflow or design scope.
- **FR-008**: System MUST keep unrelated GIS workflows unchanged.

### Business Problem *(mandatory)*

When meter supply points are updated, SPSID is not reliably populated in the same
workflow, causing inconsistent representation of meter connectivity across GIS and
integration interfaces.

### Current Pain Point *(mandatory)*

Users and downstream systems encounter mismatches between utility network connectivity
and SPSID attribute values, and legacy SUPPLY_POINT relation usage adds confusion.

### Target Users / Impacted Systems *(mandatory)*

- GIS users updating meter supply point relationships.
- GIS data owners responsible for connectivity integrity.
- Impacted interfaces: CCMS, CCS, FSS, OFS.

### Desired Future Behaviour *(mandatory)*

Any meter supply point update automatically sets SPSID to the matching new supply point,
and all listed interfaces receive consistent data while outdated SUPPLY_POINT relation is
retired for this workflow.

### In Scope *(mandatory)*

- Automatic SPSID population during meter supply point updates.
- Consistency alignment between utility network connectivity and SPSID attribute.
- Integration updates for CCMS, CCS, FSS, OFS.
- Removal of outdated SUPPLY_POINT relation dependency for this workflow.

### Out of Scope *(mandatory)*

- ADMS workflow, design, or implementation changes.
- Redesign of external systems CCMS, CCS, FSS, OFS.
- Unrelated meter workflows not tied to supply point updates.

### Business Requirements *(mandatory)*

- **BR-001**: When a supply point is updated on a meter, SPSID is automatically
  populated.
- **BR-002**: Update integration interfaces CCMS, CCS, FSS, and OFS for consistent
  downstream data.
- **BR-003**: Remove the outdated SUPPLY_POINT relation for this workflow.
- **BR-004**: Ensure the meter is correctly connected to the new supply point through
  both utility network connectivity and SPSID attribute.

### Dependencies *(mandatory)*

- **DEP-001**: Reliable supply point to SPSID mapping source within GIS scope.
- **DEP-002**: Utility network connectivity model for meter-to-supply-point linkage.
- **DEP-003**: Interface integration channels for CCMS, CCS, FSS, OFS.

### Constraints *(mandatory)*

- **CON-001**: Scope is GIS only.
- **CON-002**: Integration and data consistency requirements must remain explicit.
- **CON-003**: No ADMS scope may be introduced unless explicitly stated as a dependency.
- **CON-004**: Existing GIS operations outside this workflow must not regress.

### Interface / Data Consistency Expectations *(mandatory)*

- **IC-001**: After meter supply point update, SPSID value must represent the same
  supply point as utility network connectivity.
- **IC-002**: CCMS, CCS, FSS, and OFS must receive consistent meter supply point and
  SPSID values for the same update event.
- **IC-003**: If any interface cannot receive an update, the inconsistency must be
  detectable and flagged for correction.
- **IC-004**: SUPPLY_POINT relation must no longer be used as authoritative source for
  this workflow.

### Key Entities *(include if feature involves data)*

- **Meter Record**: GIS meter entity with supply point connectivity and SPSID attribute.
- **Supply Point Mapping**: Authoritative relationship that determines valid SPSID for a
  given supply point update.
- **Integration Update Event**: Outbound data update propagated to CCMS, CCS, FSS, OFS
  for consistency.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In 100% of tested meter supply point update cases with valid mapping,
  SPSID is automatically populated.
- **SC-002**: In 100% of tested successful updates, utility network connectivity and
  SPSID represent the same new supply point.
- **SC-003**: In at least 95% of tested integration update events, CCMS, CCS, FSS, and
  OFS receive consistent SPSID-aligned data without manual reconciliation.
- **SC-004**: Manual correction effort for meter supply point and SPSID inconsistencies
  is reduced by at least 80% compared with the current workflow.

## Assumptions

- GIS has access to current supply point mapping required for SPSID population.
- Interfaces CCMS, CCS, FSS, and OFS continue to consume GIS integration updates.
- Users performing meter supply point updates already have required GIS access rights.
- SUPPLY_POINT relation removal applies to this workflow only and does not block
  unrelated historical access requirements.

## Delivery Layer Mapping *(mandatory)*

Document each major item with an explicit layer label so reviewers can separate intent,
design, build work, and verification:

- **Business Requirement**: Meter must connect correctly to updated supply point through
  utility network connectivity and SPSID attribute.
- **Technical Design**: Populate SPSID automatically on supply point update and propagate
  aligned data through CCMS, CCS, FSS, and OFS.
- **Implementation Task**: Implement update behavior, retire outdated SUPPLY_POINT
  relation dependency, and add integration consistency checks.
- **Test Consideration**: Verify mapping validity handling, cross-interface consistency,
  and non-regression of unrelated GIS workflows.
