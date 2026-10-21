---
title: "How to automate darts board lights with ESP32"
date: 2026-10-21
series: ["ESP32", "Home Assistant"]
weight: 1
tags: ["ESP32", "ESPHome", "Home Assistant", "IoT"]
author: "Arto Salminen"
---

My dartboard has its own light and a score display, both on Tapo smart plugs. I wanted them to turn on when I start throwing and off when I stop, without touching a switch or a phone. The solution is a vibration sensor on the board, an ESP32 running ESPHome, and two Home Assistant automations.

Full source: [github.com/artosalminen/esp32-vibration-sensor](https://github.com/artosalminen/esp32-vibration-sensor)

## Hardware

- [diymore ESP32-C3 board with a 0.42" OLED](https://www.amazon.de/dp/B0DYV1TJQS). Native USB-C and a built-in 72x40 SSD1306 display.
- [SW-420 vibration sensor module](https://www.amazon.de/dp/B07XJTNRD8). A spring vibration switch with an LM393 comparator, a sensitivity potentiometer and a digital `DO` output.
- Three Dupont jumper wires.
- A smart power strip for the light and display. I use a [TP-Link Tapo P300](https://www.verkkokauppa.com/fi/product/847273/TP-LINK-Tapo-P300-alyjatkojohto), which has three individually switchable outlets. Through Home Assistant's TP-Link Smart Home integration each outlet shows up in Home Assistant as its own switch, for example `switch.office_p300_darts_board_light` and `switch.office_p300_darts_display`, the entities used in the automations below.

Wiring is three wires:

| SW-420 | ESP32-C3 |
|--------|----------|
| VCC    | 3V3      |
| GND    | GND      |
| DO     | GPIO3    |

Power the module from 3.3V so `DO` stays at a safe level for the C3. Avoid GPIO2, 8 and 9 (strapping pins) and GPIO5/6 (the OLED's I2C bus).

Mount the SW-420 firmly on the back of the board or on the cabinet frame with a screw or hot glue. A module hanging from its wires picks up much less.

## Detecting a hit

The SW-420 isn't an accelerometer. When the board is hit, the spring inside chatters and `DO` outputs a burst of pulses. A dart gives several pulses in a short time, while a door closing or someone walking past gives one or two.

So the firmware counts rising edges in 100ms windows. If a window has at least *Hit Threshold* pulses, it's a hit. The ESP32-C3 has no hardware pulse counter peripheral, so `use_pcnt: false` is required:

```yaml
sensor:
  - platform: pulse_counter
    id: vibration_pulses
    pin:
      number: GPIO3
      mode: INPUT
    use_pcnt: false
    count_mode:
      rising_edge: INCREMENT
      falling_edge: DISABLE
    internal_filter: 200us
    update_interval: 100ms
    internal: true
    filters:
      # pulse_counter reports pulses/minute; convert to pulses per 100ms window
      - multiply: !lambda return 100.0f / 60000.0f;
      - lambda: return roundf(x);
```

The hit itself is a template binary sensor. `delayed_off: 500ms` makes Home Assistant see one clean on/off per hit instead of a flicker:

```yaml
binary_sensor:
  - platform: template
    name: "Dartboard Hit"
    id: dartboard_hit
    device_class: vibration
    lambda: |-
      return id(vibration_pulses).has_state() &&
             id(vibration_pulses).state >= id(hit_threshold).state;
    filters:
      - delayed_off: 500ms
```

The threshold is a template `number` with `restore_value: true`, so it survives reboots and can be changed from Home Assistant without reflashing.

There's also a `Vibration Pulses` sensor that sends the peak pulse count per second to Home Assistant. That's what you use for calibration.

## The display and the button

The OLED shows WiFi and Home Assistant connection status, a hit counter, a big inverted **HIT!** for two seconds after each hit, and a bar for the live pulse level against the threshold. The bar fills completely when a window reaches the threshold.

The board's BOOT button (GPIO9) sets the threshold: first press opens the threshold screen, a short press is +1, a long press is −1. This makes calibration easy, since you can throw a dart, look at the bar and adjust right at the board.

Don't mix up BOOT and RST. RST reboots the board, which looks like a WiFi reconnect.

## Calibrating

1. Turn the SW-420 potentiometer until its `DO` LED is steady when nothing moves and flickers when you tap the board.
2. Watch the pulse bar or the `Vibration Pulses` history while the room is quiet, then while throwing darts.
3. Set the threshold between the two. The default is 3 pulses per 100ms.

If a gentle hit fills the bar many times over, the potentiometer is too sensitive.

## The automations

All the light logic is in Home Assistant. The ESP only reports hits.

**Lights on after three hits in 30 seconds.** One knock on the wall shouldn't switch anything on. The first hit starts the automation, then it waits for two more hits within the same 30 second window. `wait.remaining` carries the leftover time from the first wait into the second:

```yaml
alias: "Darts: lights on after 3 hits in 30 s"
mode: single
triggers:
  - trigger: state
    entity_id: binary_sensor.dartboard_impact_sensor_dartboard_hit
    from: "off"
    to: "on"
actions:
  - wait_for_trigger:
      - trigger: state
        entity_id: binary_sensor.dartboard_impact_sensor_dartboard_hit
        from: "off"
        to: "on"
    timeout: "00:00:30"
    continue_on_timeout: false
  - wait_for_trigger:
      - trigger: state
        entity_id: binary_sensor.dartboard_impact_sensor_dartboard_hit
        from: "off"
        to: "on"
    timeout:
      seconds: "{{ wait.remaining | float(0) }}"
    continue_on_timeout: false
  - action: homeassistant.turn_on
    target:
      entity_id:
        - switch.office_p300_darts_board_light
        - switch.office_p300_darts_display
        - light.darts_mood_light
```

**Lights off after 15 minutes without hits.** This triggers either when the hit sensor has been off for 15 minutes or when the lights have been on for 15 minutes. The second trigger covers the case where someone turns the lights on by hand and never plays. No timer helpers needed:

```yaml
alias: "Darts: board light and display off after 15 min without hits"
mode: single
triggers:
  - trigger: state
    entity_id: binary_sensor.dartboard_impact_sensor_dartboard_hit
    to: "off"
    for: "00:15:00"
  - trigger: state
    entity_id:
      - switch.office_p300_darts_board_light
      - switch.office_p300_darts_display
    to: "on"
    for: "00:15:00"
conditions:
  - condition: state
    entity_id: binary_sensor.dartboard_impact_sensor_dartboard_hit
    state: "off"
    for: "00:15:00"
  - condition: or
    conditions:
      - condition: state
        entity_id: switch.office_p300_darts_board_light
        state: "on"
      - condition: state
        entity_id: switch.office_p300_darts_display
        state: "on"
actions:
  - action: switch.turn_off
    target:
      entity_id:
        - switch.office_p300_darts_board_light
        - switch.office_p300_darts_display
```

The mood light is left on on purpose. Check your own entity IDs under **Settings > Devices & Services > Entities**, because Home Assistant prefixes them with the device name.

The P300 outlets also have their own auto-off timers, exposed as `switch.office_darts_*_auto_off_enabled`. Leave them disabled, or set them longer than 15 minutes, so they don't cut the light mid-game. The same goes for any smart plug with a built-in timer.

## Gotcha: the ESP32-C3 wouldn't join WiFi

The board saw my router but failed to associate with `Auth Expired` errors. These small C3 boards have a badly matched chip antenna, and at full transmit power association fails. Capping the output power fixed it. Disabling power save and skipping the scan made the connection more reliable:

```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  output_power: 8.5dB
  power_save_mode: none
  fast_connect: true
```

If you have one of these boards and it refuses to connect, try this first.

## Result

Pick up the darts, throw three, and the light and display come on. Walk away, and 15 minutes later they turn off. Flashing instructions, wiring details and the complete ESPHome config are in the [repository](https://github.com/artosalminen/esp32-vibration-sensor).
