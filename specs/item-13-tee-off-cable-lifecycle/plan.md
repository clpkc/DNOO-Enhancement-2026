# Implementation Plan: 13 - Lifecycle Status Update for Tee-Off Cable Segment

**Branch**: `item-13-tee-off-cable-lifecycle` | **Date**: 2026-06-09 | **Spec**: [spec.md](spec.md)

---

## Summary

Modify `GIS_ADMS_INTF_003` to enforce a two-step lifecycle decision on line decommission
events: (1) set GIS_Status = Decommission on the ElectricLine (all asset groups/types),
then (2) identify associated StructureJunction poles via containment association and
evaluate each for decommission eligibility. A pole is decommissioned only when no
associated elements carry GIS_Status / LifecycleStatus = In Service or Planned
Uninstalled. Updates are posted via ESRI versioning (not GDBM Queue). No ADMS or ERP
workflow scope.

> ⚠️ **Open item**: Concurrent decommission events targeting multiple ElectricLine
> features sharing the same StructureJunction pole are explicitly deferred to Detailed
> Design (EH-004). This scenario must be resolved before test design for that case can
> be finalised.

---

## Constitution Check

| Gate | Status | Notes |
|------|--------|-------|
| Scope gate — GIS only | ✅ Pass | GIS_ADMS_INTF_003 modification only; ADMS/ERP referenced as dependent outputs |
| Clarity gate — requirements documented | ✅ Pass | All BRs, FRs, DL rules, EH entries, and key entities explicit in spec |
| Data integrity gate — integrity controls defined | ✅ Pass | EH-001–005; DL-006 fail-safe on unresolved containment; blocking status covers stale states |
| Regression gate — non-regression activities defined | ✅ Pass | FR-011 / CON-005; non-regression included in Test Approach |
| Layering gate — requirement/design/build/test separated | ✅ Pass | Delivery Layer Mapping in spec; sections separated in this plan |

---

## Technical Context

| Item | Value |
|------|-------|
| Platform | GIS |
| Interface modified | GIS_ADMS_INTF_003 — Lifecycle Status Updates |
| Trigger event | Line decommission event received via GIS_ADMS_INTF_003 |
| Line entity | ElectricLine — all asset groups and asset types |
| Pole entity | StructureJunction (Asset Group 90 — Support Structure; Asset Types 731 HV Pole / 732 LV Pole) |
| Pole identification | Containment association only (start/end point lookup not used — CON-004) |
| Blocking statuses | GIS_Status / LifecycleStatus = In Service OR Planned Uninstalled |
| Update mechanism | ESRI versioning only (GDBM Queue must not be used — CON-003 / FR-007) |
| Downstream consumers | ADMS and ERP (reference only; no implementation scope) |
| Concurrent events open item | Deferred to Detailed Design (EH-004) |

---

## Implementation Approach

### Step 1 — Receive Line Decommission Event
- Accept incoming line decommission event via GIS_ADMS_INTF_003 (FR-001).
- Extract the ElectricLine feature reference from the event payload.

### Step 2 — Set ElectricLine GIS_Status = Decommission
- Set GIS_Status = Decommission on the identified ElectricLine feature.
- Apply to all asset groups and asset types without exception (FR-002 / DL-001 / SC-001).
- This step completes before pole evaluation begins.

### Step 3 — Identify Associated StructureJunction Poles via Containment Association
- Query the containment association from the ElectricLine to identify associated
  StructureJunction features (FR-003 / DL-002).
- Filter to Asset Group 90 — Support Structure; Asset Types 731 HV Pole and 732 LV Pole.
- Do NOT use start/end point lookup (CON-004 / FR-003).
- If containment association cannot be resolved: fail safe — do not decommission the
  pole; flag the record for operational review (EH-001 / FR-008 / DL-006 / SC-004).

