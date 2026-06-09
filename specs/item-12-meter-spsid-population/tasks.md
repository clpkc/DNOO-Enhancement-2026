# Tasks: 12 - Meter SPSID Attribute Automatic Population

**Branch**: `item-12-meter-spsid-population` | **Date**: 2026-06-09
**Spec**: [spec.md](spec.md) | **Plan**: [plan.md](plan.md)

---

## Overview

Implement automatic SPSID (free text Supply Point Number) population on the Meter
(Electric Junction Object) for all three supply point association lifecycle events
(create, update, delete) across eight named GIS interfaces and functions. On delete with
no replacement, SPSID is cleared to null / empty. SUPPLY_POINT relation attribute retired
from this workflow. Create Missing Associations cascade logic implemented with mandatory
ordering (delete conflicting → create → populate). No ADMS scope.

---

## Group A — Analysis and Environment Readiness

Tasks in this group can run in parallel. Complete before build work begins.

**A-1** — Confirm the `SPSID` attribute (free text Supply Point Number) is present on
the Meter (Electric Junction Object) feature class in the development GIS schema. Record
attribute name, type, and current population state.
*Expected result*: Attribute confirmed; current unpopulated state documented; DEP-001
pre-check initiated.

**A-2** — Confirm the `SUPPLY_POINT` relation attribute is present on the Meter in
the GIS schema. Audit all references to `SUPPLY_POINT` across the eight named interfaces
and any legacy reporting or external tooling. Document every reference found.
*Expected result*: Full `SUPPLY_POINT` reference inventory; no undiscovered references
remain (Open Risk mitigation).

**A-3** — Confirm the Supply Point SPSID value is accessible at association event time
in GIS (DEP-001). Document the access path (query, relationship, event payload) for
each of the three lifecycle events (create / update / delete).
*Expected result*: SPSID access path documented for all three events; DEP-001 resolved.

**A-4** — Confirm the Premise-to-Meter relationship is available in GIS for the Create
Missing Associations cascade logic (FR-006). Document how all Meters at a given Premise
are identified.
*Expected result*: Premise-to-Meter lookup path confirmed; cascade logic can be
implemented.

**A-5** — Confirm all eight named interfaces and functions are available and accessible
in the development environment (DEP-003):
GIS_CCMS_INTF_001, GIS_CCMS_INTF_003, CreateMissingAssociations, Enlight GIS CCS 1,
Enlight CreateMissingAssociations, CCMS custom task, GIS_FSS_INTF_001, GIS OFS 2.
*Expected result*: All eight confirmed accessible; any unavailable interface escalated.

**A-6** — Obtain or generate test data in the development environment covering:
- Meters with existing supply point associations (create / update / delete scenarios).
- Supply Points with valid SPSID and Supply Points with no valid SPSID (EH-002 test).
- At least one Premise with multiple Meters, including one with a conflicting existing
  supply point association (EH-003 test).
*Expected result*: Test data set confirmed ready.

---

## Group B — SPSID Population: Create and Update

*Depends on A-1, A-3, and A-5 completing first.*

**B-1** — Implement SPSID population on supply point association **create** for
`GIS_CCMS_INTF_001` (CreateUsagePoints):
- On create event: retrieve SPSID from the associated Supply Point; set on Meter (FR-001 /
  FR-002).
- If Supply Point has no valid SPSID: flag for correction; do not silently set null
  (EH-002 / FR-008).
*Expected result*: Meter SPSID populated on create; invalid-SPSID events flagged.

**B-2** — Implement SPSID population on supply point association **update** for
`GIS_CCMS_INTF_003` (UpdateUsagePoints):
- On update event: retrieve new SPSID from Supply Point; overwrite Meter SPSID (FR-001 /
  FR-002).
- If Supply Point has no valid SPSID: flag for correction (EH-002).
*Expected result*: Meter SPSID updated on update event; invalid cases flagged.

**B-3** — Implement SPSID population on association **create / update** for
`Enlight GIS CCS 1` following the same create/update logic as B-1/B-2.
*Expected result*: Consistent SPSID population behavior across Enlight GIS CCS 1.

**B-4** — Implement SPSID population on association **create / update** for
`GIS_FSS_INTF_001` (Associate Supply Point and Premise).
*Expected result*: Consistent SPSID population for FSS create/update events.

**B-5** — Implement SPSID population on association **update** for `GIS OFS 2`
(Electric Junction Object supply point association update).
*Expected result*: Meter SPSID updated via GIS OFS 2 on update event.

**B-6** — Implement SPSID population for all three lifecycle events (create / update /
delete) for `CCMS custom task`.
*Expected result*: CCMS custom task handles all three events with consistent SPSID
behavior.

