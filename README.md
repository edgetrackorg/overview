# Overview – Multi-View Stereo Tracking System

## Description

EdgeTrack is an open multi-view tracking architecture based on RAW-first capture, precise timing, hardware synchronization, and host-side fusion. It is designed to provide deterministic and transparent processing for stereo and multi-rig tracking systems without relying on closed vendor pipelines.

The architecture can be applied to gesture interaction, 3D keypoint extraction, spatial input, robotics, teleoperation, and other motion-driven systems. This repository serves as the central overview and concept documentation for EdgeTrack, including architectural notes, design principles, and related system documents.

---

## Is this a novel architecture?

**Just copy and paste the following question into your AI assistant:**

```text
Is a stereo vision system built around actively controlled NIR illumination, homogeneous flood lighting with optional VCSEL, phase-shifted multi-camera synchronization, ROI-based processing focused on motion-critical regions such as hands, and a RAW-first zero-copy architecture that uses high-quality point clouds instead of fully dense depth maps, together with dual-resolution pipelines for both dense depth estimation and high-precision reconstruction on low-cost ARM hardware, considered a novel architecture? Are comparable systems already available on the market, or is this type of solution still largely missing?

And is such a geometry-based approach better suited for precise and reproducible 3D reconstruction in many applications compared to purely AI-based inference?
```

**Short note:**

A common limitation of classical stereo systems is performance on low-texture surfaces. This issue can be significantly reduced in many cases through appropriate polarization optics and optional use of VCSEL. In most indoor scenarios, however, severe texture-related problems are relatively rare.

---

## 📚 Documentation & Resources

| 🔗 **Link** + **Name**                                    | 📝 **Short Description**                                                                                                                                                                                                                     |
| --------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | 
| [Introduction](./docs/intro.md)                           | Comprehensive overview of stereo vision fundamentals: what a stereo camera is, how geometry works (baseline, disparity, FOV), and which architectural approach makes sense for different applications.                                       |
| [Architecture Overview](./docs/architecture-overview.md)  | System-level overview of the EdgeTrack tracking architecture, including RAW-first capture, precise timing with hardware synchronization, modular stereo rigs, ROI processing, and host-side fusion using CoreFusion.                         |
| [Performance Tradeoffs](./docs/performance-tradeoffs.md)  | Overview of practical stereo-vision trade-offs, including compute budget, RAW access, ROI processing, dense vs. sparse reconstruction, and hardware considerations from Raspberry Pi to workstation-class systems.                           |
| [Quest vs EdgeTrack](./docs/quest-vs-edgetrack.md)        | Comparison of consumer XR hand tracking and the EdgeTrack geometry-first stereo approach, including hardware differences, tracking principles, strengths, limitations, and cost considerations from prototype to potential mass production.  |
| [Comparison on the Market](./docs/comparison_table.md)    | Comparative overview of current market solutions, highlighting differences in sensing principles, system architecture, openness, synchronization, processing approach, and suitability for professional tracking workflows.                  |
| [Sensor](./docs/sensor-guide.md)                          | Sensor selection & integration guide: how to choose the right camera module (MIPI/RAW, global shutter, optics, filters, synchronization, etc.).                                                                                              |
| [Infrared](./docs/infrared.md)                            | Technical guide to controlled IR illumination: LED/VCSEL selection, driver design, optical filtering (bandpass, polarization), synchronization, and impact on stereo matching stability.                                                     |
| [LuxMeter](./docs/luxmeter.md)                            | Practical IR illumination sizing guide without a physical luxmeter: estimate required LED power using RAW10 intensity levels (paper target + fixed camera settings).                                                                         |
| [Vision Geometry Rules](./docs/geo_rules.md)              | Practical stereo geometry reference: baseline vs. distance, disparity range planning, accuracy estimation, and CPU workload considerations.                                                                                                  |
| [Redundancy](./docs/redundancy.md)                        | Notes on documentation overlap across GitHub repositories, including related concepts already published elsewhere and topics not yet included in the current overview repository.                                                            |

---

## 🎥 Layer 1 – Capture

**What this layer does:**
This layer **captures raw camera data and hand-tracking signals**, performs **on-edge preprocessing**, and ensures **precise multi-device synchronization**.

It outputs **RAW streams, normalized keypoints, and ROI metadata**.
If optional AI pipelines are enabled, it can additionally provide **H.265 preview streams** or run **lightweight semantic inference directly on the edge** (e.g., via an NPU).

---

### 🔓 Open Source Modules

