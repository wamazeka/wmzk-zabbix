# Klipper by HTTP

## Overview

This template monitors a **Klipper 3D printer** (e.g. Voron, single extruder) through the **Moonraker REST API**, using only native Zabbix mechanisms: HTTP agent master items (storage period 0) and dependent items with JSONPath preprocessing. **No external scripts, no agents, no SNMP.**

The template collects: print state/progress/layers/filament/durations, toolhead coordinates and motion limits, g-code speed and M220 factor, nozzle/bed temperatures with targets and heater powers, part-cooling fan (speed + tachometer), host SoC temperature and CPU usage, Klipper and Moonraker versions.

Feature parity target: the initMAX "BambuLab by Zabbix agent2 (MQTT)" trigger set, adapted to Moonraker objects (error/shutdown/failed/paused/nodata alerts plus heating-stall checks on both heaters).

## Requirements

Zabbix version: 7.0 and higher.

## Tested versions

This template has been validated against a live Moonraker response (`/printer/objects/query`) from a single-extruder Voron-class printer (Mainsail/Fluidd stack, Klipper firmware).

## Configuration

> Zabbix should be configured according to the instructions in the [Templates out of the box](https://www.zabbix.com/documentation/7.0/manual/config/templates_out_of_the_box) section.

## Setup

1. Create a host for the printer and add an **agent interface with the printer IP address** (the interface itself is not used for polling, but `{HOST.IP}` resolves from it).

2. Link the template. Done - the URLs are built as `http://{HOST.IP}:{$MOONRAKER.PORT}/...`.

3. If your Moonraker listens on a non-standard port, change `{$MOONRAKER.PORT}`. If your `printer.cfg` does not define all default objects, adjust `{$MOONRAKER.OBJECTS}` (unknown objects make Moonraker fail the whole query).

4. Optionally tune thresholds (`{$KLIPPER.*}` macros) and the polling interval.

### Macros used

|Name|Description|Default|
|-|-|-|
|{$MOONRAKER.PORT}|<p>Moonraker API port. Change only if your installation differs.</p>|`7125`|
|{$MOONRAKER.OBJECTS}|<p>Object list for the single objects/query call. Tune to your config (e.g. add heater_fan, or drop objects your Klipper does not define).</p>|`webhooks&print_stats&toolhead&extruder&heater_bed&display_status&fan&gcode_move`|
|{$MOONRAKER.INTERVAL}|<p>Objects query polling interval.</p>|`1m`|
|{$MOONRAKER.HTTP_TIMEOUT}|<p>HTTP request timeout.</p>|`15s`|
|{$MOONRAKER.NODATA}|<p>Alert when nothing arrived for this long.</p>|`5m`|
|{$MOONRAKER.PAUSE_SLOTS}|<p>Consecutive paused polls before a long-pause warning (at {$MOONRAKER.INTERVAL}).</p>|`10`|
|{$KLIPPER.NOZZLE.TARGET_MIN}|<p>Min hotend target to arm the heating-stall check (°C).</p>|`150`|
|{$KLIPPER.NOZZLE.DELTA}|<p>Allowed shortfall below target before heating stall fires (°C).</p>|`15`|
|{$KLIPPER.NOZZLE.HEAT_T}|<p>Window in which the hotend must nearly reach target.</p>|`5m`|
|{$KLIPPER.NOZZLE.MAX}|<p>Absolute hotend temp ceiling (°C).</p>|`300`|
|{$KLIPPER.BED.TARGET_MIN}|<p>Min bed target to arm the heating-stall check (°C).</p>|`40`|
|{$KLIPPER.BED.DELTA}|<p>Bed temp shortfall tolerated (°C).</p>|`10`|
|{$KLIPPER.BED.HEAT_T}|<p>Window in which the bed must nearly reach target.</p>|`10m`|
|{$KLIPPER.BED.MAX}|<p>Absolute bed temp ceiling (°C).</p>|`120`|
|{$KLIPPER.HOST_TEMP.WARN}|<p>Host CPU warning temperature (°C).</p>|`80`|
|{$KLIPPER.HOST_TEMP.CRIT}|<p>Host CPU critical temperature (°C).</p>|`90`|

### Items collected

|Name|Description|Type|Key|
|-|-|-|-|
|Get printer objects|<p>Single JSON query of printer objects ({$MOONRAKER.OBJECTS}). Dependent items extract scalars natively via JSONPath..</p>|HTTP agent|klipper.printer_objects.get|
|Get host stats|<p>Moonraker host process statistics: CPU temperature, system memory and uptime. Endpoint /machine/proc_stats (formerly /machine/system_stats in older Moonraker)..</p>|HTTP agent|klipper.host_stats.get|
|Get printer info|<p>Printer meta information (Klipper firmware version, hostname)..</p>|HTTP agent|klipper.printer_info.get|
|Service state|<p>Moonraker webhooks state: starting / ready / shutdown / error..</p>|Dependent item|klipper.webhooks.state|
|Service state message|<p>Human-readable klipper state message (e.g. "Printer is ready", MCU error details)..</p>|Dependent item|klipper.webhooks.message|
|Print state|<p>print_stats.state: standby/printing/paused/complete/cancelled/error..</p>|Dependent item|klipper.print.state|
|Print filename|<p>Name of the currently loaded g-code file..</p>|Dependent item|klipper.print.filename|
|Print job message|<p>print_stats.message (error details when the job fails)..</p>|Dependent item|klipper.print.message|
|Print active duration|<p>Time effectively spent printing the current job (print_duration)., in s.</p>|Dependent item|klipper.print.duration.active|
|Print total duration|<p>Estimated total duration of the current job (total_duration)., in s.</p>|Dependent item|klipper.print.duration.total|
|Current layer|<p>Current layer from slicer metadata (unsupported while idle)..</p>|Dependent item|klipper.print.layer.current|
|Total layers|<p>Total layer count from slicer metadata (unsupported while idle)..</p>|Dependent item|klipper.print.layer.total|
|Filament used (job)|<p>Filament extruded for the current job (mm -> m)., in m.</p>|Dependent item|klipper.filament.used|
|Toolhead X position|<p>Toolhead X coordinate (toolhead.position[0])., in mm.</p>|Dependent item|klipper.toolhead.x|
|Toolhead Y position|<p>Toolhead Y coordinate (toolhead.position[1])., in mm.</p>|Dependent item|klipper.toolhead.y|
|Toolhead Z position|<p>Toolhead Z coordinate (toolhead.position[2])., in mm.</p>|Dependent item|klipper.toolhead.z|
|Toolhead max velocity|<p>Configured max_velocity., in mm/s.</p>|Dependent item|klipper.toolhead.max_velocity|
|Toolhead max acceleration|<p>Configured max_accel., in mm/s2.</p>|Dependent item|klipper.toolhead.max_accel|
|Active g-code speed|<p>Current active g-code speed (gcode_move.speed). Unsupported if [gcode_move] is missing from the host config., in mm/s.</p>|Dependent item|klipper.gcode.speed|
|Speed factor, in %|<p>Speed multiplier set via M220 (100% = normal)., in %.</p>|Dependent item|klipper.gcode.speed_factor|
|Nozzle temperature|<p>Hotend temperature., in °C.</p>|Dependent item|klipper.extruder.temp|
|Nozzle target temperature|<p>Hotend target temperature., in °C.</p>|Dependent item|klipper.extruder.target|
|Nozzle heater power, in %|<p>PWM duty of the hotend heater (power 0-1 -> %)., in %.</p>|Dependent item|klipper.extruder.power|
|Bed temperature|<p>Heated bed temperature., in °C.</p>|Dependent item|klipper.bed.temp|
|Bed target temperature|<p>Heated bed target temperature., in °C.</p>|Dependent item|klipper.bed.target|
|Bed heater power, in %|<p>PWM duty of the bed heater (0-1 -> %)., in %.</p>|Dependent item|klipper.bed.power|
|Part cooling fan speed, in %|<p>Part cooling fan speed (0-1 -> %)., in %.</p>|Dependent item|klipper.fan.speed|
|Part cooling fan tachometer|<p>Fan tachometer RPM (unsupported if the fan has no tach pin)., in rpm.</p>|Dependent item|klipper.fan.rpm|
|Print progress, in %|<p>display_status.progress 0-1 -> %., in %.</p>|Dependent item|klipper.print.progress|
|Display message (M117)|<p>Message shown on the UI display (M117)..</p>|Dependent item|klipper.print.message_ui|
|Host CPU temperature|<p>Raspberry Pi / host CPU temperature. Unsupported on hosts without an onboard sensor., in °C.</p>|Dependent item|klipper.host.cpu_temp|
|Host uptime|<p>Host boot uptime (system_uptime, CLOCK_BOOTTIME)., in s.</p>|Dependent item|klipper.host.uptime|
|Host memory total|<p>Total host RAM., in B.</p>|Dependent item|klipper.host.mem.total|
|Host memory available|<p>Available host RAM., in B.</p>|Dependent item|klipper.host.mem.available|
|Firmware version|<p>Klipper firmware version string..</p>|Dependent item|klipper.klipper.version|
|Hostname|<p>Printer hostname reported by Moonraker..</p>|Dependent item|klipper.host.hostname|
|Estimated remaining time|<p>Rough remaining time: total job duration minus time already spent., in s.</p>|Calculated item|klipper.print.remaining|

### Triggers

|Name|Description|Expression|Severity|Dependencies and additional info|
|-|-|-|-|-|
|Klipper: Service is in error state|<p>Klipper reported the error state via Moonraker webhooks. Check klippy.log.</p>|`last(/Klipper by HTTP/klipper.webhooks.state)="error"`|High|**Manual close**: Yes|
|Klipper: Service is in shutdown state|<p>Klipper firmware is in shutdown (firmware restart required).</p>|`last(/Klipper by HTTP/klipper.webhooks.state)="shutdown"`|High|**Manual close**: Yes|
|Klipper: No data from Moonraker API|<p>Nothing arrived from the Moonraker API for {$MOONRAKER.NODATA}. Printer may be off or a network issue.</p>|`nodata(/Klipper by HTTP/klipper.webhooks.state,{$MOONRAKER.NODATA})=1`|Average|**Manual close**: Yes|
|Klipper: Print failed|<p>print_stats moved to the error state - the current job failed.</p>|`last(/Klipper by HTTP/klipper.print.state)="error"`|High|**Manual close**: Yes<br>**Operational data**: `State: {ITEM.LASTVALUE}`|
|Klipper: Print has been paused for a long time|<p>State stayed "paused" for {$MOONRAKER.PAUSE_SLOTS} consecutive polls ({$MOONRAKER.INTERVAL} each).</p>|`last(/Klipper by HTTP/klipper.print.state)="paused" and count(/Klipper by HTTP/klipper.print.state,#{$MOONRAKER.PAUSE_SLOTS},"eq","paused")={$MOONRAKER.PAUSE_SLOTS}`|Warning||
|Klipper: Print has finished|<p>Job completed successfully on the latest state change. Manual close after collecting the part.</p>|`last(/Klipper by HTTP/klipper.print.state)="complete" and last(/Klipper by HTTP/klipper.print.state,#2)<>"complete"`|Info|**Manual close**: Yes|
|Klipper: Nozzle is overheating|<p>Nozzle temperature exceeded {$KLIPPER.NOZZLE.MAX}°C - possible thermal runaway.</p>|`last(/Klipper by HTTP/klipper.extruder.temp)>{$KLIPPER.NOZZLE.MAX}`|Average||
|Klipper: Nozzle fails to reach target (heating stalled)|<p>Target >= {$KLIPPER.NOZZLE.TARGET_MIN}°C but temperature stayed below target-{$KLIPPER.NOZZLE.DELTA}°C for the whole {$KLIPPER.NOZZLE.HEAT_T} window - broken heater/thermistor or jammed fan.</p>|`last(/Klipper by HTTP/klipper.extruder.target)>={$KLIPPER.NOZZLE.TARGET_MIN} and max(/Klipper by HTTP/klipper.extruder.temp,{$KLIPPER.NOZZLE.HEAT_T})<last(/Klipper by HTTP/klipper.extruder.target)-{$KLIPPER.NOZZLE.DELTA}`|Warning|**Operational data**: `Target: {ITEM.LASTVALUE1}, actual: {ITEM.LASTVALUE2}`|
|Klipper: Bed is overheating|<p>Bed temperature exceeded {$KLIPPER.BED.MAX}°C.</p>|`last(/Klipper by HTTP/klipper.bed.temp)>{$KLIPPER.BED.MAX}`|Average||
|Klipper: Bed fails to reach target (heating stalled)|<p>Bed target >= {$KLIPPER.BED.TARGET_MIN}°C but temperature stayed below target-{$KLIPPER.BED.DELTA}°C for the whole {$KLIPPER.BED.HEAT_T} window.</p>|`last(/Klipper by HTTP/klipper.bed.target)>={$KLIPPER.BED.TARGET_MIN} and max(/Klipper by HTTP/klipper.bed.temp,{$KLIPPER.BED.HEAT_T})<last(/Klipper by HTTP/klipper.bed.target)-{$KLIPPER.BED.DELTA}`|Warning|**Operational data**: `Target: {ITEM.LASTVALUE1}, actual: {ITEM.LASTVALUE2}`|
|Klipper: Host temperature is high|<p>Moonraker host temperature above {$KLIPPER.HOST_TEMP.WARN}°C.</p>|`last(/Klipper by HTTP/klipper.host.cpu_temp)>{$KLIPPER.HOST_TEMP.WARN}`|Warning|**Depends on**: Klipper: Host temperature is critically high|
|Klipper: Host temperature is critically high|<p>Moonraker host temperature above {$KLIPPER.HOST_TEMP.CRIT}°C - throttling likely.</p>|`last(/Klipper by HTTP/klipper.host.cpu_temp)>{$KLIPPER.HOST_TEMP.CRIT}`|High||

### Items in graphs and dashboard

- Template graphs: `Klipper: Temperatures`, `Klipper: Powers and fan`, `Klipper: Host temperature and memory`
- Template dashboard: `Klipper overview` (latest-value widgets for state/progress/temps/fan, temperature and power graphs, live **Problems** widget).

## Known issues / notes

- `Klipper: Part cooling fan tachometer` stays unsupported on fans without a tach pin (expected, harmless).
- Layer items are unsupported while idle: slicer metadata exists only inside an active job.
- `Klipper: Estimated remaining time` is a rough estimate (total duration minus time spent).
- To also cover the hotend fan, add `heater_fan` to `{$MOONRAKER.OBJECTS}` and clone the fan items.

## Feedback

Please report any issues at [`wamazeka/wmzk-zabbix` issues](https://github.com/wamazeka/wmzk-zabbix/issues).
