;===== Latest change filament code (no AMS for A1 Mini) can be found at:
;===== https://github.com/Dennis-Q/bambu
;=====
;===== Based on work of eukatree and some contributors (Hillbilly-Phil and pakonambawan)
;===== https://github.com/eukatree/Bambu_CustomGCode/
;=====
;===== Updated: 20250127
;================================================
;
;===== Install instructions (Bambu Studio / Orca Slicer):
;===== Copy this complete file (including all comments)
;===== to the 'Change filament G-code'-section which can be 
;===== found in 'Printer settings' - 'Machine code' menu.
;===== Then, save the printer and use it with multi-color prints.
;
;===== Instructions (by Hillbilly-Phil): =====
; when print has paused, go into control
; set the nozzle temp to the temp you were printing with
; unload the filament by pushing on the upper extruder-button 
; load the new filament by pushing on the lower extruder-button 
; resume the print (flushing will happen next,
; flushing volumes can be set in Bambu Studio as if using an AMS)
;
;===== machine: A1 mini =========================
;===== date: 20240830  =======================

G392 S0 ; Desactiva la función de detección de filamento (no AMS).
M1007 S0 ; Desactiva la función de recuperación de filamento (no AMS).

M204 S9000 ; Establece la aceleración máxima durante el cambio de filamento.

{if toolchange_count > 1} ; Si es un cambio de filamento posterior al primero:
G17 ; Selecciona el plano XY para movimientos circulares.
G2 Z{max_layer_z + 0.4} I0.86 J0.86 P1 F10000 ; Eleva el cabezal en espiral para evitar colisiones.
{endif}

G1 Z{max_layer_z + 3.0} F1200 ; Eleva el cabezal 3 mm por encima de la capa actual.

M400 ; Espera a que todos los comandos anteriores se completen.
M106 P1 S0 ; Apaga el ventilador de la capa.
M106 P2 S0 ; Apaga el ventilador auxiliar.

{if old_filament_temp > 142 && next_extruder < 255} ; Si la temperatura del filamento anterior es mayor a 142°C:
M104 S[old_filament_temp] ; Mantiene la temperatura del filamento anterior.
{endif}

G1 X185 F18000 ; Mueve el cabezal a la posición de corte del filamento.

M17 S ; Guarda los valores actuales de corriente del motor.
M400 ; Espera a que los comandos se completen.
M17 X1 ; Aumenta la corriente del motor X para el corte.
G1 X197 F400 ; Corta el filamento lentamente.
G1 X185 F500 ; Retorna a la posición anterior al corte.
M400 ; Espera a que los comandos se completen.
M17 R ; Restaura los valores de corriente del motor.

M620.1 E F[old_filament_e_feedrate] T{nozzle_temperature_range_high[previous_extruder]} ; Configura la extrusión para el filamento anterior.
M620.10 A0 F[old_filament_e_feedrate] ; Desactiva la purga automática (no AMS).
T[next_extruder] ; Cambia al siguiente extrusor.
M620.1 E F[new_filament_e_feedrate] T{nozzle_temperature_range_high[next_extruder]} ; Configura la extrusión para el nuevo filamento.
M620.10 A1 F[new_filament_e_feedrate] L[flush_length] H[nozzle_diameter] T[nozzle_temperature_range_high] ; Configura la purga manual.

; -- BEGIN ADDED LINES --
G1 X0 Y90 F18000 ; Mueve el cabezal a una posición accesible para el usuario.
G1 X-13.5 F9000 ; Mueve el cabezal a la posición de purga.
G1 E-13.5 F900 ; Retrae el filamento para facilitar el cambio manual.
; pause for user to load and press resume
M400 U1 ; Pausa para que el usuario retire el filamento antiguo y cargue el nuevo.
; -- END ADDED LINES --

{if next_extruder < 255} ; Si hay un siguiente extrusor:
M400 ; Espera a que los comandos se completen.
G92 E0 ; Reinicia el contador de extrusión.

{if flush_length_1 > 1} ; Si se requiere purga:
; FLUSH_START
M400 ; Espera a que los comandos se completen.
M1002 set_filament_type:UNKNOWN ; Configura el tipo de filamento como desconocido.
M109 S[nozzle_temperature_range_high] ; Calienta la boquilla a la temperatura máxima.
M106 P1 S60 ; Enciende el ventilador de la capa al 60%.

