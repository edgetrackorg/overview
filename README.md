# Overview – Multi-View Stereo Tracking System

## Description

EdgeTrack is an open multi-view tracking architecture built around RAW-first capture, precise timing, hardware synchronization, and host-side fusion. It is designed as a transparent and deterministic foundation for stereo and multi-rig tracking systems, without depending on closed vendor pipelines.

The architecture itself is application-independent and intentionally kept general-purpose. It can serve as a pure technical foundation for a wide range of use cases, including **gesture interaction, 3D keypoint extraction, spatial input, robotics, teleoperation, and more**.

This repository serves as the central overview and concept documentation for EdgeTrack, including architectural notes, design principles, and related system documents.

Alongside classical stereo pipelines, EdgeTrack may also support optional neural stereo methods for multi-view processing, including acceleration on GPU-based hardware where appropriate. These AI-assisted components are optional and complement the core geometry-first architecture rather than replacing it.

EdgeTrack is hardware-agnostic and designed to remain highly flexible across different system classes, including ARM or x86 platforms, industrial cameras, and camera modules such as MIPI CSI.

---

## 📚 Documentation & Resources

comming soon!!!

---

## ⏱️ Layer 1 – Timing

**What this layer does:** 

This layer provides the **timing backbone** of the system. It controls **trigger distribution, phase sequencing, and synchronized IR illumination** across one or more camera rigs, enabling deterministic capture timing and stable multi-device operation.

