# Guide: Home Assistant, MQTT and REST

[← Back to the overview](../../README.md)

Everything this firmware adds can be set over MQTT, REST and (via MQTT discovery) Home Assistant.
The settings are stored on the charger, so automations only have to send **raw values**; all tuning
is done on the SmartEVSE.

## Home Assistant

With MQTT configured on the SmartEVSE, these entities appear automatically next to the standard ones:

| Entity | Type | Purpose |
|---|---|---|
| Capacity Peak | sensor (W) | monthly peak used by Flanders / Peak |
| Home Battery Offset | number (W) | offset on `HomeBatteryPower` |
| Home Battery Full SoC | number (%) | battery counts as 0 at or above |
| Battery Guard | switch | keep the battery out of the car (Smart) |
| Battery Guard SoC High / Low | number (%) | SoC window |
| Battery Guard Active | binary sensor | current state |
| Mains Meter Timeout | number (s) | timeout of the API mains meter |

Typical automations:

- **Battery power** every second: `mqtt.publish` to `<prefix>/Set/HomeBatteryPower` with
  `{{ battery_power }}:{{ battery_soc }}`. No calculation needed.
- **Monthly peak** if your P1 reader cannot send it: `mqtt.publish` of the meter's
  "maximum demand running month" (W) to `<prefix>/Set/CapacityPeak`.
- **Battery peak shaving** (outside the charger): keep the battery threshold at monthly peak − 30 W.

## MQTT

Set topics (`<prefix>/Set/…`):

| Topic | Payload |
|---|---|
| `MainsMeter` | `L1:L2:L3`, `L1:L2:L3:W:Wh` or `L1:L2:L3:W:Wh:V` (V in 0.1 V) |
| `CapacityPeak` | monthly peak in W (only raises, ≥ 2500) |
| `CurrentMaxSumMains` | A, one decimal allowed (`10.4`) |
| `CapacityMargin` | W (0–1000) |
| `HomeBatteryPower` | W, optional `:SoC` |
| `HomeBatteryOffset` / `HomeBatteryFullSoC` | W / % |
| `BatteryGuard` | `On` / `Off` |
| `BatteryGuardSoCHigh` / `BatteryGuardSoCLow` | % |
| `BatteryGuardStop` | min (0 = off, max 60) |
| `MainsMeterTimeout` | s (11–120) |

Published: `CapacityPeak`, `MaxSumMains` (one decimal), `CapacityMargin`, `HomeBatteryPower`,
`HomeBatteryOffset`, `HomeBatteryFullSoC`, `BatteryGuard`, `BatteryGuardSoCHigh`, `BatteryGuardSoCLow`,
`BatteryGuardActive`, `MainsMeterTimeout`.

## REST

`POST /settings?…`: `capacity_peak`, `current_max_sum_mains` (decimal), `capacity_margin`,
`home_battery_offset`, `home_battery_full_soc`, `battery_guard`, `battery_guard_soc_high`,
`battery_guard_soc_low`, `mains_meter_timeout`.

`POST /currents?battery_power=1250&battery_soc=87.5` sends the battery power.

`GET /settings` shows among others `capacity_peak_month`, `mains_voltage`, `capacity_voltage`,
`capacity_margin`, `mains_meter_timeout` and the `home_battery` object (`power`, `soc`, `offset`,
`full_soc`, `guard`, `guard_soc_high`, `guard_soc_low`, `guard_active`).

Full reference: [CLARIJS-FIRMWARE.md → MQTT and REST reference](../../CLARIJS-FIRMWARE.md#mqtt-and-rest-reference)
