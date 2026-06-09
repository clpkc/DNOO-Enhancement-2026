# Tasks: 03 - Update Substation Task Drop Down List

**Branch**: `item-03-update-substation-task` | **Date**: 2026-06-09
**Spec**: [spec.md](spec.md) | **Plan**: [plan.md](plan.md)

---

## Overview

Extend the existing Update Substation ArcFM Task to support SSNAME / SSNUM dropdown
updates for HV PM TX Transformers, Reclosers, and Pole-Mounted HV Switches
(`PLACEMENT = Pole-Mounted`), in addition to the existing Substation support. Dropdown
values are sourced from `SAP_SUBSTATION_DATA` with IS_USED filtering and bidirectional
auto-fill. Server-side validation confirms the pair originates from the same staging
record before commit. SSNAME stored with mandatory "S/S" suffix. No ADMS scope.

---

## Group A — Analysis and Environment Readiness

Tasks in this group can run in parallel. Complete before any build work begins.

**A-1** — Confirm `SAP_SUBSTATION_DATA` staging table availability in the development
environment. Verify field names: `SUBSTATION` (→ SSNUM), `FL_EN_DESC` (→ SSNAME),
`IS_USED`. Record schema against plan assumptions.
*Expected result*: Field names and types confirmed; DEP-001 risk resolved; any schema
mismatch escalated.

**A-2** — Confirm all four supported GIS asset types expose `SSNAME` and `SSNUM` fields
in the development GIS schema:
- Substation (existing)
- `ElectricDevice.Transformer.HV PM TX`
- `ElectricDevice.HV Switch.Recloser`
- `ElectricDevice.HV Switch.Switch`
*Expected result*: Attribute presence confirmed for all four types; DEP-003 resolved.

**A-3** — Confirm the `PLACEMENT` attribute is available and correctly populated on
`ElectricDevice.HV Switch.Switch` features in the test data set. Document the expected
values and identify any null/empty cases.
*Expected result*: `PLACEMENT` data quality confirmed; behavior for null `PLACEMENT`
agreed (treat as non-Pole-Mounted — fail safe).

**A-4** — Confirm the ArcFM Task extension mechanism: identify the code path or
configuration point for adding new supported asset types to the existing Update
Substation task. Confirm the task's existing entry point remains unchanged (DEP-002).
*Expected result*: Extension approach documented; no new task entry point required.

**A-5** — Confirm the `SAP_SUBSTATION_DATA` table contains test records including:
at least one `IS_USED = true` record, at least one valid SSNAME/SSNUM pair, and a pair
that should be rejected (mismatched records). Document test data set.
*Expected result*: Test data set confirmed and available in test environment.

---

## Group B — Asset Type Extension

*Depends on A-1, A-2, A-3, and A-4 completing first.*

**B-1** — Extend the Update Substation ArcFM Task to recognise the three new asset types
alongside the existing Substation:
- `ElectricDevice.Transformer.HV PM TX`
- `ElectricDevice.HV Switch.Recloser`
- `ElectricDevice.HV Switch.Switch` where `PLACEMENT = Pole-Mounted` (FR-001)
*Expected result*: Task opens and proceeds for each of the four supported asset types.

**B-2** — Implement the `PLACEMENT` filter for HV Switch assets (FR-010 / EH-002):
when the task is invoked on `ElectricDevice.HV Switch.Switch` where `PLACEMENT` ≠
Pole-Mounted (or is null/empty), the task must not proceed and must present a clear
message.
*Expected result*: Non-Pole-Mounted HV Switch blocked; clear message presented; no data
written.

**B-3** — Implement the unsupported asset type guard (EH-002): if the task is invoked
on any asset type not in the four supported types, present a clear message and do not
proceed.
*Expected result*: Unsupported types blocked with a clear message.

---

## Group C — Dropdown Sourcing and Auto-Fill

*Depends on B-1. C-1 and C-2 can run in parallel.*

**C-1** — Implement dropdown population from `SAP_SUBSTATION_DATA` (FR-002):
- Load SSNAME values from `FL_EN_DESC` into the SSNAME dropdown.
- Load SSNUM values from `SUBSTATION` into the SSNUM dropdown.
- Exclude all records where `IS_USED = true` from both dropdowns (FR-004 / VR-005).
*Expected result*: Dropdowns populated from staging table; IS_USED = true records absent.