| 🧩 **Module** | 📝 **Short Description**                                                                                                                 | ⚖️ **License**  | 🚦 **Status**  | 🔗 **Link**                                            |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | --------------- | -------------- | ------------------------------------------------------ |
| **TDMStrobe** | **Time-division-multiplexed IR illumination and trigger system** with phase control (A/B/C/D) for precise multi-camera synchronization   | Apache-2.0      | 🟡 In progress | [TDMStrobe](https://github.com/edgetrackorg/tdmstrobe) |

---

## 🎥 Layer 2 – Capture

**What this layer does:**  

This layer handles **sensor-side image acquisition and edge-side preprocessing**. It captures **raw camera data**, prepares it for downstream stereo or fusion stages, and preserves **precise timing alignment** with the timing layer.

Depending on configuration, it can output **RAW streams, ROI metadata, preview streams, or lightweight edge-side inference results**.

| 🧩 **Module** | 📝 **Short Description**                                                                                                                 | ⚖️ **License**  | 🚦 **Status**  | 🔗 **Link**                                            |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | --------------- | -------------- | ------------------------------------------------------ |
| **EdgeTrack** | **RAW10 mono capture pipeline** running on ARM-based systems (e.g., Raspberry Pi, Jetson), designed for deterministic stereo acquisition | Apache-2.0      | 🟡 In progress | [EdgeTrack](https://github.com/edgetrackorg/edgetrack) |

When using industrial cameras with direct host output, this layer 2 can be omitted entirely, and the image data can be forwarded directly to Layer 3.

---

## ⚙️ Layer 3 – Host-side Stereo Compute

**What this layer does:**

For ARM-based edge nodes, this layer is **fully optional** and only needed when **computationally heavy stereo processing** is required.
For base industrial-camera setups, however, this layer is typically **required**.

Instead of performing stereo reconstruction directly on the edge device, RAW data is streamed to a host PC, where **dense or ROI-based disparity and depth computation** is executed before the results are forwarded to the fusion layer.

| 🧩 **Module**  | 📝 **Short Description**                                                                                                                                                                                                      |  ⚖️ **License** | 🚦 **Status** | 🔗 **Link**                                               |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |  -------------- | ------------- | --------------------------------------------------------- |
| **CoreStereo** | Host-side stereo processing module: ingests **synchronized RAW or rectified stereo streams** and performs **disparity/depth reconstruction** (dense or ROI-based), including optional **filtering and confidence estimation** |  Apache-2.0     | 🟡 Planned    | [CoreStereo](https://github.com/edgetrackorg/corestereo)  |

If not needed, this layer 3 can be **completely skipped**, and data can be sent directly to Layer 4.

---

## 🔗 Layer 4 – Multi-View Fusion

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

| 🧩 **Module**  | 📝 **Short Description**                                                                                                                                                                                                   | ⚖️ **License** | 🚦 **Status** | 🔗 **Link**                                              |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------- | ------------- | -------------------------------------------------------- |
| **CoreFusion** | Aggregates **2–4 synchronized stereo rigs** over LAN; performs **multi-view calibration**, **bundle adjustment**, **outlier rejection**, and **low-latency fusion** to produce **stable 3D keypoints and spatial signals** |  Apache-2.0    | 🟡 Planned    | [CoreFusion](https://github.com/edgetrackorg/corefusion) |

---

## 🧠 Layer 5 – Motion Interpretation (Optional)

**What this layer does:** 

It converts **poses/keypoints** into **high-level intents** using **gesture grammars**, **state machines**, and **context rules** (tool modes, constraints, safety). It handles **debounce**, **disambiguation**, and **confidence scoring**, producing **deterministic, low-latency events**.

| 🧩 **Module**        | 📝 **Short Description**                                  | ⚖️ **License** | 🚦 **Status**  | 🔗 **Link**                                                                   |
| -------------------- | --------------------------------------------------------- | -------------- |--------------- | ------------------------------------------------------------------------------ |
| **MotionCoder**      | Real-time gestures/intents, state machine, context logic. |  Apache-2.0    | 🟡 Planned     | [MotionCoder](https://github.com/xtanai/motioncoder)                           |

---

##  🕹️ Peripherals (Optional)

**What this layer does:** 

Purpose-built devices that **improve ergonomics and precision** (e.g., clutch/confirm, mode switches, haptic cues). They speak **BLE/USB** and avoid IR emission to stay **camera-safe** in NIR setups.

> Note: These peripherals **don’t require MotionCoder**. They work like **standard input devices** (e.g., HID) and can be used independently.

| 🧩 **Module**   | 📝 **Short Description**                                                |  ⚖️ **License** | 🚦 **Status** | 🔗 **Link**                                    |
| ---------------- | ---------------------------------------------------------------------- | --------------- | ------------- | ---------------------------------------------- | 
| **Pen3D**        | **Tracked 3D pen input with buttons and optional haptics.**            |  Apache-2.0     | 🟡 Planned    | [Pen3D](https://github.com/xtanai/pen3d)       |
| **HMDone**       | **Minimal VR headset with external marker-based tracking only.**       |  Apache-2.0     | 🟠 Later      | [HMDone](https://github.com/xtanai/hmdone)     |

---

## 🛠️ Example Use Cases

**What this layer does:**

This layer shows how the stack can be applied across different domains, including **robot grippers, motion capture, 3D scanning, robotics, and workspace perception**.

> Note: These are example application areas built on top of the stack. More use cases will be added over time.

| 🧩 **Module**     | 📝 **Short Description**                                                                                      | ⚖️ **License** | 🚦 **Status** | 🔗 **Link**                                                  |
|-------------------|---------------------------------------------------------------------------------------------------------------|----------------|----------------|-------------------------------------------------------------|
| **MoCap**         | **Marker-based motion capture for tracking body, hand, or object movement in 3D space.**                      | Apache-2.0     | 🟠 Later       | [MoCap](https://github.com/edgetrackorg/mocap)              |
| **3DScan**        | **Multi-view 3D scanning for geometry capture, reconstruction, and measurement.**                             | Apache-2.0     | 🟠 Later       | [3DScan](https://github.com/edgetrackorg/3dscan)            |
| **PerceptGrid**   | **External multi-camera perception layer for robots, safety monitoring, and shared workspace understanding.** | Apache-2.0     | 🟠 Later       | [PerceptGrid](https://github.com/edgetrackorg/perceptgrid)  |

More modules and application areas will be added over time.

---

## 🗺️ Roadmap

Coming soon. The project is currently in the research and prototyping phase. 🚀

---