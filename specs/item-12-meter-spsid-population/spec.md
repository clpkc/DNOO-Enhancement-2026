# 12 - Meter SPSID Attribute Automatic Population

**Feature Branch**: `item-12-meter-spsid-population`

**Created**: 2026-06-08

**Status**: Draft

**Input**: User description: "Create a specification for one DNOO GIS enhancement item only. Requirement ID: 12 - Meter SPSID attribute automatic population. Business requirement: Update multiple integration interfaces: CCMS, CCS, FSS, OFS. When a supply point is updated on a meter, the SPSID attribute should also be automatically populated. Remove the outdated SUPPLY_POINT relation. Business purpose: Ensure the meter is correctly connected to the new supply point through both utility network connectivity and SPSID attribute. Please produce: business problem, current pain point, target users / impacted systems, desired future behaviour, in-scope / out-of-scope, assumptions, dependencies, constraints, acceptance criteria, interface / data consistency expectations, edge cases / open questions. Scope notes: GIS only. Do not introduce ADMS scope unless it is an explicitly stated dependency. Keep integration and data consistency requirements explicit. Keep the wording aligned with the business requirement above."

## Clarifications

### Session 2026-06-09

- Q: What events trigger SPSID population? → A: Supply point association create, update, and delete — not update only.
- Q: What are the specific interfaces and functions in scope? → A: GIS_CCMS_INTF_001 (CreateUsagePoints), GIS_CCMS_INTF_003 (UpdateUsagePoints), CreateMissingAssociations, Enlight GIS CCS 1, Enlight CreateMissingAssociations, CCMS custom task, GIS_FSS_INTF_001 (Associate Supply Point and Premise), GIS OFS 2.
- Q: What are the two SPSID-related attributes on the Meter? → A: SPSID (free text Supply Point Number, currently not populated) and SUPPLY_POINT (reference attribute, outdated, different from the association used by integrations).
- Q: Is interface update sequencing required across all interfaces? → A: No strict sequencing; each interface/function handles its own SPSID population as part of its own supply point association lifecycle.
- Q: When a supply point association is deleted with no new replacement, what value should SPSID be set to? → A: Cleared to null / empty (field blanked; must not retain a stale value).

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Auto-Populate SPSID on Supply Point Association Create, Update, or Delete (Priority: P1)

As a GIS user managing meter supply point associations, I need SPSID (free text Supply
Point Number) to be automatically populated when the meter's supply point association
is created, updated, or deleted so meter connectivity data remains complete.

**Why this priority**: This is the core business requirement across all three lifecycle
events and prevents the SPSID free text attribute from remaining unpopulated.

**Independent Test**: Create, update, and delete a meter's supply point association and
verify SPSID is automatically set (or cleared on delete) in each case.

**Acceptance Scenarios**:

1. **Given** a meter with a supply point association, **When** the association is created
   or updated to a new supply point, **Then** SPSID is automatically populated with the
   SPSID of the associated Supply Point.
2. **Given** a meter with a supply point association, **When** the association is deleted
   with no replacement, **Then** SPSID on the meter is cleared to null / empty.
3. **Given** a successful supply point association event, **When** the meter record is
   reviewed, **Then** utility network connectivity and SPSID reflect the same supply
   point.

---

### User Story 2 - Keep Named Integration Interfaces Consistent (Priority: P2)

As an integration stakeholder, I need the named CCMS, CCS, FSS, and OFS interfaces and
functions to receive consistent SPSID-aligned meter updates for all supply point
association lifecycle events.

**Why this priority**: Interface consistency across all named functions is required to
avoid mismatched downstream meter/supply-point states.

**Independent Test**: Perform supply point association create, update, and delete events
and verify each named interface (GIS_CCMS_INTF_001, GIS_CCMS_INTF_003,
CreateMissingAssociations, Enlight GIS CCS 1, Enlight CreateMissingAssociations, CCMS
custom task, GIS_FSS_INTF_001, GIS OFS 2) receives consistent SPSID-aligned data.