**C-2** — Implement empty-dropdown behavior (EH-003): if all available records have
`IS_USED = true`, present an empty dropdown with a clear message that no eligible
substation values are available.
*Expected result*: Empty dropdown + clear message shown when no eligible values exist.

**C-3** — Implement bidirectional auto-fill (FR-003):
- Selecting SSNAME from dropdown → auto-fill SSNUM with the paired value from the same
  `SAP_SUBSTATION_DATA` record.
- Selecting SSNUM from dropdown → auto-fill SSNAME with the paired value from the same
  record.
*Expected result*: Selecting either field auto-fills the other with the correct paired
value; mismatched values cannot arise through normal dropdown interaction.

---

## Group D — Server-Side Validation and SSNAME Suffix

*Depends on C-1, C-2, and C-3. D-1 and D-2 can run in parallel.*

**D-1** — Implement server-side pair validation (FR-005 / FR-006 / VR-001–003 /
VR-006):
- On Update button press: verify submitted SSNAME and SSNUM originate from the same
  record in `SAP_SUBSTATION_DATA`.
- Reject update (do not commit) if SSNAME is present without a matching SSNUM from
  the same record (VR-002).
- Reject update (do not commit) if SSNUM is present without a matching SSNAME from
  the same record (VR-003).
- Present a clear validation failure message for any rejected update (FR-009).
*Expected result*: Only same-record pairs pass validation and are committed.

**D-2** — Implement staging table unavailability fail-safe (EH-001 / VR-007):
if `SAP_SUBSTATION_DATA` cannot be reached during server-side validation, the update
must not be committed and a clear message must be presented.
*Expected result*: Update not committed when staging table is unreachable; clear message
shown.

**D-3** — Implement SSNAME "S/S" suffix rule (VR-004 / BR-003):
- Before saving SSNAME to the GIS feature, check whether the value already ends with
  "S/S".
- If not, append " S/S" before writing the attribute.
- If already ends with "S/S", store unchanged.
*Expected result*: All saved SSNAME values end with "S/S"; duplicate suffixes never
added.

---

## Group E — Commit and Consistency

*Depends on D-1, D-2, and D-3.*

**E-1** — Wire the full commit path: if all validations pass and SSNAME suffix is
applied, commit SSNAME and SSNUM to the selected GIS feature via the ArcFM Task
framework (FR-007 / FR-008).
*Expected result*: Valid updates saved to GIS feature; SSNAME and SSNUM match the
approved SAP_SUBSTATION_DATA record.

**E-2** — Confirm all four asset types follow identical validation and commit behavior
(FR-008 / FR-001): run the full flow once per asset type in the development environment.
*Expected result*: No asset-type-specific divergence in validation or commit path.

---

## Group F — Validation and Test Evidence

*Depends on E-1 and E-2. F-1 through F-13 can run in parallel.*

**F-1** — **Evidence: Valid SSNAME + SSNUM pair — Substation.**
Execute task on Substation; select matching pair; press Update. Verify saved.
*Expected result*: Record confirming SC-001 / VR-001.

**F-2** — **Evidence: Valid SSNAME + SSNUM pair — HV PM TX Transformer.**
Execute task; select matching pair; press Update. Verify saved.
*Expected result*: Record confirming SC-001 / FR-001 (new asset type).

**F-3** — **Evidence: Valid SSNAME + SSNUM pair — Recloser.**
Execute task; select matching pair; press Update. Verify saved.
*Expected result*: Record confirming SC-001 / FR-001 (new asset type).

**F-4** — **Evidence: Valid SSNAME + SSNUM pair — Pole-Mounted HV Switch.**
Execute task on HV Switch with `PLACEMENT = Pole-Mounted`; select matching pair; press
Update. Verify saved.
*Expected result*: Record confirming SC-001 / FR-001 (new asset type).

**F-5** — **Evidence: Invalid pair (mismatched staging records).**
Attempt to save SSNAME from one record and SSNUM from a different record. Verify update
rejected; clear message shown.
*Expected result*: Record confirming SC-002 / VR-001.

**F-6** — **Evidence: Select SSNAME → SSNUM auto-fills.**
Select an SSNAME from dropdown. Verify SSNUM auto-fills with paired value.
*Expected result*: Screenshot confirming FR-003.

