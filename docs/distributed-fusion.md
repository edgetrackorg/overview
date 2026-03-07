# Distributed Fusion

Distributed fusion is a central concept of the EdgeTrack architecture.
Instead of relying on a single stereo rig, multiple rigs operate simultaneously and their observations are combined into a unified spatial model.

The fusion process is performed on the host system using **CoreFusion**.

This approach improves tracking robustness, spatial coverage, and reconstruction stability while keeping the edge hardware simple and scalable.

---

# Motivation

Single stereo rigs have inherent limitations:

* occlusions when objects block the camera view
* limited field of view
* reduced accuracy at larger distances
* sensitivity to reflections or low-texture surfaces

Distributed fusion addresses these limitations by combining observations from **multiple independent rigs**.

This creates geometric redundancy and allows the system to maintain stable tracking even if individual rigs temporarily lose reliable observations.

---

# Architecture Overview

In a distributed configuration, multiple stereo rigs capture synchronized observations of the same scene.

Example system layout:

```text
Stereo Rig 1
Stereo Rig 2
Stereo Rig 3
Stereo Rig 4
        ↓
     Ethernet
        ↓
     CoreFusion
        ↓
Unified 3D Model
```

Each rig performs local capture and optional edge processing, then streams its observations to the host.

---

# CoreFusion Responsibilities

CoreFusion is responsible for combining observations from all rigs.

Typical processing steps include:

* cross-rig alignment
* multi-view triangulation
* outlier rejection
* temporal filtering
* pose estimation

The output is a **stable, time-consistent 3D representation** of the tracked objects.

---

# Cross-Rig Alignment

Before observations from different rigs can be fused, the rigs must share a **common coordinate system**.

This is achieved through calibration procedures that estimate:

* camera intrinsics
* camera extrinsics
* rig-to-rig transformations

Once calibration is complete, observations from different rigs can be combined consistently.

---

# Multi-View Triangulation

When the same feature is observed by multiple rigs, its position can be estimated more accurately.

Multiple viewpoints allow the system to:

* reduce depth uncertainty
* improve triangulation stability
* reduce noise

This improves overall tracking accuracy compared to single-rig stereo systems.

---

# Outlier Rejection

Observations from individual rigs may occasionally contain errors.

Common causes include:

* reflections
* motion blur
* low-texture surfaces
* temporary occlusions

CoreFusion uses filtering strategies to detect and remove inconsistent observations.

This prevents unstable data from degrading the final output.

---

# Temporal Filtering

Tracking systems benefit from temporal consistency.

CoreFusion applies filtering across time to stabilize motion signals.

Typical filtering steps include:

* temporal smoothing
* motion prediction
* confidence weighting

These techniques improve stability without significantly increasing latency.

---

# Distributed Networking

Rigs communicate with the host system over **wired Ethernet**.

Ethernet provides several advantages:

* stable bandwidth
* predictable latency
* scalable multi-rig deployments

This allows rigs to be distributed across larger environments while still participating in a unified tracking system.

---

# Scalability

Distributed fusion allows the system to scale gradually.

Possible configurations include:

* single-rig setups (compact systems)
* dual-rig systems (improved robustness)
* multi-rig installations (large tracking volumes)

Additional rigs can be integrated without changing the overall architecture.

---

# Interaction with Other System Layers

Distributed fusion sits between the capture layer and the interaction layer.

Typical pipeline:

```text
EdgeTrack (edge capture)
        ↓
Optional EdgeSense
        ↓
CoreFusion (distributed fusion)
        ↓
MotionCoder (interaction layer)
        ↓
Applications
```

This modular structure allows each layer to evolve independently.

---

# Design Philosophy

The EdgeTrack architecture favors **distributed sensing** rather than a single complex sensor.

Multiple simpler stereo rigs working together provide:

* improved reliability
* better spatial coverage
* scalable system design

Distributed fusion combines these independent observations into a consistent and stable spatial representation.

This approach supports the overall EdgeTrack philosophy of **geometry-first reconstruction, deterministic timing, and modular system architecture**.

---