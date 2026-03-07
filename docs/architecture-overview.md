# EdgeTrack Architecture Overview

Author: Dmitrij Kostukovic
Project: EdgeTrack
First published: March 2026
License: Apache 2.0

---

## Introduction

EdgeTrack is an open tracking architecture designed for precise, low-latency spatial motion capture using stereo vision. The system focuses on transparency, deterministic timing behavior, and modular hardware integration.

Unlike many commercial tracking stacks that rely on closed firmware pipelines and ISP-processed image streams, EdgeTrack adopts a **RAW-first capture approach** combined with hardware synchronization and host-side processing. This allows the entire pipeline—from sensor capture to motion signals—to remain visible, controllable, and reproducible.

The goal of the project is not to build a single fixed product, but to define an **open architecture for stereo tracking systems** that can operate reliably on affordable hardware.

---

## Design Motivation

Typical off-the-shelf tracking solutions often rely on opaque processing pipelines. Camera sensors may pass through internal ISP processing, compression, and firmware layers before the data becomes accessible to the application. These stages introduce unpredictable latency, hidden filtering, and limited configurability.

EdgeTrack was developed to address these limitations by prioritizing:

* precise timing behavior through dedicated TDM-based synchronization
* primary operation with 850 nm or 940 nm NIR illumination, with optional passive operation when active illumination is not required
* direct access to RAW sensor data and low-level sensor control
* hardware-level synchronization across rigs
* modular multi-rig configurations
* host-side fusion and processing

This approach makes the full tracking pipeline easier to analyze, debug, and adapt for different research and development use cases.

---

## System-Level Architecture

At the system level, EdgeTrack follows a modular architecture that can operate in both single-rig and multi-rig configurations.

In its standard form, EdgeTrack follows a modular processing pipeline:

```
Stereo Camera Rigs
        ↓
RAW Capture
        ↓
Edge Processing
        ↓
ROI Processing
```

In multi-rig configurations, the outputs of one or more rigs are passed to the host system for fusion and higher-level processing:

```
Rig 1, Rig 2, Rig 3, ...
        ↓
CoreFusion (Host Processing)
        ├─→ Other Applications (e.g. Robotics or Teleoperation)
        └─→ Interaction Layers (e.g. MotionCoder)
```

Depending on the use case, ROI processing may be optional. In simpler configurations, RAW data can be sent directly to the host pipeline, where stereo processing can be performed either densely, if sufficient CPU or GPU resources are available, or selectively in regions of interest:
```
Stereo Camera Rigs
        ↓
RAW Capture
        ↓
CoreFusion (Host Processing)
        ├─→ Other Applications (e.g. Robotics or Teleoperation)
        └─→ Interaction Layers (e.g. MotionCoder)
```

Each stage of the pipeline has a clearly defined role and can be modified, extended, or replaced without changing the overall architectural concept.

---

## Core Design Principles

EdgeTrack is based on several architectural principles.

### RAW-first capture

Image data is captured directly from stereo sensors without ISP post-processing or compression. This preserves the full temporal and spatial information from the sensor and avoids hidden processing stages that could introduce latency, artifacts, or unpredictable behavior.

The processing concept focuses on controlled RAW pre-processing within an explicitly managed pipeline. This includes a zero-copy-oriented design wherever possible, primarily implemented in C and C++, with small performance-critical parts potentially implemented in assembly.

### Precise timing

Dedicated hardware synchronization ensures that all cameras operate with predictable and reproducible timing relationships. This makes it possible to correlate frames precisely across stereo pairs and across multiple rigs, which is essential for stable triangulation, fusion, and motion analysis. In configurations using active NIR illumination, synchronization becomes even more important, because exposure timing, strobe timing, and sensor readout can be coordinated in a controlled way. The architecture can also support phase-shifted operation between rigs, allowing different capture phases to be distributed over time. This can improve temporal sampling, reduce interference between illumination groups, and increase robustness for fast motion and multi-rig tracking scenarios.

### Modular stereo rigs

The architecture supports multiple independent stereo rigs distributed in space. Each rig captures local stereo observations that can later be combined at the host level.

### Host-side fusion

Instead of performing complex processing inside embedded camera hardware, EdgeTrack performs fusion and filtering on the host system. This enables more flexible algorithms, easier debugging, and scalable processing power.

### Transparent processing pipeline

All processing stages, from capture to motion interpretation, remain visible to the developer. The system avoids closed firmware pipelines and opaque vendor algorithms.

---

## Core Components

### EdgeTrack (capture layer)

