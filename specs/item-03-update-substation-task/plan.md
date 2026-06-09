# Implementation Plan: 03 - Update Substation Task Drop Down List

**Branch**: `item-03-update-substation-task` | **Date**: 2026-06-09 | **Spec**: [spec.md](spec.md)

---

## Summary

Extend the existing Update Substation ArcFM Task to support SSNAME and SSNUM dropdown
updates for additional GIS asset types: HV PM TX Transformers, Reclosers, and
Pole-Mounted HV Switches (`PLACEMENT = Pole-Mounted`), in addition to the existing
Substation support. Dropdown values are sourced from the `SAP_SUBSTATION_DATA` staging
table with bidirectional auto-fill (name ↔ number), IS_USED exclusion, and server-side
validation that submitted pairings originate from the same staging record. SSNAME values
are stored with a mandatory "S/S" suffix.

No ADMS scope. EWMS is a downstream dependency context only.

---

## Constitution Check

| Gate | Status | Notes |
|------|--------|-------|
| Scope gate — GIS only | ✅ Pass | ArcFM Task + GIS asset types only; EWMS is dependency context, not scope |
| Clarity gate — requirements documented | ✅ Pass | All BRs, FRs, VRs, EH, assumptions, dependencies, constraints explicit in spec |
| Data integrity gate — integrity controls defined | ✅ Pass | VR-001–007 (pair validation, IS_USED, S/S suffix); EH-001 (table unavailable fail-safe) |
| Regression gate — non-regression activities defined | ✅ Pass | FR-011 / CON-004; non-regression test activities listed in Test Approach |
| Layering gate — requirement/design/build/test separated | ✅ Pass | Delivery Layer Mapping in spec; sections separated in this plan |

---

## Technical Context

| Item | Value |
|------|-------|
| Platform | ArcGIS Pro (ArcFM Task framework) |
| Affected task | Update Substation ArcFM Task |
| Extended asset types | Substation (existing); HV PM TX Transformer; Recloser; Pole-Mounted HV Switch (`PLACEMENT = Pole-Mounted`) |
| Reference data source | `SAP_SUBSTATION_DATA` staging table |
| Key staging table fields | `SUBSTATION` → SSNUM; `FL_EN_DESC` → SSNAME; `IS_USED` |
| Dropdown behavior | Bidirectional auto-fill (name selects number; number selects name) |
| IS_USED filter | Values where `IS_USED = true` excluded from dropdown |
| Validation type | Server-side, pre-commit |
| SSNAME suffix rule | Append "S/S" if not already present before save |
| EWMS | Downstream dependency context; staging import not in scope |

---

## Implementation Approach

### Step 1 — Extend Supported Asset Types
- Update the task to recognise and process the following asset types in addition to
  Substations:
  - `ElectricDevice.Transformer.HV PM TX`
  - `ElectricDevice.HV Switch.Recloser`
  - `ElectricDevice.HV Switch.Switch` where `PLACEMENT = Pole-Mounted`
- For HV Switch assets where `PLACEMENT` ≠ Pole-Mounted, the task MUST NOT proceed and
  MUST present a clear message (FR-010 / EH-002).

### Step 2 — Dropdown Sourcing and IS_USED Filtering
- Source all dropdown values (SSNAME and SSNUM) from `SAP_SUBSTATION_DATA`.
- Filter out records where `IS_USED = true` — these are not presented as selectable
  options (FR-004 / VR-005).
- When no eligible values remain (all are IS_USED = true), display an empty dropdown
  with a clear message (EH-003).

### Step 3 — Bidirectional Auto-Fill
- Selecting an SSNAME from the dropdown: auto-fill SSNUM with the paired value from
  the same `SAP_SUBSTATION_DATA` record (FR-003).
- Selecting an SSNUM from the dropdown: auto-fill SSNAME with the paired value from
  the same `SAP_SUBSTATION_DATA` record (FR-003).
- Auto-fill operates client-side based on the data loaded from `SAP_SUBSTATION_DATA`.

### Step 4 — Server-Side Validation
- When the Update button is pressed, perform server-side validation before committing:
  - Confirm submitted SSNAME and SSNUM originate from the same record in
    `SAP_SUBSTATION_DATA` (VR-001).
  - Reject update if SSNAME is present without a matching SSNUM from the same record
    (VR-002).
  - Reject update if SSNUM is present without a matching SSNAME from the same record
    (VR-003).
- If `SAP_SUBSTATION_DATA` is unavailable during validation, fail safe: do not commit
  the update; present a clear message to the user (EH-001 / VR-007).

### Step 5 — SSNAME "S/S" Suffix Rule
- Before saving SSNAME to the GIS asset, check whether the value already ends with
  "S/S".
- If not, append " S/S" to the SSNAME value before writing to the GIS attribute
  (VR-004 / BR-003).

### Step 6 — Commit
- If all validations pass, commit the SSNAME and SSNUM values to the selected GIS
  feature through the ArcFM Task framework.
- Unrelated GIS task behavior must remain unchanged (FR-011 / CON-004).

---

## Impacted Data and Interfaces

