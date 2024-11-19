# Stealthchanger-Addons
Configuration changes for my setup v2.4 350
FILAMENT SETTINGS
On Filament TAB, Enter desired Idle Temperature
![image](https://github.com/user-attachments/assets/b5611076-c7e6-4584-8a43-210c9998d1d5)

Filament end G-code
T0_TEMP={idle_temperature[0]} ; filament end gcode Add this to the Filament end G-code for each tool modifying fo the proper Tool/Extruder number.
![image](https://github.com/user-attachments/assets/9a2b3d35-9403-4902-973b-2f5188b560b6)

Machine Start G-code
PRINT_START {if is_extruder_used[0]}T0_TEMP={idle_temperature[0]}{endif} {if is_extruder_used[1]}T1_TEMP={idle_temperature[1]}{endif} TOOL_TEMP={first_layer_temperature[initial_tool]} TOOL=[initial_tool] BED_TEMP=[first_layer_bed_temperature]
![image](https://github.com/user-attachments/assets/91860d24-2eb0-4584-bf62-5e1ca655a351)
