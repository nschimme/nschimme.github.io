---
layout: post
date: 2025-10-15
image: /assets/img/IMG_7596.jpeg
title: "Integrating the Tuya Smart Valve Controller with ESPHome "
tags:
  - Smart Valve
  - ESPHome
  - Tuya
  - Home Assistant
  - smart home
  - hacking
---
When protecting your home, reliability is non-negotiable. A smart water shut-off valve is arguably the second most important component in your smart home defense system when it comes to avoiding damage (wind and hail damage claim first place).

By integrating the Tuya Smart Valve Controller using ESPHome, we eliminate cloud dependencies, giving us complete **local control** and ensuring that the valve works the _instant_ a leak is detected. It's a cost effective solution at ~$25 that won't break the bank!

## Why Local Control?

Unlike a smart light or a thermostat, failure of this device can lead to tens of thousands of dollars in water damage. This demands maximum reliability, which cloud services simply cannot guarantee.

*   **Instant Response:** Local control means zero reliance on external servers. Commands travel across your Wi-Fi network instantly, crucial in an emergency.
    
*   **Guaranteed Stability:** Flashing with ESPHome gives you full ownership of the device's firmware. Your valve won't fail because the manufacturer shut down their cloud or pushed a buggy, forced update.
    

## Flashing OpenBeken and ESPHome

As usual, the [OpenBeken forum](https://www.elektroda.com/news/news3945101.html) was an excellent resource. I didn't have to desolder the CB3S chip and was able to connect my UART USB directly to the appropriate pins to [flash with OpenBeken](https://community.home-assistant.io/t/detailed-guide-on-how-to-flash-the-new-tuya-beken-chips-with-openbk7231t/437276?page=9) and subsequently [OTA migrate to ESPHome](https://docs.libretiny.eu/docs/flashing/esphome/#migrating-from-esphome-to-openbeken). By leveraging `assumed_state: False` and `restore_mode: RESTORE_AND_CALL`, we make up for the lack of a physical sensor detecting the state of the valve, transforming this valve into a reliable, local, and fully-owned piece of hardware. Good luck and keep your home safe! 💧

## YAML

> Note: This post contains affiliate links for the recommended hardware. I may earn a small commission from qualifying purchases, at no extra cost to you.

To follow this guide, you will will need the same model [**Tuya Smart Valve Controller**](https://amzn.to/4hS55XJ) I purchased.

```
esphome:
  name: water-shutoff
  name_add_mac_suffix: false
  friendly_name: Water Shut-Off Valve

bk72xx:
  board: cb3s

logger:

api:
  encryption:
    key: !secret encryption_key

ota:
  platform: esphome

wifi:
  power_save_mode: HIGH
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  ap:
    ssid: "Water Shut-Off Fallback Hotspot"
    password: !secret wifi_password

captive_portal:

web_server:

sensor:
  - platform: uptime
    name: Uptime
    update_interval: 60s
  - platform: wifi_signal
    name: Wifi Signal
    update_interval: 60s

text_sensor:
  - platform: wifi_info
    ip_address:
      name: IP
  - platform: libretiny
    version:
      name: LibreTiny Version

status_led:
  pin: P8

binary_sensor:
  - platform: gpio
    id: binary_switch_1
    pin:
      number: P24
      inverted: true
      mode: INPUT_PULLUP
    on_press:
      then:
        - switch.toggle: relay_1

switch:
  - platform: gpio
    id: relay_1
    pin: P6
    restore_mode: DISABLED
    internal: True

valve:
  - platform: template
    name: Valve
    id: valve_1
    lambda: |-
      if (id(relay_1).state) {
        return VALVE_OPEN;
      } else {
        return VALVE_CLOSED;
      }
    open_action:
      - switch.turn_on: relay_1
    close_action:
      - switch.turn_off: relay_1
    stop_action:
      - logger.log: "Not Supported"
    assumed_state: False
    restore_mode: RESTORE_AND_CALL
```