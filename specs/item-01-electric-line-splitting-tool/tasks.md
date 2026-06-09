# Tasks: 01 - Electric Line Splitting Tool

**Branch**: `item-01-electric-line-splitting-tool` | **Date**: 2026-06-09
**Spec**: [spec.md](spec.md) | **Plan**: [plan.md](plan.md)

---

## Overview

Build a new ArcFM task (Line folder → Tasks Pane) that splits an ElectricLine feature at
a user-selected point, redistributes LENGTHUSER proportionally to each child feature
using the geometry-derived split ratio, and enforces edit-session gating and zero-length
guards. No new columns; Segment Length untouched; no ADMS scope.

---

## Group A — Analysis and Environment Readiness

Tasks in this group can run in parallel. Complete before any build work begins.

**A-1** — Confirm OOT split-at-point function availability in the target ArcGIS Pro /
ArcFM environment. Record the exact API entry point and parameters used to call it.
*Expected result*: Confirmed function name and call signature documented; DEP-004 risk
resolved.

**A-2** — Confirm `LENGTHUSER` and `Shape_Length` attributes exist on the ElectricLine
feature class in the development GIS schema. Record attribute names, types, and
nullability.
*Expected result*: Attribute schema verified; DEP-002 resolved; no schema gap found (or
gap escalated).

**A-3** — Confirm `GlobalId` attribute availability on ElectricLine in the target schema.
*Expected result*: GlobalId confirmed present and readable for dialog display.

**A-4** — Identify all ElectricLine AssetGroup / AssetType combinations present in the
test data set and confirm no asset-type-specific branching is required.
*Expected result*: Asset combination list documented; Assumption (all types treated
identically) validated.

**A-5** — Define the degenerate split point tolerance (how close to an endpoint constitutes
a zero-length risk). Agree with team and document the threshold value.
*Expected result*: Numeric tolerance confirmed; EH-001 implementation can reference it.

---

## Group B — Task Registration and Shell

*Depends on A-1 and A-2 completing first.*

**B-1** — Register a new ArcFM task entry in the Line folder of the Tasks Pane.
Confirm the task appears in the correct folder when ArcGIS Pro is launched.
*Expected result*: Task visible in Line folder; no conflict with existing tasks.

**B-2** — Implement the edit-session guard (EH-002 / FR-009): when the task is invoked
outside an active edit session, it must not execute and must display a clear message.
*Expected result*: Task is inert and shows message when no edit session is active.

---

## Group C — Dialog Implementation

*Depends on B-1. C-1 and C-2 can run in parallel.*

**C-1** — Build the task dialog. Display the following fields:
- **Electric Line ID**: read-only; populated from the selected feature's `GlobalId`.
- **User Length**: editable; pre-populated per FR-003 logic (see C-2).
*Expected result*: Dialog opens on task invocation; Electric Line ID is read-only and
correct; User Length field is editable.

**C-2** — Implement User Length pre-population logic (FR-003):
- If `LENGTHUSER > 0` → pre-populate with `LENGTHUSER`.
- Else → pre-populate with `Shape_Length`.
*Expected result*: Correct source used for each pre-population case; verified against
test lines with LENGTHUSER = 0/null and LENGTHUSER > 0.

---

## Group D — Split Ratio and LENGTHUSER Redistribution

*Depends on C-1 and C-2. D-1 and D-2 can run in parallel.*

**D-1** — Implement geometry-derived split ratio computation (FR-004):
- `ratio_B = length_child_B / length_parent_geometry`
- `ratio_C = 1 - ratio_B`
*Expected result*: Ratios computed from actual geometry split point; user does not input
the ratio.

**D-2** — Implement LENGTHUSER redistribution for each child (FR-005 / FR-006 / FR-008):
- `LENGTHUSER_B = round(ratio_B × user_length, 2)`
- `LENGTHUSER_C = round(ratio_C × user_length, 2)`
- Assign rounding residual to the longer child so that child totals = parent total.
*Expected result*: Both children have correct LENGTHUSER at 2dp; totals always match.

---

## Group E — Split Execution and Edge-Case Guards

*Depends on D-1 and D-2.*

**E-1** — Implement zero-length child guard (EH-001): before invoking the OOT
split-at-point function, check the split point against the tolerance from A-5. If the
split would produce a zero-length child, halt and display a clear message; do not execute
the split.
*Expected result*: Split is blocked with a message when split point is at or near an
endpoint; no zero-length child feature is created.

**E-2** — Invoke the OOT split-at-point function (DEP-004) to physically split the parent
ElectricLine geometry into two child ElectricLine features. Write computed LENGTHUSER
values to each child. Do not write to Segment Length.
*Expected result*: Two child ElectricLine features created with correct geometry and
LENGTHUSER; Segment Length unchanged; no ADMS event triggered.

