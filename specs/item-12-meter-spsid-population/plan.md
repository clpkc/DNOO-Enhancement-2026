# Implementation Plan: 12 - Meter SPSID Attribute Automatic Population

**Branch**: `item-12-meter-spsid-population` | **Date**: 2026-06-09 | **Spec**: [spec.md](spec.md)

---

## Summary

Implement automatic SPSID (free text Supply Point Number) population on the Meter for
all three supply point association lifecycle events (create, update, delete) across eight
named GIS interfaces and functions. On delete with no replacement, SPSID must be cleared
to null / empty. Retire the SUPPLY_POINT relation attribute from this workflow. No ADMS
scope. All changes are GIS-only.

---

## Constitution Check

| Gate | Status | Notes |
|------|--------|-------|
| Scope gate — GIS only | ✅ Pass | All interfaces named are GIS integrations; no ADMS scope |
| Clarity gate — requirements documented | ✅ Pass | All BRs, FRs, ICs, data entities, and EH entries explicit in spec |
| Data integrity gate — integrity controls defined | ✅ Pass | EH-001–005; IC-001–005; conflicting association deletion before write |
| Regression gate — non-regression activities defined | ✅ Pass | FR-011 / CON-004; non-regression listed in Test Approach |
| Layering gate — requirement/design/build/test separated | ✅ Pass | Delivery Layer Mapping in spec; sections separated in this plan |

---

## Technical Context

| Item | Value |
|------|-------|
| Platform | GIS |
| Key entity | Meter (Electric Junction Object) |
| Trigger events | Supply point association create, update, delete |
| SPSID attribute | Free text Supply Point Number on Meter (currently not populated) |
| SUPPLY_POINT relation | Outdated relation attribute — to be retired from this workflow |
| Supply Point | Source of SPSID value for population onto Meter |
| Premise | Container for meters; triggers Create Missing Associations cascade |
| Affected interfaces | GIS_CCMS_INTF_001, GIS_CCMS_INTF_003, CreateMissingAssociations, Enlight GIS CCS 1, Enlight CreateMissingAssociations, CCMS custom task, GIS_FSS_INTF_001, GIS OFS 2 |
| No strict sequencing | Each interface handles its own SPSID population independently |

---

## Implementation Approach

### Step 1 — SPSID Population on Association Create or Update
- On supply point association **create** or **update** on a Meter:
  - Retrieve the SPSID value from the associated Supply Point (DEP-001).
  - Set the Meter's SPSID attribute to that value (FR-001 / FR-002).
  - Ensure SPSID matches the same supply point as utility network connectivity (FR-004 / IC-001).
- If the Supply Point has no valid SPSID value: flag for correction; do not silently
  set null or a partial value (EH-002 / FR-008).

### Step 2 — SPSID Clearing on Association Delete
- On supply point association **delete** with no replacement:
  - Set the Meter's SPSID attribute to null / empty (FR-003 / IC-001 / EH-001).
  - Must not retain the stale previous value.
- This step applies within each named interface that processes the delete event.

### Step 3 — Create Missing Associations Logic (FR-006 / IC-003)
Implement for `CreateMissingAssociations` and `Enlight CreateMissingAssociations`:
1. When a Premise is associated to a Supply Point:
   a. Identify all Meters at that Premise.
   b. Delete any conflicting Supply Point associations on those Meters **before** writing
      the new association (EH-003).
   c. Create the supply point association for each Meter at the Premise.
   d. Populate SPSID on each Meter from the Supply Point's SPSID.
- Order is mandatory: conflicting association deletion → new association creation →
  SPSID population.

### Step 4 — Apply SPSID Logic Across All Named Interfaces and Functions
Apply Steps 1–3 as applicable to each named interface/function:

| Interface / Function | Applicable Events |
|---------------------|------------------|
| GIS_CCMS_INTF_001 — CreateUsagePoints | Create |
| GIS_CCMS_INTF_003 — UpdateUsagePoints | Update |
| CreateMissingAssociations | Create (with cascade per FR-006) |
| Enlight GIS CCS 1 | Create / Update |
| Enlight CreateMissingAssociations | Create (with cascade per FR-006) |
| CCMS custom task | Create / Update / Delete |
| GIS_FSS_INTF_001 — Associate Supply Point and Premise | Create / Update |
| GIS OFS 2 | Update (supply point association on Electric Junction Object) |

Each interface handles its own SPSID population independently (no cross-interface
sequencing required — IC-002 / Assumption).

### Step 5 — Retire SUPPLY_POINT Relation Attribute
- Remove the dependency on the SUPPLY_POINT relation attribute from all interfaces and
  functions in scope (FR-007 / IC-005).
- The SUPPLY_POINT relation attribute is outdated and differs from the integration
  association; it must not be used as authoritative source in this workflow.
- Verify no in-scope workflow logic reads or writes SUPPLY_POINT after retirement.

### Step 6 — Interface Inconsistency Flagging
- If any named interface cannot deliver its update event, the inconsistency must be
  detectable and flagged (EH-004 / IC-004).
- Silent partial-update states are not permitted.

### Step 7 — Non-Regression
- Unrelated GIS workflows must not be affected (FR-011 / CON-004).
- Verify SUPPLY_POINT removal does not break unrelated historical access paths outside
  this workflow (Assumption).

---

## Impacted Data and Interfaces