EdgeTrack refers to the capture layer responsible for synchronized stereo acquisition. This layer handles sensor configuration, exposure control, and timing synchronization between cameras.

### CoreFusion (host fusion layer)

CoreFusion is responsible for combining observations from multiple stereo rigs. This stage performs tasks such as:

* multi-rig fusion
* outlier rejection
* temporal smoothing
* confidence weighting

The output of CoreFusion consists of stable fused representations such as 3D keypoints, structured motion signals, or other spatial data products.

These outputs can be used by higher-level motion layers, but they can also be consumed directly by other applications, such as SLAM, teleoperation, robotics, or custom host-side processing systems.

### Motion layers

Higher-level layers, such as MotionCoder, can interpret these signals as gestures, interaction commands, or other motion-based inputs.

These layers are intentionally separated from the tracking system itself, allowing the capture architecture to remain general-purpose.

---

## Multi-Rig Operation

EdgeTrack is designed to support multiple synchronized rigs observing the same workspace. This configuration enables several advantages:

* improved robustness against occlusion
* better spatial coverage
* improved measurement stability

Fusion algorithms on the host system combine observations from different viewpoints to produce stable motion outputs.

---

## Timing and Temporal Sampling

EdgeTrack also supports timing configurations in which different rigs operate with phase offsets. By distributing capture timing across multiple rigs, the system can increase the effective temporal sampling rate beyond the frame rate of a single camera.

This approach can improve motion tracking performance in situations involving rapid motion or transient events.

---

## Relation to Other Tracking Systems

EdgeTrack differs from many existing tracking systems in several ways.

Typical inside-out XR tracking systems rely on head-mounted cameras and SLAM pipelines tightly integrated with headset firmware.

EdgeTrack instead focuses on **externally mounted stereo rigs** operating in a deterministic and transparent capture pipeline. The architecture emphasizes modularity and open processing rather than tightly integrated device-specific stacks.

---

## Open Architecture

EdgeTrack is intended to be an open architecture rather than a closed product. Both hardware and software components are designed to be adaptable to different configurations and hardware platforms.

The system is released under open licenses to encourage experimentation, modification, and community development.

---

## Design and Features

### 1. Clear Separation of Capture and Processing

#### Problem

Native **MIPI CSI** camera interfaces offer excellent bandwidth, low latency, and precise timing, but they suffer from a major limitation: **very short cable lengths**, which makes larger or distributed camera setups impractical.

**USB-based** camera solutions allow longer cables, but USB is **interrupt-driven and host-dependent**, which typically results in **higher latency, increased jitter, and less deterministic timing**—especially problematic for tightly synchronized multi-camera systems.

**Ethernet/LAN-based** cameras improve distance and deployment flexibility, but many implementations still rely on **OS-level interrupts, buffering, and packet scheduling**, which are not inherently optimized for **hard real-time capture** or frame-accurate multi-camera synchronization. In practice, achieving truly deterministic behavior often requires a **real-time–tuned OS and network stack**, along with careful system-level optimization.

In addition, purpose-built **GigE / 2.5GigE machine-vision cameras** remain relatively expensive—often **€500+ per unit**—which makes large multi-camera arrays **cost-heavy** and limits scalability from a hardware-budget perspective.

At the high end, **CoaXPress** can deliver outstanding performance with direct, low-latency data paths into CPU/GPU memory. However, it comes with **very high hardware cost**, requires dedicated **frame grabbers**, and scales poorly **in terms of system cost and integration complexity**. Beyond the capture hardware itself, processing **multiple high-resolution cameras on a single host** can place a substantial load on the CPU/GPU—often pushing systems toward high-end workstation-class hardware (e.g., Threadripper-class systems) and significant engineering effort to optimize the processing pipeline.

#### Solution

EdgeTrack separates **image capture** from **high-level processing** by moving **reconstruction and preprocessing directly to the edge**. Instead of concentrating the entire workload on a single, expensive host system, each edge device performs its **local reconstruction tasks** using native MIPI CSI—where it performs best—and exports only **processed, compact 3D data** (e.g., keypoints, tool poses, sparse geometry) over the network.

This architecture preserves the **timing fidelity and signal quality** of native CSI capture while remaining **cost-efficient, scalable, and deployment-friendly**.

---

### 2. Ethernet-Native Alternative to CoaXPress

A **CoaXPress-based** camera infrastructure is a high-end solution in terms of **bandwidth and timing precision**, but it is **cost-intensive** and requires substantial **integration effort**, including dedicated **frame grabbers** and complex host-side pipelines.