**E-3** — Confirm that all other ElectricLine attributes (other than LENGTHUSER) are
carried to child features as expected by the OOT split function. Flag any unexpected
attribute behavior.
*Expected result*: Attribute carry-over confirmed; no unintended attribute changes on
child features.

---

## Group F — Validation and Test Evidence

*Depends on E-1 and E-2. F-1 through F-8 can run in parallel.*

**F-1** — **Evidence: Equal split (50:50), LENGTHUSER = 100.**
Run split; verify child LENGTHUSER = 50 each; total = 100.
*Expected result*: Screenshot / test record confirming values.

**F-2** — **Evidence: Non-equal split (2:3), LENGTHUSER = 100.**
Run split; verify children LENGTHUSER = 40 and 60; total = 100.
*Expected result*: Screenshot / test record confirming values.

**F-3** — **Evidence: LENGTHUSER = 0 → Shape_Length fallback.**
Run split on line where LENGTHUSER = 0 or null; verify children redistributed from
Shape_Length; total = Shape_Length.
*Expected result*: Screenshot / test record confirming fallback used.

**F-4** — **Evidence: Editor override.**
Pre-populated value = 80; editor types 100; run split. Verify children redistributed
from 100, not 80.
*Expected result*: Screenshot / test record confirming override respected.

**F-5** — **Evidence: Rounding residual case.**
Use a ratio that produces a 3dp result (e.g., 1:2 ratio on LENGTHUSER = 100 → 33.33 /
66.67). Verify residual assigned to longer child; totals match exactly.
*Expected result*: Test record showing 33.33 / 66.67 and total = 100.00.

**F-6** — **Evidence: Degenerate split guard.**
Attempt split with point at or very close to an existing endpoint. Verify task halts,
message is presented, and no zero-length child is created.
*Expected result*: Message confirmed; no zero-length feature in GIS.

**F-7** — **Evidence: Outside edit session guard.**
Invoke task with no active edit session. Verify task does not execute; message presented.
*Expected result*: Screenshot / test record confirming behavior.

**F-8** — **Evidence: All AssetGroup / AssetType combinations.**
Run at least one split per major AssetGroup in the test data set. Verify consistent
behavior across all combinations.
*Expected result*: Test matrix completed; no combination produces unexpected results.

**F-9** — **Evidence: Non-regression.**
Run regression test pass for existing GIS editing tools (Line folder and general editing).
Verify no change in behavior.
*Expected result*: Regression pass record; no defects related to this change.

---

## Group G — Deployment Preparation

*Depends on all Group F evidence gathered and approved.*

**G-1** — Package the ArcFM task for deployment to the test / UAT environment. Confirm
the task appears in the Line folder and behaves identically to the development build.
*Expected result*: Task deployed and smoke-tested in UAT; no configuration delta.

**G-2** — Prepare release notes identifying: new task name, location (Line folder →
Tasks Pane), LENGTHUSER redistribution behavior, edit-session requirement, and
Segment Length exclusion.
*Expected result*: Release notes reviewed and approved by GIS lead.

**G-3** — Confirm production deployment checklist covers: task registration, attribute
schema prerequisites (LENGTHUSER, Shape_Length, GlobalId), and rollback steps if needed.
*Expected result*: Deployment checklist signed off.

---

## Group H — Documentation Update

*Can run in parallel with Group G.*

**H-1** — Update spec.md Status field from "Draft" to "Ready for Implementation" (or
equivalent agreed status) once all validation evidence is gathered.
*Expected result*: Spec status reflects implementation readiness.

**H-2** — Record final test evidence file paths and test pass/fail outcomes in the spec
or linked test artifact.
*Expected result*: Evidence traceable from spec to test records for BA / QA review.

---

## Dependency Summary

```
A-1, A-2, A-3, A-4, A-5   (parallel — no dependencies)
      ↓
   B-1, B-2               (parallel after A complete)
      ↓
   C-1, C-2               (parallel after B-1)
      ↓
   D-1, D-2               (parallel after C complete)
      ↓
   E-1 → E-2 → E-3        (sequential)
      ↓
F-1 through F-9            (parallel after E complete)
      ↓
   G-1, G-2, G-3 / H-1, H-2   (G and H parallel after F approved)
```

---

## Open Items Carried from Plan

| Item | Status | Action Required |
|------|--------|----------------|
| Degenerate split threshold | Open | Resolved in A-5 before build |
| OOT split-at-point function availability | Risk | Confirmed in A-1; escalate if unavailable |
| LENGTHUSER / Shape_Length schema presence | Risk | Confirmed in A-2; escalate if missing |
