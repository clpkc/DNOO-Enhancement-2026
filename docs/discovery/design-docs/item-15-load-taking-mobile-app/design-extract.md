# 15 - Load taking mobile app enhancement

## What the enhancement is
This item focuses on the population of SS_NO in TX_LOAD_READING.
The clarification explicitly says the enhancement scope is only the population of SS_NO / SSNUM in the load table; changes to the rest of the logic are not in scope.

## Business purpose / problem statement
The SS_NO (SUBSTATION NUMBER) field in TX_LOAD_READING is populated using the SSNUM of the associated transformer feature (via Feature GUID).

## Detailed logic / behaviour
Explicit field / table mapping from design:
- SS_NO
- type: String
- description: Substation number of a transformer feature
- target table: TX_LOAD_READING
- mapped attribute: SSNUM

Asset-type-specific logic explicitly stated in clarification:
- for HV SS Transformer:
  - derive the value from the containing HV Substation
- for HV PM Transformer:
  - take the value directly from the transformer's own SSNUM
- for all other Transformer asset types:
  - SSNUM will not be populated in TX_LOAD_READING

Explicit note on field usage:
- enhancement will not extend TX_LOAD_READING with a new field
- instead it will use the existing field
- Field name: SS_NO
- Alias: SUBSTATION_NO

Explicit null-handling rule:
- if SSNUM is empty in the Transformer table or Structure Boundary
- then the SS_NO / SSNUM field in TX_LOAD_READING will be null

## Assumptions / constraints
- Scope is limited to SS_NO / SSNUM population in TX_LOAD_READING only.
- Changes to other logic are explicitly out of scope.

## Notes from review / clarification
- Clarification provides explicit transformer asset-type branching rules for SSNUM source.
