# 02 - User length alignment tool

## What the enhancement is
This is an attribute rule enhancement on ElectricLine.

## Business purpose / problem statement
The business purpose states that LENGTHUSER must always be populated on Electric Line.

## Detailed logic / behaviour
When a new ElectricLine feature is created:
- if LENGTHUSER is null or less than or equal to 0
- GIS automatically populates LENGTHUSER using Shape_Length

The design states this applies to:
- ElectricLine feature class
- all Asset Groups
- all Asset Types

## Assumptions / constraints
- Updating existing values is out of scope.
- The design explicitly says the attribute rule itself does not prevent later modification.
- Whether the user can change the field after auto-population depends on whether the field is configured as editable or read-only in the project.

## Notes from review / clarification
- None specified beyond the stated scope limitations.
