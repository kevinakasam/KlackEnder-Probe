# KlackEnder Probe Optimized

This repository contains a modified version of the KlackEnder Probe project. The original project had issues with Adaptive Meshes and unnecessary sensor movements. This version includes fixes and optimizations to address these issues.

## Modifications and Improvements

### Adaptive Meshes Fix
The original KlackEnder Probe did not work correctly with Adaptive Meshes. This modification ensures proper functionality and improves the bed leveling process.

### Sensor Handling Optimization
In the original project, the sensor was always stored, even if it was needed in the next movement or already positioned. This caused many unnecessary movements. The modification optimizes sensor usage, reducing redundant operations.

## How to Use

### Installation
1. Download the modified files from this repository.
2. Follow the original installation instructions of the KlackEnder Probe, but use the modified files from this repository.

### Macro for Starting Print

Below is an example of a G-code macro to start a print, utilizing the optimizations made:

```gcode
[gcode_macro START_PRINT]
gcode:
    {% set BED_TEMP = params.BED_TEMP|default(60)|float %}
    {% set EXTRUDER_TEMP = params.EXTRUDER_TEMP|default(210)|float %}
    #{% set BED_TEMP = params.BED_TEMP|default(80)|float %}
    #{% set EXTRUDER_TEMP = params.EXTRUDER_TEMP|default(240)|float %}
    # Start heating
    M140 S{BED_TEMP}
    M104 S{EXTRUDER_TEMP}
    # Wait for bed to reach temperature
    M190 S{BED_TEMP}
    # Use absolute coordinates
    G90
    # Reset the G-Code Z offset (adjust Z offset if needed)
    SET_GCODE_OFFSET Z=0.0
    # Home the printer
    G28 P
    # Bed Level
    BED_MESH_CALIBRATE ADAPTIVE=1
    # Wait for nozzle to reach temperature
    M109 S{EXTRUDER_TEMP}
    G92 E0
    # Reset extruder
    G92 E0
    #Move Z Axis up little to prevent scratching of Heat Bed
    G0 Z2.0 F8000
