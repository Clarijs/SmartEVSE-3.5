# Guide: charging under the Flemish capacity tariff

[← Back to the overview](../../README.md)

Since 2023 the Flemish grid operator bills the **highest 15-minute average power of the month**
(minimum 2.5 kW). Charging an EV can easily set a new peak. This guide sets up the SmartEVSE so the
car charges as fast as possible **without ever raising that peak**.

## 1. What you need

| | |
|---|---|
| Charger | SmartEVSE v3 with this firmware, mode **Smart** |
| Mains measurement | A reader on the P1 port of the digital meter, sending to the SmartEVSE every second |
| Network | An MQTT broker (e.g. Mosquitto) that both the P1 reader and the SmartEVSE use |

Any P1 reader that can publish MQTT works. The example below uses an ESP8266 with ESPHome. A
HomeWizard P1 meter also works without MQTT (`MAINS MET = Homewizrd`): this firmware reads the voltage
and the official monthly peak from it as well.

## 2. How the charger limits the power

```
monthly peak (billed, from the meter)      2755 W
SmartEVSE ceiling = peak − margin           2700 W   (margin 55 W)
EV current = ceiling − rest of the house, converted with the measured voltage
```

**Mode Peak** keeps a stable ceiling just under the monthly peak. Every mains measurement
(every second) the SmartEVSE recalculates how much is left for the car.

> **Why not mode Flanders?** Flanders spreads a 15-minute energy budget. When the house used little at
> the start of a quarter, it allows more than the peak for the rest of the quarter. The average stays
> correct, but the instantaneous power goes above the peak. With a home battery doing peak shaving,
> that extra comes from the battery (measured: −2.2 kW into the car). Peak does not have this problem.
> The difference in charging time is small: about 5–7 minutes on a ~10 kWh night session.

## 3. Send the measurements from the P1 port

The SmartEVSE needs two things over MQTT:

| Topic | Payload | How often |
|---|---|---|
| `<prefix>/Set/MainsMeter` | `L1:L2:L3:W:Wh:V`: current per phase (0.1 A, + = import), power (W), total import (Wh), voltage L1 (0.1 V) | every second |
| `<prefix>/Set/CapacityPeak` | official monthly peak in W (OBIS 1-0:1.6.0), at least 2500 | every minute |

`<prefix>` is `SmartEVSE-<serial>`, e.g. `SmartEVSE-08512`.

ESPHome example (single phase; give the `dsmr` sensors the ids used here):

```yaml
wifi:
  power_save_mode: none        # recommended on an ESP8266: fewer delayed messages

mqtt:
  broker: 192.168.0.129
  username: esphome
  password: !secret mqtt_password
  discovery: false
  topic_prefix: null
  log_topic: null
  birth_message:
  will_message:

interval:
  - interval: 1s
    then:
      - if:
          condition:
            lambda: 'return !isnan(id(p_del).get_raw_state()) && !isnan(id(p_ret).get_raw_state());'
          then:
            - mqtt.publish:
                topic: SmartEVSE-08512/Set/MainsMeter
                payload: !lambda |-
                  float v = id(v_l1).get_raw_state();
                  if (isnan(v) || v < 180 || v > 270) v = 230.0;
                  float w = (id(p_del).get_raw_state() - id(p_ret).get_raw_state()) * 1000.0;
                  int l1 = (int) lroundf(w / v * 10.0);
                  float e1 = id(e_t1).get_raw_state(), e2 = id(e_t2).get_raw_state();
                  long wh = (isnan(e1) || isnan(e2)) ? -1 : lround((e1 + e2) * 1000.0);
                  char buf[64];
                  snprintf(buf, sizeof(buf), "%d:0:0:%d:%ld:%d",
                           l1, (int) lroundf(w), wh, (int) lroundf(v * 10.0));
                  return std::string(buf);
  - interval: 60s
    then:
      - if:
          condition:
            lambda: 'return !isnan(id(piek_maand).get_raw_state());'
          then:
            - mqtt.publish:
                topic: SmartEVSE-08512/Set/CapacityPeak
                payload: !lambda |-
                  int w = (int) lroundf(id(piek_maand).get_raw_state() * 1000.0);
                  return to_string(w < 2500 ? 2500 : w);
```

`get_raw_state()` uses the value of every telegram, also when you throttle the same sensors towards
Home Assistant (e.g. `throttle: 5s`), which keeps the ESP lighter.

## 4. Settings on the SmartEVSE

| Setting | Value | Where |
|---|---|---|
| `MAINS MET` | **API** | LCD menu |
| `MODE` | **Smart** | LCD, web, MQTT |
| `CAP MODE` | **Peak** | web interface → All settings → Capacity, or LCD |
| Margin | **50–100 W** (55 W with a peak-shaving battery, see the battery guide) | `Set/CapacityMargin`, REST `capacity_margin` |
| `CONTACT 2` | **Always Off** for a single-phase installation | LCD menu |
| Mains meter timeout | **20–30 s** for a P1 reader on Wi-Fi | `Set/MainsMeterTimeout` |

Check `GET /settings`: `capacity_peak_month` should show your meter's peak and `capacity_voltage`
the measured voltage.

## 5. What to expect

- At the start of a month the peak is 2500 W (the minimum). It rises automatically when the meter
  reports a higher quarter.
- The car starts at the current that is actually free, not at full current.
- Night charging settles about 50–110 W under the peak. Short peaks in the house (kettle) are
  corrected within a second or two.

Next: [combine it with a home battery](guide-home-battery.md) ·
[problems?](troubleshooting.md)
