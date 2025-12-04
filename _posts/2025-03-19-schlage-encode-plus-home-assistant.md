---
layout: post
date: 2025-03-19
title: Integrating Schlage Encode Plus with Home Assistant via HomeKit Actions
  and Automations
tags:
  - Home Assistant
  - HomeKit
  - Smart Locks
  - Schlage
---
The [Schlage Encode Plus](https://amzn.to/4rEgKxE) is a powerful smart lock that integrates seamlessly with Apple HomeKit. However, Home Assistant lacks native support for this lock. Fortunately, by leveraging HomeKit Actions, the Home Assistant HomeKit bridge, and automations, we can achieve a reliable bidirectional sync—while keeping **Apple Home Key functionality** and **Thread battery life benefits** intact.

> _Note: This post contains affiliate links. I may earn a small commission from qualifying purchases, at no extra cost to you._

## Why This Approach?

This method was originally detailed in the Home Assistant Community [forums](https://community.home-assistant.io/t/schlage-encode-plus-in-apple-home-w-homekey-and-home-assistant/684159/24).

By using this approach, you can:

1.  **Retain Apple Home Key** for fast and secure lock access.
    
2.  **Continue using Thread**, ensuring low-power, high-speed communication. With Thread, battery life exceeds one year compared to less than three months over WiFi.
    
3.  **Avoid the Schlage Wi-Fi integration**, which significantly reduces battery life.
    

The only limitation of this method is that Home Assistant cannot display battery status or detect if the lock is jammed.

## How It Works

We will create three `input_boolean` entities in Home Assistant:

*   `input_boolean.schlage_state`: Represents the current state of the lock.
    
*   `input_boolean.schlage_lock`: A momentary switch that triggers the lock action.
    
*   `input_boolean.schlage_unlock`: A momentary switch that triggers the unlock action.
    

Then, we will define a **template lock** in Home Assistant, expose the `input_boolean` entities to HomeKit, and use **automations** to synchronize lock state changes in both directions.

* * *

## Step 1: Create Input Booleans

Instead of modifying `configuration.yaml`, we will create these input booleans via the **Helpers UI**:

1.  Open Home Assistant and navigate to **Settings > Devices & Services > Helpers**.
    
2.  Click **Create Helper** and select **Toggle**.
    
3.  Name it **Schlage State**, assign an appropriate icon (e.g., `mdi:lock`), and click **Create**.
    
4.  Repeat this process to create two additional toggles:
    
    *   **Schlage Lock** (`mdi:lock`)
        
    *   **Schlage Unlock** (`mdi:lock-open`)
        

### Hiding Input Booleans from the UI

To prevent unnecessary clutter in your dashboard:

1.  Go to **Settings > Devices & Services > Entities**.
    
2.  Locate the three `input_boolean` entities.
    
3.  Click each one and enable **"Hide from UI"**.
    
4.  Click **Save**.
    

These entities will remain available for automations but will not appear on the dashboard.

* * *

## Step 2: Expose Input Booleans to HomeKit

To enable HomeKit to interact with these booleans, expose them through the **HomeKit Bridge**:

1.  Open **Settings > Devices & Services**.
    
2.  Locate **HomeKit Bridge** and click **Configure**.
    
3.  Click **Include Entities**, then add:
    
    *   `input_boolean.schlage_state`
        
    *   `input_boolean.schlage_lock`
        
    *   `input_boolean.schlage_unlock`
        
4.  Click **Submit** and restart Home Assistant if prompted.
    

If Home Assistant is not yet paired with HomeKit, follow the on-screen instructions to complete the setup.

* * *

## Step 3: Create the Lock Template

Now, define a `lock` entity using a template in `configuration.yaml`:

```yaml
template:
- lock:
  - unique_id: lock.front_door
    name: Front Door Lock
    optimistic: false
    lock:
    # Sanity test
    - target:
        entity_id:
        - input_boolean.schlage_unlock
      action: input_boolean.turn_off
    # Reset the momentary switch if its stuck
    - choose:
      - conditions:
        - condition: state
          entity_id:
          - input_boolean.schlage_lock
          state: 'on'
          match: all
        sequence:
        - target:
            entity_id:
            - input_boolean.schlage_lock
          action: input_boolean.turn_off
        - delay: 0:00:01
    # Start the momentary switch to lock
    - target:
        entity_id:
        - input_boolean.schlage_lock
      action: input_boolean.turn_on
    - delay: 0:00:01
    # Stop the momentary switch to lock
    - target:
        entity_id:
        - input_boolean.schlage_lock
      action: input_boolean.turn_off
    unlock:
    # Sanity test
    - target:
        entity_id:
        - input_boolean.schlage_lock
      action: input_boolean.turn_off
    # Reset the momentary switch if its stuck
    - choose:
      - conditions:
        - condition: state
          entity_id:
          - input_boolean.schlage_unlock
          state: 'on'
          match: all
        sequence:
        - target:
            entity_id:
            - input_boolean.schlage_unlock
          action: input_boolean.turn_off
        - delay: 0:00:01
    # Start the momentary switch to unlock
    - target:
        entity_id:
        - input_boolean.schlage_unlock
      action: input_boolean.turn_on
    - delay: 0:00:01
    # Stop the momentary switch to unlock
    - target:
        entity_id:
        - input_boolean.schlage_unlock
      action: input_boolean.turn_off
    state: "{% raw %}{{ is_state('input_boolean.schlage_state', 'on') }}{% endraw %}" # Sync lock state
```

This template lock:

*   Appears as locked when `input_boolean.schlage_state` is `on`.
    
*   Locks when `input_boolean.schlage_lock` is triggered.
    
*   Unlocks when `input_boolean.schlage_unlock` is triggered.
    

Restart Home Assistant to apply the changes.

* * *

## Step 4: Configure HomeKit Automations

In the **Apple Home app**, create **four automations**:

1.  When the **lock is locked**, turn `input_boolean.schlage_state` **on**.
    
2.  When the **lock is unlocked**, turn `input_boolean.schlage_state` **off**.
    
3.  When `input_boolean.schlage_lock` turns **on**, lock the door.
    
4.  When `input_boolean.schlage_unlock` turns **on**, unlock the door.
    

These automations ensure Home Assistant stays synchronized with the actual lock state. The input boolean will show up as a switch in your HomeKit home.

## Conclusion

With this setup, your Schlage Encode Plus smart lock is now fully integrated with Home Assistant. The lock state remains synchronized, and you can control it just like a native entity.

You can also repeat this process with different input\_boolean names for multiple locks if needed. For example, `input_boolean.front_door_state`, `input_boolean.front_door_lock` , and `input_boolean.front_door_unlock`. Enjoy seamless integration of your Schlage Encode Plus with Home Assistant!