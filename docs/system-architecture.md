# System Architecture

This document describes the high-level architecture of the EdgeTrack ecosystem. The system is designed as a modular pipeline that separates **sensor capture**, **geometry reconstruction**, **multi-rig fusion**, and **interaction layers**.

The architecture prioritizes:

* deterministic timing
* RAW-first image capture
* modular stereo rigs
* scalable multi-rig setups
* transparent processing pipelines

Each stage of the system has a clearly defined role and can be modified or replaced without affecting the overall architecture.

---

# High-Level Pipeline

At a system level, the EdgeTrack ecosystem follows a layered architecture:

```
Stereo Sensors
      ↓
EdgeTrack (edge capture)
      ↓
Optional EdgeSense
      ↓
CoreFusion (host fusion)
      ↓
MotionCoder (interaction layer)
      ↓
Applications
```

This separation allows each layer to focus on a specific responsibility.

---

# Layer 1 – Edge Capture (EdgeTrack)

EdgeTrack units operate on the **edge device**, typically based on embedded hardware such as Raspberry Pi platforms.

Their primary responsibilities include:

* synchronized stereo capture
* RAW image acquisition
* optional rectification
* ROI tracking
* optional local stereo processing

Each unit produces **time-consistent local observations** that can later be fused with other rigs.

Multiple EdgeTrack rigs can operate simultaneously to increase spatial coverage and reduce occlusion.

---

# Optional Edge Processing (EdgeSense)

EdgeSense is an optional assist layer that runs on the edge device.

It does **not replace stereo reconstruction**, but improves robustness by providing:

* confidence estimation
* ambiguity detection
* lightweight classification
* failure detection

EdgeSense can run on small AI accelerators when available.

The core geometry pipeline remains fully functional without it.

---

# Host Fusion Layer (CoreFusion)

CoreFusion runs on the host system and aggregates data from multiple EdgeTrack units.

Its responsibilities include:

* multi-view triangulation
* cross-rig alignment
* outlier rejection
* temporal smoothing
* pose estimation

The result is a **stable, time-consistent 3D representation** such as keypoints or structured motion signals.

CoreFusion transforms independent rigs into a **distributed perception system** connected via Ethernet.

---

# Interaction Layer (MotionCoder)

MotionCoder interprets motion signals and converts them into structured commands for applications.

It processes time-consistent 3D keypoints and produces:

* gesture recognition
* interaction commands
* editor actions
* structured control signals

MotionCoder focuses on **deterministic interaction**, not purely visual hand tracking.

Typical target environments include:

* Blender
* Unreal Engine
* XR authoring tools
* spatial editing workflows

---

# Optional Host Stereo Processing (CoreStereo)

In some configurations, stereo disparity computation can be moved to the host.

CoreStereo ingests synchronized RAW frames and performs:

* dense disparity reconstruction
* extended disparity search
* advanced filtering
* experimental stereo algorithms

This allows high-performance compute platforms (e.g., GPUs or workstation CPUs) to perform more expensive stereo processing tasks.

---

# Networking and Synchronization

EdgeTrack systems are designed around **wired Ethernet networking**.

Advantages include:

* predictable latency
* stable bandwidth
* scalable multi-rig setups
* robust synchronization

Hardware timing is handled through **dedicated synchronization hardware** such as TDMStrobe.

This ensures that exposure timing across cameras remains deterministic.

---

# Design Philosophy

The EdgeTrack system follows several core design principles:

* **Geometry-first reconstruction** rather than AI-inferred depth
* **RAW-first capture** instead of ISP-processed pipelines
* **Deterministic timing** through hardware synchronization
* **Distributed multi-rig architecture**
* **Open and transparent processing pipeline**

This approach enables reproducible and scalable 3D tracking systems that can be adapted to different research and industrial use cases.

---