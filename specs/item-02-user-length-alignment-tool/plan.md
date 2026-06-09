# Implementation Plan: 02 - User Length Alignment Tool

**Branch**: `item-02-user-length-alignment-tool` | **Date**: 2026-06-09 | **Spec**: [spec.md](spec.md)

---

## Summary

Add an attribute rule to the ElectricLine feature class (all Asset Groups, all Asset
Types) that automatically sets `LENGTHUSER` = `SHAPE_Length` at feature creation time
when `LENGTHUSER` is null or ≤ 0. If the user supplies a valid `LENGTHUSER` > 0 at
creation, the value is preserved unchanged. The rule fires only at creation — not on
edits to existing features. Users retain the ability to update `LENGTHUSER` after
creation for business reasons.

No ADMS scope. Oracle WACS is referenced as a downstream dependency context only.

---

## Constitution Check

| Gate | Status | Notes |
|------|--------|-------|
| Scope gate — GIS only | ✅ Pass | ElectricLine feature class only; Oracle WACS is dependency context, not scope |
| Clarity gate — requirements documented | ✅ Pass | All BRs, FRs, assumptions, dependencies, constraints explicit in spec |
| Data integrity gate — integrity controls defined | ✅ Pass | EH-001 (invalid/null geometry fail-safe); FR-001–FR-003 (null prevention) |
| Regression gate — non-regression activities defined | ✅ Pass | FR-008 / CON-004; non-regression test activities listed in Test Approach |
| Layering gate — requirement/design/build/test separated | ✅ Pass | Delivery Layer Mapping in spec; sections separated in this plan |

---

## Technical Context

| Item | Value |
|------|-------|
| Platform | ArcGIS Pro (GIS attribute rule framework) |
| Enhancement type | Attribute rule — creation-time default |
| Affected feature class | ElectricLine (all Asset Groups and Asset Types) |
| Affected attributes | `LENGTHUSER` (write on creation), `SHAPE_Length` (read) |
| Rule trigger | Feature creation only — not edits to existing features |
| Rule condition | `LENGTHUSER` is null or ≤ 0 |
| Rule action | Set `LENGTHUSER` = `SHAPE_Length` |
| Creation methods in scope | Interactive, batch, and import |
| Post-creation edits | `LENGTHUSER` remains editable; rule does not re-fire |
| Oracle WACS | Downstream dependency context only — not in scope |

---

## Implementation Approach

### Step 1 — Attribute Rule Definition
- Define a new attribute rule on the `ElectricLine` feature class scoped to the
  **creation** event only (not update/delete).
- Rule logic:
  - Condition: `LENGTHUSER IS NULL OR LENGTHUSER <= 0`
  - Action: `LENGTHUSER = SHAPE_Length`
- Rule must **not** fire during edits to existing ElectricLine features (FR-005).

### Step 2 — Scope Configuration
- Apply the rule across **all Asset Groups and Asset Types** for ElectricLine (FR-006).
- No asset-type-specific branching required.

### Step 3 — User Value Preservation
- Rule is conditional: if `LENGTHUSER > 0` at creation time, the rule does not execute
  and the user-supplied value is preserved (FR-002).
- Rule fires only when `LENGTHUSER` is null or ≤ 0.

### Step 4 — Fail-Safe for Invalid Geometry
- If `SHAPE_Length` is unavailable or null at creation time (invalid/null geometry), the
  creation must fail safe:
  - The record is NOT saved with a null `LENGTHUSER`.
  - A clear error message is presented to the user (EH-001).

### Step 5 — Post-Creation Edit Behaviour
- After the rule fires at creation, `LENGTHUSER` remains a user-editable field.
- Subsequent edits to `LENGTHUSER` do not trigger the attribute rule (FR-004 / FR-005).

### Step 6 — Deployment
- Rule is deployed to the GIS schema/geodatabase configuration.
- No application code changes expected beyond the attribute rule configuration.
- Standard GIS schema deployment process applies.

---

## Impacted Data and Interfaces

| Item | Impact |
|------|--------|
| `ElectricLine` feature class | Attribute rule added; creation behavior changes |
| `LENGTHUSER` attribute | Written at creation when null or ≤ 0; editable after creation |
| `SHAPE_Length` attribute | Read at creation as default source; never written by rule |
| All other ElectricLine attributes | Not modified by this rule |
| Oracle WACS | Downstream consumer; receives non-null `LENGTHUSER` — no interface change |
| Other GIS feature classes | Not impacted |
| Existing GIS editing tools | Must not regress (FR-008 / CON-004) |
| Batch / import creation paths | Rule fires for all creation events including batch/import (FR-007) |