| Item | Impact |
|------|--------|
| Meter (Electric Junction Object) — SPSID attribute | Written on create/update; cleared on delete |
| Meter — SUPPLY_POINT relation attribute | Retired from this workflow; dependency removed |
| Supply Point — SPSID value | Read as source for Meter SPSID population |
| Supply Point Association | Trigger for all three lifecycle events |
| Premise | Read for Create Missing Associations cascade logic |
| GIS_CCMS_INTF_001 — CreateUsagePoints | Updated to populate SPSID on create |
| GIS_CCMS_INTF_003 — UpdateUsagePoints | Updated to populate SPSID on update |
| CreateMissingAssociations | Updated with cascade logic + SPSID population |
| Enlight GIS CCS 1 | Updated to populate SPSID on association lifecycle |
| Enlight CreateMissingAssociations | Updated with cascade logic + SPSID population |
| CCMS custom task | Updated for all three lifecycle events |
| GIS_FSS_INTF_001 — Associate Supply Point and Premise | Updated to populate SPSID |
| GIS OFS 2 | Updated to populate SPSID on Electric Junction Object update |
| Existing unrelated GIS workflows | Must not regress |

---

## Dependencies

| ID | Dependency | Risk if Unavailable |
|----|-----------|-------------------|
| DEP-001 | Supply Point SPSID value available in GIS at association event time | SPSID cannot be populated; EH-002 flagging activates |
| DEP-002 | Utility network connectivity model for meter-to-supply-point linkage | IC-001 consistency check fails; connectivity and SPSID cannot align |
| DEP-003 | All named integration channels available and consuming GIS events | Interface consistency (IC-002 / EH-004) breaks; partial-update state possible |

---

## Assumptions

- GIS has access to the Supply Point's SPSID value at association create/update/delete
  time for automatic population onto the Meter.
- Each named interface/function independently handles its own SPSID population as part
  of its supply point association lifecycle; no cross-interface sequencing is required.
- SUPPLY_POINT relation attribute removal applies to this workflow only and does not
  block unrelated historical access requirements outside this workflow.
- Users performing supply point association changes already have required GIS access
  rights.
- Named integration channels remain available and continue consuming GIS events for all
  three supply point association lifecycle events.

---

## Constraints

| ID | Constraint |
|----|-----------|
| CON-001 | Scope is GIS only |
| CON-002 | Integration and data consistency requirements must remain explicit |
| CON-003 | No ADMS scope |
| CON-004 | Existing GIS operations outside this workflow must not regress |

---

## Environments

| Environment | Purpose |
|-------------|---------|
| Development | GIS environment with all named interfaces available; Meter, Supply Point, Premise test data |
| Test / UAT | Scenarios for create / update / delete lifecycle; Create Missing Associations cascade; conflicting association cases; Supply Point with/without valid SPSID; interface unavailability simulation |
| Production | Live GIS environment; deployed via standard GIS release process |

---

## Test Approach and Validation Evidence

| Test Case | Expected Evidence |
|-----------|------------------|
| Association create — valid SPSID | Meter SPSID set to Supply Point's SPSID (SC-001 / FR-002) |
| Association update — valid SPSID | Meter SPSID updated to new Supply Point's SPSID (SC-001 / FR-002) |
| Association delete — no replacement | Meter SPSID cleared to null / empty; no stale value (SC-002 / FR-003 / EH-001) |
| Association create — Supply Point has no valid SPSID | Update flagged for correction; Meter SPSID not silently set to null (EH-002 / FR-008) |
| Utility network + SPSID alignment | Meter SPSID and connectivity reference same supply point (SC-003 / FR-004 / IC-001) |
| Create Missing Associations — Premise with clean meters | All meters at Premise get association and SPSID populated (FR-006 / IC-003) |
| Create Missing Associations — conflicting association on meter | Conflicting association deleted first; then new association + SPSID written (EH-003 / FR-006) |
| GIS_CCMS_INTF_001 (CreateUsagePoints) | SPSID-aligned data received on create event (FR-005) |
| GIS_CCMS_INTF_003 (UpdateUsagePoints) | SPSID-aligned data received on update event (FR-005) |
| Enlight GIS CCS 1 | SPSID-aligned data received (FR-005) |
| Enlight CreateMissingAssociations | Cascade + SPSID data received (FR-005 / FR-006) |
| CCMS custom task | All three lifecycle events produce consistent SPSID (FR-005) |
| GIS_FSS_INTF_001 (Associate Supply Point and Premise) | SPSID-aligned data received (FR-005) |
| GIS OFS 2 | Meter SPSID updated on Electric Junction Object (FR-005) |
| SUPPLY_POINT relation not used | No interface/function reads or writes SUPPLY_POINT in workflow (FR-007 / IC-005) |
| Interface unavailable | Inconsistency detected and flagged; no silent partial-update (EH-004 / IC-004) |
| Cross-interface consistency (≥95%) | All named interfaces receive consistent SPSID-aligned data for same event (SC-004) |
| Non-regression — unrelated GIS workflows | No change in behavior (FR-011 / CON-004) |

---

## Open Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| Supply Point missing valid SPSID in production GIS data | Medium | High | EH-002 flagging mandatory; data quality pre-check recommended before rollout |
| SUPPLY_POINT relation removal has undiscovered references in legacy reporting or external tooling | Low | Medium | Audit all SUPPLY_POINT references before retirement; confirm Assumption holds |
| Partial-update state if one named interface is temporarily unavailable at event time | Medium | Medium | EH-004 / IC-004 detection and flagging required; retry/reconciliation approach to be confirmed in Detailed Design if needed |
| Conflicting associations on multiple meters at same Premise in rapid succession | Low | Medium | EH-003 ordering (delete → create → populate) must be transaction-safe; confirm with GIS data model capability |
