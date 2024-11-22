# Stealthchanger-Addons
Automatic Temperature control Configuration for Stealthchanger using Orca Slicer 2.2.0

This is my current configuration to prevent heating all tools to first layer temperature when a print is starting, which prevents heating tools unnecessarily.

Printer: Voron 2.4 350 configured with two tools.

In this example I am using ASA with a first layer temperature of 255C, Normal printing Temp 250C and idle temp 190C.
Probing/Meshing Nozzle temp is set to 150C

Nozzle temp will be set to 150C throughout the homing and meshing sequence then tools will be set to idle temperature if not the initial tool used. Initial tool is set to first layer temperature.

On tool change the next tool is heated to temp and the current tool is set to idle temperature and the Tool is changed.

Under Process - Global on Multimaterial tab Enable Ooze prevention and set Temperature Variation and Preheat Time.

In this example I used -65 at 10s which is just enough time for the Rapido hotend I am using to heat from 190C to 250C. (this will vary based on hotend/filaments used) 

<i/>I do not believe the Ooze prevention Temperature variation is used since 'idle_temperature' is called directly. </i>

![image](https://github.com/user-attachments/assets/ade8d327-338a-4497-8a19-8557186d81ee)

FILAMENT SETTINGS.

On Filament TAB, Enter desired Idle Temperature (Must be done for all Filament types used)

In this example I am using Polymaker ASA and have set the idle temperature to 190C.

![image](https://github.com/user-attachments/assets/b5611076-c7e6-4584-8a43-210c9998d1d5)

Filament end G-code
```
T0_TEMP={idle_temperature[0]} ; filament end gcode 
```
Add this to the Filament end G-code for each tool modifying for the proper Tool/Extruder number.

![image](https://github.com/user-attachments/assets/9a2b3d35-9403-4902-973b-2f5188b560b6)

Machine Start G-code
```
PRINT_START {if is_extruder_used[0]}T0_TEMP={idle_temperature[0]}{endif} {if is_extruder_used[1]}T1_TEMP={idle_temperature[1]}{endif} TOOL_TEMP={first_layer_temperature[initial_tool]} TOOL=[initial_tool] BED_TEMP=[first_layer_bed_temperature]
```
![image](https://github.com/user-attachments/assets/91860d24-2eb0-4584-bf62-5e1ca655a351)

This is my 'print_start' macro

```
[gcode_macro PRINT_START]
gcode:
  # This part fetches data from your slicer. Such as bed temp, extruder temp, chamber temp and size of your printer.
  {% set target_bed = params.BED_TEMP|int %}
  {% set TOOL = params.TOOL|int %}
  #{% set target_extruder = params.TOOL_TEMP|int %} Moved to later in start procedure.
  #{% set target_chamber = params.CHAMBER|default("40")|int %}
  {% set x_wait = printer.toolhead.axis_maximum.x|float / 2 %}
  {% set y_wait = printer.toolhead.axis_maximum.y|float / 2 %}
  {% set initial_tool = params.TOOL|int %}
  {% set initial_temperature = 150 %}                          #Nozzle temp variable for homing and probing

### HEAT FOR HOMING AND PROBING ###
  SET_DISPLAY_TEXT MSG="Bed: {target_bed}c"                    # Displays info
  STATUS_HEATING                                               # Sets Tn-leds to heating-mode
  M106 S255                                                    # Turns on the PT-fan
  # Preheat all the hotends in use
  
  M190 S{target_bed}                                           # Sets the target temp for the bed  

  INITIALIZE_TOOLCHANGER

  SET_DISPLAY_TEXT MSG="Hotend: 150c"                          # Displays info
  STATUS_HEATING                                               # LED status

 # Heats the tools to 150c - set by initial_temperature parameter above
  {% for tool_nr in printer.toolchanger.tool_numbers %}       
    {% if tool_nr == 0 or tool_nr == 1 tool_nr == 2 or tool_nr == 3 tool_nr == 4 or tool_nr == 5 %}              
        # Set initial temperature for Tools
        M104 T{tool_nr} S{initial_temperature}
    {% endif %}
  {% endfor %}                                       
  
  _CQGL                                                        # conditional homing and QGL

  {% if initial_tool == 0 %}                                   # Conditional check to call T0 or T1 based on the value of initial_tool 
        T0
    {% elif initial_tool == 1 %}                               
        T1
    {% elif initial_tool == 2 %}                               
        T2
    {% elif initial_tool == 3 %}                               
        T3
    {% elif initial_tool == 4 %}                               
        T4
    {% elif initial_tool == 5 %}                               
        T5
    {% else %}
        M117 Error: Unsupported tool selection
    {% endif %}
   
  G90                                                         # Absolute position
  BED_MESH_CLEAR                                              # Clears old saved bed mesh (if any)
###  BED MESH ###
  SET_DISPLAY_TEXT MSG="Bed mesh"                             # Displays info
  STATUS_MESHING                                              # Sets SB-leds to bed mesh-mode
  BED_MESH_CALIBRATE                                          # Starts bed mesh
  {% for tool_nr in printer.toolchanger.tool_numbers %}
    {% set tooltemp_param = 'T' ~ tool_nr|string ~ '_TEMP' %}
    {% if tooltemp_param in params %}
      M104 T{tool_nr} S{params[tooltemp_param]}
    {% endif %}
  {% endfor %}
  Smart_Park                                                  # Parks nozzle closer to print area

### HEAT NOZZLE FOR PRINTING ###  
  {% set target_extruder = params.TOOL_TEMP|int %}            # Heats up the nozzle up to target via data from slicer
  SET_DISPLAY_TEXT MSG="Hotend: {target_extruder}c"           # Displays info
  STATUS_HEATING                                              # Sets SB-leds to heating-mode
  #G1 X{x_wait} Y{y_wait} Z15 F9000                           # Goes to center of the bed  
  M109 S{ params.TOOL_TEMP }                                  # Heats the nozzle to printing temp
  
### CLEAN NOZZLE AND PURGE LINE THEN BEGIN PRINT ###
  SET_DISPLAY_TEXT MSG="Printing"                             # Displays info
  STATUS_PRINTING                                             # Sets SB-leds to printing-mode
  #CLEAN_NOZZLE
  
  #G1 X{x_wait - 50} Y10 F10000                               # Moves to starting point 
  #G1 Z0.6                                                    # Raises Z to 0.6
  G91                                                         # Incremental positioning for Satndard purge line
  #G1 X100 E20 F1500                                          # Standard Purge line
  #VORON_PURGE                                                # KAMP Macro
  Line_Purge                                                  # KAMP Macro
  M107                                                        # Turns off partcooling fan
  G90                                                         # Absolute position
```
