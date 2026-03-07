# Introduction and Motivation

## What’s Different

Born out of frustration with the limitations of typical off-the-shelf systems, **EdgeTrack** introduces a tracking architecture designed for determinism, robustness, and cost efficiency. The system is built around a **RAW-first capture pipeline**, **hardware synchronization**, and **host-side fusion**, keeping the entire pipeline transparent and controllable.

By avoiding ISP-processed or compressed camera streams, timing and processing behavior remain predictable. On the host side, **CoreFusion** aggregates multiple rigs, rejects outliers, smooths motion, and processes ROI-based stereo data to produce stable outputs such as **3D keypoints** and **structured motion signals** for higher-level layers like MotionCoder.

With **multi-rig fusion** and **phase-offset timing**, the system can increase the effective temporal sampling rate beyond the frame rate of a single camera, enabling very high effective FPS for fast motion.

The result is **robust, low-latency tracking on affordable hardware without vendor lock-in**.

---

## What is NIR?

EdgeTrack primarily uses **synchronized near-infrared (NIR) illumination** at **850 nm or 940 nm** to improve stereo reconstruction stability.

NIR illumination:

* increases surface contrast
* reduces sensitivity to ambient light
* allows controlled lighting conditions
* improves stereo matching reliability

In combination with global-shutter sensors and synchronized exposure control, NIR illumination enables **stable geometry-based 3D perception**.

---

## System Architecture

### Ecosystem Overview

The EdgeTrack ecosystem consists of several modular components:

```
EdgeTrack   – edge-side stereo capture (RAW-first, synchronized)
EdgeSense   – optional edge-side AI assist layer
CoreStereo  – optional host-side stereo processing
CoreFusion  – host-side multi-rig fusion
TDMStrobe   – deterministic timing and illumination control
MotionCoder – optional gesture and interaction layer
```

The project is **fully open source under the Apache License**.
The goal is not to create a black-box product, but a **transparent and extensible spatial tracking system**.

---

### What is EdgeTrack?

EdgeTrack is a **modular edge-side tracking system** based on **physical NIR stereo vision**.

Each EdgeTrack unit:

* captures synchronized global-shutter stereo images
* provides direct RAW access
* performs local stereo reconstruction
* outputs metric 3D data such as keypoints or ROI point clouds

The design prioritizes **stable reconstruction**, **low latency**, and **predictable timing**.

EdgeTrack can operate:

* as a **single stereo unit**
* or as part of a **multi-rig system** covering larger volumes

Multi-rig configurations reduce occlusions and improve tracking stability using **multi-view geometry**.

**Mechanical analogy:**
EdgeTrack is the **piston** — it generates the primary force of the system.

---

### What is EdgeSense?

EdgeSense is the **optional AI assist layer** in the EdgeTrack ecosystem.

It **does not replace stereo geometry**. Instead, it improves robustness on top of the geometry-based pipeline.

Typical tasks include:

* confidence estimation
* ambiguity resolution
* failure detection
* lightweight classification

In difficult situations — such as low texture, fast motion, partial occlusion, or difficult lighting — EdgeSense can help stabilize tracking.

In multi-view setups, geometric redundancy often reduces the amount of AI assistance required.

EdgeSense can run on available **GPU or NPU hardware**, such as:

* Raspberry Pi AI HAT+
* Radxa AI platforms

**Mechanical analogy:**
EdgeSense is the **turbocharger** — it increases performance when needed.

---

### What is CoreStereo?

CoreStereo is an **optional host-side stereo computation layer**.

While EdgeTrack can compute disparity locally (ROI-based or sparse), CoreStereo allows higher compute workloads such as:

* dense disparity
* extended disparity ranges
* experimental stereo pipelines

CoreStereo processes synchronized stereo frames streamed from EdgeTrack and performs reconstruction on **workstation-class hardware** such as CUDA GPUs or high-core-count CPUs.

**Mechanical analogy:**
CoreStereo is like a **larger engine block** — the same principles with more computational horsepower.

---

### What is CoreFusion?

CoreFusion is the **host-side fusion layer**.

Each EdgeTrack unit produces synchronized local 3D observations. CoreFusion merges these into a single consistent spatial model.

CoreFusion performs:

* multi-view triangulation
* cross-rig alignment
* outlier rejection
* temporal filtering

The output is a **stable, time-consistent 3D pose stream**.

This transforms EdgeTrack from a single device into a **distributed perception system over Ethernet**.

**Mechanical analogy:**
CoreFusion is the **crankshaft** — combining multiple pistons into smooth power.

#### Transport Interface

EdgeTrack and EdgeSense connect to CoreFusion via **wired Ethernet**.

Wired networking ensures:

* stable bandwidth
* predictable latency
* reliable synchronization

For timing-critical systems, this approach is more reliable than USB-based camera pipelines.

---

### What is TDMStrobe?

TDMStrobe is the **hardware timing and illumination control system**.

TDM stands for **Time Division Multiplexing**.

Instead of continuous illumination, the system uses **short synchronized pulses** to:

* reduce motion blur
* maximize illumination efficiency
* avoid cross-rig interference

In multi-rig setups, rigs operate in **separate timing slots**, preventing optical interference.

**Mechanical analogy:**
TDMStrobe is the **camshaft** — precise timing control is essential.

---

### What is MotionCoder?

MotionCoder converts **time-consistent 3D keypoints** into structured gesture commands for deterministic interaction with digital tools.

Unlike consumer hand-tracking systems that prioritize visual realism, MotionCoder focuses on:

* spatial accuracy
* timing consistency
* reproducibility

This enables controlled, repeatable interaction in software environments such as Blender, Unreal Engine, and other VR authoring tools.

**Mechanical analogy:**
MotionCoder is the **cockpit**. Precise control requires a stable and balanced engine underneath - like a radial engine with multiple cylinders working in harmony. A single-cylinder setup may run, but it cannot deliver the same smooth, consistent performance.

More detailed documentation is available at [xtanai/overview](https://github.com/xtanai/overview).

---

## Data Flow

### EdgeTrack (edge device)

* rectification
* ROI tracking
* disparity computation inside ROI
* optional RAW streaming

### EdgeSense (optional edge AI)

* confidence estimation
* ambiguity resolution
* tracking quality monitoring

### CoreStereo (optional host compute)

* dense disparity reconstruction
* confidence maps
* advanced filtering

### CoreFusion (host processing)

* outlier rejection
* temporal smoothing
* multi-view fusion
* 3D keypoint fitting

### MotionCoder (optional interaction layer)

* gesture classification
* deterministic command semantics
* domain-specific mappings
* confidence-aware behavior

---