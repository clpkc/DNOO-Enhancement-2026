# Implementation Plan: 01 - Electric Line Splitting Tool

**Branch**: `item-01-electric-line-splitting-tool` | **Date**: 2026-06-09 | **Spec**: [spec.md](spec.md)

---

## Summary

Build a new ArcFM task (under the Line folder in the Tasks Pane in ArcGIS Pro) that
allows a GIS editor to select an ElectricLine feature, review and optionally override the
pre-populated User Length, choose a split point on the geometry, and have the system
automatically split the parent line into two child ElectricLine features with LENGTHUSER
redistributed proportionally from the geometry split ratio.

Segment Length (onsite installed length) remains a manual user input and is out of scope.
The task is only executable within an active GIS edit session. No ADMS scope is
introduced.

---

## Constitution Check

| Gate | Status | Notes |
|------|--------|-------|
| Scope gate — GIS only | ✅ Pass | ElectricLine feature class only; ADMS not referenced as scope |
| Clarity gate — requirements documented | ✅ Pass | All BRs, FRs, assumptions, dependencies, constraints explicit in spec |
| Data integrity gate — integrity controls defined | ✅ Pass | FR-008 (rounding rule), EH-001 (zero-length guard), FR-003 (pre-population fallback) |
| Regression gate — non-regression activities defined | ✅ Pass | FR-007 and CON-005; non-regression test activities listed in Test Approach |
| Layering gate — requirement/design/build/test separated | ✅ Pass | Delivery Layer Mapping in spec; sections separated in this plan |

---

## Technical Context

| Item | Value |
|------|-------|
| Platform | ArcGIS Pro (ArcFM task framework) |
| Task location | Line folder — Tasks Pane |
| Affected feature class | ElectricLine (all AssetGroups and AssetTypes) |
| Affected attributes | LENGTHUSER, Shape_Length (read), GlobalId (read) |
| Edit mechanism | Active GIS edit session (ESRI versioning context) |
| Split function | Out-of-the-box ArcFM/ArcGIS split-at-point function (OOT call) |
| Segment Length | Manual user input — not affected |
| ADMS | Not in scope; dependency reference only |

---

## Implementation Approach

### Step 1 — Task Registration
- Register a new ArcFM task in the Line folder of the Tasks Pane.
- Task is only accessible and executable when a GIS edit session is active (EH-002 /
  FR-009).

### Step 2 — Line Selection and Dialog
- User selects (or the task prompts selection of) the ElectricLine feature to split.
- Task dialog opens and displays:
  - **Electric Line ID**: read-only field, populated from the selected feature's
    `GlobalId`.
  - **User Length**: editable field, pre-populated using: `LENGTHUSER > 0 → LENGTHUSER`
    else `Shape_Length` (FR-003).

### Step 3 — Split Point Selection
- User selects the division point on the selected geometry.
- The system validates the split point is not at or near an existing endpoint (EH-001).
  If the split would produce a zero-length child, the task halts and presents a clear
  message; the split is not executed.

### Step 4 — Ratio Computation and LENGTHUSER Redistribution
- Derive split ratio from geometry: `ratio_B = length_child_B / length_parent_geometry`;
  `ratio_C = 1 - ratio_B`.
- Apply ratio to parent User Length (stored or editor-supplied):
  - `LENGTHUSER_B = round(ratio_B × user_length, 2)`
  - `LENGTHUSER_C = round(ratio_C × user_length, 2)`
- Assign any rounding residual to the longer child so that
  `LENGTHUSER_B + LENGTHUSER_C = user_length` exactly (FR-008).

### Step 5 — Split Execution
- Call the OOT ArcFM/ArcGIS split-at-point function to physically split the parent
  ElectricLine geometry into two child ElectricLine features.
- Write computed `LENGTHUSER` values to each child feature.
- Segment Length field is not written by this task.

### Step 6 — Edit Session Completion
- Changes are staged within the active edit session.
- The editor saves or discards changes through standard GIS edit session workflow.
- No GDBM Queue or external interface event is triggered by this task.

---

## Impacted Data and Interfaces

| Item | Impact |
|------|--------|
| `ElectricLine` feature class | Split operation creates two child features; LENGTHUSER written on each |
| `LENGTHUSER` attribute | Read from parent; computed and written to each child |
| `Shape_Length` attribute | Read as fallback when LENGTHUSER = 0 or null; never written by task |
| `GlobalId` attribute | Read to populate Electric Line ID in dialog; never written |
| Segment Length field | Read-only context; not modified by this task |
| ArcFM Tasks Pane (Line folder) | New task entry registered |
| Other ElectricLine attributes | Not modified by this task |
| ADMS interfaces | Not impacted; no interface events triggered |
| Existing GIS editing tools | Must not regress (FR-007 / CON-005) |

