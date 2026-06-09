# Tasks: 02 - User Length Alignment Tool

**Branch**: `item-02-user-length-alignment-tool` | **Date**: 2026-06-09
**Spec**: [spec.md](spec.md) | **Plan**: [plan.md](plan.md)

---

## Overview

Add a creation-time attribute rule to the ElectricLine feature class (all Asset Groups,
all Asset Types) that automatically sets `LENGTHUSER` = `SHAPE_Length` when `LENGTHUSER`
is null or ≤ 0. Rule fires on creation only — never on edits. User-supplied
`LENGTHUSER` > 0 at creation is preserved. No new columns; no ADMS scope.

---

## Group A — Analysis and Environment Readiness

Tasks in this group can run in parallel. Complete before any build work begins.

**A-1** — Confirm `LENGTHUSER` and `SHAPE_Length` attributes are present on the
ElectricLine feature class in the development GIS schema. Record attribute names, types,
nullability, and current default values.
*Expected result*: Attribute schema verified; DEP-001 resolved; no schema gap found (or
gap escalated).

**A-2** — Confirm the GIS attribute rule framework supports a creation-only trigger
(i.e., the rule can be configured to fire on creation events and NOT on edit/update
events). Document how to constrain scope to creation.
*Expected result*: Framework capability confirmed; DEP-002 resolved; implementation
approach for creation-only trigger documented.

**A-3** — Confirm `SHAPE_Length` is system-calculated and available at rule execution
time during feature creation (i.e., geometry is committed before the rule fires).
*Expected result*: Calculation order confirmed; Assumption validated; EH-001 scope
understood.

**A-4** — Identify all ElectricLine AssetGroup / AssetType combinations present in the
test data set. Confirm no asset-type-specific branching is required.
*Expected result*: Asset combination list documented; FR-006 scope confirmed.

**A-5** — Identify and document the creation methods in use in the target environment:
interactive editing, batch creation, and data import. Confirm the attribute rule
framework fires on all creation paths (FR-007).
*Expected result*: Creation method inventory complete; batch/import rule-trigger behavior
confirmed or flagged as risk.

---

## Group B — Attribute Rule Development

*Depends on A-1, A-2, and A-3 completing first.*

**B-1** — Author the attribute rule definition for the ElectricLine feature class:
- **Trigger**: Creation event only (not update/delete).
- **Condition**: `LENGTHUSER IS NULL OR LENGTHUSER <= 0`
- **Action**: `LENGTHUSER = SHAPE_Length`
*Expected result*: Rule definition authored; condition and action logic confirmed against
spec FR-001 and FR-002.

**B-2** — Verify the rule is scoped to all Asset Groups and all Asset Types on
ElectricLine — no asset-type filter applied (FR-006).
*Expected result*: Rule scope confirmed as unrestricted across all asset combinations.

**B-3** — Implement and verify the user-value preservation logic (FR-002 / FR-003):
when `LENGTHUSER > 0` at creation time, the rule must not execute and the supplied value
must be preserved. Confirm by inspection of rule condition.
*Expected result*: Rule condition correctly evaluates; user-supplied valid values are
not overwritten.

**B-4** — Implement and verify the fail-safe for invalid or null geometry (EH-001):
if `SHAPE_Length` is unavailable at creation time, the creation must fail and present a
clear error message; the record must not be saved with `LENGTHUSER` = null.
*Expected result*: Fail-safe behavior verified in development; error message text
confirmed with GIS lead.

**B-5** — Confirm the rule does NOT fire on edits to existing ElectricLine features
(FR-005). Test by editing an existing record and verifying `LENGTHUSER` is unchanged by
the rule.
*Expected result*: `LENGTHUSER` not reset on edit; rule fire confirmed absent.

---

## Group C — Deployment to Development / Test

*Depends on B-1 through B-5.*

**C-1** — Deploy the attribute rule to the development/test GIS geodatabase using the
standard GIS schema deployment process.
*Expected result*: Rule active in test environment; ElectricLine feature class updated.

**C-2** — Smoke test the rule in the test environment: create one ElectricLine with no
`LENGTHUSER` and verify `LENGTHUSER` = `SHAPE_Length` after save.
*Expected result*: Smoke test passes; rule fires correctly in deployed state.

---

## Group D — Validation and Test Evidence

*Depends on C-1 and C-2. D-1 through D-9 can run in parallel.*

**D-1** — **Evidence: No LENGTHUSER supplied at creation.**
Create ElectricLine with no `LENGTHUSER` value. Verify `LENGTHUSER` = `SHAPE_Length`
after save.
*Expected result*: Test record / screenshot confirming SC-001.

