# Tasks: 15 - Load Taking Mobile App Enhancement

**Branch**: `item-15-load-taking-mobile-app` | **Date**: 2026-06-09
**Spec**: [spec.md](spec.md) | **Plan**: [plan.md](plan.md)

---

## Overview

Populate the existing `SS_NO` field (alias: `SUBSTATION_NO`) in `TX_LOAD_READING` via
`GIS_SPS_INTF_001` using asset-type-specific SSNUM source rules:
- **HV SS Transformer** → containing HV Substation's SSNUM
- **HV PM Transformer** → transformer's own SSNUM
- **All other types** → SS_NO not populated
- **Null source** → SS_NO written as null

No new column is added. `TX_CORE_LOAD` is out of scope. SQL injection remediation for
`GIS_SPS_INTF_004` is explicitly deferred pending a separate GIS enhancement item.

> ⚠️ **Open items — not in scope for these tasks**:
> - **EH-004**: HV SS Transformer reassigned to different HV Substation between load
>   cycles — SS_NO behavior deferred to Detailed Design. Test case excluded.
> - **SQL injection / GIS_SPS_INTF_004** (FR-006 / SEC-001–003): Deferred pending
>   separate GIS enhancement item or explicit re-scoping approval.

---

## Group A — Analysis and Environment Readiness

Tasks in this group can run in parallel. Complete before build work begins.

**A-1** — Confirm the existing `SS_NO` field (alias: `SUBSTATION_NO`) is present in
`TX_LOAD_READING` in the development GIS schema (CON-004). Confirm no schema change is
required. Document the field name, data type, and current population state.
*Expected result*: SS_NO field confirmed present; no new column required; field
characteristics documented.

**A-2** — Confirm `GIS_SPS_INTF_001` is accessible in the development environment
(DEP-003). Identify the location in the interface logic where SS_NO is (or should be)
written to TX_LOAD_READING.
*Expected result*: Interface path confirmed; write location identified.

**A-3** — Confirm how Transformer asset type (HV SS, HV PM, or other) is determined at
load processing time via the GIS data model. Document the attribute name, field, or
classification mechanism used for asset-type detection.
*Expected result*: Asset-type detection mechanism documented; branching logic can be
implemented.

**A-4** — Confirm the transformer-to-containing-HV-Substation association is resolvable
at load processing time for HV SS Transformer features (DEP-001). Document the
relationship or query path used.
*Expected result*: HV SS → HV Substation lookup path confirmed and documented.

**A-5** — Confirm the `SSNUM` attribute is present and readable on:
- HV Substation features (for HV SS Transformer path).
- HV PM Transformer features (for their own SSNUM path).
Document the attribute name, type, and whether null/empty values exist in the test data
(DEP-002).
*Expected result*: SSNUM attribute confirmed on both entity types; null/empty coverage
noted.

**A-6** — Obtain or generate test data in the development environment covering:
- HV SS Transformer with a containing HV Substation with non-null SSNUM.
- HV SS Transformer with a containing HV Substation with null/empty SSNUM.
- HV SS Transformer with no resolvable containing HV Substation (EH-001).
- HV PM Transformer with non-null own SSNUM.
- HV PM Transformer with null/empty own SSNUM.
- Transformer of an unsupported asset type (neither HV SS nor HV PM).
- Any type with an empty-string SSNUM source (to verify null write, not empty string).
*Expected result*: Test data set confirmed covering all branches and EH cases.

---

## Group B — Asset-Type Detection

*Depends on A-2 and A-3 completing first.*

**B-1** — Implement asset-type detection in `GIS_SPS_INTF_001`: for each Transformer
feature processed, identify whether the asset type is HV SS Transformer, HV PM
Transformer, or another type (FR-001 / plan Step 1).
*Expected result*: Asset-type classification correctly identifies each category for all
test data; branching entry point confirmed.

---

## Group C — SSNUM Source Lookup

*Depends on B-1 completing. C-1 and C-2 can run in parallel.*

