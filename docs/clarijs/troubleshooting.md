# Troubleshooting

[← Back to the overview](../../README.md)

## "Communication Error", charging stops and restarts

**Symptom:** the charger shows *Communication Error*, stops, and ~35 s later starts again. Happens
several times per hour.

**Cause:** with `MAINS MET = API` the charger needs a mains measurement at least every 11 s. A P1
reader on Wi-Fi (especially an ESP8266 with a signal around −75 dBm or weaker) sometimes holds its
messages back for 5–20 s: they all arrive afterwards, but too late.

**Check:** subscribe to `<prefix>/Set/MainsMeter` on your broker and look at the time between
messages. Gaps of 5–20 s followed by a burst of messages = Wi-Fi delay at the reader.

**Fix, in this order:**
1. Improve the reader's Wi-Fi: access point closer, lock the reader to one access point, higher
   transmit power, or an ESP32 / ESP with external antenna. Aim for better than −70 dBm.
2. On an ESP8266: `wifi: power_save_mode: none`, and throttle the sensors you send to Home Assistant
   (use `get_raw_state()` for the SmartEVSE).
3. Set `Set/MainsMeterTimeout` to 20–30 s, so a short hiccup no longer stops charging.

## The home battery charges the car

| Situation | Cause | Fix |
|---|---|---|
| At the start of each quarter, with `CAP MODE = Flanders` | Flanders allows more than the peak when the quarter started quiet; the battery shaves it | use **mode Peak** |
| The battery holds the grid exactly at its threshold while the car charges | charger ceiling too close to the battery threshold | battery threshold closer to the peak than the charger ceiling (e.g. battery peak − 10 W, charger margin 0–55 W), measured voltage on |
| In daytime, grid at 0 W, battery discharging | battery covers house and car (self-consumption) | mode Solar, or block the battery's discharge while charging in Home Assistant (see the [battery guide](guide-home-battery.md)) |
| One night per year, battery goes to ~0 % | battery SOH calibration (e.g. Huawei): reserve is ignored | nothing to fix; the battery recharges afterwards |
| The battery goes below its peak-shaving level (e.g. 15 %) while the car charges | battery still in self-consumption below that level (e.g. after a calibration) | set **Peak Shaving SoC** to that level and send the SoC with the battery power (`W:SoC`) |

## New monthly peak while charging (no home battery)

**Cause:** a big load in the house (oven, hob) for several minutes leaves less than 6 A for the car.
The charger keeps the car at 6 A for `CAP STOP` minutes before it stops; during that time the grid is
above the peak. **Fix:** set `CAP STOP` to Off (0) or 1 minute, see
[capacity tariff guide, section 6](guide-capacity-tariff.md#6-without-a-home-battery-set-cap-stop-short).

## The car starts at full current and overshoots

Fixed in this firmware (start current is calculated, setpoint capped at the measured headroom). If it
still happens, check that an **EV meter** is configured: the headroom cap uses it.

## The mains stays ~100 W under the limit

Most cars draw a little less than the setpoint. This firmware allows up to 1 A of that under-draw on
top of the headroom, once the car follows the setpoint. With a larger margin you also get more room:
check `capacity_margin`.

## Monthly peak wrong after the first of the month

During the first quarter of a month an external peak is ignored (the meter may still report last
month's value). From 00:15 the peak follows the meter again.

## Going back to the official firmware

This firmware is unsigned. Flash an official *unsigned* build of dingo35 first, or flash over USB
(`HowToFlash.txt` in the release zip).
