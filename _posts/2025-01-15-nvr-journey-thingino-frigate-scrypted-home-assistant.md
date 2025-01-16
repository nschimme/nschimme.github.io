---
layout: post
title: "My DIY NVR: From Raspberry Pi Aquarium Streams to Frigate AI Magic"
date: 2025-01-15
tags: [home automation, security, DIY, Frigate, Wyze, Scrypted, Thingino, Home Assistant]
youtube_id: TEYpFXql8hQ
---
<iframe style="height: auto; width: 100%; aspect-ratio: 16 / 9;" id="youtube_iframe" src="https://www.youtube.com/embed/{{ page.youtube_id }}?feature=oembed&amp;enablejsapi=1&amp;wmode=opaque" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe><a href="https://www.youtube.com/watch?v={{ page.youtube_id }}&amp;vq=hd720"></a>
What began as a fun experiment to stream my aquarium footage on a Raspberry Pi soon evolved into a full-blown exploration of smart home security. From simple motion JPEG streams to AI-driven object detection, this is the story of my journey to a custom NVR setup powered by cutting-edge tools like Frigate, Scrypted, and Home Assistant.

## Getting Started with a Raspberry Pi and Aquarium Streams

My first step into video monitoring was simple—streaming my aquarium using a Raspberry Pi and a basic webcam. Watching my fish remotely sparked curiosity about home video streaming’s potential for security. If I could observe fish, why not secure my front door?

I began by streaming to [YouTube](https://www.youtube.com/@thatguppyaquarium1900), learning how to manage video feeds and troubleshoot streaming issues. This small-scale project opened my eyes to the power of local video streaming.

## Unlocking RTSP and Motion Events with Dafang Hacks

When I grew curious about motion detection and automation, I upgraded to a Xiaomi Dafang camera and flashed it with [Dafang Hacks](https://github.com/EliasKotlyar/Xiaomi-Dafang-Hacks) firmware. This added RTSP support, enabling local live video streaming and bypassing cloud-based services for lower latency and better privacy.

To detect motion, I integrated [MotionEye](https://github.com/motioneye-project/motioneye) with Home Assistant. Although the system was functional, the complex interface didn’t pass the family usability test—my wife found it too cumbersome. It became clear I needed a more user-friendly alternative.

## A More User-Friendly Solution

My search for simplicity led me to Wyze cameras. Their easy-to-use app, affordable pricing, and hackability via Dafang Hacks made them ideal for my needs. To integrate Wyze cameras into my setup, I used [docker-wyze-bridge](https://github.com/mrlt8/docker-wyze-bridge) to stream RTSP video directly into MotionEye and Home Assistant.

However, my cloud-centric approach had a major drawback: Comcast’s data cap. Every camera feed was sent to the cloud and downloaded back, quickly exhausting my data allowance. Wyze’s RTSP beta firmware solved this by allowing direct, local streaming.

Around this time, I discovered [Scrypted](https://www.scrypted.app). It offered seamless HomeKit Secure Video (HKSV) integration with Wyze cameras, leveraging Apple’s ecosystem for privacy and ease of use. Scrypted replaced MotionEye in my setup, making the system more intuitive and family-friendly while integrating smoothly with Home Assistant.

## Thingino to the Rescue

Wyze’s decision to block RTSP support was a setback, but I wasn’t ready to give up. I turned to [Thingino](https://www.thingino.com), a custom firmware restoring RTSP functionality and enabling local streaming once again. With my growing number of cameras, my home server’s processing power was stretched thin. I needed a more efficient solution for motion detection.

Thingino’s ONVIF support allowed me to offload motion detection to the cameras themselves, reducing server load. I contributed to the Thingino project by enabling motion events over ONVIF and adding AAC and OPUS audio support, which enhanced both performance and flexibility.

## Frigate: Bringing AI-Powered Object Detection to the Table

The final piece of the puzzle was [Frigate](https://frigate.video), an AI-driven NVR solution that transformed my cameras into smart security devices. Frigate’s object detection identified people, vehicles, and animals, making the system proactive rather than reactive.

With Frigate integrated into Home Assistant, I automated actions based on AI events. Combining Thingino, Scrypted, Frigate, and Home Assistant resulted in a powerful, intelligent, and privacy-focused security system.

If you're interested in replicating my setup, check out my guide [here](https://github.com/themactep/thingino-firmware/wiki/Integration:-Frigate).

## Conclusion: A Fully Integrated Home Security System

Today, my home security system is a finely tuned machine. Wyze cameras running Thingino firmware stream locally, Scrypted provides HomeKit integration, and Frigate adds AI-powered object detection—all coordinated through Home Assistant. Looking back, what began with a simple aquarium stream on a Raspberry Pi has grown into a sophisticated, automated security solution.

Whether you’re just starting or looking to enhance your setup, the tools and platforms I’ve used offer endless possibilities. Explore, experiment, and build your smarter home!