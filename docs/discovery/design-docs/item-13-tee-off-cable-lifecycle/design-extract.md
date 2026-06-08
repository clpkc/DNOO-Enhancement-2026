# 13 - Lifecycle status update for tee-off cable segment

## What the enhancement is
This modifies [GIS_ADMS_INTF_003] Lifecycle Status Updates.
The requirement clarification says the change is needed when decommission for the line is received and the related connectivity node from ADMS (pole) is involved.

## Business purpose / problem statement
If a lifecycle status change is received, set GIS_Status = Decommission for the affected Electric Line (all asset groups/types), while preventing incorrect decommission outcomes for associated poles.

## Detailed logic / behaviour
Updated activity logic explicitly says:
- If a lifecycle status change is received that decommissions an Electric Line:
  - update GIS_Status of the Electric Line to Decommission
  - identify the associated poles using containment association

Poles are represented by:
- Table: StructureJunction
- Asset Group: 90 - Support Structure
- Asset Types:
  - 731 - HV Pole
  - 732 - LV Pole

For each identified pole:
- check whether the pole has an association with any other element
- if at least one associated element has:
  - GIS_Status / LifecycleStatus = In Service
  - or Planned Uninstalled
  then do not decommission the pole
- if there is no associated equipment, or all associated equipment has GIS_Status other than In Service or Planned Uninstalled
  - update the pole's GIS_Status to Decommission

Additional implementation detail explicitly found:
- The design comments explicitly say this used to refer to poles differently, but was updated to align with the StructureJunction support structure model.
- The same comments explicitly say the logic uses containment association rather than electric line start/end points.
- The design also explicitly says the lifecycle status update is posted via ESRI versioning.
- A comment/reply explicitly confirms that it is not posted through the GDBM Queue.

## Assumptions / constraints
- Pole decision logic depends on containment association and associated element lifecycle states.
- Scope of line status update applies to all Electric Line asset groups/types.

## Notes from review / clarification
- Clarification and comments emphasize model alignment to StructureJunction and containment-based pole lookup.