**C-1** — Implement Branch A (HV SS Transformer): resolve the containing HV Substation
for the transformer and read its SSNUM (FR-002 / DP-001).
- If HV Substation cannot be resolved: SS_NO = null; record available for operational
  review (EH-001). Do not fail silently.
- If HV Substation SSNUM is null or empty: SS_NO = null (EH-002 / FR-005 / DP-004).
*Expected result*: SSNUM correctly read from HV Substation for non-null cases; null
written for all null/unresolvable cases.

**C-2** — Implement Branch B (HV PM Transformer): read the transformer's own SSNUM
attribute directly (FR-003 / DP-002).
- If transformer SSNUM is null or empty: SS_NO = null (EH-002 / FR-005 / DP-004).
*Expected result*: SSNUM correctly read from transformer feature; null written for null
or empty cases.

**C-3** — Implement Branch C (all other Transformer asset types): do NOT populate SS_NO
in TX_LOAD_READING. No default or fallback SSNUM is substituted (FR-004 / DP-003 /
EH-003).
*Expected result*: SS_NO field is not written for unsupported asset types; no default
value appears.

---

## Group D — SS_NO Write and Null Handling

*Depends on C-1, C-2, and C-3 completing first.*

**D-1** — Implement the SS_NO write step (plan Step 3): write the derived SSNUM value
(or null) to the existing SS_NO field in TX_LOAD_READING for all in-scope branches
(CON-004).
- Confirm no new column is added.
- Confirm TX_CORE_LOAD is not modified.
*Expected result*: SS_NO written correctly per asset-type branch; TX_LOAD_READING schema
unchanged; TX_CORE_LOAD untouched.

**D-2** — Implement and verify the null handling rule (plan Step 4 / FR-005 / EH-002):
when the applicable SSNUM source is null or empty for any reason (including empty string),
SS_NO is written as null, not as empty string or a placeholder value.
*Expected result*: Null written for all null/empty-source scenarios; empty string not
written.

---

## Group E — Validation and Test Evidence

*Depends on Groups B–D completing. E-1 through E-11 can run in parallel.*

**E-1** — **Evidence: HV SS Transformer — HV Substation has non-null SSNUM.**
Process a load record for HV SS Transformer with a known HV Substation SSNUM; verify
SS_NO = HV Substation SSNUM.
*Expected result*: Record confirming SC-001 / FR-002 / DP-001.

**E-2** — **Evidence: HV SS Transformer — HV Substation SSNUM is null.**
Process load record; verify SS_NO = null in TX_LOAD_READING.
*Expected result*: Record confirming SC-003 / EH-002 / FR-005.

**E-3** — **Evidence: HV SS Transformer — no resolvable HV Substation.**
Process load record where HV Substation cannot be resolved; verify SS_NO = null and
record is available for operational review; no silent skip.
*Expected result*: Record confirming EH-001.

**E-4** — **Evidence: HV PM Transformer — transformer SSNUM non-null.**
Process load record for HV PM Transformer with known own SSNUM; verify SS_NO =
transformer's own SSNUM.
*Expected result*: Record confirming SC-002 / FR-003 / DP-002.

**E-5** — **Evidence: HV PM Transformer — transformer SSNUM is null.**
Process load record; verify SS_NO = null.
*Expected result*: Record confirming SC-003 / EH-002 / FR-005.

**E-6** — **Evidence: Unsupported transformer asset type.**
Process load record for a Transformer that is neither HV SS nor HV PM; verify SS_NO is
not populated; no default written.
*Expected result*: Record confirming SC-004 / FR-004 / EH-003 / DP-003.

**E-7** — **Evidence: Any type — SSNUM source is empty string.**
Provide a transformer where the SSNUM source is an empty string; verify SS_NO = null
(not empty string) in TX_LOAD_READING.
*Expected result*: Record confirming FR-005 / EH-002 null write behavior.

**E-8** — **Evidence: TX_CORE_LOAD not modified.**
Run interface for all test data; verify no changes to TX_CORE_LOAD records.
*Expected result*: Record confirming TX_CORE_LOAD out-of-scope constraint.

