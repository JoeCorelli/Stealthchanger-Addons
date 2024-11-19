# Stealthchanger-Addons
Automatic Temperature control Configuration for Stealthchanger

This is my current configuration to prevent heating all tools to first layer temperature when a print is starting, which prevents heating tools unnecessarily.

In this example I am using ASA with a first layer temperature of 255C, Normal printing Temp 250C and idle temp 190C.
Probing/Meshing Nozzle temp is set to 150C

Nozzle temp will be set to 150C throughout the homing and meshing sequence then tools will be set to idle temperature if not the initial tool used. Initial tool is set to first layer temperature.

On tool change the next tool is heated to temp and the current tool is set to idle temperature and the Tool is changed.

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
