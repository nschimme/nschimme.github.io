---
layout: post
date: 2026-07-23
title: "FAAC 2.0: A Free, Fast AAC Encoder Built for the Edge"
tags:
  - FAAC
  - Thingino
  - open source
---
FAAC has been around for a long time as a lightweight, dependency-free AAC encoder. What pushed a fresh 2.0 release wasn't a plan to chase Apple or Fraunhofer's quality crown — it was **Thingino**, an open embedded-Linux firmware project for IP cameras. Thingino uses FAAC to encode audio on-device, on hardware where every kilobyte of binary size and every CPU cycle is budgeted, and where a license with any legal ambiguity is a non-starter. Those constraints — footprint, speed, and clean licensing — are what actually drove FAAC 2.0. Better audio quality came along with it.

Here's what changed, why, and how it stacks up against Apple AAC, Fraunhofer FDK, and FFmpeg.

---

## 1. Clearing the License: 100% LGPL, and a New Codec Mode to Go With It

Older FAAC releases carried a legacy problem: despite the project calling itself LGPL, the codebase was actually a patchwork — some source files carried restrictive wording inherited from the original ISO/IEC MPEG reference code, and others were under GPL-3.0 or other terms entirely. A library marketed as LGPL but built from a mix of GPL and other-licensed code internally means a downstream project can't trust the label, and can end up with GPL obligations it never signed up for just by linking against it. Thingino, itself fully open source, needed to depend on FAAC without that risk.

So FAAC 2.0 rewrote every non-compliant module — stereo, quantize, channels, bitstream, filterbank, TNS, Huffman coding, and the MP4 output path — as original work under LGPL-2.1-or-later, and dropped the old MPEG reference-code preamble from the docs and license headers. Every source file is now consistently, unambiguously LGPL.

The rewrite wasn't just defensive, though. With clean ownership of the codec paths, FAAC 2.0 also added **HE-AAC v1** support — a mode that adds Spectral Band Replication (SBR) on top of the standard AAC-LC core, which meaningfully improves quality at low bitrates. That's a real capability upgrade delivered as a direct benefit of doing the license cleanup properly instead of working around the old code.

There's also a new "auto" mode that just picks the right encoding profile for you based on your target bitrate, so integrators don't need to understand AAC's internals to get a sensible result — you tell FAAC the bitrate you want, and it makes the reasonable choice.

Supporting HE-AAC's SBR mode meant the encoder now runs at two different sample rates internally, and the old API had no clean way to expose that to callers — on top of a fair amount of accumulated cruft from years of incremental changes. Rather than bolt SBR onto the old interface, FAAC 2.0 replaces it with a simpler one: set your options once when you open the encoder, get consistent error codes back, and let the library handle housekeeping that used to be the caller's job. It's a breaking change for existing integrations, but a much easier library to use correctly going forward.

---

## 2. Quieter Fixes That Add Up to a Higher Quality Floor

The pre-2.0 line had a reputation for the occasional harsh, glitchy-sounding frame. Most of that turned out to be plain bugs rather than an inherent limit of the design, and 2.0 fixes several of them:

- **Smarter stereo decisions, not a bigger stereo model.** The old joint-stereo mode leaned on one fixed rule for when to blend the left and right channels together. 2.0 makes that call independently for different frequency ranges instead of applying one rule to the whole track — so it can preserve stereo separation where it matters and simplify where it doesn't. A couple of related bugs got fixed along the way, including one that could occasionally cause dead silence in a channel on playback.
- **A more careful quantizer.** The quantizer — the part that decides how much detail to keep or discard per frame — had a few bugs letting quiet passages sound thinner than they should, and in rare cases producing a file a decoder would reject outright. Those are fixed.
- **A lookahead bug, fixed.** The encoder is supposed to look slightly ahead in the audio to anticipate sharp transients. A bug had shrunk that lookahead down to almost nothing, causing a smeared "pre-echo" sound before sharp hits and hurting stereo quality. That's restored and the buffering behind it was cleaned up.
- **Better compatibility.** A filter setting that could occasionally produce files some decoders (like FFmpeg and FAAD) would refuse to play is now capped to a safe range.
- **Faster math, same output.** The core FFT — the transform at the heart of every AAC encoder — now uses a more efficient algorithm, about 18% faster than before. This is an algorithmic change, not new SIMD/vector code.

---

## 3. Where FAAC 2.0 Stands Today

