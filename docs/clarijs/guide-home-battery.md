# Guide: SmartEVSE with a home battery

[← Back to the overview](../../README.md)

Home batteries (Huawei LUNA2000, and others) often do **peak shaving on the instantaneous grid
power**: above a threshold they discharge. Without care the charger and the battery work against
each other, and the energy you stored for the expensive hours ends up in the car. This guide makes
them cooperate.

## 1. The principle

```
monthly peak (billed)                2755 W
battery peak shaving (peak − 30 W)   2725 W   ← battery only discharges above this
SmartEVSE ceiling (peak − 55 W)      2700 W   ← mode Peak, measured voltage
```

The charger always stays **below** the battery's threshold. The battery then only reacts to real
peaks in the house (oven, kettle), never to the car.

1. Use **mode Peak** (see the [capacity tariff guide](guide-capacity-tariff.md)), not Flanders.
2. Set the battery's peak shaving a little **closer** to the peak than the SmartEVSE margin.
3. Send the battery power to the SmartEVSE and turn on **Battery Guard**.

## 2. Send the battery power

```
<prefix>/Set/HomeBatteryPower    1250          W, + charging / − discharging
<prefix>/Set/HomeBatteryPower    1250:87.5     … with the state of charge in %
```

Send it every second or every few seconds (after 60 s without an update the battery counts as 0).
The SmartEVSE does the rest itself:

| Setting | Meaning | Example |
|---|---|---|
| `Set/HomeBatteryOffset` | W subtracted from the power (sensor offset, own consumption) | 25 |
| `Set/HomeBatteryFullSoC` | at or above this SoC the battery counts as 0 | 99.7 |

Both are also in the Home battery card of the web interface and as Home Assistant numbers. The old
`Set/HomeBatteryCurrent` (in 0.1 A) still works.

## 3. Battery Guard

**Battery Guard** (`Set/BatteryGuard On`) keeps the battery out of the car in **Smart** mode:

- **Peak shaving:** a discharging battery counts as grid import, so the charger lowers the car as if
  the battery were idle.
- **Self-consumption in daytime** (battery covers the house, grid at 0 W): the charger works like
  Solar. It lowers the car by what the battery discharges and raises it only slowly afterwards, so
  the car runs on solar + grid, not on the battery. Below 6 A the car keeps 6 A.
- At night, when the battery is below its reserve and idle, nothing changes.

**SoC window (optional):** let a well-charged battery feed the car after all.

| Setting | Effect |
|---|---|
| `Set/BatteryGuardSoCHigh` (e.g. 80) | at or above: Battery Guard paused, battery may feed the car |
| `Set/BatteryGuardSoCLow` (e.g. 50) | at or below: Battery Guard active again |

Between both values the last state is kept. `/BatteryGuardActive` shows the current state.

## 4. Result (measured)

| | Before | After |
|---|---|---|
| 15-min average while charging | 2300–2500 W | 2617–2663 W |
| Margin to the peak | 255–455 W | ~110 W |
| Home battery while charging | ~0.7 kWh into the car, peaks up to 4.7 kW | practically idle |

## 5. Good to know

- **Big loads in the house:** when the house alone goes above the ceiling, Battery Guard lowers the
  car to 6 A; after `MaxSumMainsTime` charging stops until there is room again.
- **Battery calibration:** some batteries (e.g. Huawei, about once a year) run an SOH calibration:
  they discharge to ~0 % ignoring the reserve and then charge from the grid. During that night the
  battery may cover the car; nothing is wrong with the charger. See [troubleshooting](troubleshooting.md).
- **Grid charging to the reserve:** a battery below its peak-shaving reserve may charge from the grid.
  That is normal load for the charger: the car gets less until the battery is back at its reserve.

Next: [integration with Home Assistant / MQTT / REST](guide-integration.md)