**Acceptance Scenarios**:

1. **Given** a supply point association lifecycle event (create, update, or delete),
   **When** data is transmitted through the named interfaces and functions, **Then** each
   reflects the updated SPSID and aligned meter connectivity state.
2. **Given** a Premise has an association to a Supply Point (Create Missing Associations
   logic), **When** processed, **Then** the association is created for all meters at that
   Premise, conflicting Supply Point associations are deleted, and SPSID is populated on
   each meter from the associated Supply Point's SPSID.

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

- Meter supply point association is deleted with no new association: SPSID on the meter
  must be cleared to null / empty; it must not retain a stale value.
- Meter supply point association is created or updated to a supply point with no valid
  SPSID value: the outcome must be flagged for correction.
- Existing meter has stale SPSID that conflicts with current utility network connectivity.
- Create Missing Associations runs against a Premise where some meters already have a
  conflicting Supply Point association: conflicting associations must be deleted before
  SPSID is populated.
- Multiple association lifecycle events are applied to the same meter in a short interval.
- Removal of SUPPLY_POINT relation attribute affects legacy reporting references outside
  this workflow.

### Error Handling

- **EH-001**: When a supply point association is deleted with no replacement, SPSID on
  the Meter is cleared to null / empty; it is never left with the previous stale value.
- **EH-002**: When a supply point association is created or updated but the Supply Point
  has no valid SPSID value, the update is flagged for correction; the Meter SPSID is not
  silently set to null or a partial value.
- **EH-003**: When Create Missing Associations runs against a Premise where a meter
  already has a conflicting Supply Point association, the conflicting association is
  deleted before the new association and SPSID are populated; the meter is not left with
  two competing associations.
- **EH-004**: When an interface update event cannot be delivered to one or more of the
  named interfaces, the inconsistency is flagged and detectable; silent partial-update
  states are not permitted.
- **EH-005**: When an existing Meter has a stale SPSID that conflicts with current
  utility network connectivity, the inconsistency is detectable and correctable through
  the same supply point association lifecycle workflow.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST automatically populate SPSID (free text Supply Point Number)
  on the Meter when the supply point association is created, updated, or deleted.
- **FR-002**: System MUST populate SPSID with the SPSID value from the associated Supply
  Point when an association is created or updated.
- **FR-003**: System MUST clear SPSID to null / empty when a supply point association is
  deleted with no replacement; the attribute must not retain a stale value.
- **FR-004**: System MUST ensure populated SPSID corresponds to the new supply point
  connection in utility network connectivity.
- **FR-005**: System MUST update the following interfaces and functions with the updated
  meter and SPSID state: GIS_CCMS_INTF_001 (CreateUsagePoints),
  GIS_CCMS_INTF_003 (UpdateUsagePoints), CreateMissingAssociations,
  Enlight GIS CCS 1, Enlight CreateMissingAssociations, CCMS custom task,
  GIS_FSS_INTF_001 (Associate Supply Point and Premise), GIS OFS 2.
- **FR-006**: System MUST implement Create Missing Associations logic: when a Premise has
  an association to a Supply Point, create the association for all meters at that Premise;
  delete any conflicting Supply Point associations on those meters; and populate SPSID
  on each meter from the associated Supply Point's SPSID.
- **FR-007**: System MUST remove the SUPPLY_POINT relation attribute dependency for this
  workflow; SUPPLY_POINT is considered outdated and differs from the integration
  association.
- **FR-008**: System MUST flag or reject updates where the supply point to SPSID mapping
  is invalid or unavailable.
- **FR-009**: System MUST preserve consistent meter-to-supply-point representation across
  utility network connectivity and SPSID attribute.
- **FR-010**: System MUST keep scope limited to GIS integration/data consistency and
  MUST NOT introduce ADMS workflow or design scope.
- **FR-011**: System MUST keep unrelated GIS workflows unchanged.

