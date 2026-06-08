# 01 - Electric line splitting tool

## What the enhancement is
This is an ArcGIS Pro task for splitting an Electric Line.
In the design, it sits under the Line folder in the Tasks Pane.

## Business purpose / problem statement
The business purpose says the user needs to split an Electric Line at a specific point of division, and the user length of the resulting child lines must be recalculated proportionally according to the geometry split.

## Detailed logic / behaviour
Task behaviour / flow:
- The user starts the task.
- If the Electric Line is not already selected, the user selects the line to be split.
- A dialog window then appears and shows:
  - Electric Line ID, populated with the selected line's GlobalId
  - User Length, pre-populated as follows:
    - if LENGTHUSER > 0, use LENGTHUSER
    - otherwise, use Shape_Length
- The design explicitly says the User Length field is editable at this stage.
- The user then selects the division point on the selected line.
- The task updates the current session by:
  - splitting the parent line into two child electric lines
  - assigning LENGTHUSER to the child lines based on the same proportion as the geometric split

Explicit example:
- if the geometry is split in a 2:3 ratio, the User Length is also distributed in the same 2:3 ratio

Requirement clarification explicitly states:
- the user is not supposed to input the split ratio
- the system must compute the ratio from the actual split point

## Assumptions / constraints
- The task can only be executed in a session where editing is possible.
- GIS can capture only 2-dimensional circuit length.
- CLP still requires business users to input the actual installed onsite length into the segment length field.
- The enhancement is intended for ElectricLine feature class, all AssetGroups and AssetTypes.

## Notes from review / clarification
- A design review comment explicitly states that the user does not need to call a separate split tool manually.
- The reply says the implementation will call an OOT function for split-at-point.
