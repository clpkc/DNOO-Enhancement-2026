# Tasks: 13 - Lifecycle Status Update for Tee-Off Cable Segment

**Branch**: `item-13-tee-off-cable-lifecycle` | **Date**: 2026-06-09
**Spec**: [spec.md](spec.md) | **Plan**: [plan.md](plan.md)

---

## Overview

Modify `GIS_ADMS_INTF_003` to enforce a two-step lifecycle decision on line decommission
events: (1) set GIS_Status = Decommission on the ElectricLine (all asset groups/types),
then (2) identify associated StructureJunction poles via containment association and
evaluate each for decommission eligibility. A pole is decommissioned only when no
associated elements carry GIS_Status / LifecycleStatus = In Service or Planned
Uninstalled. Updates are posted via ESRI versioning (not GDBM Queue). GIS-only scope.

> ⚠️ **Open item — EH-004**: Concurrent decommission events targeting multiple
> ElectricLine features sharing the same StructureJunction pole are deferred to Detailed
> Design. The test case for that scenario is explicitly excluded from Group G and must be
> added once EH-004 is resolved.

---

## Group A — Analysis and Environment Readiness

Tasks in this group can run in parallel. Complete before build work begins.

**A-1** — Confirm `GIS_ADMS_INTF_003` is accessible in the development environment and
that the line decommission event payload can be received and inspected. Document the
event payload structure and the ElectricLine feature reference field.
*Expected result*: Interface confirmed accessible; event payload structure documented
(DEP-003).

**A-2** — Confirm the containment association between ElectricLine and StructureJunction
is present in the GIS data model in the development environment (DEP-001). Document the
association name, direction, and how to query it from an ElectricLine instance.
*Expected result*: Containment association confirmed; query path documented.

**A-3** — Confirm the `GIS_Status` attribute is present and writable on ElectricLine
features (all asset groups and asset types) and on StructureJunction (Asset Group 90;
Asset Types 731 HV Pole / 732 LV Pole) in the development schema. Document field name,
allowable values, and current Decommission status code.
*Expected result*: GIS_Status attribute confirmed on both entity types; Decommission
code value documented.

**A-4** — Confirm the `GIS_Status` / `LifecycleStatus` attribute on elements associated
to StructureJunction poles is readable at event processing time (DEP-002). Identify
which entity types can be associated to poles and which attribute name (`GIS_Status` vs
`LifecycleStatus`) is authoritative per entity type.
*Expected result*: Blocking status attribute confirmed readable; entity types and
attribute names documented.

**A-5** — Confirm ESRI versioning is available and is the approved write mechanism for
lifecycle status changes in the development environment (DEP-004). Confirm the GDBM
Queue is explicitly not used for this event path.
*Expected result*: ESRI versioning confirmed available; GDBM exclusion confirmed.

**A-6** — Obtain or generate test data in the development environment covering:
- An ElectricLine with multiple asset groups and asset types.
- StructureJunction poles (HV Pole 731 and LV Pole 732) linked via containment
  association to the line.
- Poles with: at least one In Service element; at least one Planned Uninstalled element;
  mixed states (Planned Uninstalled + Decommissioned); all elements at non-blocking
  status; no associated elements.
- A line where the containment association cannot be resolved (for fail-safe testing).
- A line associated to multiple poles with mixed eligibility.
*Expected result*: Test data set confirmed and documented; covers all DL rules and EH
cases.

---

## Group B — ElectricLine Status Update

*Depends on A-1 and A-3 completing first.*

**B-1** — Implement Step 1 of the event processor in `GIS_ADMS_INTF_003`: receive the
line decommission event and extract the ElectricLine feature reference from the event
payload (FR-001).
*Expected result*: Event received and ElectricLine feature reference correctly
identified.

**B-2** — Implement Step 2: set `GIS_Status = Decommission` on the identified
ElectricLine feature. Apply to all asset groups and asset types without exception
(FR-002 / DL-001). This step must complete before pole evaluation begins.
*Expected result*: ElectricLine GIS_Status = Decommission written for all tested asset
groups/types; pole evaluation does not start until this write completes.

---

## Group C — Pole Identification via Containment Association

*Depends on A-2, A-3, and B-2 completing first.*

**C-1** — Implement Step 3: query the containment association from the ElectricLine to
identify all associated StructureJunction features. Filter results to Asset Group 90
(Support Structure), Asset Types 731 (HV Pole) and 732 (LV Pole) only (FR-003 / DL-002).
Do NOT use start/end point lookup (CON-004).
*Expected result*: Associated StructureJunction poles correctly identified by
containment association; non-pole features excluded.

