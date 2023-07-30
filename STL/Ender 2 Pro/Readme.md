# KlackEnderPro2

A modified version of the KlackEnder project to work with an Ender Pro 2.
Its a remix from daaar on Thingiverse.com



There weren't step files included so I had to modify some of the models creatively. Tolerances are between 5.3mm and 5.4mm depending on the hole. This is specific to the accuracy of my printer (which had damaged kinematics at the time).

The original Ender 2 pro version used a Retainer that didnt leave enough clearance against the bed. I did modify that design but ended up making one which attaches to the base screws for the gantry arm.



I made use of Klipper and a replacement board 4.2.7 board on my Ender 2 Pro. Also I have a custom head configuration. So some tolerances might be specific to my setup.



The 10mm variant is experimental and does not currently work. I made it before I had 5mm magnets. Make sure the magnets you use are conductive too.



## Printing order:

The tolerance test is helpful to know if the existing models will work for your selected 5mm magnets and printer.

1. Probe Block (Insert Magnets - the two side-by-side magnets need to have opposite polarity … see reference documents)
2. Probe Dock (Use the Block to determine polarity of magnets to determine the correct polarity here)
3. Probe Mount (Use the block to determine the correct polarity of magnets. Take note of the correct orientation). The tolerance of the rectangular hole used to hold the 4. piece might be too small. I ended up scraping the hole to the correct dimension.
5. Probe retainer (Can also be printed first. Has one flat side so make sure of orientation. Needs supports. I suggest organic supports anywhere on the build plate)



## Modified Klippper Configuration

One of the useful links contains my actively maintained Klipper configurations. However it is very specific to my setup. Im hoping to merge these files into the main GitHub repo by Kevinakasam's with exact instructions on how to get the probe to work.



## Configuration Notes:

Ensure you have a functional Klipper configuration. Some of the X and Y values in the macros needed to be modified for the correct xy values