{if flush_length_1 > 23.7} ; Si la longitud de purga es mayor a 23.7 mm:
G1 E23.7 F{old_filament_e_feedrate} ; Purga inicial sin pulsaciones.
G1 E{(flush_length_1 - 23.7) * 0.02} F50 ; Purga con pulsaciones lentas.
G1 E{(flush_length_1 - 23.7) * 0.23} F{old_filament_e_feedrate} ; Purga con pulsaciones rápidas.
{else} ; Si la longitud de purga es menor o igual a 23.7 mm:
G1 E{flush_length_1} F{old_filament_e_feedrate} ; Purga simple.
{endif}
; FLUSH_END
G1 E-[old_retract_length_toolchange] F1800 ; Retrae el filamento.
G1 E[old_retract_length_toolchange] F300 ; Extruye el filamento.
M400 ; Espera a que los comandos se completen.
M1002 set_filament_type:{filament_type[next_extruder]} ; Configura el tipo de filamento nuevo.
{endif}

{if flush_length_1 > 45 && flush_length_2 > 1} ; Si se requiere purga adicional:
; WIPE
M400 ; Espera a que los comandos se completen.
M106 P1 S178 ; Enciende el ventilador de la capa al 70%.
M400 S3 ; Espera 3 segundos.
G1 X-3.5 F18000 ; Mueve el cabezal para limpiar la boquilla.
G1 X-13.5 F3000 ; Retorna a la posición de purga.
M400 ; Espera a que los comandos se completen.
M106 P1 S0 ; Apaga el ventilador de la capa.
{endif}

M629 ; Finaliza la purga.

M400 ; Espera a que los comandos se completen.
M106 P1 S60 ; Enciende el ventilador de la capa al 60%.
M109 S[new_filament_temp] ; Calienta la boquilla a la temperatura del nuevo filamento.
G1 E5 F{new_filament_e_feedrate} ; Compensa el derrame de filamento durante el calentamiento.
M400 ; Espera a que los comandos se completen.
G92 E0 ; Reinicia el contador de extrusión.
G1 E-[new_retract_length_toolchange] F1800 ; Retrae el filamento.
M400 ; Espera a que los comandos se completen.
M106 P1 S178 ; Enciende el ventilador de la capa al 70%.
M400 S3 ; Espera 3 segundos.
G1 X-3.5 F18000 ; Mueve el cabezal para limpiar la boquilla.
G1 X-13.5 F3000 ; Retorna a la posición de purga.
M400 ; Espera a que los comandos se completen.
M106 P1 S0 ; Apaga el ventilador de la capa.

G1 Z{max_layer_z + 3.0} F3000 ; Eleva el cabezal 3 mm por encima de la capa actual.
{if layer_z <= (initial_layer_print_height + 0.001)} ; Si es la primera capa:
M204 S[initial_layer_acceleration] ; Ajusta la aceleración para la primera capa.
{else} ; Si no es la primera capa:
M204 S[default_acceleration] ; Usa la aceleración por defecto.
{endif}

M620 S[next_extruder]A ; Habilita el siguiente extrusor.
T[next_extruder] ; Cambia al siguiente extrusor.
M621 S[next_extruder]A ; Finaliza el cambio de extrusor.

M622.1 S0 ; Desactiva la compensación dinámica de extrusión.
M9833 F{outer_wall_volumetric_speed/2.4} A0.3 ; Ajusta la compensación dinámica de extrusión.
M1002 judge_flag filament_need_cali_flag ; Verifica si se necesita calibración de filamento.
M622 J1 ; Habilita la compensación dinámica de extrusión.
G92 E0 ; Reinicia el contador de extrusión.
G1 E-[new_retract_length_toolchange] F1800 ; Retrae el filamento.
M400 ; Espera a que los comandos se completen.
M106 P1 S178 ; Enciende el ventilador de la capa al 70%.
M400 S7 ; Espera 7 segundos.
G1 X0 F18000 ; Mueve el cabezal para limpiar la boquilla.
G1 X-13.5 F3000 ; Retorna a la posición de purga.
M400 ; Espera a que los comandos se completen.
M106 P1 S0 ; Apaga el ventilador de la capa.
M623 ; Finaliza la compensación dinámica de extrusión.

G392 S0 ; Desactiva la detección de filamento (no AMS).
M1007 S1 ; Reactiva la función de recuperación de filamento (no AMS).