---

## Dependencies

| ID | Dependency | Risk if Unavailable |
|----|-----------|-------------------|
| DEP-001 | `LENGTHUSER` and `SHAPE_Length` attributes present on ElectricLine feature class in GIS schema | Rule cannot reference required fields; build blocked |
| DEP-002 | GIS attribute rule framework supports creation-event-only rules with conditional logic | Rule cannot be restricted to creation; fires on edits — regression risk |
| DEP-003 | Oracle WACS as downstream context only; no interface change required | No build risk; downstream consumption relies on non-null `LENGTHUSER` being delivered |

---

## Assumptions

- `SHAPE_Length` is system-calculated and available before the attribute rule fires at
  feature creation.
- `LENGTHUSER` = 0 or negative is treated equivalently to null for the defaulting
  condition.
- The attribute rule framework supports conditional logic (`IS NULL OR <= 0` condition).
- The rule does not re-fire on subsequent edits to existing features — this is a creation
  trigger only.
- Existing ElectricLine records are not retroactively updated; out of scope.
- Editor permissions for creating and updating ElectricLine features remain unchanged.
- Oracle WACS consumes `LENGTHUSER` without requiring any interface-level change.

---

## Constraints

| ID | Constraint |
|----|-----------|
| CON-001 | Scope is GIS only; ElectricLine feature class only |
| CON-002 | Wording and scope aligned with stated business requirement |
| CON-003 | No ADMS scope |
| CON-004 | No regression to existing GIS creation and edit workflows |

---

## Environments

| Environment | Purpose |
|-------------|---------|
| Development | GIS schema with ElectricLine feature class; attribute rule authoring and local verification |
| Test / UAT | Representative ElectricLine data; all Asset Group / Asset Type combinations; batch and import creation testing |
| Production | Live GIS environment; rule deployed via standard GIS schema release process |

---

## Test Approach and Validation Evidence

| Test Case | Expected Evidence |
|-----------|------------------|
| Create ElectricLine with no LENGTHUSER supplied | LENGTHUSER = SHAPE_Length after save (SC-001) |
| Create ElectricLine with LENGTHUSER = 0 | LENGTHUSER = SHAPE_Length after save (FR-001) |
| Create ElectricLine with LENGTHUSER = null explicitly | LENGTHUSER = SHAPE_Length after save (FR-001) |
| Create ElectricLine with LENGTHUSER > 0 (user-supplied) | LENGTHUSER unchanged after save (SC-002 / FR-002) |
| Edit existing ElectricLine — change unrelated attribute | LENGTHUSER not reset; rule does not fire (FR-005) |
| Edit existing ElectricLine — update LENGTHUSER manually | Updated value retained; rule does not re-fire (SC-004 / FR-004) |
| Batch / import creation of multiple ElectricLines | All records have non-null LENGTHUSER; rule fires per feature (FR-007) |
| All Asset Group / Asset Type combinations (sample set) | Rule fires consistently for all combinations (SC-003 / FR-006) |
| Invalid or null geometry — SHAPE_Length unavailable | Creation fails safe; clear message; LENGTHUSER not saved as null (EH-001) |
| Non-regression — other GIS editing tools | No change in behavior; regression test pass |

---

## Open Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| Attribute rule framework does not support creation-only trigger (fires on edits too) | Low | High | Verify rule trigger scope during dev environment setup; add explicit edit-trigger suppression logic if needed |
| `SHAPE_Length` not available at rule fire time during creation | Low | Medium | Confirm calculation order in GIS schema; EH-001 defines fail-safe if unavailable |
| Batch/import path bypasses attribute rule framework | Medium | Medium | Test batch and import creation paths explicitly during UAT; confirm rule fires for all creation methods |
| Existing ElectricLine records with null `LENGTHUSER` surface as a migration concern | Low | Low | Out of scope per spec; flag for separate remediation if downstream impact is identified |
| Oracle WACS receives null `LENGTHUSER` from pre-existing records not covered by this rule | Low | Low | Downstream; not in scope; note as known gap if raised |
