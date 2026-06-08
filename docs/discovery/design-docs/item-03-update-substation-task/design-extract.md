# 03 - Update substation task drop down list

## What the enhancement is
This enhancement extends the existing Update substation ArcFM Task so users can update SSNAME and SSNUM using values from a maintained source table.
The source of valid substation values is the SAP_SUBSTATION_DATA staging table.

## Business purpose / problem statement
ArcFM Task reads valid Substation Number-Name pairs from the SAP_Substation_Data staging table (populated and maintained by the EWMS integration) and applies these values to GIS substation-related features.

## Detailed logic / behaviour
Data source / backend logic from EWMS design says GIS:
- picks up a substation data file from a well-known network share location
- loads the incoming substation data into a substation staging table
- marks the file as processed

Import logic aligns IS_Used attribute with used SSNAME and SSNUM instances currently in the model.

Incoming substation file fields explicitly listed:
- SUBSTATION (Substation Number)
- FL_EN_DESC
- FL_ZF_DESC
- ABC_IND
- ADD_NAME1
- ADD_NAME2
- CURCOM_DATE
- DECOM_DATE
- SCADA_CODE
- LOCATION
- PLANT_SECTION
- MAP_REF
- SCH_REF
- CLASS
- PLANNER_GRP
- STATUS_TEXT
- MANUF_1~5
- MODEL_1~5
- PM_FLOOD_INFO

GIS field mapping explicitly found:
- SUBSTATION -> SSNUM
- FL_EN_DESC -> SSNAME

It is explicitly stated:
- SSNAME should be concatenated with "S/S" if the value from EWMS does not already end with "S/S".

Front-end task behavior explicitly found:
- Substation name dropdown:
  - values come from SAP_Substation_data
  - selecting a name auto-fills the substation number
- Substation number dropdown:
  - values also come from SAP_Substation_data
  - selecting a number auto-fills the substation name
- Update button applies the selected name/number values

Enhancement scope from clarification document:
- should allow task updates for:
  - ElectricDevice.Transformer.HV PM TX
  - ElectricDevice.HV Switch.Recloser
  - ElectricDevice.HV Switch.Switch where PLACEMENT = Pole-Mounted
- background validation must check that SSNAME and SSNUM come from the same record
- previously used substation values should not appear again in future drop-down selection
- this is managed by IS_USED attribute

## Assumptions / constraints
- Source values are maintained via SAP_SUBSTATION_DATA staging import process.
- Used values are controlled through IS_USED alignment behavior.

## Notes from review / clarification
- Clarification explicitly extends applicability beyond substations to listed transformer/switch device types.
