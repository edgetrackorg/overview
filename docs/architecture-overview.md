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

Image data is acquired in RAW form directly from the stereo sensors and processed in a controlled software pipeline, avoiding unnecessary ISP intervention and compression wherever possible. This architecture is intended to maintain temporal consistency, reduce avoidable latency, and provide deterministic control over the early image-processing stages.

The pipeline follows a zero-copy-oriented design where feasible and is implemented primarily in C and C++, with low-level optimization reserved for selected bottlenecks.

See more in the architecture documentation: [Architektur Pipeline](https://github.com/edgetrackorg/edgetrack/blob/main/docs/architecture_pipeline.md)

### Precise timing

Dedicated hardware synchronization ensures that all cameras operate with predictable and reproducible timing relationships. This makes it possible to correlate frames precisely across stereo pairs and across multiple rigs, which is essential for stable triangulation, fusion, and motion analysis. In configurations using active NIR illumination, synchronization becomes even more important, because exposure timing, strobe timing, and sensor readout can be coordinated in a controlled way. The architecture can also support phase-shifted operation between rigs, allowing different capture phases to be distributed over time. This can improve temporal sampling, reduce interference between illumination groups, and increase robustness for fast motion and multi-rig tracking scenarios.

See more in the documentation: [TDMStrobe](https://github.com/edgetrackorg/tdmstrobe)

### Modular stereo rigs

The architecture supports multiple independent stereo rigs distributed in space. Each rig captures local stereo observations that can later be combined at the host level.

### Host-side fusion

Instead of performing complex processing inside embedded camera hardware, EdgeTrack performs fusion and filtering on the host system. This enables more flexible algorithms, easier debugging, and scalable processing power.

See more in the documentation: [CoreFusion](https://github.com/edgetrackorg/corefusion)

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