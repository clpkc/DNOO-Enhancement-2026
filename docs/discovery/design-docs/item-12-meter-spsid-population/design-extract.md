# 12 - Meter SPSID attribute automatic population

## What the enhancement is
This item affects:
- GIS CCMS Integration
- GIS custom CCMS task
- GIS FSS Integration
- Enlight CCS
- Enlight OFS

## Business purpose / problem statement
Automatic update of Supply Point number (SPSID) on Meter.
When the association with a Supply Point is created, updated, or deleted, the Supply Point number (SPSID) on the Meter should be updated accordingly.

Problem statement explicitly stated in clarification:
- Meter has two supply-point-related attributes:
  - SPSID: free text Supply Point Number, not populated
  - SUPPLY_POINT: reference to Supply Point, outdated and different from the association used by integrations

## Detailed logic / behaviour
Explicit requirement:
- when the supply point on the meter is updated, SPSID on the meter must also be updated
- the relation attribute SUPPLY_POINT between Meter and Supply Point should be removed

Explicit systems/interfaces/functions to be updated:
- [GIS_CCMS_INTF_001] CreateUsagePoints
- [GIS_CCMS_INTF_003] UpdateUsagePoints
- CreateMissingAssociations
- Enlight GIS CCS 1
- Enlight CreateMissingAssociations
- CCMS custom task
- [GIS_FSS_INTF_001] Associate Supply Point and Premise
- GIS OFS 2

Explicit behavior found in CCMS-GIS design (Create Missing Associations logic):
- if a Premise has an association to a Supply Point:
  - create the association for all meters associated to that Premise
  - if some meters are associated with another Supply Point, delete that association
  - populate the SPSID attribute of the meter using the SPSID of the associated Supply Point

Explicit behavior found in Enlight CCS design:
- an electrical association to the identified Supply Point is created/re-established for the relevant meters
- the SPSID attribute of the Meter is populated with the SPSID from the newly connected Supply Point

Explicit behavior found in Enlight OFS design:
- enhancement "Meter SPSID attribute automatic population"
- populate SPSID on the meter (Electric Junction Object) during update of its association with Supply Point, using the Supply Point's SPSID

## Assumptions / constraints
- Scope is specifically SPSID update behavior linked to Supply Point association lifecycle.
- SUPPLY_POINT relation attribute is considered outdated for integration usage and is to be removed per requirement clarification.

## Notes from review / clarification
- Clarification explicitly names both data consistency issue (SPSID not populated) and legacy attribute issue (SUPPLY_POINT outdated).
