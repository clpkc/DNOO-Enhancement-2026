# 09 - 11kV cable overview report generation

## What the enhancement is
This is a new reporting interface.
Affected functionality is explicitly stated as CLP GIS Report integration.
The design document intended for it is stated as CLP DNOO GIS-Reporting Batch 1 Integration Design Document.

## Business purpose / problem statement
Support the 11kV cable joint replacement initiative with detailed distribution data for planning and government reporting.

## Detailed logic / behaviour
Scope explicitly stated:
- Scope covers all 11kV in-service cable.

Required report output explicitly listed:
- All 11kV cable sections with:
  - length
  - material
  - cable size
  - insulation
  - commissioning data
  - connected substation
- All cable joint locations with:
  - latitude
  - longitude
  - commissioning data
- Network topology indicating connectivity between:
  - cable
  - joint

Frequency / format:
- Frequency: monthly extraction, expected by month end
- Format: CSV

Report logic explicitly found:
- identify each unique terminated substation
- the terminated substation must be part of the underground circuit
- no OHL circuit elements are required
- identification is done by tracing
- traces start from Device features that already have:
  - SOM_SS
  - SOM_CCT
  and are contained by a Substation

Additional clarification from comments:
- this report intends to cover:
  - HV cable and joints
  - for closed ring circuits
  - but not overhead line circuits
  - and these circuits always have one SOM_SS and SOM_CCT

## Assumptions / constraints
- Monthly extraction is expected by month end.
- Output format is CSV.
- OHL circuit elements are not required.

## Notes from review / clarification
- Summary in extract states: Pre-Design.
- Explicit gap: Detailed Design is needed.