---

## Dependencies

| ID | Dependency | Risk if Unavailable |
|----|-----------|-------------------|
| DEP-001 | ArcGIS Pro ArcFM task framework; Line folder in Tasks Pane | Task cannot be built or registered |
| DEP-002 | `LENGTHUSER` and `Shape_Length` attributes on ElectricLine in GIS data model | Pre-population and redistribution logic cannot execute |
| DEP-003 | Active GIS edit session | Task cannot run; EH-002 enforces this |
| DEP-004 | OOT split-at-point function (ArcFM/ArcGIS) | Custom geometry split must be implemented if OOT function unavailable — scope risk |

---

## Assumptions

- The OOT ArcFM/ArcGIS split-at-point function is available in the target environment
  and produces two valid child ElectricLine features.
- `LENGTHUSER` and `Shape_Length` are present on the ElectricLine feature class in the
  production GIS schema.
- All ElectricLine AssetGroups and AssetTypes are handled identically by the task; no
  asset-type-specific branching is required.
- The task creates a new entry in the Line folder; it does not replace or modify an
  existing split tool.
- GIS captures only 2-dimensional circuit length; `LENGTHUSER` and `Shape_Length`
  reflect 2D geometry only.
- Editor access rights and GIS edit permissions remain unchanged.

---

## Constraints

| ID | Constraint |
|----|-----------|
| CON-001 | Scope is GIS only; ElectricLine feature class only |
| CON-002 | Segment Length is not automatically redistributed |
| CON-003 | LENGTHUSER and Shape_Length reflect 2D geometry only |
| CON-004 | Task must only execute within an active GIS edit session |
| CON-005 | No regression to existing GIS editing behavior |
| CON-006 | No ADMS scope |

---

## Environments

| Environment | Purpose |
|-------------|---------|
| Development | ArcGIS Pro + ArcFM task framework; GIS data model with ElectricLine feature class |
| Test / UAT | Representative ElectricLine data set; all AssetGroup / AssetType combinations; edit session environment |
| Production | Live GIS environment; change deployed via standard GIS release process |

---

## Test Approach and Validation Evidence

| Test Case | Expected Evidence |
|-----------|------------------|
| Equal split (50:50) on parent LENGTHUSER = 100 | Both children LENGTHUSER = 50; total = 100 |
| Non-equal split (2:3) on parent LENGTHUSER = 100 | Children LENGTHUSER = 40 and 60; total = 100 |
| Parent LENGTHUSER = 0 → Shape_Length fallback | Children redistributed from Shape_Length; total = Shape_Length |
| Editor overrides pre-populated value | Children redistributed from editor-supplied value |
| Rounding case with decimal ratio | Residual assigned to longer child; totals match exactly |
| Split point at or near endpoint (degenerate) | Task halts; clear message presented; no zero-length child created |
| Task invoked outside edit session | Task does not execute; clear message presented |
| All AssetGroup / AssetType combinations | Task behaves consistently across all combinations |
| Non-regression — other GIS editing tools | No change in behavior; verified by regression test pass |

---

## Open Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| OOT split-at-point function not available in target environment | Low | High | Verify during dev environment setup; fallback requires custom geometry split (scope escalation) |
| `LENGTHUSER` or `Shape_Length` missing from production schema | Low | High | Confirm attribute availability in GIS data model before build starts (DEP-002) |
| Degenerate split threshold definition (how close is "near endpoint") | Medium | Medium | Define tolerance during detailed design; align with GIS geometry precision settings |
| Rounding edge cases for unusual ratios | Low | Low | FR-008 defines the rule (2dp, residual to longer child); cover in unit tests |
| User override with value ≠ geometry-derived length causes confusion | Low | Low | Dialog field is explicitly editable per spec; no validation required beyond non-zero |

---

## Delivery Layer Summary

| Layer | Artifact |
|-------|---------|
| Business Requirement | [spec.md — Business Requirements](spec.md#business-requirements) |
| Technical Design | [spec.md — Delivery Layer Mapping](spec.md#delivery-layer-mapping) |
| Implementation | This plan — Implementation Approach section |
| Test | This plan — Test Approach and Validation Evidence section |
| Tasks | `tasks.md` (generated by `/speckit.tasks` — not yet created) |
