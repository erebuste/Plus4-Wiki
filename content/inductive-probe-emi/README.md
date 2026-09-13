# Inductive probe EMI

The inductive sensor sometimes triggers almost "randomly." The bed map is significantly distorted, and it can be measured at the cost of very large point repetitions.

I've noticed that the situation changes significantly during head maintenance, and that the problem depends on the location of the sensor cable.

The original cable is unshielded and without twisted pair. However, the extruder heater is always running, holding the PWM at 140 degrees (to stabilize for piezoelectric sensor measurements).

Disabling the heater during meshing stopped the random sensor activations and reduced its reading repeatability to 0.05 mm. Apparently, the inductive sensor cable is picking up interference from the heater's PWM controller.

This problem was on the original unit and on the Phaetus Conch.

## WARNING

Stock inductive probe will fail when heated to ~65c. In this case, the table will ram your head on probe.
Apparently, cooling the thermal barrier also cools the sensor.

## Possible solutions

1. Route the sensor cable away from the heater cables. It should also be secured to prevent vibration during printing.
2. Use a different type of sensor.
3. Turn off the extruder heater while the sensor is operating. But this has some problems described in "warning".

### "Disable" extruder heater while probing

`smart_effector` can run custom gcode before and after probe.

We cannot turn off extruder - need keep `hotend_fan` enable for cooling stock sensor what controlled by `heater_fan` and i don`t know how enable it manual. So heater will set to 50c - in most cases heater on 50c will turned on with low frequency - table will keep it heated.

50c is default `heater_temp` in `[heater_fan hotend_fan]`.

For changes has 2 options to write changes:
1. To end of `printer.cfg`
2. To separate `.cfg` file in printer config folder, and include in end of printer.cfg:
```
[include my_custom_config.cfg]
```

Custom gcode:
```
#-------- probe heaters activate/deactivate ---------
[gcode_macro _PROBE_HEATERS_ACTIVATE]
description: Induction sensor/probe activate heaters
variable_extruder_temp: 0
gcode:
    {% if extruder_temp > 0 %}
        M104 S{extruder_temp} # extruder
    {% endif %}

    
[gcode_macro _PROBE_HEATERS_DEACTIVATE]
description: Induction sensor/probe deactivate heaters for EMI
gcode:
    SET_GCODE_VARIABLE MACRO=_PROBE_HEATERS_ACTIVATE VARIABLE=extruder_temp VALUE={printer["extruder"].target}
    M104 S50 # extruder

[smart_effector]
deactivate_on_each_sample: True
activate_gcode:
    _PROBE_HEATERS_ACTIVATE
deactivate_gcode:
    _PROBE_HEATERS_DEACTIVATE
```

### Optional additional config

After changes my printers have good repeatability of measurements. I use bigger mesh and lower probe tolerance like on [Better Bed Meshing](../more-accurate-bed-meshing/README.md), and tested what:
1. No motors tweaks need anymore, but i still use "interpolate: False".
2. Probe z-speeds can bee increased back to default (5), lift-speed - more then default.
3. In my case 5 samples per point has "99%" difference in values >0.08 and near measurement points while meshing has adequate difference. We cannot disable extruder, so `samples` keep to stock value "2".

```
# Bigger mesh
[bed_mesh]
horizontal_move_z:10
probe_count:11,11
bicubic_tension:0.3

[smart_effector]
speed:5
lift_speed: 10
samples: 2 # default - 2
sample_retract_dist: 10
samples_tolerance: 0.013
```