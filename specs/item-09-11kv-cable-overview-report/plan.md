# Implementation Plan: 09 - 11kV Cable Overview Report Generation

**Branch**: `item-09-11kv-cable-overview-report` | **Date**: 2026-06-09 | **Spec**: [spec.md](spec.md)

---

> ⚠️ **PLANNING GATE — PRE-DESIGN**
>
> This item is **Batch 2 / Pre-Design** status. Detailed Design must be completed and
> approved before implementation planning can proceed to execution (CON-006 / DEP-005).
>
> This document is a **pre-design planning skeleton** that captures known scope,
> approach intent, data requirements, dependencies, and open design questions. It is
> intended to support Detailed Design scoping and will be updated once Detailed Design
> is available.
>
> **Do not proceed to `/speckit.tasks` or implementation until Detailed Design is
> delivered and this plan is updated.**

---

## Summary

Deliver a new monthly CLP GIS Report integration that exports all in-service 11kV
underground cable sections (closed ring circuits only) in CSV format, delivered by month
end. The export covers cable section attributes (length, material, cable size, insulation,
commissioning data, connected substation), joint location records (latitude, longitude,
commissioning data), and cable-to-joint topology. OHL elements and non-in-service cable
are excluded. Terminated substations are identified by tracing from Device features with
SOM_SS and SOM_CCT contained by a Substation.

No ADMS scope. GIS-only output via CLP GIS Report integration.

---

## Constitution Check

| Gate | Status | Notes |
|------|--------|-------|
| Scope gate — GIS only | ✅ Pass | CLP GIS Report integration only; no ADMS scope |
| Clarity gate — requirements documented | ✅ Pass | All BRs, FRs, data fields, assumptions, dependencies, constraints explicit in spec |
| Data integrity gate — integrity controls defined | ✅ Pass | EH-001–005 (flagging, exclusion rules, non-closed-ring deferral) |
| Regression gate — non-regression activities defined | ✅ Pass | FR-009 / CON-005; non-regression listed in Test Approach |
| Layering gate — requirement/design/build/test separated | ✅ Pass | Delivery Layer Mapping in spec; sections separated in this plan |
| **Pre-Design gate** | ⚠️ **Blocked** | CON-006 / DEP-005 — Detailed Design required before proceeding |

---

## Technical Context

| Item | Value |
|------|-------|
| Platform | GIS (CLP GIS Report integration) |
| Output format | CSV |
| Delivery schedule | Monthly, by month end |
| Cable scope | In-service 11kV HV underground cable sections; closed ring circuits only |
| OHL | Explicitly excluded |
| Substation identification | Trace from Device features with SOM_SS + SOM_CCT + Substation containment |
| Batch status | **Batch 2 — Pre-Design; Detailed Design required** |
| ADMS | Not in scope |

---

## Intended Implementation Approach *(subject to Detailed Design)*

The following approach is based on the current spec and design extract. All steps are
provisional until Detailed Design confirms or revises them.

### Step 1 — In-Service Filter
- Query the GIS data model for 11kV cable sections with lifecycle status = In Service.
- Exclude all other lifecycle states (decommissioned, planned, etc.) (CON-002).

### Step 2 — Closed Ring Underground Circuit Identification
- Identify underground closed ring circuits.
- Exclude OHL circuit elements from the result set (FR-001 / CON-002).
- Behavior for non-closed-ring circuits must be confirmed in Detailed Design (EH-005).

### Step 3 — Terminated Substation Tracing
- Trace from Device features that have both SOM_SS and SOM_CCT and are spatially
  contained by a Substation feature.
- Exclude Device features with SOM_SS and SOM_CCT that are NOT contained by a
  Substation (EH-003).
- Identify each unique terminated substation that is part of the underground circuit.

### Step 4 — Cable Section Data Assembly
- For each identified in-service cable section, collect required fields:
  - DF-001 Length, DF-002 Material, DF-003 Cable size, DF-004 Insulation,
    DF-005 Commissioning data, DF-006 Connected substation identifier (FR-002).
- Flag rows with missing required fields; do not silently drop them (EH-001).
- Flag duplicate cable identifiers; do not merge silently (EH-004).

### Step 5 — Joint Location Data Assembly
- For each cable section, collect all associated joint location records:
  - DF-007 Latitude, DF-008 Longitude, DF-009 Commissioning data (FR-003).
- Flag joint rows with missing fields; do not silently drop them (EH-002).

### Step 6 — Network Topology
- Include DF-010 connectivity reference between cable sections and joints (FR-004).
- Topology detail (exact fields, row structure) to be confirmed in Detailed Design.

### Step 7 — CSV Generation and Month-End Delivery
- Produce the assembled data as a CSV output (FR-006 / CON-003).
- Deliver by month end per the agreed reporting schedule.

### Step 8 — Non-Regression
- Existing GIS reporting workflows must remain unchanged (FR-009 / CON-005).

---

## Impacted Data and Interfaces

| Item | Impact |
|------|--------|
| CLP GIS Report integration | New monthly reporting interface added |
| In-service 11kV cable section attributes | Read; exported to CSV |
| Cable joint location data (lat/long, commissioning) | Read; exported to CSV |
| Network topology (cable-to-joint) | Read; exported to CSV |
| Device features (SOM_SS, SOM_CCT, containment) | Read for trace start point identification |
| Substation features | Read for containment check |
| OHL circuit elements | Identified and excluded; not written |
| Existing GIS reporting workflows | Must not regress (FR-009 / CON-005) |