**E-9** — **Evidence: No new column added.**
Inspect TX_LOAD_READING schema after changes; verify schema unchanged (no new column).
*Expected result*: Record confirming CON-004.

**E-10** — **Evidence: Non-regression — existing GIS_SPS_INTF_001 logic.**
Run regression pass for existing interface behavior on other fields and operations;
verify no change.
*Expected result*: Regression pass confirming CON-005.

**E-11** — **Evidence: Non-regression — other tables.**
Verify no modification to TX_CORE_LOAD or other load-taking tables.
*Expected result*: Record confirming out-of-scope constraint and CON-005.

> ⚠️ **Deferred evidence tasks**:
> - HV SS Transformer reassigned to different HV Substation between load cycles (EH-004)
>   — deferred to Detailed Design.
> - SQL injection payloads to GIS_SPS_INTF_004 (FR-006 / SEC-001–003 / SC-005)
>   — deferred pending separate GIS enhancement item.

---

## Group F — Deployment Preparation

*Depends on all Group E evidence gathered and approved.*

**F-1** — Package `GIS_SPS_INTF_001` changes for deployment to the UAT environment.
Smoke test: process one HV SS Transformer record and one HV PM Transformer record;
verify SS_NO populated correctly in TX_LOAD_READING.
*Expected result*: Interface operational in UAT; smoke tests pass.

**F-2** — Prepare release notes covering: SS_NO population logic (HV SS, HV PM, other),
null handling rule, TX_CORE_LOAD exclusion, no schema change, and what is unchanged
(existing interface behavior, other tables). Note EH-004 and SQL injection item as
explicitly deferred.
*Expected result*: Release notes reviewed and approved by GIS lead.

**F-3** — Confirm production deployment checklist covers: GIS_SPS_INTF_001 updated
configuration, SSNUM data quality pre-checks for HV Substation and HV PM Transformer
features (DEP-002), HV SS mapping readiness (DEP-001), and rollback steps.
*Expected result*: Deployment checklist signed off.

**F-4** — Raise or confirm status of a separate GIS enhancement item for SQL injection
remediation of GIS_SPS_INTF_004 (Open Risk — High/High). This item must not be deployed
to production without a resolution plan for the SQL injection risk.
*Expected result*: Separate item raised (or in-flight); status documented.

---

## Group G — Documentation Update

*Can run in parallel with Group F.*

**G-1** — Update spec.md Status field from "Draft" to "Ready for Implementation" once
all validation evidence is gathered.
*Expected result*: Spec status reflects implementation readiness.

**G-2** — Record final test evidence file paths and pass/fail outcomes in the spec or
linked test artifact for BA / QA review. Note EH-004 and SQL injection as explicitly
deferred and link to separate item.
*Expected result*: Evidence traceable from spec; deferred items clearly flagged with
next-action owner.

---

## Dependency Summary

```
A-1, A-2, A-3, A-4, A-5, A-6   (parallel — no dependencies)
            ↓
           B-1                   (after A-2, A-3)
            ↓
      C-1, C-2, C-3              (parallel after B-1 + A-4, A-5)
            ↓
         D-1 → D-2               (sequential after C-1, C-2, C-3)
            ↓
E-1 through E-11                 (parallel after Groups B–D)
            ↓
F-1, F-2, F-3, F-4 / G-1, G-2   (F and G parallel after E approved)
```

---

## Open Items Carried from Plan

| Item | Status | Action Required |
|------|--------|----------------|
| SQL injection — GIS_SPS_INTF_004 (FR-006 / SEC-001–003) | Deferred (out of scope) | Raise separate GIS enhancement item; mandatory before production (F-4) |
| EH-004 — HV SS Transformer reassignment between cycles | Deferred to Detailed Design | Add test case once resolved; currently excluded |
| HV Substation mapping data quality (DEP-001) | Risk (Medium) | Pre-check in A-4 and F-3 deployment checklist |
| SSNUM data quality on transformer features (DEP-002) | Risk (Low/Medium) | Pre-check in A-5 and F-3 deployment checklist |