**F-7** — **Evidence: Select SSNUM → SSNAME auto-fills.**
Select an SSNUM from dropdown. Verify SSNAME auto-fills with paired value.
*Expected result*: Screenshot confirming FR-003.

**F-8** — **Evidence: IS_USED = true values excluded from dropdown.**
Open task; confirm IS_USED = true staging records are not selectable.
*Expected result*: Screenshot / record confirming SC-004 / VR-005.

**F-9** — **Evidence: All dropdown values IS_USED = true.**
Use test data set where all records have IS_USED = true. Verify empty dropdown + clear
message.
*Expected result*: Screenshot confirming EH-003.

**F-10** — **Evidence: SSNAME without "S/S" suffix.**
Save an SSNAME value that does not end with "S/S". Verify "S/S" appended in stored value.
*Expected result*: Record confirming SC-005 / VR-004.

**F-11** — **Evidence: SSNAME already ending in "S/S".**
Save an SSNAME already ending with "S/S". Verify no duplicate suffix added.
*Expected result*: Record confirming VR-004 idempotency.

**F-12** — **Evidence: HV Switch PLACEMENT ≠ Pole-Mounted — blocked.**
Invoke task on HV Switch with non-Pole-Mounted PLACEMENT. Verify task does not proceed;
clear message shown.
*Expected result*: Screenshot confirming EH-002 / FR-010.

**F-13** — **Evidence: SAP_SUBSTATION_DATA unavailable.**
Simulate staging table unavailability at validation time. Verify update not committed;
clear fail-safe message.
*Expected result*: Screenshot confirming EH-001 / VR-007.

**F-14** — **Evidence: Non-regression — existing Substation task behavior.**
Run regression pass on existing Substation task workflow. Verify no change in behavior.
*Expected result*: Regression pass record; no defects related to this change (FR-011).

---

## Group G — Deployment Preparation

*Depends on all Group F evidence gathered and approved.*

**G-1** — Package the task extension for deployment to the UAT environment. Confirm all
four asset types are supported, validation rules fire correctly, and SSNAME suffix is
applied. Run smoke test.
*Expected result*: Task deployed to UAT; smoke test passed; no configuration delta.

**G-2** — Prepare release notes covering: newly supported asset types, PLACEMENT filter
for HV Switch, IS_USED exclusion, auto-fill behavior, "S/S" suffix rule, server-side
validation, and fail-safe behavior.
*Expected result*: Release notes reviewed and approved by GIS lead.

**G-3** — Confirm production deployment checklist covers: task configuration, schema
prerequisites (SSNAME, SSNUM on all four types; `SAP_SUBSTATION_DATA` accessible),
and rollback steps.
*Expected result*: Deployment checklist signed off.

---

## Group H — Documentation Update

*Can run in parallel with Group G.*

**H-1** — Update spec.md Status field from "Draft" to "Ready for Implementation" once
all validation evidence is gathered.
*Expected result*: Spec status reflects implementation readiness.

**H-2** — Record final test evidence file paths and test pass/fail outcomes in the spec
or linked test artifact for BA / QA review.
*Expected result*: Evidence traceable from spec to test records.

---

## Dependency Summary

```
A-1, A-2, A-3, A-4, A-5        (parallel — no dependencies)
         ↓
B-1 → B-2, B-3                 (B-1 first; B-2 and B-3 parallel after B-1)
         ↓
C-1, C-2 → C-3                 (C-1 and C-2 parallel after B-1; C-3 after C-1)
         ↓
D-1, D-2, D-3                  (parallel after C-3)
         ↓
E-1 → E-2                      (sequential)
         ↓
F-1 through F-14               (parallel after E-2)
         ↓
G-1, G-2, G-3 / H-1, H-2      (G and H parallel after F approved)
```

---

## Open Items Carried from Plan

| Item | Status | Action Required |
|------|--------|----------------|
| SAP_SUBSTATION_DATA field names | Risk | Confirmed in A-1 before build |
| PLACEMENT attribute data quality (HV Switch) | Risk | Confirmed in A-3; null behavior agreed before B-2 |
| IS_USED maintenance currency | Operational dependency | Note for EWMS integration owner; not a build blocker |
| Auto-fill performance with large staging table | Low risk | Assess table size in A-5; add query filter if needed |
