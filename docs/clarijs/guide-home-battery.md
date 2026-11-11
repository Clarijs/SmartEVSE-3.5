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
2. Send the **measured voltage** with the mains meter (`L1:L2:L3:W:Wh:V`).
3. Set the battery's peak shaving a little **closer** to the peak than the SmartEVSE ceiling.

Measured (margin 0, battery threshold peak − 10 W, 6.7 kWh night charge): highest 15-minute average
2706 W at a peak of 2755 W; the battery delivered 7 Wh in total.

## 2. Send the battery power

```
<prefix>/Set/HomeBatteryPower    1250          W, + charging / − discharging
<prefix>/Set/HomeBatteryPower    1250:48       … with the state of charge in % (needed for the peak-shaving SoC)
```

Send it every second or every few seconds (after 60 s without an update the battery counts as 0).
The SmartEVSE does the rest itself:

| Setting | Meaning | Example |
|---|---|---|
| `Set/HomeBatteryOffset` | W subtracted from the power (sensor offset, own consumption) | 25 |
| `Set/HomeBatteryPeakShavingSoC` | below this SoC the battery only does peak shaving; see section 3 | 15 |

Both are also in the Home battery card of the web interface (All settings) and Home Assistant numbers.
The old `Set/HomeBatteryCurrent` (in 0.1 A) still works. As in the original firmware the battery is used
in **Solar** mode; in **Smart** mode only below the Peak Shaving SoC.

## 3. Keep the peak-shaving part of the battery out of the car (Smart mode)

Batteries that do peak shaving keep the lowest part (e.g. below 15 %) for peak shaving only. When the
battery is still discharging below that level (e.g. after a calibration, or in self-consumption
mode), the charger only sees a grid under the peak and gives the car more: the peak-shaving part ends
up in the car. Set **Peak Shaving SoC** to the SoC from which your battery only does peak shaving:

| Setting | Effect |
|---|---|
| `Set/HomeBatteryPeakShavingSoC` (e.g. 15) | below this SoC a discharging battery counts as grid import in Smart mode; 0 = off |

It needs the SoC in `Set/HomeBatteryPower` (`W:SoC`). Above it nothing changes. Also on All
settings (Home battery → *Peak shaving below*) and as Home Assistant number *Home Battery Peak Shaving SoC*.

## 3b. Block the battery completely (optional)

The charger does not control the battery. To be sure it never feeds the car, do it in Home
Assistant: set the battery's maximum discharge power to 0 W while the SmartEVSE state is *Charging*,
and back afterwards. (An earlier version of this fork had *Battery Guard* for this; it is removed.)

## 4. Result (measured)

| | Before | After |
|---|---|---|
| 15-min average while charging | 2300–2500 W | 2617–2663 W |
| Margin to the peak | 255–455 W | ~110 W |
| Home battery while charging | ~0.7 kWh into the car, peaks up to 4.7 kW | practically idle |

## 5. Good to know

- **Battery calibration:** some batteries (e.g. Huawei) run an SOH calibration, pack by pack: they
  discharge to ~0 % ignoring the reserve and then charge from the grid, sometimes up to well above the
  reserve. Afterwards, in self-consumption mode, the battery may feed the car again until it is back at
  its peak-shaving level; **Peak Shaving SoC** stops it there. See [troubleshooting](troubleshooting.md).
- **Grid charging to the reserve:** a battery below its peak-shaving reserve may charge from the grid.
  That is normal load for the charger: the car gets less until the battery is back at its reserve.

Next: [integration with Home Assistant / MQTT / REST](guide-integration.md)
