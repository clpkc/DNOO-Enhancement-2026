# Feature Specification: Lifecycle Status Update for Tee-Off Cable Segment

**Feature Branch**: `008-update-teeoff-lifecycle`

**Created**: 2026-06-08

**Status**: Draft

**Input**: User description: "Create a specification for one DNOO GIS enhancement item only. Requirement ID: 13 - Lifecycle status update for tee-off cable segment. Business requirement: Modify the GIS_ADMS_INTF_003 interface so that when a line decommission is received, the system checks whether the associated pole has other in-service features before decommissioning it. Business purpose: Prevent incorrect decommissioning of poles that still serve active equipment, which would otherwise cause wrong views in ADMS and incorrect asset status updates to ERP. Please produce: business problem, current pain point, target users / impacted systems, desired future behaviour, in-scope / out-of-scope, assumptions, dependencies, constraints, acceptance criteria, decision logic / validation expectations, edge cases / open questions. Scope notes: GIS only; ADMS and ERP can be referenced only as impacted or dependent systems; keep wording aligned with business requirement above."

## Clarifications

### Session 2026-06-09

- Q: What is the first action taken on the Electric Line when a decommission event arrives? → A: GIS_Status of the Electric Line is set to Decommission for all asset groups/types before pole evaluation.
- Q: How are poles identified from the line event? → A: Via containment association; poles are StructureJunction features (Asset Group 90, Asset Types 731 HV Pole / 732 LV Pole). Start/end point lookup is not used.
- Q: Which lifecycle status values on associated elements block pole decommission? → A: GIS_Status / LifecycleStatus = In Service OR Planned Uninstalled.
- Q: How is the lifecycle update posted? → A: Via ESRI versioning; not through the GDBM Queue.

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

### User Story 1 - Set Electric Line Status to Decommission and Evaluate Associated Poles (Priority: P1)

As a GIS integration user, when a line decommission event is received, the system first
sets GIS_Status = Decommission on the ElectricLine (all asset groups/types), then
identifies associated StructureJunction poles via containment association and evaluates
each for decommission eligibility.

**Why this priority**: The two-step sequence (line status update → pole validation) is
the core behavior and directly prevents incorrect pole decommissioning.

**Independent Test**: Submit a decommission event for an ElectricLine and verify
GIS_Status is set to Decommission on the line. Then verify that associated
StructureJunction poles (Asset Types 731/732) are evaluated via containment association.

**Acceptance Scenarios**:

1. **Given** a decommission event is received for an ElectricLine, **When** processed,
   **Then** GIS_Status of the ElectricLine is set to Decommission for all asset
   groups/types.
2. **Given** the ElectricLine's GIS_Status is set, **When** the system evaluates
   associated StructureJunction poles via containment association, **Then** each pole
   is checked for in-service or Planned Uninstalled associated elements before
   decommissioning.
3. **Given** an associated StructureJunction pole has at least one element with
   GIS_Status / LifecycleStatus = In Service or Planned Uninstalled, **When**
   evaluated, **Then** the pole is not decommissioned.
4. **Given** an associated StructureJunction pole has no elements with GIS_Status /
   LifecycleStatus = In Service or Planned Uninstalled, **When** evaluated, **Then**
   the pole's GIS_Status is set to Decommission.

---

### User Story 2 - Preserve Accurate Downstream Views (Priority: P2)

As a downstream consumer stakeholder, I need GIS lifecycle updates posted via ESRI
versioning to avoid sending incorrect decommission status to impacted systems.

**Why this priority**: Accurate downstream state prevents wrong ADMS views and ERP asset
status updates.

**Independent Test**: Verify resulting lifecycle status output for events involving shared
active StructureJunction poles (with In Service or Planned Uninstalled elements) and
fully inactive poles.

**Acceptance Scenarios**:

1. **Given** a StructureJunction pole retains elements with GIS_Status = In Service or
   Planned Uninstalled, **When** lifecycle status is posted via ESRI versioning, **Then**
   impacted systems receive non-decommissioned pole status.

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

- Associated StructureJunction pole cannot be resolved via containment association from
  the incoming line decommission event: fail safe — do not decommission the pole;
  flag for review.
- Pole has mixed element states including both Planned Uninstalled and Decommissioned
  elements: Planned Uninstalled elements count as blocking; pole is not decommissioned.
- Pole has elements with stale or unresolved GIS_Status: fail safe — treat as blocking
  to prevent incorrect decommission.
- Concurrent events update multiple ElectricLine features associated to the same
  StructureJunction pole.
- Lifecycle update is accepted in GIS via ESRI versioning but not yet consumed by one
  impacted downstream system.