**D-2** — **Evidence: LENGTHUSER = 0 at creation.**
Create ElectricLine with `LENGTHUSER` = 0. Verify `LENGTHUSER` = `SHAPE_Length`
after save.
*Expected result*: Test record confirming FR-001 null-equivalent rule.

**D-3** — **Evidence: LENGTHUSER explicitly null at creation.**
Create ElectricLine with `LENGTHUSER` = null. Verify `LENGTHUSER` = `SHAPE_Length`
after save.
*Expected result*: Test record confirming FR-001 null case.

**D-4** — **Evidence: LENGTHUSER > 0 supplied at creation.**
Create ElectricLine with `LENGTHUSER` = 50 (or any positive value). Verify
`LENGTHUSER` remains 50 after save; rule does not overwrite.
*Expected result*: Test record confirming SC-002 / FR-002.

**D-5** — **Evidence: Edit existing ElectricLine — rule does not fire.**
Edit an existing ElectricLine (change an unrelated attribute). Verify `LENGTHUSER` is
unchanged and the rule has not re-triggered.
*Expected result*: Test record confirming FR-005 / SC-004.

**D-6** — **Evidence: Post-creation LENGTHUSER update.**
Create ElectricLine (rule fires), then manually update `LENGTHUSER` to a different
value. Verify updated value is retained without rule re-triggering.
*Expected result*: Test record confirming SC-004 / FR-004.

**D-7** — **Evidence: Batch / import creation.**
Create multiple ElectricLine features via batch or import with null/zero `LENGTHUSER`.
Verify all records have non-null `LENGTHUSER` = `SHAPE_Length` after save.
*Expected result*: Batch test record confirming FR-007 / SC-003.

**D-8** — **Evidence: All AssetGroup / AssetType combinations (sample).**
Create at least one ElectricLine per major AssetGroup in the test data set. Verify
consistent defaulting behavior across all combinations.
*Expected result*: Test matrix completed; SC-003 / FR-006 confirmed.

**D-9** — **Evidence: Invalid geometry / SHAPE_Length unavailable.**
Attempt to create ElectricLine with invalid or null geometry. Verify creation fails safe
with a clear error message; no null `LENGTHUSER` record saved.
*Expected result*: Screenshot / test record confirming EH-001.

**D-10** — **Evidence: Non-regression.**
Run regression test pass for existing GIS ElectricLine creation and editing workflows.
Verify no change in unrelated behavior.
*Expected result*: Regression pass record; no defects related to this change.

---

## Group E — Deployment Preparation

*Depends on all Group D evidence gathered and approved.*

**E-1** — Package the attribute rule configuration for deployment to the UAT environment.
Confirm the rule is present, fires correctly, and scope matches development build.
*Expected result*: Rule deployed to UAT; smoke test passed; no configuration delta.

**E-2** — Prepare release notes identifying: rule name, trigger (creation only),
condition (`LENGTHUSER` null or ≤ 0), action (`LENGTHUSER` = `SHAPE_Length`), scope
(all Asset Groups / Types), and what is explicitly unchanged (no edits affected, no
existing records updated).
*Expected result*: Release notes reviewed and approved by GIS lead.

**E-3** — Confirm production deployment checklist covers: attribute rule registration,
schema prerequisites (`LENGTHUSER`, `SHAPE_Length`), and rollback steps.
*Expected result*: Deployment checklist signed off.

---

## Group F — Documentation Update

*Can run in parallel with Group E.*

**F-1** — Update spec.md Status field from "Draft" to "Ready for Implementation" (or
agreed status) once all validation evidence is gathered.
*Expected result*: Spec status reflects implementation readiness.

**F-2** — Record final test evidence file paths and test pass/fail outcomes in the spec
or linked test artifact.
*Expected result*: Evidence traceable from spec to test records for BA / QA review.

---

## Dependency Summary

```
A-1, A-2, A-3, A-4, A-5   (parallel — no dependencies)
         ↓
   B-1 → B-2, B-3, B-4, B-5   (B-1 first; B-2 through B-5 parallel after B-1)
         ↓
      C-1 → C-2             (sequential)
         ↓
D-1 through D-10            (parallel after C-2)
         ↓
E-1, E-2, E-3 / F-1, F-2   (E and F parallel after D approved)
```

---

## Open Items Carried from Plan

| Item | Status | Action Required |
|------|--------|----------------|
| Attribute rule creation-only trigger support | Risk | Confirmed in A-2 before build |
| SHAPE_Length availability at rule fire time | Risk | Confirmed in A-3; escalate if order-of-operations gap found |
| Batch / import path rule coverage | Risk | Confirmed in A-5; tested explicitly in D-7 |
| Existing null LENGTHUSER records | Out of scope | Note as known gap if downstream impact raised; do not retroactively fix |
