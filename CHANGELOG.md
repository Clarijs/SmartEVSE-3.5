# Changelog — Clarijs edition

[← Back to the overview](README.md)

Changes of this fork on top of [dingo35/SmartEVSE-3.5](https://github.com/dingo35/SmartEVSE-3.5).
Every change is added to **Unreleased** first; at the next release that section gets the version number.

## Unreleased

- **Sensorbox 2 on the P1 port:** the voltage of L1 from the smart meter (register 0x0002, already part
  of every reading) is used by the capacity modes instead of 230 V. With CTs only nothing changes.
  *Not tested on real hardware.*
- **Web dashboard (test, only on branch `release/personal-dashboard`, not in a release yet):** new page
  `/dashboard.html` with a simple visual overview: charging status, capacity bar (house / car / room
  under the ceiling), home battery with Battery Guard state, a 10-minute chart, big mode buttons and quick
  settings. Uses only the existing `/settings` API; no change to the charging logic. The dashboard is
  now the start page (`/`); the former main page with every setting moved to `/advanced.html`
  (*All settings*). All pages (advanced, capacity, update) got the same style as the dashboard (dark
  status banner, larger cards and buttons, toggles). Theme button (auto/light/dark) also on the
  dashboard; dark mode with more contrast.
  Without a home battery (or when no battery power arrived for 2 minutes) the dashboard hides the
  battery card, the battery line in the chart and the Battery Guard switch.
  "Grid now", the capacity bar and the chart use the real meter values (`original_data`), not the
  values with the battery discharge added by Battery Guard.
  "Other on the meter" = mains minus car (the charger does not know the solar production).
  With an RFID reader the dashboard asks to present the card while access is off.

- **Battery Guard stop (only on `release/personal-dashboard`):** Battery Guard lowers the EV while the
  home battery discharges, but never below the minimum current (6 A), so a battery in self-consumption
  mode still covers ~1.4 kW of the EV. New setting `BatteryGuardStop` (minutes, 0 = off): when the
  battery keeps discharging with the EV at the minimum current for that long, charging stops
  ("No Power Available") and restarts once the battery has been idle for 60 s. MQTT
  `Set/BatteryGuardStop` (+ HA number "Battery Guard Stop"), REST `battery_guard_stop`, `/settings`
  `home_battery.guard_stop`, `guard_stop_timer`, `guard_stopped`. Field *Stop after* on All settings;
  the dashboard shows "Paused by Battery Guard". *Not tested on real hardware yet.*

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
