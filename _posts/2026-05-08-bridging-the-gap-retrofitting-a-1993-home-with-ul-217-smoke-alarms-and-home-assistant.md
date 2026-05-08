---
layout: post
date: 2026-05-08
title: "Bridging the Gap: Retrofitting a 1993 Home with UL 217 Smoke Alarms and
  Home Assistant"
tags:
  - smart home
  - home assistant
  - smoke alarm
---
My old smoke alarms finally reached their 10-year expiration mark, and the replacement process turned into a full-blown engineering project. If you live in a home built in the early 90s—specifically around 1993—you’re likely in the same "infrastructure gap" I was. My house in Houston was originally hardwired for only three spots in the hallways, but modern fire codes now mandate smoke alarms in **every bedroom**.

Here’s the rub: **Building code (NFPA 72) and insurance requirements generally prohibit replacing a hardwired alarm with a battery-only unit.** You must maintain hardwired power where it exists, but you also need a way to link those hallway units to new bedroom alarms so that if one sounds, they all sound.

Here is how I spent ~$380 to build a code-compliant, UL 217-certified mesh that bridges perfectly into Home Assistant and HomeKit.

---

## Why I Skipped "Smart" Smoke Alarms

One thing that surprised me during this project was how disappointing most consumer "smart" smoke alarms still are in 2026. Most major ecosystems are:

- **Cloud Dependent:** If your internet goes down, your remote alerts go down.
- **Battery Hogs:** Radios for WiFi or Z-Wave often chew through batteries, leading to chirping every few months.
- **Expensive:** Scaling to 7+ units for a full home can cost an extra $400-$600 just for "smart" features.

The approach I took is arguably **more reliable**. The actual smoke alarm system remains a standalone, UL-listed life safety network. Home Assistant only *monitors* the relay output; it is not a point of failure in the alarm chain itself. No subscriptions, no vendor lock-in, and dramatically better battery life.

## Why Kidde Beats First Alert for Power-Users

I initially looked at First Alert, but their ecosystem has a major technical bottleneck: their wireless-interconnect units often fail to trigger their own hardwired relay (the RM4). For a smart home setup, that’s a dealbreaker.

Kidde’s **VRF (Wire-Free Interconnect)** series acts as a true bridge. When a battery-powered unit in a bedroom detects smoke, it wirelessly triggers the hardwired units in the hall. The hardwired unit then sends a physical signal to the **SM120X relay module**, which trips the Zigbee bridge for Home Assistant. It’s the most reliable way to mesh legacy wiring with modern "wire-free" requirements.

### The Bill of Materials (BOM)

- **3x Kidde 20SAR-VRF:** Hardwired units for the original 1993 hallway spots.
- **4x Kidde 20SDR-VRF:** Battery-powered units for the bedrooms (satisfying modern code without fishing wires through drywall).
- **1x Kidde SM120X:** The "Brain." This relay module mounts in a junction box and provides the dry contact output you need.
- **1x Zigbee Mini Switch:** Used as a binary input to bridge the relay signal into Home Assistant via ZHA.

> **Pro-Tip:** If your home already has a red traveler wire (interconnect wire) between all boxes, skip the "VRF" models and get standard **Kidde 20SAR** units to save roughly **$30 per alarm**.

---

## The Technical Setup: From Relay to Dashboard

Getting a 120V relay to talk to Home Assistant requires a few specific steps to ensure it’s recognized as a safety device.

### 1. The Physical Wiring

The **SM120X** relay provides several wires. For the signaling bridge, we care about the **Blue (Common)** and **Orange (Normally Open)** wires. 

- Connect the **Blue wire** to the **S1** terminal on your Zigbee mini switch.
- Connect the **Orange wire** to the **S2** terminal.
When the smoke mesh triggers, the relay closes the circuit, and the Zigbee switch reports "on."

### 2. The ZHA "Quirk" & Template Helper

When I paired the switch via **ZHA (Zigbee Home Automation)**, it appeared as a generic "light" or "switch." To get proper behavior, I used a **Template Binary Sensor**:

- **Navigate to:** Settings > Devices > Helpers > Create Helper > Binary Sensor.
- **State Template:** Set it to track when your Zigbee switch is `on`.
- **Device Class:** Set this specifically to `**smoke**`.

### 3. The "Wife-Approval" Factor: HomeKit Critical Alerts

The real magic happens when you bridge this binary sensor to **Apple HomeKit**. By classifying the device as a `smoke` sensor in Home Assistant, iOS recognizes it as a **Life Safety** device.

In the Apple Home app, you can enable **Critical Alerts**. This is a game-changer for peace of mind: if a fire starts at 3 AM, the notification will **bypass "Do Not Disturb"** and the physical mute switch on my wife’s phone, playing a loud, distinct siren sound. That is the difference between a "neat hobby project" and a "household safety essential."

---

## The UL 217 Standard: Why It Matters

As of 2026, the **UL 217 (9th Edition)** standards are the gold standard. These new alarms are designed to be much more resistant to "nuisance alarms" (like burnt toast) while being significantly faster at detecting smoldering synthetic fires common in modern furniture. Saving a few bucks on non-UL-listed Zigbee sensors from a random import brand isn't worth the risk of an insurance claim denial or a sensor that fails to wake the family in time.

## Final Thoughts

This project cost less than $400 and a Saturday afternoon, but the result is a home that is fully up to code, provides total bedroom coverage, and features instant, local-only smart home alerts. It solves the 1993 "no traveler wire" problem without a single patch of drywall repair, and the HomeKit integration ensures my family is protected even when their phones are on silent.