Instead, comparable practical precision **for the target outputs** can be achieved through a combination of:

- **Well-defined multi-view geometry**
- **Calibrated stereo triangulation**
- **Edge-side preprocessing**
- **Early fusion in CoreFusion**

By transmitting **stable 3D primitives** (e.g., keypoints, tool poses, sparse geometry) rather than raw video streams, the system delivers **reproducible, low-jitter 3D signals** with significantly lower bandwidth requirements and reduced host-side complexity.

For the intended application, this architecture can **approach CoaXPress-class results** for **pose/keypoint accuracy and temporal stability**, while offering:

- **Simpler integration**
- **Lower hardware and maintenance costs**
- **Much better scalability**

Additional rigs can be added via **standard LAN connections**, rather than consuming limited frame-grabber channels and centralized capture resources.

---

### 3. TDM Phase-Offset Capture for Deterministic Timing

EdgeTrack uses **phase-offset global-shutter capture via Time-Division Multiplexing (TDM)**.
Instead of exposing all cameras simultaneously, multiple stereo rigs are triggered in **time-interleaved phases**.

This design:

* **Reduces occlusion**
* Improves **temporal consistency**
* Enables more **stable, repeatable input**

—especially important in **close-range, tool-centric workflows** where precision matters more than visual realism.

EdgeTrack is **markerless by default**, but supports **optional, minimal markers** when additional robustness is required.
Examples include subtle fingertip markers or markers placed directly on a tool. A “3D pencil” assisted by two small markers can provide **pen-like precision**, enabling reliable writing gestures or even **virtual keyboard interaction**.

A small **MCU-based trigger controller** generates deterministic, phase-shifted triggers for **up to eight stereo rigs** at **120 FPS per rig**.
When fused in **CoreFusion**, this results in an **effective aggregate update rate of up to ~960 Hz**, while maintaining **low jitter and high temporal stability**, depending on configuration and synchronization.


#### Why TDM Is Not Distributed Over LAN (and Why an MCU Can Still Make Sense)

A dedicated MCU (e.g., **RP2040**) is **not strictly required**—on a **Raspberry Pi 5**, TDM trigger signals can be generated locally using **hardware timers and/or DMA-driven GPIO** with sufficient precision for **120 FPS**.

The key distinction is **where timing is generated**:

* **Ethernet/LAN is excellent for data transport** (payload streaming, timestamps, configuration/control messages).
* But it is **not ideal as a real-time trigger bus**, because packet delivery depends on **OS scheduling, buffering, interrupts/NAPI, NIC behavior, and switch latency**, which introduces **variable jitter**.

Therefore, EdgeTrack uses **LAN for payload and timestamps**, while **TDM phase triggering is generated locally on each edge device** (Pi-side or MCU-side). If multiple edge devices must share a common phase reference, synchronization is handled via a **deterministic wired sync bus** (e.g., **RS-485**) or a **shared time base** (e.g., clock sync + scheduled start times), rather than sending **per-frame triggers** over the network.

> In short: **LAN transports data; the edge generates timing.**

### 4. Short Features

#### Stereo-First, Not AI-First

EdgeTrack uses **NIR stereo vision** as the primary tracking method. Depth is computed through **triangulation**, which means the system measures real geometry instead of guessing it. This makes the output predictable, repeatable, and easier to validate—especially important in professional workflows where stability matters more than “impressive demos.”

#### Designed for Short-Exposure Motion Freeze

The front-facing NIR illumination is not decorative. It enables **ultra-short exposure times** that freeze fast motion and reduce blur. This improves stereo correspondence, reduces noise, and stabilizes tracking under real-world movement—something many consumer-grade depth solutions struggle with.

#### MultiView as the Main Upgrade Path

Instead of relying on heavier models to “fix” failures, EdgeTrack scales through **MultiView geometry**. With **2–3 stereo rigs**, occlusions are reduced and robustness increases dramatically. In many scenarios, MultiView delivers reliability that would otherwise require complex AI, but without losing determinism.

#### Edge Processing, Minimal Data Output

EdgeTrack is designed to process data on the edge and export only what matters:

* 3D keypoints
* ROI point clouds
* compact geometric results

This reduces bandwidth, lowers latency, and keeps the system scalable for multi-sensor setups.

#### Optional AI as a Support Layer

AI can be added where it truly helps—such as plausibility checks, lightweight classification, or recovery in difficult edge cases. But AI is not the core. The core is a system you can measure, tune, and trust.

---