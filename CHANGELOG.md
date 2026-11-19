# Changelog — Clarijs edition

[← Back to the overview](README.md)

Changes of this fork on top of [dingo35/SmartEVSE-3.5](https://github.com/dingo35/SmartEVSE-3.5).
Every change is added to **Unreleased** first; at the next release that section gets the version number.

## Unreleased

- **Smart mode: no longer stays up to 0.3 A below the limit.** When there is room, the charge current
  rises by 1/4 of the free headroom per mains measurement. Currents are whole tenths of an ampere, so with
  0.1-0.3 A of headroom that quarter was 0 and the current never rose further: up to ~70 W unused under a
  capacity ceiling (median ~80 W below the monthly peak over 129 charging quarters). It now rises by at
  least 0.1 A while there is room; going down is still immediate.
- **MQTT: publish only what changed.** The status topics are still checked every 10 seconds, but a topic
  is only sent again when its value changed, plus once a minute as a heartbeat. After a (re)connect
  everything is sent once. Far fewer messages for the broker and Home Assistant; values that change
  still arrive just as fast. Idea from [basmeerman/SmartEVSE-3.5](https://github.com/basmeerman/SmartEVSE-3.5).
- Merged dingo35 master: **Ethernet (CH390D) no longer stalls under load.** A missed interrupt could stop
  receiving for 1 s, and a corrupt packet could cause a loop of resets while traffic continued.
- **Ethernet diagnostics.** New diagnostic sensors `Eth Rx Overflow`, `Eth Rx Resync`, `Eth Tx Busy` and
  `Eth Rx Busy Loops` (also under `eth.diag` in `/settings`). When nothing is received or sent for 60 s
  while the cable is connected, the chip registers and counters are stored once in flash and shown after
  the next restart as `Eth Last Stall`. Meant to find the cause of a network lockup after the CH390D fix.
- **Fix: Ethernet could stay deaf until a power cycle** (found with the diagnostics above). After a bad
  frame the CH390D could still hold received data while its "packet received" interrupt was already
  cleared; nothing read the buffer again, it filled up and every new frame was dropped. The driver now
  also drains the buffer when the chip reports data waiting, keeps reading after a bad frame, and
  restarts the receiver when nothing arrives for 30 s with the cable connected. New diagnostic sensors
  `Eth Rx Kick` and `Eth Rx Reset` count how often that safety net was needed.
- **Fix: `Eth Last Stall` stayed empty when the 30 s receiver restart kicked in.** The snapshot was only
  taken after 60 s without traffic, but the restart already brings the network back after 30 s, so a
  stall that was caught never left a trace (seen on 17 Nov: network gone for 32 s, `Eth Rx Reset` 1,
  `Eth Last Stall` still from the previous firmware). The chip registers and counters are now stored just
  before each receiver restart (`why=rxwd`, once per stall), the 60 s snapshot stays for other stalls
  (`why=idle`). The flash write happens in the main loop, not in the Ethernet task.
- **Fix: Ethernet stall recovered after 1 s instead of 30 s** (found with the snapshot above, 17 Nov 18:09).
  The CH390D reported a frame waiting while the byte at its read pointer was 0x00; the driver took that
  as "no frame", never resynced, the buffer filled up and nothing was received until the 30 s restart —
  long enough for a mains-meter communication error. When this lasts 1 s the receiver is now restarted at
  once. New diagnostic sensor `Eth Rx Zero` (REST `eth.diag.rx_zero`) counts it; the snapshot gets
  `why=zero` and a `zero=` counter. The 30 s restart stays as a last resort.
- **Ethernet diagnostics: traffic mix.** New sensors `Eth Rx Unicast`, `Eth Rx Multicast` and `Eth Rx Broadcast`
  (REST `eth.diag.rx_ucast` / `rx_mcast` / `rx_bcast`) count received frames by destination. The CH390D
  accepts all multicast and broadcast; this shows how much of the ~10 frames/s is not meant for the charger.
  `Eth Rx Broadcast Arp` counts the ARP part of the broadcast, and `Eth Top Broadcast` / `Eth Top Multicast`
  (REST `top_bcast` / `top_mcast`) show the three senders (MAC address = frames since boot) that send the most,
  so the source can be looked up in the router or access-point client list.
- **Dashboard: "Live power" replaces the 10-minute chart.** One row House · Grid · Battery · Car with
  moving dots in the direction the energy flows (line thickness follows the power) and a status line in
  plain words, e.g. "The home battery helps the car with 1.29 kW" or "the battery stays out of the car".
  Without a home battery the battery circle is hidden. House is calculated (grid − car − battery), so it
  turns negative and green when solar produces more than the house uses.

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