```
[stepper_x]
position_endstop: -15
position_max: 165 # Ensure the maximum position is set here
... # Other configuration for stepper_x here

[stepper_y]
position_min: -5
position_max: 165
... # Other configuration for stepper_x here

# Make note of the endstop pin here. Will be specific to the board you are using
[stepper_z]
... # Other configuration for stepper_z here
position_max: 180
endstop_pin: probe:z_virtual_endstop #if you want to use the Prove as z-endstop (You can unsinstall the stock z endstop then. If not, remove the [homing_override])
position_min: -8 # set a negative value (minimum as the probe z_offset), this needs to be lower than the offset

... # Other configuration up before Mainsail parts of the config

#####################################################################
# KlackEnder- Settings
#####################################################################

# !! Change your Z endstop pin from 'endstop_pin: Pin123' to 'endstop_pin: probe:z_virtual_endstop'
# !! Also add in [stepper_y] 'position_min: -8'. Idk why but most configs mave this wrong. For the Stock Ender 3 the homed Y position is -8.

[probe]
pin: ^PA7 #^PC14 #Probe-Stop Connection on Skr Mini E3 V1.2
z_offset = 2.105 #Measure per your specific setup. Can be done by homing x, y, and z and then move to 20,30,0 and use a piece of paper to see if the distance is correct
x_offset: -4 # negative = left of the nozzle
y_offset: 21 # negative = in front of of the nozzle
speed: 5.0
lift_speed: 15.0
sample_retract_dist: 1
samples: 2
samples_tolerance_retries: 6

##[(7x7)-1] / 2 = 24
##[(5x5)-1] / 2 = 12
[bed_mesh]
speed: 300
horizontal_move_z: 4 #2
mesh_min: 20,20
mesh_max: 140,140
probe_count: 5,5
# relative_reference_index: 12 # Deprecated
algorithm: bicubic
fade_start: 1
fade_end: 10
#fade_target:
#   The z position in which fade should converge. When this value is set
#   to a non-zero value it must be within the range of z-values in the mesh.
#   Users that wish to converge to the z homing position should set this to 0.
#   Default is the average z value of the mesh.
split_delta_z: 0.015
#   The amount of Z difference (in mm) along a move that will
#   trigger a split. Default is .025.
move_check_distance: 3
#   The distance (in mm) along a move to check for split_delta_z.
#   This is also the minimum length that a move can be split. Default
#   is 5.0.
mesh_pps: 4,4
#   A comma separated pair of integers (X,Y) defining the number of
#   points per segment to interpolate in the mesh along each axis. A
#   "segment" can be defined as the space between each probed
#   point. The user may enter a single value which will be applied
#   to both axes.  Default is 2,2.
#bicubic_tension: .2
#   When using the bicubic algorithm the tension parameter above
#   may be applied to change the amount of slope interpolated.
#   Larger numbers will increase the amount of slope, which
#   results in more curvature in the mesh. Default is .2.

[delayed_gcode bed_mesh_init]
initial_duration: .01
gcode:
  BED_MESH_PROFILE LOAD=default

[homing_override]
set_position_z:0 # Make printer think Z axis is at zero, so we can force a move upwards away from build plate
gcode:
  G90
  G1 Z10 F3000 ; move up to prevent accidentally scratching build plate
  G28 X
  G28 Y
  PROBE_OUT
  G1 X80 Y60 F6000
  G28 Z
  PROBE_IN

##For Dual Z setups only!! (with independent motors, no Y splitters or dual Z port on board!)##
#[z_tilt]
#z_positions:
#  25,117
#  210,117
#   A list of X, Y coordinates (one per line; subsequent lines
#   indented) describing the location of each bed "pivot point". The
#   "pivot point" is the point where the bed attaches to the given Z
#   stepper. It is described using nozzle coordinates (the X, Y position
#   of the nozzle if it could move directly above the point). The
#   first entry corresponds to stepper_z, the second to stepper_z1,
#   the third to stepper_z2, etc. This parameter must be provided.
#points:
#  4,96.5
#  219,96.5
#   A list of X, Y coordinates (one per line; subsequent lines
#   indented) that should be probed during a Z_TILT_ADJUST command.
#   Specify coordinates of the nozzle and be sure the probe is above
#   the bed at the given nozzle coordinates. This parameter must be
#   provided.
#speed: 100
#   The speed (in mm/s) of non-probing moves during the calibration.
#   The default is 50.
#horizontal_move_z: 15
#   The height (in mm) that the head should be commanded to move to
#   just prior to starting a probe operation. The default is 5.
#retries: 10
#   Number of times to retry if the probed points aren't within
#   tolerance.
#retry_tolerance: 0.01
#   If retries are enabled then retry if largest and smallest probed
#   points differ more than retry_tolerance. Note the smallest unit of
#   change here would be a single step. However if you are probing
#   more points than steppers then you will likely have a fixed
#   minimum value for the range of probed points which you can learn
#   by observing command output.

#####################################################################
# KlackEnder- Macros
#####################################################################

[gcode_macro PROBE_OUT]
gcode:
  G90
  G1 X-15 F4000
  G4 P300
  G1 Z15
  G1 X20 #X0

[gcode_macro PROBE_IN]
gcode:
  G90
  G1 Z20
  G1 X-15 F20000
  G1 Y-5
  G1 Z0
  G4 P300
  G1 X100 F6000 # 35 130 Move right to catch the probe
  G1 Z10
  G1 X20 #X0 #X20

[gcode_macro AUTO_BED_MESH]
gcode:
  PROBE_OUT
  G1 Z15
  G1 X25
  BED_MESH_CALIBRATE
  #G1 Y0 F20000
  PROBE_IN

[gcode_macro G29]
gcode:
  PROBE_OUT
  G1 Z15
  G1 X25
  BED_MESH_CALIBRATE
  #G1 Y0 F20000
  PROBE_IN

[gcode_macro Accuracy_Test]
gcode:
  PROBE_OUT
  G1 Z15
  G1 X25
  G90
  G1 Y80 X80 F20000
  PROBE_ACCURACY
  PROBE_IN

[gcode_macro AUTO_Z_TILT_ADJUST]
gcode:
  PROBE_OUT
  Z_TILT_ADJUST
  PROBE_IN


#####################################################################
# KlackEnder- Menu - Only if you have a display installed!
#####################################################################

[menu __main]
type: list
name: Main

[menu __main __KlackEnder]
type: list
enable: True
name: KlackEnder

[menu __main __KlackEnder __ProbeOut]
type: command
name: Probe Out
gcode:
  PROBE_OUT

[menu __main __KlackEnder __ProbeIn]
type: command
name: Probe In
gcode:
  PROBE_IN

[menu __main __KlackEnder __AutoBedMesh]
type: command
name: Auto Bed Mesh
gcode:
  G28
  AUTO_BED_MESH
```


## Useful Links:

https://www.thingiverse.com/kevinakasam/designs
https://www.thingiverse.com/thing:5778174
https://kevinakasam.com/klackender/ (You can click on the images for further instructions)
https://github.com/kevinakasam/KlackEnder-Probe
https://github.com/TRex22/3d-models/tree/main/klipper_printer_configs
Kevinakasam's Discord Server: https://discord.gg/xqpKrxt9FC