- ElectricLine asset group or asset type is an unusual or rarely tested combination:
  the update applies to all ElectricLine asset groups and asset types.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST process line decommission events through GIS_ADMS_INTF_003.
- **FR-002**: System MUST set GIS_Status = Decommission on the ElectricLine (all asset
  groups and asset types) when a decommission event is received.
- **FR-003**: System MUST identify associated StructureJunction poles using containment
  association; start/end point lookup MUST NOT be used.
- **FR-004**: System MUST evaluate each identified StructureJunction pole
  (Asset Group 90 — Support Structure; Asset Types 731 HV Pole and 732 LV Pole) for
  decommission eligibility.
- **FR-005**: System MUST prevent pole decommission when one or more elements associated
  with the pole have GIS_Status / LifecycleStatus = In Service or Planned Uninstalled.
- **FR-006**: System MUST set GIS_Status = Decommission on the pole when no associated
  elements have GIS_Status / LifecycleStatus = In Service or Planned Uninstalled.
- **FR-007**: System MUST post lifecycle status updates via ESRI versioning; updates
  MUST NOT be posted through the GDBM Queue.
- **FR-008**: System MUST fail safe when the containment association cannot be resolved:
  do not decommission the pole and flag for review.
- **FR-009**: System MUST propagate resulting lifecycle status to impacted downstream
  systems as dependency outputs (ADMS and ERP references only).
- **FR-010**: System MUST keep decision behavior within GIS scope and MUST NOT introduce
  ADMS or ERP workflow/design scope.
- **FR-011**: System MUST keep unrelated GIS workflows unchanged.

### Business Problem *(mandatory)*

Current decommission handling can incorrectly decommission StructureJunction poles
(HV Pole / LV Pole) that still support active or planned-uninstalled equipment when a
line decommission event is received.

### Current Pain Point *(mandatory)*

Incorrect pole status updates create mismatches between real equipment service state and
reported lifecycle state, requiring manual correction and reconciliation.

### Target Users / Impacted Systems *(mandatory)*

- GIS integration and operations users managing lifecycle updates.
- GIS data owners responsible for lifecycle status accuracy.
- Impacted/dependent systems: ADMS and ERP.

### Desired Future Behaviour *(mandatory)*

On each line decommission event, GIS sets GIS_Status = Decommission on the ElectricLine
(all asset groups/types), then identifies associated StructureJunction poles via
containment association and evaluates each. A pole is decommissioned only when no
associated elements have GIS_Status / LifecycleStatus = In Service or Planned
Uninstalled. Updates are posted via ESRI versioning.

### In Scope *(mandatory)*

- Setting GIS_Status = Decommission on ElectricLine features (all asset groups/types)
  when a decommission event is received via GIS_ADMS_INTF_003.
- Containment-association-based identification of StructureJunction poles
  (Asset Group 90; Asset Types 731 HV Pole / 732 LV Pole).
- Decision logic for pole decommission eligibility based on associated element
  GIS_Status / LifecycleStatus (In Service or Planned Uninstalled).
- Lifecycle status updates posted via ESRI versioning.
- Lifecycle status output consistency for dependent ADMS/ERP consumption.

### Out of Scope *(mandatory)*

- ADMS or ERP workflow redesign or implementation changes.
- Non-decommission lifecycle workflows outside the described event path.
- Broader asset model redesign unrelated to pole decommission validation.

### Business Requirements *(mandatory)*

- **BR-001**: Modify GIS_ADMS_INTF_003 so that when a decommission event is received,
  GIS_Status = Decommission is set on the ElectricLine (all asset groups/types) and
  associated StructureJunction poles are evaluated before decommissioning.
- **BR-002**: Prevent incorrect pole decommissioning that causes wrong views in ADMS.
- **BR-003**: Prevent incorrect asset status updates to ERP caused by invalid pole
  decommission decisions.

### Dependencies *(mandatory)*

- **DEP-001**: GIS data model provides containment association between ElectricLine
  and StructureJunction poles.
- **DEP-002**: GIS_Status / LifecycleStatus data on associated elements accurately
  indicates In Service or Planned Uninstalled state.
- **DEP-003**: GIS_ADMS_INTF_003 integration path is available for lifecycle update
  processing.
- **DEP-004**: ESRI versioning mechanism is available for posting lifecycle updates;
  GDBM Queue is not used.
- **DEP-005**: ADMS and ERP consume lifecycle status as dependent outputs.

### Constraints *(mandatory)*

- **CON-001**: Scope is GIS only.
- **CON-002**: ADMS and ERP are referenced only as impacted/dependent systems.
- **CON-003**: Lifecycle updates must be posted via ESRI versioning; GDBM Queue must
  not be used.
- **CON-004**: Pole lookup must use containment association; start/end point lookup
  is not used.
