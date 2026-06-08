# Feature Specification: Lifecycle Status Update for Tee-Off Cable Segment

**Feature Branch**: `008-update-teeoff-lifecycle`

**Created**: 2026-06-08

**Status**: Draft

**Input**: User description: "Create a specification for one DNOO GIS enhancement item only. Requirement ID: 13 - Lifecycle status update for tee-off cable segment. Business requirement: Modify the GIS_ADMS_INTF_003 interface so that when a line decommission is received, the system checks whether the associated pole has other in-service features before decommissioning it. Business purpose: Prevent incorrect decommissioning of poles that still serve active equipment, which would otherwise cause wrong views in ADMS and incorrect asset status updates to ERP. Please produce: business problem, current pain point, target users / impacted systems, desired future behaviour, in-scope / out-of-scope, assumptions, dependencies, constraints, acceptance criteria, decision logic / validation expectations, edge cases / open questions. Scope notes: GIS only; ADMS and ERP can be referenced only as impacted or dependent systems; keep wording aligned with business requirement above."

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

### User Story 1 - Validate Pole Before Decommission (Priority: P1)

As a GIS integration user, when a line decommission event is received, the system checks
whether the associated pole still has in-service features before changing pole status.

**Why this priority**: This is the core control that prevents incorrect pole
decommissioning.

**Independent Test**: Submit decommission events for poles with and without other
in-service features and verify lifecycle update behavior.

**Acceptance Scenarios**:

1. **Given** a decommission event is received for a line, **When** the associated pole
  has other in-service features, **Then** the pole is not decommissioned.
2. **Given** a decommission event is received for a line, **When** the associated pole
  has no other in-service features, **Then** the pole is decommissioned.

---

### User Story 2 - Preserve Accurate Downstream Views (Priority: P2)

As a downstream consumer stakeholder, I need GIS lifecycle updates to avoid sending
incorrect decommission status to impacted systems.

**Why this priority**: Accurate downstream state prevents wrong ADMS views and ERP asset
status updates.

**Independent Test**: Verify resulting lifecycle status output for events involving shared
active poles and fully inactive poles.

**Acceptance Scenarios**:

1. **Given** a pole remains in service due to other active features, **When** lifecycle
  status is propagated, **Then** impacted systems receive non-decommissioned pole status.

---

### User Story 3 - Improve Decision Transparency (Priority: P3)

As a GIS data owner, I need decision outcomes on pole decommission eligibility to be
traceable for review and reconciliation.

**Why this priority**: Traceability supports data integrity monitoring and issue
resolution.

**Independent Test**: Review processed events and confirm the decision rationale can be
determined from available output state.

**Acceptance Scenarios**:

1. **Given** a line decommission event is processed, **When** the outcome is reviewed,
   **Then** it is clear whether the pole was retained or decommissioned based on active
   feature presence.

---

### Edge Cases

- Associated pole cannot be resolved from the incoming line decommission event.
- Pole has mixed feature states with stale status timestamps.
- Concurrent events update multiple connected features for the same pole.
- In-service feature inventory returns partial data at decision time.
- Lifecycle update is accepted in GIS but not applied by one impacted system.
- Open question: Should the active-feature check include only directly connected
  equipment, or all in-service features associated to the pole in GIS?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST process line decommission events through GIS_ADMS_INTF_003 and
  evaluate associated pole decommission eligibility.
- **FR-002**: System MUST check whether the associated pole has other in-service
  features before decommissioning the pole.
- **FR-003**: System MUST prevent pole decommission when one or more other in-service
  features remain associated with that pole.
- **FR-004**: System MUST allow pole decommission when no other in-service features are
  associated with that pole.
- **FR-005**: System MUST propagate resulting lifecycle status to impacted downstream
  systems as dependency outputs (ADMS and ERP references only).
- **FR-006**: System MUST keep decision behavior within GIS scope and MUST NOT introduce
  ADMS or ERP workflow/design scope.
- **FR-007**: System MUST keep unrelated GIS workflows unchanged.

### Business Problem *(mandatory)*

Current decommission handling can incorrectly decommission poles that still support
active equipment when a line decommission event is received.

### Current Pain Point *(mandatory)*

Incorrect pole status updates create mismatches between real equipment service state and
reported lifecycle state, requiring manual correction and reconciliation.