**C-2** — Implement fail-safe handling for unresolved containment association (EH-001 /
FR-008 / DL-006): if the containment association cannot be resolved, do not decommission
the pole and flag the record for operational review. Ensure the event does not fail
silently.
*Expected result*: Unresolved containment triggers flag; pole not decommissioned; no
silent failure.

---

## Group D — Pole Decommission Eligibility Evaluation

*Depends on C-1 completing first. D-1 through D-3 can run in parallel.*

**D-1** — Implement blocking check logic: for each identified StructureJunction pole,
query all associated elements and check whether any have GIS_Status / LifecycleStatus =
**In Service** (FR-005 / DL-003–DL-004).
*Expected result*: In Service elements correctly identified as blocking; pole retained.

**D-2** — Extend blocking check: **Planned Uninstalled** elements must also block pole
decommission (EH-003 / FR-005). Confirm Planned Uninstalled is included in the blocking
status evaluation alongside In Service.
*Expected result*: Planned Uninstalled correctly treated as blocking; pole retained when
this status is present.

**D-3** — Implement fail-safe for elements with stale or unresolved GIS_Status (EH-002):
treat as blocking; do not decommission the pole until status is resolved.
*Expected result*: Stale/unresolved status elements cause pole to be retained; no false
decommission.

**D-4** — Implement the decommission path: when no associated elements have GIS_Status /
LifecycleStatus = In Service or Planned Uninstalled (and no stale statuses), set the
pole's `GIS_Status = Decommission` (FR-006 / DL-005).
*Expected result*: Pole GIS_Status correctly set to Decommission when all associated
elements are at non-blocking status.

**D-5** — Verify that each StructureJunction pole identified in C-1 is evaluated
independently. When a single ElectricLine is associated to multiple poles of mixed
eligibility, only eligible poles are decommissioned; retained poles are unaffected.
*Expected result*: Independent per-pole evaluation confirmed; mixed-eligibility scenario
handled correctly.

---

## Group E — ESRI Versioning Post

*Depends on B-2 and D-4 completing first. E-1 and E-2 can run in parallel.*

**E-1** — Implement Step 5: post all GIS_Status changes (ElectricLine and eligible poles)
via ESRI versioning (FR-007 / CON-003 / DL-007).
*Expected result*: All lifecycle changes committed through ESRI versioning.

**E-2** — Confirm the GDBM Queue is not written to during this event path (CON-003 /
FR-007). Add an explicit assertion or test check that no GDBM Queue writes occur.
*Expected result*: No GDBM Queue write observed for any event in this flow.

---

## Group F — Validation and Test Evidence

*Depends on Groups B–E completing. F-1 through F-14 can run in parallel.*

**F-1** — **Evidence: Decommission event received — ElectricLine status.**
Submit decommission event; verify GIS_Status = Decommission set on ElectricLine for all
tested asset groups/types.
*Expected result*: Record confirming SC-001 / FR-002 / DL-001.

**F-2** — **Evidence: Pole with In Service element — retained.**
Process event for line with a pole having an In Service associated element; verify pole
not decommissioned.
*Expected result*: Record confirming SC-002 / FR-005 / DL-004.

**F-3** — **Evidence: Pole with Planned Uninstalled element — retained.**
Process event for line with a pole having a Planned Uninstalled element; verify pole not
decommissioned.
*Expected result*: Record confirming EH-003 / FR-005 / DL-004.

**F-4** — **Evidence: Pole with mixed Planned Uninstalled + Decommissioned elements —
retained.**
Verify Planned Uninstalled counts as blocking even when mixed with Decommissioned
elements.
*Expected result*: Record confirming mixed-state blocking behavior.

**F-5** — **Evidence: Pole with no associated elements — decommissioned.**
Process event for line where pole has no associated elements; verify pole GIS_Status =
Decommission.
*Expected result*: Record confirming SC-003 / FR-006 / DL-005.

**F-6** — **Evidence: Pole with all elements at non-blocking status — decommissioned.**
Process event where all associated elements are at status other than In Service or
Planned Uninstalled; verify pole decommissioned.
*Expected result*: Record confirming SC-003 / DL-005.

**F-7** — **Evidence: Containment association unresolved — fail safe.**
Process event where containment association cannot be resolved; verify pole not
decommissioned and record flagged for review.
*Expected result*: Record confirming SC-004 / EH-001 / DL-006 / C-2.