### Business Problem *(mandatory)*

When supply point associations are created, updated, or deleted on a meter, SPSID (free
text Supply Point Number) is not reliably populated in the same workflow, causing
inconsistent representation of meter connectivity across GIS and integration interfaces.
The SUPPLY_POINT relation attribute is outdated and differs from the association used by
integrations, adding further confusion.

### Current Pain Point *(mandatory)*

SPSID (free text Supply Point Number) on the Meter is not populated, causing mismatches
between utility network connectivity and SPSID attribute values. The SUPPLY_POINT
relation attribute (a separate reference, different from the integration association)
adds legacy confusion that must be retired.

### Target Users / Impacted Systems *(mandatory)*

- GIS users managing meter supply point associations (create, update, delete).
- GIS data owners responsible for connectivity integrity.
- Impacted interfaces and functions: GIS_CCMS_INTF_001, GIS_CCMS_INTF_003,
  CreateMissingAssociations, Enlight GIS CCS 1, Enlight CreateMissingAssociations,
  CCMS custom task, GIS_FSS_INTF_001, GIS OFS 2.

### Desired Future Behaviour *(mandatory)*

Any supply point association create, update, or delete on a Meter automatically sets
SPSID (free text Supply Point Number) to the matching supply point value. All named
interfaces and functions receive consistent data. The outdated SUPPLY_POINT relation
attribute is retired from this workflow.

### In Scope *(mandatory)*

- Automatic SPSID (free text Supply Point Number) population during supply point
  association create, update, and delete on the Meter.
- Consistency alignment between utility network connectivity and SPSID attribute.
- Updates to: GIS_CCMS_INTF_001, GIS_CCMS_INTF_003, CreateMissingAssociations,
  Enlight GIS CCS 1, Enlight CreateMissingAssociations, CCMS custom task,
  GIS_FSS_INTF_001, GIS OFS 2.
- Removal of SUPPLY_POINT relation attribute dependency for this workflow.

### Out of Scope *(mandatory)*

- ADMS workflow, design, or implementation changes.
- Redesign of external systems CCMS, CCS, FSS, OFS.
- Unrelated meter workflows not tied to supply point updates.

### Business Requirements *(mandatory)*

- **BR-001**: When the supply point association on a meter is created, updated, or
  deleted, SPSID (free text Supply Point Number) is automatically populated accordingly.
- **BR-002**: Update the named interfaces and functions (GIS_CCMS_INTF_001,
  GIS_CCMS_INTF_003, CreateMissingAssociations, Enlight GIS CCS 1, Enlight
  CreateMissingAssociations, CCMS custom task, GIS_FSS_INTF_001, GIS OFS 2) for
  consistent downstream data.
- **BR-003**: Remove the SUPPLY_POINT relation attribute for this workflow; it is
  outdated and different from the association used by integrations.
- **BR-004**: Ensure the meter is correctly connected to the supply point through both
  utility network connectivity and SPSID attribute.

### Dependencies *(mandatory)*

- **DEP-001**: Reliable supply point to SPSID mapping source within GIS scope;
  SPSID must be available on the Supply Point for population onto the Meter.
- **DEP-002**: Utility network connectivity model for meter-to-supply-point linkage.
- **DEP-003**: Integration channels available for all named interfaces and functions:
  GIS_CCMS_INTF_001, GIS_CCMS_INTF_003, CreateMissingAssociations,
  Enlight GIS CCS 1, Enlight CreateMissingAssociations, CCMS custom task,
  GIS_FSS_INTF_001, GIS OFS 2.

### Constraints *(mandatory)*

- **CON-001**: Scope is GIS only.
- **CON-002**: Integration and data consistency requirements must remain explicit.
- **CON-003**: No ADMS scope may be introduced unless explicitly stated as a dependency.
- **CON-004**: Existing GIS operations outside this workflow must not regress.

### Interface / Data Consistency Expectations *(mandatory)*