### Step 4 — Evaluate Each Pole for Decommission Eligibility
For each identified StructureJunction pole (FR-004 / DL-003–DL-005):

**Blocking check**:
- Query all elements associated with the pole.
- If any associated element has GIS_Status / LifecycleStatus = **In Service** or
  **Planned Uninstalled**: retain the pole — do not set GIS_Status = Decommission
  (FR-005 / DL-004 / SC-002 / EH-003).
- If the pole has no associated elements, or all associated elements have GIS_Status
  other than In Service or Planned Uninstalled: set the pole's GIS_Status = Decommission
  (FR-006 / DL-005 / SC-003).

**Fail-safe cases** (retain pole, flag for review):
- Containment association not resolved (EH-001 / DL-006).
- Associated element has stale or unresolved GIS_Status — treat as blocking (EH-002).
- Mixed states including Planned Uninstalled — Planned Uninstalled is blocking (EH-003).

### Step 5 — Post Lifecycle Updates via ESRI Versioning
- Post all GIS_Status changes (ElectricLine and poles where eligible) via ESRI versioning
  (FR-007 / CON-003 / DL-007).
- Do NOT use the GDBM Queue for this event path.

### Step 6 — Downstream Dependency Output
- Resulting lifecycle status is consumed by ADMS and ERP as dependency outputs
  (FR-009 / DEP-005).
- Downstream consumption lag is not a GIS failure condition (EH-005).
- No ADMS or ERP implementation scope is introduced (FR-010 / CON-001 / CON-002).

### Step 7 — Non-Regression
- Unrelated GIS workflows must not be affected (FR-011 / CON-005).

---

## Decision Logic Summary

| Condition | Pole Outcome |
|-----------|-------------|
| At least one associated element: In Service or Planned Uninstalled | Retain (do not decommission) |
| No associated elements, or all elements other than In Service / Planned Uninstalled | Decommission (GIS_Status = Decommission) |
| Containment association cannot be resolved | Fail safe — retain; flag for review |
| Associated element has stale / unresolved GIS_Status | Fail safe — treat as blocking; retain |
| Concurrent events on shared pole | **Deferred to Detailed Design (EH-004)** |

---

## Impacted Data and Interfaces

| Item | Impact |
|------|--------|
| GIS_ADMS_INTF_003 | Modified — new two-step decommission logic |
| ElectricLine — GIS_Status | Written: set to Decommission on decommission event |
| StructureJunction (Pole) — GIS_Status | Conditionally written: Decommission or retained |
| Containment Association | Read — used to identify poles from ElectricLine |
| Associated Elements — GIS_Status / LifecycleStatus | Read — used for blocking evaluation |
| ESRI versioning | Write mechanism for all lifecycle status changes |
| ADMS | Dependent consumer of lifecycle output (reference only) |
| ERP | Dependent consumer of lifecycle output (reference only) |
| Existing unrelated GIS workflows | Must not regress |

---

## Dependencies

| ID | Dependency | Risk if Unavailable |
|----|-----------|-------------------|
| DEP-001 | Containment association in GIS reliably maps ElectricLine to StructureJunction poles | Pole identification fails; EH-001 fail-safe activates |
| DEP-002 | GIS_Status / LifecycleStatus on associated elements accurately reflects In Service or Planned Uninstalled at event time | Incorrect blocking evaluation; EH-002 stale-status fail-safe activates |
| DEP-003 | GIS_ADMS_INTF_003 integration path available | Line decommission events cannot be processed |
| DEP-004 | ESRI versioning available for posting lifecycle updates | GIS_Status changes cannot be committed |
| DEP-005 | ADMS and ERP consume lifecycle outputs | Downstream views and asset status remain incorrect; GIS-side processing remains complete |

---

## Assumptions

- ElectricLine features belong to all asset groups and asset types; GIS_Status
  Decommission update applies across the board without asset-type filtering.