| Item | Impact |
|------|--------|
| Update Substation ArcFM Task | Extended to cover additional asset types; validation and suffix logic added |
| `SAP_SUBSTATION_DATA` staging table | Read-only reference for dropdown values and server-side validation |
| `SSNAME` attribute (all supported asset types) | Written with "S/S" suffix applied; validated against staging record |
| `SSNUM` attribute (all supported asset types) | Written; validated against staging record |
| `IS_USED` attribute in `SAP_SUBSTATION_DATA` | Read to filter dropdown; not written by this enhancement |
| `PLACEMENT` attribute on HV Switch | Read to determine task eligibility; not written |
| EWMS integration | Downstream; populates `SAP_SUBSTATION_DATA` via file import — not modified |
| Existing Substation task behavior | Must not regress |
| Other GIS tasks and workflows | Must not regress (FR-011) |

---

## Dependencies

| ID | Dependency | Risk if Unavailable |
|----|-----------|-------------------|
| DEP-001 | `SAP_SUBSTATION_DATA` staging table available and current | Dropdown empty; validation cannot complete; fail-safe activates (EH-001) |
| DEP-002 | Update Substation ArcFM Task remains the GIS workflow entry point | Task extension cannot be built on existing entry point |
| DEP-003 | Supported GIS asset types expose `SSNAME` and `SSNUM` fields in GIS schema | Attributes cannot be written; build blocked |
| DEP-004 | `IS_USED` attribute in `SAP_SUBSTATION_DATA` maintained by import alignment | Filtering incorrect; used values may appear in dropdown |
| DEP-005 | EWMS file feed and network share import process | Staging table not refreshed; downstream context only; no build risk |

---

## Assumptions

- `SAP_SUBSTATION_DATA` is the single authoritative source for valid SSNAME/SSNUM pairings.
  The import process (EWMS file → network share → staging table) is outside this
  enhancement's scope.
- `IS_USED` is correctly maintained by the import alignment process at the time the task
  is used.
- All four supported asset types expose `SSNAME` and `SSNUM` fields in the existing GIS
  schema.
- The `PLACEMENT` attribute is available on HV Switch features and correctly populated.
- EWMS consumes GIS output as a downstream dependency; no EWMS interface change is required.
- User access and permissions for the Update Substation task remain unchanged.

---

## Constraints

| ID | Constraint |
|----|-----------|
| CON-001 | Scope is GIS only |
| CON-002 | Validation and data integrity requirements must remain explicit |
| CON-003 | No ADMS scope |
| CON-004 | No regression to existing GIS task behavior |

---

## Environments

| Environment | Purpose |
|-------------|---------|
| Development | ArcGIS Pro + ArcFM Task framework; `SAP_SUBSTATION_DATA` staging table available with test data |
| Test / UAT | All four supported asset types; representative staging data including IS_USED = true records; test cases for valid/invalid pairings, unavailable table, and SSNAME suffix rule |
| Production | Live GIS environment; change deployed via standard GIS release process |

---

## Test Approach and Validation Evidence

| Test Case | Expected Evidence |
|-----------|------------------|
| Valid SSNAME + SSNUM pair — all supported asset types | Update saved; SSNAME/SSNUM match staging record (SC-001/SC-003) |
| Invalid SSNAME + SSNUM pair (mismatched records) | Update rejected server-side; clear message presented (SC-002 / VR-001) |
| Select SSNAME from dropdown | SSNUM auto-fills with paired value (FR-003) |
| Select SSNUM from dropdown | SSNAME auto-fills with paired value (FR-003) |
| Dropdown — IS_USED = true records | Values not presented as selectable options (SC-004 / VR-005) |
| All dropdown values IS_USED = true | Empty dropdown with clear message (EH-003) |
| SSNAME without "S/S" suffix from EWMS | "S/S" appended before save; stored correctly (SC-005 / VR-004) |
| SSNAME already ending in "S/S" | Value stored unchanged (VR-004) |
| HV Switch with PLACEMENT = Pole-Mounted | Task proceeds normally (FR-001) |
| HV Switch with PLACEMENT ≠ Pole-Mounted | Task does not proceed; clear message (EH-002 / FR-010) |
| Unsupported asset type | Task does not proceed; clear message (EH-002) |
| SAP_SUBSTATION_DATA unavailable at validation | Update not committed; fail-safe message (EH-001 / VR-007) |
| Non-regression — existing Substation task behavior | No change in behavior; regression test pass (FR-011) |

---

## Open Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| `SAP_SUBSTATION_DATA` schema or field names differ in production | Low | High | Verify field names (`SUBSTATION`, `FL_EN_DESC`, `IS_USED`) against production schema before build |
| `IS_USED` maintenance lag — stale values appear in dropdown | Medium | Medium | Note as operational dependency (DEP-004); not a build issue; raise with EWMS integration owner |
| `PLACEMENT` attribute not populated on all HV Switch features in production | Medium | Medium | Verify data completeness in UAT; document behavior for null/empty `PLACEMENT` (treat as non-Pole-Mounted — fail safe) |
| Auto-fill performance with large `SAP_SUBSTATION_DATA` table | Low | Low | Confirm table size; add query filter at load time if needed |
| EWMS file feed delay causes stale staging data at task use time | Low | Medium | Downstream operational risk; outside scope; note as known gap |
