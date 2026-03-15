# Overview – Multi-View Stereo Tracking System

## Description

EdgeTrack is an open multi-view tracking architecture based on RAW-first capture, precise timing, hardware synchronization, and host-side fusion. It is designed to provide deterministic and transparent processing for stereo and multi-rig tracking systems without relying on closed vendor pipelines.

The architecture can be applied to gesture interaction, 3D keypoint extraction, spatial input, robotics, teleoperation, and other motion-driven systems. This repository serves as the central overview and concept documentation for EdgeTrack, including architectural notes, design principles, and related system documents.

---

## 📚 Documentation & Resources

| 📔 **Name**                  | 📝 **Short Description**                                                                                                                                                                                                                     | ⚖️ **License** | 🔗 **Link**                                               |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------- | --------------------------------------------------------- |
| **Introduction**             | Comprehensive overview of stereo vision fundamentals: what a stereo camera is, how geometry works (baseline, disparity, FOV), and which architectural approach makes sense for different applications.                                       | Apache-2.0     | [Introduction](./docs/intro.md)                           |
| **Architecture Overview**    | System-level overview of the EdgeTrack tracking architecture, including RAW-first capture, precise timing with hardware synchronization, modular stereo rigs, ROI processing, and host-side fusion using CoreFusion.                         | Apache-2.0     | [Architecture Overview](./docs/architecture-overview.md)  |
| **Performance Tradeoffs**    | Overview of practical stereo-vision trade-offs, including compute budget, RAW access, ROI processing, dense vs. sparse reconstruction, and hardware considerations from Raspberry Pi to workstation-class systems.                           | Apache-2.0     | [Performance Tradeoffs](./docs/performance-tradeoffs.md)  |
| **Quest vs EdgeTrack**       | Comparison of consumer XR hand tracking and the EdgeTrack geometry-first stereo approach, including hardware differences, tracking principles, strengths, limitations, and cost considerations from prototype to potential mass production.  | Apache-2.0     | [Quest vs EdgeTrack](./docs/quest-vs-edgetrack.md)        |
| **Comparison on the Market** | Comparative overview of current market solutions, highlighting differences in sensing principles, system architecture, openness, synchronization, processing approach, and suitability for professional tracking workflows.                  | Apache-2.0     | [Comparison on the Market](./docs/comparison_table.md)    |
| **Haptics Tradeoffs**        | Analysis of force-feedback and haptic system trade-offs, including cost, complexity, scalability, ergonomics, and why a non-haptic workflow can be the more practical choice for desktop 3D authoring and professional interaction.          | Apache-2.0     | [Haptics Tradeoffs](./docs/haptics-tradeoffs.md)          |
| **Sensor Guide**             | Sensor selection & integration guide: how to choose the right camera module (MIPI/RAW, global shutter, optics, filters, synchronization, etc.).                                                                                              | Apache-2.0     | [Sensor](./docs/sensor-guide.md)                          |
| **Infrared**                 | Technical guide to controlled IR illumination: LED/VCSEL selection, driver design, optical filtering (bandpass, polarization), synchronization, and impact on stereo matching stability.                                                     | Apache-2.0     | [Infrared](./docs/infrared.md)                            |
| **LuxMeter**                 | Practical IR illumination sizing guide without a physical luxmeter: estimate required LED power using RAW10 intensity levels (paper target + fixed camera settings).                                                                         | Apache-2.0     | [LuxMeter](./docs/luxmeter.md)                            |
| **GeoRules**                 | Practical stereo geometry reference: baseline vs. distance, disparity range planning, accuracy estimation, and CPU workload considerations.                                                                                                  | Apache-2.0     | [Vision Geometry Rules](./docs/geo_rules.md)              |
| **Redundancy**               | Notes on documentation overlap across GitHub repositories, including related concepts already published elsewhere and topics not yet included in the current overview repository.                                                            | Apache-2.0     | [Redundancy](./docs/redundancy.md)                        |

---

## 🎥 Layer 1 – Capture

**What this layer does:** This layer **ingests camera/hand-tracking streams**, performs **on-edge preprocessing**, and **synchronizes** frames across devices. It outputs **normalized pose/keypoint streams and ROIs**; if optional AI pipelines are enabled, it can also publish an **H.265 video stream** for preview/segmentation or run **semantic inference directly on the edge** (e.g., via an NPU accelerator).

| 🧩 **Module**      | 📝 **Short Description**                                                                                                                                                      | 🔌 **Hardware / Deps**                                                       | ⚖️ **License** | ⚠️ **Notes**                                                            | 🚦 **Status**         | 🔗 **Link**                                             |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------- | -------------- | ----------------------------------------------------------------------- | --------------------- | ------------------------------------------------------ |
| **EdgeTrack**      | **RAW10 mono ingest** on Pi 5; GPU/NEON-optimized preproc                                                                                                                      | **Raspberry Pi 5** (4/8 GB), 2× MIPI-CSI (OV9281 etc.)                       | Apache-2.0     | —                                                                       | 🟡 In progress        | [EdgeTrack](https://github.com/edgetrackorg/edgetrack) |
| **TDMStrobe**      | **Time-Division-Multiplexed IR strobe & camera trigger** (phase control A/B/C/D) for EdgeTrack                                                                                 | IR-LED/VCSEL arrays, LED-drivers, MCU (RP2040)                               | Apache-2.0     | —                                                                       | 🟡 In progress        | [TDMStrobe](https://github.com/edgetrackorg/tdmstrobe) |
| **EdgeSense**      | AI semantic segmentation & scene understanding on edge (optional pipeline beside geometry-based capture)                                                                       | RGB Camera, optional NPU/AI accelerator                                      | Apache-2.0     | —                                                                       | 🟡 Planned            | coming soon                                            |

---

## 🔗 Layer 2 – Host-side fusion

**What this layer does:** It runs on the host PC and sits between capture and higher-level interpretation. It aggregates 2–4 stereo pairs over LAN, performs time synchronization, multi-view calibration refinement, bundle adjustment, and low-latency fusion/filtering to produce stable spatial outputs such as 3D keypoints, poses, dense depth, or structured motion signals. These outputs can be passed directly to your application, for example robotics, teleoperation, SLAM, spatial input, or gesture-based interaction systems.

| 🧩 **Module**   | 📝 **Short Description**                                                                                                                                                                                                                                      | 🖥️ **Host**                                                | ⚖️ **License** | ⚠️ **Notes**  | 🚦 **Status** | 🔗 **Link**                                                |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -----------------------------------------------------------|--------------- | -------------- | ------------- | ---------------------------------------------------------- |
| **CoreFusion**  | Aggregates **2–4 synchronized stereo rigs** over LAN; performs **multi-view calibration**, **bundle adjustment**, **outlier rejection**, and **low-latency fusion** to produce **stable 3D keypoints / keyposes** (joints + confidences + reference anchors). | Host PC (CUDA-capable GPU recommended); ZeroMQ / UDP / TCP | Apache-2.0     | —              | 🟡 Planned    | [CoreFusion](https://github.com/edgetrackorg/corefusion)   |
| **CoreStereo**   | Host-side stereo compute layer: ingests **synchronized RAW/rectified stereo frames** from EdgeTrack and performs **disparity / depth reconstruction** (dense or ROI-based), plus optional **filters and confidence maps** for downstream modules.             | High-performance CPU example: AMD Threadripper             | Apache-2.0     | —              | 🟡 Planned    | coming soon                                                | 

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