- Containment association in GIS reliably maps ElectricLine to StructureJunction poles
  (Asset Group 90; Asset Types 731 / 732).
- GIS_Status / LifecycleStatus data on associated elements is sufficiently current at
  event processing time to determine blocking status.
- ESRI versioning is the approved and available mechanism for posting lifecycle updates.
- Dependent systems (ADMS / ERP) consume lifecycle outputs without requiring scope
  expansion in those systems.
- Users handling exceptions have an operational process to review records flagged where
  containment association could not be resolved.

---

## Constraints

| ID | Constraint |
|----|-----------|
| CON-001 | Scope is GIS only |
| CON-002 | ADMS and ERP referenced only as impacted/dependent systems |
| CON-003 | Lifecycle updates via ESRI versioning; GDBM Queue must not be used |
| CON-004 | Pole identification via containment association only; start/end point lookup not used |
| CON-005 | Existing GIS processes outside this event path must not regress |

---

## Environments

| Environment | Purpose |
|-------------|---------|
| Development | GIS environment with GIS_ADMS_INTF_003; ElectricLine, StructureJunction, containment association test data |
| Test / UAT | Scenarios: single pole (blocking / non-blocking), Planned Uninstalled blocking, stale status fail-safe, unresolved containment fail-safe, mixed element states, all asset groups/types, ESRI versioning path |
| Production | Live GIS environment; deployed via standard GIS release process |

---

## Test Approach and Validation Evidence

| Test Case | Expected Evidence |
|-----------|------------------|
| Decommission event received — ElectricLine status | GIS_Status = Decommission set on ElectricLine; all asset groups/types (SC-001 / FR-002) |
| Pole with In Service element | Pole retained; GIS_Status not set to Decommission (SC-002 / FR-005 / DL-004) |
| Pole with Planned Uninstalled element | Pole retained; Planned Uninstalled is blocking (EH-003 / FR-005 / DL-004) |
| Pole with mixed Planned Uninstalled + Decommissioned | Pole retained (Planned Uninstalled counts) |
| Pole with no associated elements | Pole GIS_Status set to Decommission (SC-003 / FR-006 / DL-005) |
| Pole with all elements at non-blocking status | Pole GIS_Status set to Decommission (SC-003 / DL-005) |
| Containment association unresolved | Pole not decommissioned; flagged for review (SC-004 / EH-001 / DL-006) |
| Element with stale / unresolved GIS_Status | Treated as blocking; pole retained (EH-002) |
| Multiple poles on same line — mixed eligibility | Each pole evaluated independently; only eligible poles decommissioned |
| ElectricLine with unusual asset group/type | GIS_Status Decommission applied regardless of asset group/type (FR-002) |
| ESRI versioning used | All lifecycle changes committed via ESRI versioning; not via GDBM Queue (FR-007 / CON-003) |
| GDBM Queue not used | No GDBM Queue write observed for this event path |
| Downstream output state | ADMS/ERP receive non-decommissioned pole status where pole retained (FR-009 / SC-005) |
| Non-regression — unrelated GIS workflows | No change in behavior (FR-011 / CON-005) |

> ⚠️ **Deferred test case**: Concurrent decommission events on multiple ElectricLine
> features sharing the same StructureJunction pole — test design for this scenario
> is deferred until Detailed Design resolves EH-004.

---

## Open Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| Concurrent decommission events on shared pole — behavior undefined | Medium | High | EH-004 explicitly deferred; must be resolved in Detailed Design before this test case is written |
| Containment association data quality gaps in production GIS | Medium | High | EH-001 fail-safe mandatory; data quality pre-check recommended before rollout (DEP-001) |
| GIS_Status data staleness at event processing time | Medium | Medium | EH-002 fail-safe (treat as blocking) limits exposure; DEP-002 data currency dependency documented |
| ESRI versioning availability or performance at high event volume | Low | Medium | DEP-004 dependency; confirm versioning capacity as part of environment readiness |