### Target Users / Impacted Systems *(mandatory)*

- GIS integration and operations users managing lifecycle updates.
- GIS data owners responsible for lifecycle status accuracy.
- Impacted/dependent systems: ADMS and ERP.

### Desired Future Behaviour *(mandatory)*

On each line decommission event, GIS evaluates whether the associated pole still serves
other in-service features and only decommissions poles when no active features remain.

### In Scope *(mandatory)*

- Decision logic in GIS_ADMS_INTF_003 for pole decommission eligibility.
- Active-feature validation for associated poles during line decommission events.
- Lifecycle status output consistency for dependent ADMS/ERP consumption.

### Out of Scope *(mandatory)*

- ADMS or ERP workflow redesign or implementation changes.
- Non-decommission lifecycle workflows outside the described event path.
- Broader asset model redesign unrelated to pole decommission validation.

### Business Requirements *(mandatory)*

- **BR-001**: Modify GIS_ADMS_INTF_003 so line decommission processing checks for other
  in-service features on the associated pole before decommissioning it.
- **BR-002**: Prevent incorrect pole decommissioning that causes wrong views in ADMS.
- **BR-003**: Prevent incorrect asset status updates to ERP caused by invalid pole
  decommission decisions.

### Dependencies *(mandatory)*

- **DEP-001**: GIS data model provides reliable association between line events and poles.
- **DEP-002**: GIS feature status data accurately indicates in-service state.
- **DEP-003**: GIS_ADMS_INTF_003 integration path is available for lifecycle update
  processing.
- **DEP-004**: ADMS and ERP consume lifecycle status as dependent outputs.

### Constraints *(mandatory)*

- **CON-001**: Scope is GIS only.
- **CON-002**: ADMS and ERP are referenced only as impacted/dependent systems.
- **CON-003**: Wording and logic remain aligned with stated business requirement.
- **CON-004**: Existing GIS processes outside this event path must not regress.

### Decision Logic / Validation Expectations *(mandatory)*

- **DL-001**: For each line decommission event, identify associated pole.
- **DL-002**: Evaluate whether any other features associated to that pole are in-service.
- **DL-003**: If one or more in-service features exist, do not decommission the pole.
- **DL-004**: If no in-service features exist, decommission the pole.
- **DL-005**: If association or status evidence is unavailable, fail safe by preventing
  automatic pole decommission and flagging for review.

### Key Entities *(include if feature involves data)*

- **Line Decommission Event**: Incoming lifecycle update event indicating a line is being
  decommissioned.
- **Associated Pole**: Pole linked to the decommissioned line and evaluated for
  decommission eligibility.
- **In-Service Feature Set**: Collection of features associated with the pole that may
  remain active and block pole decommission.
- **Lifecycle Decision Outcome**: Result indicating pole retained or decommissioned based
  on validation logic.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In 100% of tested events where associated poles have other in-service
  features, pole decommission is prevented.
- **SC-002**: In 100% of tested events where associated poles have no other in-service
  features, pole decommission proceeds.
- **SC-003**: Incorrect pole status updates propagated to dependent ADMS/ERP feeds from
  this event path are reduced by at least 90% versus current baseline.
- **SC-004**: Manual correction cases caused by false pole decommission decisions in this
  workflow are reduced by at least 80%.

## Assumptions

- Feature status data in GIS is sufficiently current to determine in-service state.
- Association between line and pole can be resolved from existing GIS relationships.
- Dependent systems consume lifecycle outputs without requiring scope expansion in those
  systems.
- Users handling exceptions have an operational process to review flagged decisions.

## Delivery Layer Mapping *(mandatory)*

Document each major item with an explicit layer label so reviewers can separate intent,
design, build work, and verification:

- **Business Requirement**: Prevent incorrect pole decommissioning during line
  decommission processing while maintaining accurate dependent status outputs.
- **Technical Design**: Add pole active-feature validation decision logic in
  GIS_ADMS_INTF_003 before applying pole decommission lifecycle state.
- **Implementation Task**: Process event, resolve pole, evaluate in-service associations,
  apply retain/decommission decision, and emit consistent status output.
- **Test Consideration**: Validate retain/decommission decisions, fail-safe behavior,
  dependent output consistency, and non-regression of unrelated GIS workflows.
