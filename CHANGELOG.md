# Changelog — Clarijs edition

[← Back to the overview](README.md)

Changes of this fork on top of [dingo35/SmartEVSE-3.5](https://github.com/dingo35/SmartEVSE-3.5).
Every change is added to **Unreleased** first; at the next release that section gets the version number.

## Unreleased

Nothing yet.

## v3.11.3-Clarijs.5 — 11 Nov 2026

- **Home battery: peak-shaving SoC (Smart mode).** The state of charge is back in `Set/HomeBatteryPower`
  (`1250:48`, REST `battery_soc`) and there is one new setting, **Peak Shaving SoC** (`Set/HomeBatteryPeakShavingSoC`,
  REST `home_battery_peak_shaving_soc`, Home Assistant number *Home Battery Peak Shaving SoC*, field *Peak shaving below* on
  All settings; 0 = off). Below that SoC a discharging battery counts as grid import in Smart mode, so
  the EV cannot empty the part of the battery kept for peak shaving. Above it nothing changes.
  Measured why: after a Huawei calibration the battery (35 %) fed the EV at ~2.5 kW down to 12 %,
  below the 15 % it keeps for peak shaving. Named *Peak Shaving SoC* (first build: *Reserve SoC*; the stored value is kept). `/settings` shows `home_battery.soc`, `peak_shaving_soc`,
  `peak_shaving_active`; MQTT publishes `/HomeBatteryPeakShavingSoC` and `/HomeBatteryPeakShavingActive`.
- All settings: the home battery fields are aligned (label, field, unit).
- **CAP STOP in every capacity mode.** The LCD menu showed `CAP STOP` (Max Sum Mains Time) only in
  `CAP MODE = Fixed`, although the timer also applies to Interval, Flanders and Peak. It is now shown for
  every capacity mode, and there is a *CAP STOP (min)* field on the Capacity page of the web interface.
  Without a home battery set it to 0 or 1 minute (see the capacity tariff guide, section 6).

## v3.11.3-Clarijs.4 — 11 Nov 2026

- **New web interface.** The start page (`/`) is a visual dashboard: charging status, a capacity bar
  (other on the meter / car / room under the ceiling), the home battery (only when battery power is
  received), a 10-minute chart, big mode buttons with a short explanation and the charge current. The
  former main page with every setting is now *All settings* (`/advanced.html`). All pages (dashboard,
  all settings, capacity, update) share the same style: dark status banner, larger cards and buttons,
  toggles, a theme button (auto / light / dark) and a clearer dark mode. With an RFID reader the
  dashboard asks to present the card while access is off. Uses only the existing `/settings` API; no
  change to the charging logic.
- **Removed: Battery Guard.** `Set/BatteryGuard`, `BatteryGuardSoCHigh/Low` (MQTT, REST `battery_guard…`,
  Home Assistant entities *Battery Guard …*, the web fields) are gone; in Smart mode the home battery is
  ignored again, as in the original firmware. Not needed: with mode Peak, the measured voltage and the
  battery's peak-shaving threshold at peak − 10 W the battery stays out of the EV (measured night: 7 Wh
  from the battery during a 6.7 kWh charge, highest quarter 2706 W at a peak of 2755 W).
- **Home battery: power only.** The state of charge (`Set/HomeBatteryPower 1250:87.5`, REST `battery_soc`)
  and the setting *Full SoC* are removed. Send only the power in W; a payload with `:SoC` still works
  (the SoC is ignored). The offset stays. Used in Solar mode, as in the original firmware.
- **Sensorbox 2 on the P1 port:** the voltage of L1 from the smart meter (register 0x0002, already part
  of every reading) is used by the capacity modes instead of 230 V. With CTs only nothing changes.
  *Not tested on real hardware.*

**After updating:** old Home Assistant entities *Battery Guard …* and *Home Battery Full SoC* stay as
"unavailable"; delete them in Home Assistant. Your settings are kept.

## v3.11.3-Clarijs.3 — 10 Nov 2026

- Web: the **Help** button opens the documentation of this fork (was a 404 on dingo35).

## v3.11.3-Clarijs.2 — 10 Nov 2026

- **Battery Guard** (renamed from *Home battery protect*): in Smart mode the home battery does not charge
  the EV; while the battery discharges the charger works like Solar. MQTT `Set/BatteryGuard`,
  `Set/BatteryGuardSoCHigh`, `Set/BatteryGuardSoCLow`; REST `battery_guard…`; HA entities *Battery Guard …*.
- **Configurable timeout for the API mains meter** (`Set/MainsMeterTimeout`, 11–120 s, default 11 s).
- Smart mode: the EV's under-draw (max 1 A) is only added once the EV follows the setpoint.
- Flanders: remaining budget spread over at least 2 minutes at the end of a quarter (experimental).
- Merged dingo35 master (#433 regulation per switched phase, #415 HomeWizard signed power).
- New documentation: README, guides and troubleshooting.

## v3.11.3-clarijs.1 — 9 Nov 2026

First release of this fork:

- Monthly capacity peak synced from the meter (`Set/CapacityPeak`), published and stored.
- Max Sum Mains in 0.1 A steps.
- Regulation per switched phase.
- Measured mains voltage (API 6th value, Modbus `U_REG`, HomeWizard); monthly peak from a HomeWizard P1.
- CAP MODE **Peak** and configurable margin.
- Home battery in watts (`Set/HomeBatteryPower`, offset, full SoC) and home battery protect with SoC window.
- Smart mode start current fixed; setpoint capped at the measured headroom.