---

## Group C — SPSID Clearing: Delete

*Can run in parallel with Group B. Depends on A-1 and A-3.*

**C-1** — Implement SPSID clearing on supply point association **delete** (no
replacement) for all interfaces that process delete events:
- Set Meter SPSID to null / empty (FR-003 / IC-001 / EH-001).
- Must not retain the previous stale value.
- Applies at minimum to: CCMS custom task (confirmed delete scope); verify which other
  named interfaces process delete events and apply accordingly.
*Expected result*: Meter SPSID = null/empty after delete; no stale value in any tested
interface.

---

## Group D — Create Missing Associations Cascade Logic

*Depends on A-3, A-4, and A-5 completing first. D-1 and D-2 can run in parallel.*

**D-1** — Implement Create Missing Associations logic for `CreateMissingAssociations`
(FR-006 / IC-003 / EH-003):
1. When a Premise is associated to a Supply Point:
   a. Identify all Meters at the Premise.
   b. **Delete** any conflicting Supply Point associations on those Meters first.
   c. Create the supply point association for each Meter.
   d. Populate SPSID on each Meter from the Supply Point's SPSID.
- Ordering is mandatory: delete conflicting → create → populate (EH-003).
- Transaction safety: confirm the GIS data model supports atomic execution of this
  sequence (Open Risk mitigation).
*Expected result*: Cascade logic executes in correct order; conflicting associations
removed before new ones written.

**D-2** — Implement Create Missing Associations logic for `Enlight CreateMissingAssociations`
using the identical cascade ordering as D-1 (FR-006 / IC-003 / EH-003).
*Expected result*: Identical cascade behavior confirmed for Enlight function.

---

## Group E — SUPPLY_POINT Relation Attribute Retirement

*Depends on A-2 completing first.*

**E-1** — Remove all references to `SUPPLY_POINT` relation attribute from all eight
named interfaces and functions in scope (FR-007 / IC-005). Use the reference inventory
from A-2 as the checklist.
*Expected result*: No in-scope interface/function reads or writes `SUPPLY_POINT` after
this change.

**E-2** — Verify that SUPPLY_POINT retirement does not break any unrelated historical
access paths identified in A-2 (Assumption). Document any unrelated references and
confirm they are outside this workflow's scope.
*Expected result*: No unrelated workflow broken; Assumption confirmed or gap documented.

---

## Group F — Interface Consistency and Flagging

*Depends on Groups B, C, D, E completing. F-1 can run in parallel with E.*

**F-1** — Implement interface inconsistency flagging (EH-004 / IC-004): if any named
interface cannot deliver its update event, the inconsistency must be detectable and
flagged. Silent partial-update states must not be permitted.
*Expected result*: Inconsistency is detectable and observable when any interface fails
to deliver; no silent partial-update state.

---

## Group G — Validation and Test Evidence

*Depends on all Groups B–F completing. G-1 through G-16 can run in parallel.*

**G-1** — **Evidence: Association create — valid SPSID (GIS_CCMS_INTF_001).**
Trigger create event; verify Meter SPSID = Supply Point's SPSID.
*Expected result*: Record confirming SC-001 / FR-002 / B-1.

**G-2** — **Evidence: Association update — valid SPSID (GIS_CCMS_INTF_003).**
Trigger update event; verify Meter SPSID updated to new Supply Point's SPSID.
*Expected result*: Record confirming SC-001 / FR-002 / B-2.

**G-3** — **Evidence: Association delete — SPSID cleared.**
Trigger delete event with no replacement; verify Meter SPSID = null / empty; no stale
value.
*Expected result*: Record confirming SC-002 / FR-003 / EH-001.

**G-4** — **Evidence: Supply Point has no valid SPSID.**
Trigger create/update with Supply Point that has no valid SPSID; verify event flagged
for correction; Meter SPSID not silently null.
*Expected result*: Record confirming EH-002 / FR-008.

**G-5** — **Evidence: Utility network + SPSID alignment.**
After create/update, verify Meter SPSID and utility network connectivity reference the
same supply point.
*Expected result*: Record confirming SC-003 / FR-004 / IC-001.

**G-6** — **Evidence: Create Missing Associations — clean Premise.**
Run cascade on Premise with clean Meters; verify all Meters get association + SPSID.
*Expected result*: Record confirming FR-006 / IC-003 / D-1.

**G-7** — **Evidence: Create Missing Associations — conflicting association.**
Run cascade on Meter with existing conflicting association; verify conflicting association
deleted first, then new association + SPSID written in correct order.
*Expected result*: Record confirming EH-003 / FR-006 ordering.

