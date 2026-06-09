# Implementation Plan: 15 - Load Taking Mobile App Enhancement

**Branch**: `item-15-load-taking-mobile-app` | **Date**: 2026-06-09 | **Spec**: [spec.md](spec.md)

---

## Summary

Populate the existing `SS_NO` field (alias: `SUBSTATION_NO`) in `TX_LOAD_READING` via
`GIS_SPS_INTF_001` using asset-type-specific SSNUM source rules. No new column is added.
`TX_CORE_LOAD` is out of scope. SQL injection remediation for `GIS_SPS_INTF_004` is
explicitly deferred pending separate GIS enhancement item or re-scoping approval.

> ⚠️ **Open items**:
> - **SQL injection remediation** (GIS_SPS_INTF_004 / FR-006–007 / SEC-001–003 / SC-005):
>   Flagged out of scope for this item; a separate GIS enhancement item or explicit
>   re-scoping approval is required before it can be actioned.
> - **Reassignment behavior** (HV SS Transformer moved to different HV Substation between
>   load cycles): SS_NO behavior deferred to Detailed Design (EH-004).

---

## Constitution Check

| Gate | Status | Notes |
|------|--------|-------|
| Scope gate — GIS only | ✅ Pass | GIS_SPS_INTF_001 only; no ADMS scope |
| Clarity gate — requirements documented | ✅ Pass | All BRs, FRs, DPs, EH entries, asset-type rules explicit in spec |
| Data integrity gate — integrity controls defined | ✅ Pass | EH-001–004; null-handling (EH-002); unsupported types excluded (EH-003) |
| Regression gate — non-regression activities defined | ✅ Pass | CON-005; non-regression listed in Test Approach |
| Layering gate — requirement/design/build/test separated | ✅ Pass | Delivery Layer Mapping in spec; sections separated in this plan |

---

## Technical Context

| Item | Value |
|------|-------|
| Platform | GIS |
| Interface | GIS_SPS_INTF_001 (SS_NO population) |
| Target table | TX_LOAD_READING |
| Target field | SS_NO (alias: SUBSTATION_NO) — existing field; no new column |
| Out-of-scope table | TX_CORE_LOAD (explicitly excluded) |
| Asset-type branching | HV SS Transformer, HV PM Transformer, all other types |
| SSNUM source — HV SS | Containing HV Substation's SSNUM |
| SSNUM source — HV PM | Transformer's own SSNUM |
| SSNUM source — other types | Not populated |
| Null rule | If source SSNUM is null or empty → SS_NO written as null |
| SQL injection remediation | GIS_SPS_INTF_004 — **Deferred** (out of scope; separate item required) |
| Reassignment behavior | **Deferred to Detailed Design** (EH-004) |

---

## Implementation Approach

### Step 1 — Determine Transformer Asset Type
- For each Transformer feature processed by `GIS_SPS_INTF_001`:
  - Identify whether the asset type is **HV SS Transformer**, **HV PM Transformer**, or
    another type (FR-001).
  - Asset type must be determinable at load processing time via the GIS data model
    (Assumption).

### Step 2 — Derive SSNUM by Asset Type

**Branch A — HV SS Transformer** (FR-002 / DP-001):
- Resolve the containing HV Substation for the transformer.
- Read the SSNUM from the containing HV Substation.
- If the HV Substation cannot be resolved: SS_NO = null; record available for review
  (EH-001).
- If the HV Substation's SSNUM is null or empty: SS_NO = null (EH-002 / DP-004 / FR-005).

**Branch B — HV PM Transformer** (FR-003 / DP-002):
- Read the transformer's own SSNUM attribute directly.
- If the transformer's own SSNUM is null or empty: SS_NO = null (EH-002 / DP-004 / FR-005).

**Branch C — All other Transformer asset types** (FR-004 / DP-003 / EH-003):
- Do NOT populate SS_NO.
- No default or fallback SSNUM is substituted.

### Step 3 — Write SS_NO to TX_LOAD_READING
- Write the derived value (non-null SSNUM or null) to the existing SS_NO field in
  TX_LOAD_READING (CON-004).
- No new column is added.
- TX_CORE_LOAD is not modified (Out of Scope).

### Step 4 — Null Handling (all branches)
- If the applicable SSNUM source is null or empty for any reason, write SS_NO as null
  (FR-005 / EH-002 / DP-004).
- No partial or placeholder value is used.

### Step 5 — Non-Regression
- All existing GIS integration behavior outside this SS_NO population logic must remain
  unchanged (CON-005).
- TX_CORE_LOAD and all other load-taking interface logic must not be affected.

### Step 6 — SQL Injection Remediation *(Deferred)*
- GIS_SPS_INTF_004 search API SQL injection remediation is **not in scope** for this
  item (FR-006 / FR-007 deferred).
- No implementation steps are performed under this item.
- A separate GIS enhancement item must be raised, or explicit re-scoping approval
  obtained, before this work can proceed.

---

## Asset-Type Decision Table

| Transformer Asset Type | SSNUM Source | SS_NO Written As |
|------------------------|-------------|-----------------|
| HV SS Transformer | Containing HV Substation SSNUM | SSNUM value (or null if unavailable) |
| HV PM Transformer | Transformer's own SSNUM | SSNUM value (or null if empty) |
| All other asset types | Not applicable | Not populated |
| Any (null source) | Source is null or empty | null |

