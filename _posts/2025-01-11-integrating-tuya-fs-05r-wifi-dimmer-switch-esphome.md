---
layout: post
title: "Integrating the Tuya FS-05R WiFi Dimmer Switch with ESPHome"
date: 2025-01-11
tags: [FS-05R, smart home, OpenBeken, ESPHome, hacking, Home Assistant]
---

When I moved to Texas and started setting up my smart home, I encountered a specific challenge: a combination switch in my kitchen controlled two separate lights. One of the lights was too bright, so I needed a dimmer solution. The issue? The existing tan-colored switch was built into the tile backsplash, and replacing it with a larger two-gang box wasn’t an option. It had to be hackable, open-source, retain the same look, and be smart—all while keeping my wife happy with it. After some research, I found a way to meet these requirements using the Tuya FS-05R dimmer switch, ESPHome, and a little ingenuity. This post walks you through the process.

## Build Versus Buy

My first instinct was to look for a Matter-compatible device, but I quickly realized that no suitable options existed on the market. That left me with a DIY approach. After some research, I considered two options: the $4 [Tuya FS-05R](https://www.aliexpress.us/item/3256806679229437.html) and the $31 [Shelly Dimmer2](https://us.shelly.com/products/shelly-dimmer2). While the Shelly Dimmer2 offered premium open-source capabilities, the challenge and learning experience of working with the FS-05R intrigued me. Since the FS-05R was compatible with OpenBeken, I expected it to be a cost-effective and reasonable solution. I was wrong.

## Initial Concerns

The FS-05R's wiring diagram appeared simple enough for connecting to mains and integrating with the existing switch. However, after purchasing the switch, I discovered through [Reddit](https://www.reddit.com/r/homeassistant/comments/17bnkvt/fs05r_zigbee_dimmer_anyone_used_it_with_zha/) that it required a momentary push button rather than a standard toggle switch. This created a challenge: I needed to maintain the familiar combination switch, which was not a momentary push button.

### Inspiration Strikes

I got inspiration from a [Sonoff WiFi Switch](https://www.aliexpress.us/item/3256806718368541.html) and decided to replicate this setup using two low-voltage sensors labeled as S1 and S2 in their wiring diagram. By hooking up the FS-05R's GPIO pins to the existing combination switch, I could bypass the momentary push buttons entirely with some custom code.

But I still had to figure out how to adjust the brightness manually. After considering my options, I decided that an interval timer could be used to cycle through different brightness levels each time the switch was toggled off and on. This approach would allow for manual control without relying on any smart interface. In short, if my wife toggled the lights within 5 seconds, it would toggle between brightness settings. Otherwise, it would stay at a constant level.

## Flashing OpenBeken and ESPHome

I began by desoldering the [CBS2 module](https://developer.tuya.com/en/docs/iot/cb2s-module-datasheet?id=Kafgfsa2aaypq) from the PCB and [flashing it with OpenBeken](https://community.home-assistant.io/t/detailed-guide-on-how-to-flash-the-new-tuya-beken-chips-with-openbk7231t/437276?page=9). Afterward, I performed an [OTA migration from OpenBeken to ESPHome](https://docs.libretiny.eu/docs/flashing/esphome/#migrating-from-esphome-to-openbeken), which I prefer for its ease of use and seamless integration with Home Assistant.

This process liberated the device from the Tuya app, giving me complete control over its functionality. I next had to scour the [OpenBeken's forums](https://www.elektroda.com/rtvforum/topic4027492-30.html) to understand the proper GPIO pins and how to interface with the [fake TuyaMCU](https://www.elektroda.com/rtvforum/topic4039890.html) on the FS-05R. It was painful but I succeeded by testing the CBS2 and the GPIO pins in isolation on my desk before resoldering the PCB to the device and verifying the behavior on mains. Be safe!

Below is the YAML configuration I used. I configured GPIO pins P23 and P6 to be hooked up to the combination switch. It defines GPIO-based physical controls and includes an interval timer for cycling brightness when toggling the switch.

```yaml
esphome:
  name: fs05r
  name_add_mac_suffix: false
  friendly_name: FS-05R Mini Dimmer Switch
  on_boot:
    priority: -10
    then:
      - globals.set:
            id: boot_complete
            value: "true"
      - light.turn_off: out

bk72xx:
  board: cb2s

# Enable logging
logger:

# Enable Home Assistant API
api:
  encryption:
    key: !secret encryption_key

ota:
  - platform: esphome

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "FS-05R Fallback Hotspot"
    password: !secret fallback_hotspot

captive_portal:

globals:
  - id: brightness_index
    type: int
    restore_value: no
    initial_value: '0'
  - id: last_press_time
    type: unsigned long
    restore_value: no
    initial_value: '0'
  - id: boot_complete
    type: bool
    restore_value: no
    initial_value: "false"

# https://www.elektroda.com/rtvforum/topic4039890.html
status_led:
  pin:
    number: P8
    inverted: true

uart:
  id: uartbus
  rx_pin: RX1
  tx_pin: TX1
  baud_rate: 115200
  debug:
    direction: BOTH

time:
  - platform: homeassistant
    id: ha_time
    on_time_sync:
      then:
        - lambda: |-
            id(last_press_time) = id(ha_time).now().timestamp;

output:
  - platform: gpio
    pin: P23
    id: S0
  - platform: template
    id: fake_tuya
    type: float
    min_power: 0.0
    max_power: 1.0
    write_action:
      - uart.write:
          # https://www.elektroda.com/rtvforum/topic3880546.html
          id: uartbus
          data: !lambda |-
            uint8_t brightness_level = int(id(out).remote_values.get_brightness() * 255);
            uint8_t toggle_value = int(id(out).remote_values.is_on());

            // Calculate the value to send to the Tuya device
            uint16_t val = brightness_level * 3 * toggle_value;

            ESP_LOGD("fake_tuya", "Brightness: %d, Toggle: %d, val: %d", brightness_level, toggle_value, val);

            // Split the value into two bytes
            uint8_t high = (val >> 8) & 0xFF;
            uint8_t low = val & 0xFF;

            // Prepare TuyaMCU data packet
            // [0] 0x55       - Start byte
            // [1] 0xAA       - Second byte
            // [2] 0x00       - Protocol version or command type (can vary)
            // [4] 0x30       - Command type of package (content)
            // [5] 0x00       - The size of the packet`s data content in bytes (high)
            // [6] 0x03       - The size of the packet`s data content in bytes (low)
            // [7] 0x00       - Unknown
            // [8] high       - High byte of the calculated value (brightness * 3 * toggle_value)
            // [9] low        - Low byte of the calculated value
            // [10] checksum  - Checksum
            std::vector<uint8_t> data = {0x55, 0xAA, 0x00, 0x30, 0x00, 0x03, 0x00, high, low, 0x00};

            // Sum all bytes except the checksum itself (last byte)
            uint16_t sum = 0;
            for (size_t i = 0; i < data.size() - 1; ++i) {
              sum += data[i];
            }

            // Take the result modulo 256 to get the checksum
            uint8_t checksum = static_cast<uint8_t>(sum % 256);
            data[data.size() - 1] = checksum;

            ESP_LOGD("fake_tuya", "High: %d, Low: %d, Checksum: %d", high, low, checksum);

            return data;

light:
  - platform: monochromatic
    id: out
    name: "Dimmer"
    restore_mode: ALWAYS_OFF
    default_transition_length: 0s
    output: fake_tuya

number:
  - platform: template
    name: "Brightness Adjustment Interval"
    id: brightness_adjustment_interval
    min_value: 0.0
    max_value: 10.0
    initial_value: 5.0
    step: 1.0
    unit_of_measurement: "s"
    restore_value: yes
    optimistic: true
    icon: "mdi:timer-sand"
    entity_category: config

binary_sensor:
  - platform: gpio
    pin: P7
    id: button
    name: "Button"
    internal: true
    on_press:
      - script.execute: cycle
  - platform: gpio
    pin: P24
    id: dimmer
    name: "Dimmer"
    internal: true
    on_press:
      then:
        - light.dim_relative:
            id: out
            relative_brightness: -10%
  - platform: gpio
    pin: P26
    id: brighter
    name: "Brighter"
    internal: true
    on_press:
      then:
        - light.dim_relative:
            id: out
            relative_brightness: 10%
  - platform: gpio
    pin:
      number: P6
      inverted: true
    id: dumb_switch
    name: "Dumb Switch"
    internal: true
    filters:
      - delayed_on_off: 100ms
    on_state:
        - if:
            condition:
              - lambda: "return id(boot_complete);"
            then:
              - script.execute: cycle

script:
  - id: cycle
    mode: restart
    then:
      if:
        condition:
          light.is_on: out
        then:
          - light.turn_off: out
        else:
          - lambda: |-
              uint32_t current_time = id(ha_time).now().timestamp;
              float cycle_interval = id(brightness_adjustment_interval).state;

              auto call = id(out).turn_on();

              // Check if the switch is pressed within the cycle interval
              uint32_t time_difference = current_time - id(last_press_time);
              if (time_difference < cycle_interval) {
                // Switch the brightness index
                id(brightness_index) = (id(brightness_index) + 1) % 4;  // Cycle through brightness levels

                // Set brightness based on the profile
                if (id(brightness_index) == 0) {
                  call.set_brightness(1.0);
                  ESP_LOGD("cycle", "Brightness set to 100%");
                } else if (id(brightness_index) == 1) {
                  call.set_brightness(0.66);
                  ESP_LOGD("cycle", "Brightness set to 66%");
                } else if (id(brightness_index) == 2) {
                  call.set_brightness(0.33);
                  ESP_LOGD("cycle", "Brightness set to 33%");
                } else {
                  call.set_brightness(0.15);
                  ESP_LOGD("cycle", "Brightness set to 15%");
                }
              }

              call.perform();

              ESP_LOGI("cycle", "Time difference: %lu seconds, Brightness index: %d", time_difference, id(brightness_index));

              // Update the last press time
              id(last_press_time) = current_time;
```

## Troubleshooting and Tips

Note for others down the road: unshielded 3.3V wires near mains voltage can pick up interference, causing unstable readings or erratic behavior due to electromagnetic or capacitive coupling and result in the light turning off or on unexpectedly. To avoid this, keep 3.3V wires away from mains wiring—just a few centimeters of separation can make a big difference!

## Conclusion

Integrating the Tuya FS-05R with OpenBeken and ESPHome was a cost-effective solution that met my dimming needs without altering the existing switch interface. By leveraging GPIO pins and an interval timer, I added smart functionality while maintaining a clean aesthetic.

This project is a testament to the power of DIY home automation. It’s not always smooth sailing, but with patience and creativity, the end result is incredibly rewarding.