- **IC-001**: After any supply point association create or update on a Meter, SPSID must
  represent the same supply point as utility network connectivity. On association delete
  with no replacement, SPSID must be cleared to null / empty.
- **IC-002**: All named interfaces and functions must receive consistent Meter supply
  point and SPSID values for the same association lifecycle event.
- **IC-003**: Create Missing Associations logic: when a Premise is associated to a
  Supply Point, all meters at that Premise must have the association created, any
  conflicting Supply Point associations deleted, and SPSID populated from the Supply
  Point's SPSID.
- **IC-004**: If any interface cannot receive an update, the inconsistency must be
  detectable and flagged for correction.
- **IC-005**: SUPPLY_POINT relation attribute must no longer be used as authoritative
  source for this workflow; it is separate from and different to the integration
  association.

### Key Entities *(include if feature involves data)*

- **Meter (Electric Junction Object)**: GIS meter entity with supply point connectivity,
  SPSID attribute (free text Supply Point Number, currently not populated), and
  SUPPLY_POINT relation attribute (outdated, to be retired from this workflow).
- **Supply Point**: Source of the SPSID value used to populate the Meter's SPSID
  attribute.
- **Supply Point Association**: The integration association (create/update/delete) that
  triggers SPSID population; distinct from the outdated SUPPLY_POINT relation attribute.
- **Premise**: Spatial or logical grouping; when associated to a Supply Point, triggers
  Create Missing Associations logic for all meters at that Premise.
- **Integration Update Event**: Outbound data update propagated to named interfaces and
  functions for consistency.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In 100% of tested supply point association create and update cases with
  valid SPSID mapping, SPSID is automatically populated on the Meter.
- **SC-002**: In 100% of tested supply point association delete cases, SPSID on the
  Meter is cleared to null / empty and no stale value remains.
- **SC-003**: In 100% of tested successful events, utility network connectivity and
  SPSID represent the same supply point.
- **SC-004**: In at least 95% of tested integration events, all named interfaces and
  functions receive consistent SPSID-aligned data without manual reconciliation.
- **SC-005**: Manual correction effort for Meter supply point and SPSID inconsistencies
  is reduced by at least 80% compared with the current workflow.

## Assumptions

- GIS has access to the Supply Point's SPSID value at association create/update/delete
  time for automatic population onto the Meter.
- The named interfaces and functions continue to consume GIS integration events for all
  three supply point association lifecycle events.
- Users performing supply point association changes already have required GIS access
  rights.
- SUPPLY_POINT relation attribute removal applies to this workflow only and does not
  block unrelated historical access requirements.
- No strict sequencing is required across the named interfaces; each handles its own
  SPSID population as part of its supply point association lifecycle.

## Delivery Layer Mapping *(mandatory)*

Document each major item with an explicit layer label so reviewers can separate intent,
design, build work, and verification:

- **Business Requirement**: Meter must reflect supply point changes through automatic
  SPSID population for all association lifecycle events (create, update, delete); the
  SUPPLY_POINT relation attribute is retired from this workflow.
- **Technical Design**: On supply point association create/update, populate SPSID on the
  Meter from the Supply Point's SPSID; on delete, update SPSID accordingly. Apply Create
  Missing Associations logic for Premise-level cascades. Retire SUPPLY_POINT relation
  attribute. Propagate consistently through all named interfaces and functions.
- **Implementation Task**: Implement SPSID population for all three lifecycle events
  across GIS_CCMS_INTF_001, GIS_CCMS_INTF_003, CreateMissingAssociations, Enlight GIS
  CCS 1, Enlight CreateMissingAssociations, CCMS custom task, GIS_FSS_INTF_001, and
  GIS OFS 2; retire SUPPLY_POINT relation attribute dependency.
- **Test Consideration**: Verify SPSID population for create/update/delete events,
  delete-then-clear behavior, Create Missing Associations cascade logic (including
  conflicting association removal), cross-interface consistency for all named functions,
  and non-regression of unrelated GIS workflows.