**F-8** — **Evidence: Stale / unresolved GIS_Status — treated as blocking.**
Process event where an associated element has stale GIS_Status; verify treated as
blocking; pole retained.
*Expected result*: Record confirming EH-002 / D-3.

**F-9** — **Evidence: Multiple poles on same line — mixed eligibility.**
Process event where line has two poles, one eligible and one retained; verify only
eligible pole decommissioned.
*Expected result*: Record confirming independent per-pole evaluation / D-5.

**F-10** — **Evidence: ElectricLine with unusual asset group/type.**
Trigger decommission for a less-common ElectricLine asset group/type; verify GIS_Status
Decommission applied.
*Expected result*: Record confirming FR-002 all-asset-group/type scope.

**F-11** — **Evidence: ESRI versioning used.**
Verify all lifecycle status changes are committed via ESRI versioning.
*Expected result*: Record confirming FR-007 / CON-003 / E-1.

**F-12** — **Evidence: GDBM Queue not used.**
Verify no GDBM Queue write occurs for any event in this flow.
*Expected result*: Record confirming CON-003 / E-2.

**F-13** — **Evidence: Downstream output state.**
Verify ADMS/ERP receive non-decommissioned pole status for retained poles after lifecycle
output is posted.
*Expected result*: Record confirming FR-009 / SC-005.

**F-14** — **Evidence: Non-regression — unrelated GIS workflows.**
Run regression pass on unrelated GIS workflows; verify no change in behavior.
*Expected result*: Regression pass record confirming FR-011 / CON-005.

> ⚠️ **Deferred evidence task — EH-004**: Concurrent decommission events on multiple
> ElectricLine features sharing the same StructureJunction pole. Test design deferred
> until Detailed Design resolves EH-004.

---

## Group G — Deployment Preparation

*Depends on all Group F evidence gathered and approved.*

**G-1** — Package `GIS_ADMS_INTF_003` changes for deployment to the UAT environment.
Smoke test: submit one decommission event and verify GIS_Status set on ElectricLine and
at least one eligible pole decommissioned via ESRI versioning.
*Expected result*: Interface operational in UAT; smoke test passes.

**G-2** — Prepare release notes covering: two-step decommission logic, containment-based
pole identification, blocking statuses (In Service / Planned Uninstalled), fail-safe
behavior, ESRI versioning path, GDBM Queue exclusion, and what is unchanged (unrelated
GIS workflows). Note EH-004 as a deferred open item pending Detailed Design.
*Expected result*: Release notes reviewed and approved by GIS lead.

**G-3** — Confirm production deployment checklist covers: `GIS_ADMS_INTF_003` updated
configuration, containment association data quality pre-check (DEP-001), ESRI versioning
availability confirmation, and rollback steps.
*Expected result*: Deployment checklist signed off.

---

## Group H — Documentation Update

*Can run in parallel with Group G.*

**H-1** — Update spec.md Status field from "Draft" to "Ready for Implementation" once
all validation evidence is gathered.
*Expected result*: Spec status reflects implementation readiness.

**H-2** — Record final test evidence file paths and pass/fail outcomes in the spec or
linked test artifact for BA / QA review. Note EH-004 deferred test case explicitly.
*Expected result*: Evidence traceable from spec to test records; deferred item clearly
flagged.

---

## Dependency Summary

```
A-1, A-2, A-3, A-4, A-5, A-6   (parallel — no dependencies)
            ↓
          B-1 → B-2              (sequential; A-1, A-3 required)
            ↓
         C-1, C-2                (parallel after B-2 + A-2)
            ↓
   D-1, D-2, D-3, D-5            (parallel after C-1)
         D-4                     (after D-1, D-2, D-3)
            ↓
         E-1, E-2                (parallel after B-2, D-4)
            ↓
F-1 through F-14                 (parallel after Groups B–E)
            ↓
G-1, G-2, G-3 / H-1, H-2        (G and H parallel after F approved)
```

---

## Open Items Carried from Plan

| Item | Status | Action Required |
|------|--------|----------------|
| EH-004 — Concurrent events on shared pole | Deferred to Detailed Design | Do not write test case until resolved; flag in release notes |
| Containment association data quality (DEP-001) | Risk (Medium/High) | Pre-check in A-2 and G-3 deployment checklist |
| GIS_Status data staleness (DEP-002) | Risk (Medium/Medium) | EH-002 fail-safe in D-3; treated as blocking |
| ESRI versioning capacity at high event volume | Risk (Low/Medium) | Confirm in A-5 environment readiness |
