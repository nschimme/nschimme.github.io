---
layout: post
date: 2025-05-18
title: Enabling Open Source Two-Way Audio in Thingino
---
I'm excited to share a significant new feature in Thingino: two-way audio support! One-way audio support already existed, but this allows for real-time audio communication from anNVR to the Thingino device to be played back over the remote speaker. In this post, I'll walk through the architecture of this backchannel audio pipeline and discuss some of the key considerations that went into its development, keeping the resource-constrained environment in mind.

## Backchannel Audio Pipeline: Architecture Overview

The core of this new functionality lies in the backchannel audio pipeline, designed to manage audio flowing from an RTSP client to the device.

Here’s a breakdown of the key stages in the pipeline:

1\. **RTSP Setup and SDP (Session Description Protocol):**

The journey begins with the [Prudynt-T RTSP server](https://github.com/gtxaspec/prudynt-t), which utilizes Live555. For each supported audio format, it creates a `BackchannelServerMediaSubsession`. The Session Description Protocol (SDP) plays a crucial role here, defining the audio stream parameters for the client. This includes essential details like the codec being used (e.g., AAC, PCMU), the sample rate, and other audio characteristics.

2\. **Audio Reception and Timeout Mechanism:**

The `BackchannelSink` is responsible for receiving the encoded audio data streamed from the RTSP client. A critical feature implemented here is a timeout mechanism. If no audio data is received for a predefined period, the `BackchannelSink` intelligently sends a "stop" frame. This signals the end of the current audio session.

3\. **Frame Queueing:**

Once audio data is received, the `BackchannelSink` wraps the audio into `BackchannelFrame` objects. These frames are then enqueued into a global queue. It's important to note that these frames can be one of two types:

\* **Playback frames:** These contain the actual audio data.

\* **Stop frames:** These have an empty payload and are used to signal the termination of an audio session, often triggered by the timeout mechanism mentioned earlier.

4\. **Audio Processing and Session Management:**

The `BackchannelWorker` continuously monitors the input queue. When a frame is dequeued, the worker takes over. Its primary responsibilities include:

\* **Decoding:** Using the IMP audio SDK, it decodes the audio data.

\* **Resampling:** If necessary, the audio is resampled to match the requirements of the output stage.

\* **Session Management:** The `BackchannelWorker` maintains the concept of a "current" session. This is vital for handling multiple potential clients or connection interruptions. It processes only frames belonging to the active session, discarding any frames from other or outdated sessions.

5\. **Audio Output:**

After decoding and resampling, the resulting audio is sent to a pipe. From this pipe, the `/bin/iac` (Ingenic Audio Client/Control) program, which is part of the [Ingenic Audio Daemon (IAD)](https://github.com/gtxaspec/ingenic-audiodaemon), picks up the data and handles the final audio output on the device.

The `IMPBackchannel` class underpins much of this, tasked with registering and managing audio decoders with the IMP audio SDK, as well as creating and destroying the IMP audio channels used for the decoding process.

## Key Development Considerations

Beyond the core architecture, several specific challenges and decisions shaped the final implementation, always with an eye on the tight flash memory budget:

### 1\. Keeping the AAC Decoder Lean with libhelix-aac

For AAC (Advanced Audio Coding) decoding, size was a major constraint, **especially critical given the 8MB flash target.** After evaluating options, I selected `libhelix-aac`, primarily due to its impressively small footprint.

To put this in perspective:

\* **libhelix-aac:** `128K`  
\* FAAD2: `460K`  
\* FDK-AAC: `1.2M`

The [libhelix-aac](https://github.com/earlephilhower/ESP8266Audio/blob/master/src/libhelix-aac/readme.txt) decoder, maintained by the ESP8266Audio project, offered the best balance of functionality and size. This choice was paramount to fitting advanced features like AAC support within the stringent memory limits. However, its upstream version doesn't provide a standard Makefile. This required me to implement some minor patches to adapt it from its Arduino-centric assumptions and then compile it as a library using some Buildroot magic.

### 2\. Managing Multiple Clients and Inactivity

A crucial aspect of a robust two-way audio system is handling scenarios like multiple clients attempting to stream audio or a user leaving a stream active and walking away. To address this, I implemented an activity monitoring system. If the audio stream becomes inactive for a certain duration (as detected by the `BackchannelSink`'s timeout), the system automatically "turns off" that stream. This frees up the audio channel, allowing another client to take control, ensuring fair and efficient use of the resource.

### 3\. Reusing the Ingenic Audio Daemon (IAD)

To streamline development and leverage existing robust components, I decided to reuse the Ingenic Audio Daemon (IAD) for the final audio output stage. The primary task was to handle the audio decoding. I was able to utilize codecs already available within the Ingenic SDK for PCM U-law (G.711 μ-law) and PCM A-law (G.711 A-law), and as mentioned, `libhelix-aac` for AAC. This approach allowed me to focus on the complexities of the backchannel pipeline without reinventing the wheel for audio playback, again, helping me manage the overall firmware size.

## Integration and Testing

I've put this new two-way audio functionality through its paces, and I'm pleased to report it has been successfully tested with popular NVR (Network Video Recorder) and home automation platforms like **Scrypted** and **Frigate** using both ONVIF and RTSP. Ensuring compatibility with these systems was a key goal to make this feature immediately useful for many of its users.

For those using Frigate, detailed instructions on how to configure and use the two-way audio feature can be found on the [Thingino Wiki](https://github.com/themactep/thingino-firmware/wiki/Integration:-Frigate).

* * *

Adding two-way audio to Thingino, especially with the challenge of supporting devices with only 8MB of flash, has been a demanding but rewarding endeavor. This new capability significantly enhances Thingino's interactivity, and I'm eager to see how users will leverage it with systems like Frigate and Scrypted. As always, feedback and contributions are welcome!