*(Note on benchmarks: results come from `[faac-benchmark](https://github.com/nschimme/faac-benchmark)`, the same GitHub Action-based harness that now gates every PR against regressions in CI — these aren't one-off marketing numbers, and the methodology is inspectable. All encoders below were run on the same Apple Silicon M1 machine: FAAC 2.0.0, fdk-aac 2.0.3, FFmpeg 8.1.2, and Apple AAC 26.5.2 (25F84). Full results are posted in [this discussion](https://github.com/nschimme/faac-benchmark/discussions/33), and each metric (MOS, Stereo Fidelity, etc.) is defined in [the benchmark's docs](https://github.com/nschimme/faac-benchmark/blob/master/docs/metrics.md).)*

### FAAC 2.0.0 vs. Apple AAC, FDK, and FFmpeg


| Rank | Encoder | Avg MOS (Quality) | Worst MOS Floor | Stereo Fidelity | Throughput Speed | Binary Footprint |
| ----- | ----------------- | ----------------- | ------------------------ | ----------------- | ------------------------- | ------------------------- |
| **1** | **Apple AAC** | **3.963** *(#1)* | 1.390 | 0.9360 | 111.9x | 353.3 KB |
| **2** | **aac-enc (FDK)** | 3.912 *(#2)* | 1.334 | 0.9548 | 159.6x | 1124.9 KB |
| **3** | **FAAC 2.0.0** | 3.876 *(#3)* | **2.165** *(Best Floor)* | **0.9755** *(#1)* | **544.9x** *(Best Speed)* | **116.0 KB** *(Smallest)* |
| **4** | **FFmpeg AAC** | 3.258 | 1.369 | 0.9460 | 40.6x | 431.4 KB |


### The Takeaways

1. **Speed & footprint domination:** At **544.9x real-time**, FAAC 2.0 is roughly 3.4x faster than FDK and 5x faster than Apple AAC, in a **116 KB** binary — a tenth the size of FDK's library.
2. **Quality that holds up, not just a footnote:** Apple and FDK still lead on average MOS, particularly at mid-bitrates (40k–64k stereo). But at 128k and above, FAAC 2.0 is competitive with — and in places ahead of — both, delivering effectively transparent audio without their CPU overhead. Given FAAC's constraints, that's the number that matters, not the overall average.

---

## 4. Proving Quality Was Actually Improving

Rewriting a codec's guts is easy to get wrong quietly — a change can sound fine on a couple of test clips and still be a regression across the board. So building FAAC 2.0 meant building a way to actually measure that, not just assert it: `faac-benchmark`, the same CI suite mentioned above, was created to run every candidate change against a fixed set of test material and catch quality regressions before they shipped.

Running it against the pre-2.0 line turned up a real one: stereo quality had quietly regressed at some point after the 1.30 release. That regression is fixed in 2.0.0, and it shows up directly in the numbers — **FAAC 2.0 has the best Stereo Fidelity score in the table, 0.9755**, ahead of both Apple AAC and FDK.

Apple and FDK both run heavier models that decide, moment to moment, how much they can blend or distort the left and right channels before a listener would notice, then spend the bits they save on making the rest of the track sound better. That's a real part of why they score higher on average quality, but it takes real CPU time and code complexity to do well. FAAC's stereo handling was already simple before 2.0 — that's part of what kept it fast and small — so the fix here was to make that existing simple approach smarter (the per-band decision described above), not to bolt on a heavier model. It keeps more of the original stereo image intact than the bigger encoders do, at the cost of a few tenths of a point in average quality score, since FAAC isn't reallocating those bits the way Apple and FDK do.

---

## 5. The Verdict: When Should You Reach for FAAC 2.0?

FAAC 2.0 is a precision-built encoder for environments where CPU budget, binary size, and license clarity are the actual requirements — exactly the brief Thingino needed filled for encoding audio on embedded IP cameras.

With clean LGPL licensing, a zero-dependency C architecture, a 116 KB footprint, and 544.9x encoding speed, FAAC 2.0 is a strong fit for:

- **Embedded Linux devices and IoT hardware** — cameras, recorders, and similar targets — where binary size and CPU budget are hard limits, not preferences.
- **Edge servers and real-time broadcast pipelines** where CPU cycles are scarce and shared across many concurrent streams.
- **High-bitrate archival encoding (128k–192k)**, where FAAC reaches perceptual transparency faster and lighter than any alternative here.

FAAC 2.0 is open source and available now at [github.com/knik0/faac](https://github.com/knik0/faac).