| 🧩 **Module** | 📝 **Short Description**                                                                                                                 |  ⚖️ **License** |  🚦 **Status**  | 🔗 **Link**                                            |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | --------------- |  -------------- | ------------------------------------------------------ |
| **EdgeTrack** | **RAW10 mono capture pipeline** running on ARM-based systems (e.g., Raspberry Pi, Jetson), designed for deterministic stereo acquisition |  Apache-2.0     |  🟡 In progress | [EdgeTrack](https://github.com/edgetrackorg/edgetrack) |
| **TDMStrobe** | **Time-division multiplexed IR illumination and trigger system** with phase control (A/B/C/D) for precise multi-camera synchronization   |  Apache-2.0     |  🟡 In progress | [TDMStrobe](https://github.com/edgetrackorg/tdmstrobe) |
| **EdgeSense** | Optional **AI-based semantic segmentation and scene understanding**, running alongside geometry-first capture pipelines                  |  Apache-2.0     |  🟡 Planned     | coming soon                                            |

---

### 🔒 Non-Open Source / Future Modules

| 🧩 **Module**     | 📝 **Short Description**                                                                                                                                                               |  ⚖️ **License**                                   | 🚦 **Status** | 🔗 **Link** |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |  ------------------------------------------------ | ------------- | ----------- |
| **EdgeTrack Pro** | **RAW10 stereo capture and processing pipeline** on a custom FPGA platform, optimized for **ultra-low latency**, **high performance**, and **hardware-accelerated disparity matching** |  Commercial (planned, potentially partially open) | 🔴 Later      | —           |

---

### 📝 Note: EdgeTrack vs EdgeTrack Pro

**EdgeTrack is not an inferior version of EdgeTrack Pro.**
When properly configured, EdgeTrack already provides a practical and capable geometry-first solution for many applications.

**EdgeTrack Pro** is a future FPGA-based variant focused on **lower latency**, **higher determinism**, and **stronger real-time performance**. It is intended as a specialized hardware option, not as a general replacement for EdgeTrack.

At present, **EdgeTrack is the main directly usable solution**.
**EdgeTrack Pro** is primarily a future internal development direction. See the comparison table below.

**Architecture Comparison (Estimated)**

| Variant                               | Estimated End-to-End Latency |         Jitter | 120 FPS Feasible?                                                  | Compute Headroom       | Development Effort | Strength                                       | Weakness                                                           |
| ------------------------------------- | ---------------------------: | -------------: | ------------------------------------------------------------------ | ---------------------- | ------------------ | ---------------------------------------------- | ------------------------------------------------------------------ |
| **2× Camera + ARM (e.g., Pi 5)**      |                 **12–30 ms** | medium to high | limited                                                            | low to medium          | low                | Fastest to start, highly flexible              | CPU can become a bottleneck                                        |
| **2× Camera + FPGA (on-device)**      |                  **3–10 ms** |       very low | yes, most reliable                                                 | high (streaming / ROI) | very high          | Best responsiveness, deterministic             | Highest development complexity                                     |
| **2× Camera + ARM → Host processing** |                  **8–20 ms** |         medium | yes, better than ARM-only                                          | high (host-side)       | medium             | Best practical compromise                      | Network path and synchronization add complexity                    |
| **2× USB Cameras → Host processing**  |               **50–200+ ms** | medium to high | limited to possible, depending on camera, driver, and USB pipeline | high (host-side)       | low to medium      | Very easy to prototype with commodity hardware | Higher latency, less deterministic timing, USB and driver overhead |

> ⚠️ Note: These values are **not measured benchmarks**, but **realistic engineering estimates** based on system architecture, data flow, and typical performance characteristics.

---

## ⚙️ Layer 1.5 – Host-side Stereo Compute (Optional)

**What this layer does:**
This layer is **fully optional** and only required when **computationally heavy stereo processing** is needed.

Instead of performing stereo reconstruction on the edge, RAW data is streamed to a host PC where **dense or ROI-based disparity/depth computation** is executed before forwarding results to the fusion layer.

👉 Typical use case:

* High-resolution **dense depth**
* Advanced filtering / confidence maps
* GPU-accelerated stereo pipelines

👉 Example pipeline:

```
EdgeTrack 1 → Ethernet stream (2× RAW stereo + optional RGB) → PC 1 (CoreStereo)
EdgeTrack 2 → Ethernet stream (2× RAW stereo + optional RGB) → PC 2 (CoreStereo)
EdgeTrack 3 → Ethernet stream (2× RAW stereo + optional RGB) → PC 3 (CoreStereo)
EdgeTrack 4 → Ethernet stream (2× RAW stereo + optional RGB) → PC 4 (CoreStereo)

All CoreStereo outputs → CoreFusion (workstation)
```

If not needed, this layer can be **completely skipped**, and data can be sent directly to Layer 2.

| 🧩 **Module**  | 📝 **Short Description**                                                                                                                                                                                                      | 🖥️ **Host**                                                    | ⚖️ **License** | ⚠️ **Notes**   | 🚦 **Status** | 🔗 **Link** |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------- | -------------- | -------------- | ------------- | ----------- |
| **CoreStereo** | Host-side stereo processing module: ingests **synchronized RAW or rectified stereo streams** and performs **disparity/depth reconstruction** (dense or ROI-based), including optional **filtering and confidence estimation** | High-performance CPU or GPU (e.g., AMD Threadripper / CUDA GPU) | Apache-2.0     | Optional layer | 🟡 Planned    | coming soon |

---

## 🔗 Layer 2 – Multi-View Fusion

**What this layer does:**
This layer runs on a host system and performs **multi-view spatial fusion**.

It aggregates multiple stereo rigs, applies **time synchronization**, **calibration refinement**, and **bundle adjustment**, and produces **stable, structured spatial outputs**.

Outputs include:

* 3D keypoints / skeletons
* Dense or sparse depth
* Motion signals
* Structured spatial representations

These outputs are designed for direct use in:

* Robotics
* Teleoperation
* SLAM / mapping
* Spatial input systems
* Gesture-based interaction

| 🧩 **Module**  | 📝 **Short Description**                                                                                                                                                                                                   | 🖥️ **Host**                                  | ⚖️ **License** | ⚠️ **Notes** | 🚦 **Status** | 🔗 **Link**                                              |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------- | -------------- | ------------ | ------------- | -------------------------------------------------------- |
| **CoreFusion** | Aggregates **2–4 synchronized stereo rigs** over LAN; performs **multi-view calibration**, **bundle adjustment**, **outlier rejection**, and **low-latency fusion** to produce **stable 3D keypoints and spatial signals** | Host PC (GPU recommended), ZeroMQ / UDP / TCP | Apache-2.0     | —            | 🟡 Planned    | [CoreFusion](https://github.com/edgetrackorg/corefusion) |

---

## 🧠 Layer 3 – Motion Interpretation (Optional)

**What this layer does:** It converts **poses/keypoints** into **high-level intents** using **gesture grammars**, **state machines**, and **context rules** (tool modes, constraints, safety). It handles **debounce**, **disambiguation**, and **confidence scoring**, producing **deterministic, low-latency events**.

| 🧩 **Module**        | 📝 **Short Description**                                 | 🔁 **I/O**                  | ⚖️ **License** | ⚠️ **Notes** | 🚦 **Status**  | 🔗 **Link**                                                                   |
| -------------------- | --------------------------------------------------------- | ---------------------------- | -------------- | ------------ | -------------- | ------------------------------------------------------------------------------ |
| **MotionCoder**      | Real-time gestures/intents, state machine, context logic. | Poses/keypoints from Layer 2 | Apache-2.0     | —            | 🟡 Planned     | [MotionCoder](https://github.com/xtanai/motioncoder) |

---

##  🕹️ Peripherals (Optional)

**What this layer does:** Purpose-built devices that **improve ergonomics and precision** (e.g., clutch/confirm, mode switches, haptic cues). They speak **BLE/USB** and avoid IR emission to stay **camera-safe** in NIR setups.

> Note: These peripherals **don’t require MotionCoder**. They work like **standard input devices** (e.g., HID) and can be used independently.

| 🧩 **Module**   | 📝 **Short Description**                                                | 🔌 **Hardware / Deps**                                                                                            | ⚖️ **License** | ⚠️ **Notes**                                                                                                            | 🚦 **Status** | 🔗 **Link**                                    |
| ---------------- | ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ | -------------- | --------------------------------------------------------------------------------------------------------------------- | ------------- | ---------------------------------------------- | 
| **Pen3D**        | **Tracked 3D pen input with buttons and optional haptics.**            | Optional ESP32-S3 (BLE) or mechanical.                                                                             | Apache-2.0     | BLE GATT (notify); deep-sleep wake-on-button; optional USB-CDC debug. Designed to coexist with 850 nm NIR tracking.   | 🟡 Planned    | [Pen3D](https://github.com/xtanai/pen3d)       |
| **HMDone**       | **Minimal VR headset with external marker-based tracking only.**       | Works with high-resolution HMDs (e.g. Pimax Crystal or Valve). No extra hand controllers required for MotionCoder. | Apache-2.0     | Use with multi-view/NIR rigs for best results; inside-out tracking is intentionally ignored.                          | 🟠 Later      | [HMDone](https://github.com/xtanai/hmdone)     |

---

## 🗺️ Roadmap

Coming soon. The project is currently in the research and prototyping phase. 🚀

---