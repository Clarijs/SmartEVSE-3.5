# Changelog — Clarijs edition

[← Back to the overview](README.md)

Changes of this fork on top of [dingo35/SmartEVSE-3.5](https://github.com/dingo35/SmartEVSE-3.5).
Every change is added to **Unreleased** first; at the next release that section gets the version number.

## Unreleased

- **Removed: Battery Guard.** `Set/BatteryGuard`, `BatteryGuardSoCHigh/Low` (MQTT, REST `battery_guard…`,
  Home Assistant entities *Battery Guard …*, the web fields) are gone; in Smart mode the home battery is
  ignored again, as in the original firmware. Not needed: with mode Peak, the measured voltage and the
  battery's peak-shaving threshold at peak − 10 W the battery stays out of the EV (measured night: 7 Wh
  from the battery during a 6.7 kWh charge). The home battery in watts (`Set/HomeBatteryPower`, offset,
  full SoC; used in Solar mode) stays. The old *Battery Guard* entities can be deleted in Home Assistant.

- **Sensorbox 2 on the P1 port:** the voltage of L1 from the smart meter (register 0x0002, already part
  of every reading) is used by the capacity modes instead of 230 V. With CTs only nothing changes.
  *Not tested on real hardware.*
- **Web dashboard (test, only on branch `release/personal-dashboard`, not in a release yet):** the
  start page (`/`) is a simple visual dashboard: charging status, capacity bar (other on the meter / car
  / room under the ceiling), home battery (only when battery power is received), a 10-minute chart, big
  mode buttons and quick settings (charge current). Uses only the existing `/settings` API; no change to
  the charging logic. The former main page with every setting moved to `/advanced.html`
  (*All settings*). All pages (advanced, capacity, update) got the same style (dark status banner,
  larger cards and buttons, toggles); theme button (auto/light/dark) on every page. "Grid now" uses the
  real meter values (`original_data`). With an RFID reader the dashboard asks to present the card while
  access is off.

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