---

## Impacted Data and Interfaces

| Item | Impact |
|------|--------|
| GIS_SPS_INTF_001 | Modified — SS_NO population logic added |
| TX_LOAD_READING — SS_NO field | Written: SSNUM value or null based on asset type |
| TX_CORE_LOAD | Not modified (explicitly out of scope) |
| HV Substation — SSNUM | Read for HV SS Transformer path |
| HV SS Transformer | Read: asset type + containing HV Substation lookup |
| HV PM Transformer | Read: asset type + own SSNUM attribute |
| Other Transformer asset types | Identified; SS_NO not written |
| GIS_SPS_INTF_004 | **Not modified** — SQL injection remediation deferred |
| Existing GIS integration logic | Must not regress (CON-005) |

---

## Dependencies

| ID | Dependency | Risk if Unavailable |
|----|-----------|-------------------|
| DEP-001 | GIS mapping between HV SS Transformer and its containing HV Substation resolvable at load processing time | HV SS path fails; EH-001 null + flag activates |
| DEP-002 | SSNUM maintained on HV Substation (for HV SS path) and on transformer feature (for HV PM path) | SS_NO written as null; DP-004 null rule applies |
| DEP-003 | GIS_SPS_INTF_001 interface path available | SS_NO population cannot occur |

---

## Assumptions

- The existing SS_NO field in TX_LOAD_READING is available for population; no schema
  change is needed.
- Transformer asset type (HV SS, HV PM, or other) is determinable at load processing
  time via the GIS data model.
- The transformer-to-containing-HV-Substation association is resolvable at load
  processing time for HV SS Transformer features.
- SSNUM is a stable identifier on HV Substation features and HV PM Transformer features
  in GIS data.
- Operational teams can review records with null SS_NO for follow-up where applicable.

---

## Constraints

| ID | Constraint |
|----|-----------|
| CON-001 | Scope is GIS only |
| CON-002 | Data handling and security requirements must remain explicit |
| CON-003 | No ADMS scope |
| CON-004 | Uses existing SS_NO field; no new column; no other TX_LOAD_READING logic changes |
| CON-005 | Existing GIS integration behavior outside this requirement must not regress |

---

## Environments

| Environment | Purpose |
|-------------|---------|
| Development | GIS environment with GIS_SPS_INTF_001; HV SS Transformer, HV PM Transformer, and other-type test data; TX_LOAD_READING test records |
| Test / UAT | Asset-type scenarios (HV SS, HV PM, other); null SSNUM sources; unresolvable HV Substation; non-regression of existing interface logic |
| Production | Live GIS environment; deployed via standard GIS release process |

---

## Test Approach and Validation Evidence

| Test Case | Expected Evidence |
|-----------|------------------|
| HV SS Transformer — HV Substation has non-null SSNUM | SS_NO = HV Substation SSNUM (SC-001 / FR-002 / DP-001) |
| HV SS Transformer — HV Substation SSNUM is null | SS_NO = null (SC-003 / EH-002 / FR-005) |
| HV SS Transformer — no resolvable HV Substation | SS_NO = null; record available for review (EH-001) |
| HV PM Transformer — transformer SSNUM non-null | SS_NO = transformer's own SSNUM (SC-002 / FR-003 / DP-002) |
| HV PM Transformer — transformer SSNUM is null | SS_NO = null (SC-003 / EH-002 / FR-005) |
| Transformer — unsupported asset type | SS_NO not populated; no default written (SC-004 / FR-004 / EH-003 / DP-003) |
| Any type — SSNUM source empty string | SS_NO = null (not empty string) (FR-005 / EH-002) |
| TX_CORE_LOAD | No change to TX_CORE_LOAD records (Out of Scope) |
| No new column added | TX_LOAD_READING schema unchanged (CON-004) |
| Non-regression — existing GIS_SPS_INTF_001 logic | No change in behavior for other fields/operations (CON-005) |
| Non-regression — TX_CORE_LOAD and other tables | No modification observed (Out of Scope) |

> ⚠️ **Deferred test cases**:
> - HV SS Transformer reassigned to different HV Substation between load cycles — deferred
>   to Detailed Design (EH-004).
> - SQL injection payloads to GIS_SPS_INTF_004 — deferred pending scope confirmation
>   (FR-006 / SEC-001–003 / SC-005).

---

## Open Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| SQL injection risk in GIS_SPS_INTF_004 unmitigated until separate item raised | High | High | SEC-001–003 flagged; separate GIS enhancement item must be raised promptly; do not deploy without resolution plan |
| HV SS Transformer reassignment behavior undefined between load cycles | Medium | Medium | EH-004 deferred; must be resolved in Detailed Design before reassignment test case can be written |
| HV Substation mapping gaps in production GIS data (HV SS path) | Medium | Medium | EH-001 null + available-for-review rule limits silent data loss; DEP-001 data readiness check recommended before rollout |
| SSNUM data quality on transformer features (HV PM path) | Low | Medium | EH-002 null rule applies; DEP-002 data quality pre-check recommended |