---

## Dependencies

| ID | Dependency | Risk if Unavailable |
|----|-----------|-------------------|
| DEP-001 | GIS source records for in-service 11kV cable sections and attribute data | Report cannot be generated |
| DEP-002 | Joint location coordinates and commissioning data available in GIS | Joint rows cannot be populated; EH-002 flagging activates |
| DEP-003 | Device features have SOM_SS and SOM_CCT populated and Substation containment established | Trace cannot identify terminated substations; report incomplete |
| DEP-004 | Network topology links between cable sections and joints maintained in GIS | Topology data cannot be included (DF-010) |
| **DEP-005** | **Detailed Design for this item (Batch 2) must be completed** | **Implementation planning cannot proceed; this plan remains a skeleton** |

---

## Assumptions

- GIS data model includes identifiable in-service 11kV cable sections for closed ring
  underground circuits.
- Device features have SOM_SS and SOM_CCT populated and are spatially contained by
  Substation features at report generation time.
- OHL circuit elements are identifiable and separable from underground cable data.
- Monthly schedule and month-end deadline are agreed with planning and government
  reporting stakeholders.
- Government reporting consumers accept CSV format.
- Existing GIS reporting user access controls remain unchanged.

---

## Constraints

| ID | Constraint |
|----|-----------|
| CON-001 | Scope is GIS only; CLP GIS Report integration only |
| CON-002 | In-service HV underground closed ring circuits only; OHL and non-in-service excluded |
| CON-003 | Output format is CSV; delivered by month end |
| CON-004 | No ADMS scope |
| CON-005 | No regression to existing GIS reporting workflows |
| **CON-006** | **Batch 2 / Pre-Design — Detailed Design must be completed before proceeding** |

---

## Environments

| Environment | Purpose |
|-------------|---------|
| Development | GIS environment with CLP GIS Report integration; 11kV test cable data |
| Test / UAT | Representative in-service and excluded cable data; OHL circuits; closed ring and non-closed-ring scenarios; Device features with/without SOM_SS+SOM_CCT+containment |
| Production | Live GIS environment; report deployed via standard GIS release process |

---

## Provisional Test Approach *(to be confirmed after Detailed Design)*

| Test Case | Expected Evidence |
|-----------|------------------|
| Monthly CSV run — in-service 11kV closed ring circuits | All in-service cable sections present; OHL absent (SC-001) |
| In-service filter | Non-in-service cable excluded |
| OHL exclusion | OHL elements not present in CSV (SC-001) |
| Cable section required fields complete | All 6 cable section fields populated; rows not dropped (SC-002) |
| Cable section with missing field | Row flagged; not dropped (EH-001) |
| Joint location with all fields complete | Latitude, longitude, commissioning data populated (SC-003) |
| Joint location with missing field | Row flagged; not dropped (EH-002) |
| Topology reference included | DF-010 present and linkable (FR-004) |
| Trace — Device with SOM_SS + SOM_CCT + containment | Substation identified as trace start |
| Trace — Device with SOM_SS + SOM_CCT but no Substation containment | Excluded from trace; report continues (EH-003) |
| Duplicate cable identifiers | Flagged in output; not merged (EH-004) |
| Non-closed-ring circuit | Excluded and logged; behavior confirmed per Detailed Design (EH-005) |
| Month-end delivery | CSV delivered within schedule (SC-005) |
| Non-regression — existing GIS reports | No change in behavior (FR-009 / CON-005) |

---

## Open Design Questions *(to be resolved in Detailed Design)*

| Question | Impact | Status |
|----------|--------|--------|
| Exact behavior for non-closed-ring circuits (exclude and log, or error?) | Test design; EH-005 wording | Deferred to Detailed Design |
| DF-010 topology row structure (one row per cable, or one row per cable-joint link?) | CSV schema; row count | Deferred to Detailed Design |
| Concurrent update handling (monthly run during active GIS edits) | Data consistency | Deferred to Detailed Design |
| Flagging mechanism format in CSV (extra column, separate flag file?) | CSV schema | Deferred to Detailed Design |
| Report scheduling mechanism (scheduled task, manual trigger, CLP GIS Report schedule?) | Implementation and operations | Deferred to Detailed Design |

---

## Open Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| Detailed Design not delivered before implementation pressure | Medium | High | CON-006 / DEP-005 documented; escalate if delivery timeline at risk |
| SOM_SS / SOM_CCT / containment data not fully populated in production GIS | Medium | High | Verify data completeness as part of pre-implementation readiness check (DEP-003) |
| Non-closed-ring circuit behavior undefined until Detailed Design | Medium | Medium | EH-005 documents the deferral; must be resolved before test design is finalised |
| Topology field structure (DF-010) not confirmed | Medium | Medium | Deferred to Detailed Design; CSV schema cannot be finalised without this |
| Month-end delivery scheduling mechanism not yet selected | Low | Medium | Deferred to Detailed Design; confirm scheduling approach early in design phase |