- **CON-005**: Existing GIS processes outside this event path must not regress.

### Decision Logic / Validation Expectations *(mandatory)*

- **DL-001**: For each line decommission event, set GIS_Status = Decommission on the
  ElectricLine (all asset groups and asset types).
- **DL-002**: Identify all associated StructureJunction poles using containment
  association (Asset Group 90 — Support Structure; Asset Types 731 HV Pole / 732 LV
  Pole).
- **DL-003**: For each identified pole, evaluate whether any associated element has
  GIS_Status / LifecycleStatus = In Service or Planned Uninstalled.
- **DL-004**: If one or more such elements exist, do not decommission the pole.
- **DL-005**: If no associated elements exist, or all have GIS_Status other than
  In Service or Planned Uninstalled, set the pole's GIS_Status = Decommission.
- **DL-006**: If the containment association cannot be resolved, fail safe: do not
  decommission the pole and flag the record for operational review.
- **DL-007**: Post all lifecycle status changes via ESRI versioning.

### Key Entities *(include if feature involves data)*

- **ElectricLine**: Feature whose GIS_Status is set to Decommission for all asset groups
  and asset types when a decommission event is received.
- **Line Decommission Event**: Incoming lifecycle update event received via
  GIS_ADMS_INTF_003 indicating an ElectricLine is being decommissioned.
- **StructureJunction (Pole)**: Support Structure feature (Asset Group 90; Asset Types
  731 HV Pole / 732 LV Pole) associated to the line via containment association and
  evaluated for decommission eligibility.
- **Containment Association**: GIS relationship used to identify StructureJunction poles
  from an ElectricLine decommission event.
- **Blocking Element**: Any element associated to the pole whose GIS_Status /
  LifecycleStatus = In Service or Planned Uninstalled, which prevents pole
  decommissioning.
- **Lifecycle Decision Outcome**: Result indicating pole retained or decommissioned
  based on blocking element evaluation.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In 100% of tested events, GIS_Status is set to Decommission on the
  ElectricLine (all asset groups/types) when a decommission event is received.
- **SC-002**: In 100% of tested events where an associated StructureJunction pole has
  elements with GIS_Status / LifecycleStatus = In Service or Planned Uninstalled, pole
  decommission is prevented.
- **SC-003**: In 100% of tested events where an associated StructureJunction pole has no
  elements with GIS_Status / LifecycleStatus = In Service or Planned Uninstalled, pole
  GIS_Status is set to Decommission.
- **SC-004**: In 100% of tested events where containment association cannot be resolved,
  pole decommission is prevented and the record is flagged for review.
- **SC-005**: Incorrect pole status updates propagated to dependent ADMS/ERP feeds from
  this event path are reduced by at least 90% versus current baseline.
- **SC-006**: Manual correction cases caused by false pole decommission decisions in this
  workflow are reduced by at least 80%.

## Assumptions

- ElectricLine features belong to all asset groups and asset types; the GIS_Status
  Decommission update applies across the board.
- Containment association in GIS reliably maps ElectricLine features to
  StructureJunction poles (Asset Group 90; Asset Types 731 / 732).
- GIS_Status / LifecycleStatus data on associated elements is sufficiently current to
  determine blocking status (In Service or Planned Uninstalled) at event processing time.
- ESRI versioning is the approved mechanism for posting lifecycle updates; the GDBM
  Queue is not used for this flow.
- Dependent systems consume lifecycle outputs without requiring scope expansion in those
  systems.
- Users handling exceptions have an operational process to review flagged decisions
  where containment association could not be resolved.

## Delivery Layer Mapping *(mandatory)*

Document each major item with an explicit layer label so reviewers can separate intent,
design, build work, and verification:

- **Business Requirement**: When a decommission event is received, set GIS_Status =
  Decommission on the ElectricLine (all asset groups/types) and prevent incorrect
  StructureJunction pole decommissioning while maintaining accurate dependent status
  outputs.
- **Technical Design**: Set ElectricLine GIS_Status, then use containment association to
  identify StructureJunction poles (Asset Group 90, Asset Types 731/732), evaluate each
  for elements with GIS_Status / LifecycleStatus = In Service or Planned Uninstalled,
  apply retain/decommission decision, and post via ESRI versioning (not GDBM Queue).
- **Implementation Task**: Implement event processing, ElectricLine status update,
  containment-based pole identification, blocking-element evaluation, fail-safe handling,
  and ESRI versioning post.
- **Test Consideration**: Validate ElectricLine status update for all asset groups/types,
  retain/decommission decisions for HV Pole and LV Pole, Planned Uninstalled blocking
  behavior, fail-safe on unresolved containment association, ESRI versioning path, and
  non-regression of unrelated GIS workflows.