**G-8** — **Evidence: GIS_CCMS_INTF_001 consistent SPSID data.**
Verify SPSID-aligned data received by GIS_CCMS_INTF_001 on create event.
*Expected result*: Record confirming FR-005.

**G-9** — **Evidence: GIS_CCMS_INTF_003 consistent SPSID data.**
Verify SPSID-aligned data received by GIS_CCMS_INTF_003 on update event.
*Expected result*: Record confirming FR-005.

**G-10** — **Evidence: Enlight GIS CCS 1.**
Verify SPSID-aligned data received by Enlight GIS CCS 1.
*Expected result*: Record confirming FR-005 / B-3.

**G-11** — **Evidence: Enlight CreateMissingAssociations.**
Verify cascade + SPSID data received correctly via Enlight CreateMissingAssociations.
*Expected result*: Record confirming FR-005 / FR-006 / D-2.

**G-12** — **Evidence: CCMS custom task — all three lifecycle events.**
Verify all three events (create / update / delete) produce consistent SPSID via CCMS
custom task.
*Expected result*: Record confirming FR-005 / B-6 / C-1.

**G-13** — **Evidence: GIS_FSS_INTF_001.**
Verify SPSID-aligned data received by GIS_FSS_INTF_001 on create/update.
*Expected result*: Record confirming FR-005 / B-4.

**G-14** — **Evidence: GIS OFS 2.**
Verify Meter SPSID updated via GIS OFS 2 on Electric Junction Object update event.
*Expected result*: Record confirming FR-005 / B-5.

**G-15** — **Evidence: SUPPLY_POINT relation not used.**
Verify no interface or function reads or writes `SUPPLY_POINT` in the workflow after
retirement.
*Expected result*: Record confirming FR-007 / IC-005 / E-1.

**G-16** — **Evidence: Interface unavailable — flagging.**
Simulate one named interface being temporarily unavailable. Verify inconsistency is
detected and flagged; no silent partial-update.
*Expected result*: Record confirming EH-004 / IC-004 / F-1.

**G-17** — **Evidence: Non-regression — unrelated GIS workflows.**
Run regression pass on unrelated GIS workflows. Verify no change in behavior.
*Expected result*: Regression pass record confirming FR-011 / CON-004.

---

## Group H — Deployment Preparation

*Depends on all Group G evidence gathered and approved.*

**H-1** — Package all interface/function changes for deployment to the UAT environment.
Smoke test each of the eight interfaces: trigger one create event per interface and
verify SPSID population.
*Expected result*: All eight interfaces operational in UAT; smoke tests pass.

**H-2** — Prepare release notes covering: SPSID auto-population (all three lifecycle
events), SUPPLY_POINT retirement, Create Missing Associations cascade ordering, eight
named interfaces affected, and what is unchanged (unrelated GIS workflows).
*Expected result*: Release notes reviewed and approved by GIS lead.

**H-3** — Confirm production deployment checklist covers: SPSID attribute availability,
SUPPLY_POINT reference audit completion, all eight interface configurations, and rollback
steps.
*Expected result*: Deployment checklist signed off.

---

## Group I — Documentation Update

*Can run in parallel with Group H.*

**I-1** — Update spec.md Status field from "Draft" to "Ready for Implementation" once
all validation evidence is gathered.
*Expected result*: Spec status reflects implementation readiness.

**I-2** — Record final test evidence file paths and test pass/fail outcomes in the spec
or linked test artifact for BA / QA review.
*Expected result*: Evidence traceable from spec to test records.

---

## Dependency Summary

```
A-1, A-2, A-3, A-4, A-5, A-6   (parallel — no dependencies)
            ↓
B-1, B-2, B-3, B-4, B-5, B-6   (parallel after A-1, A-3, A-5)
C-1                              (parallel with B, after A-1, A-3)
D-1, D-2                         (parallel after A-3, A-4, A-5)
E-1 → E-2                        (sequential after A-2)
            ↓
           F-1                   (after B, C, D, E complete)
            ↓
G-1 through G-17                 (parallel after F-1)
            ↓
H-1, H-2, H-3 / I-1, I-2        (H and I parallel after G approved)
```

---

## Open Items Carried from Plan

| Item | Status | Action Required |
|------|--------|----------------|
| Supply Point SPSID data quality | Risk (Medium/High) | Data quality pre-check in A-3; EH-002 flagging mandatory |
| SUPPLY_POINT undiscovered references | Risk (Low/Medium) | Full audit in A-2 before E-1 |
| Interface partial-update retry/reconciliation | Risk (Medium) | EH-004 flagging implemented in F-1; retry approach to be confirmed if needed |
| Create Missing Associations transaction safety | Risk (Low/Medium) | Confirm atomic execution capability in A-4 before D-1 |
