# Multi-Rig Architecture

The EdgeTrack system is designed to operate not only as a single stereo unit but also as a **distributed multi-rig tracking architecture**.

Multiple stereo rigs can be deployed within the same environment and fused into a single spatial model. This approach improves robustness, spatial coverage, and tracking stability, especially in complex scenes where occlusion or difficult lighting conditions occur.

Instead of relying on a single sensor pair, the system uses **geometric redundancy across multiple viewpoints**.

---

# Motivation

Single stereo rigs have inherent limitations:

* occlusions when objects block the view
* reduced accuracy at long distances
* limited field of view
* sensitivity to reflections or low-texture surfaces

A multi-rig architecture addresses these issues by combining observations from **multiple independent stereo systems**.

This allows the system to maintain stable tracking even when individual rigs temporarily lose reliable observations.

---

# Multi-Rig System Layout

A typical configuration may include **2–4 stereo rigs** positioned around the tracking volume.

Example configuration:

```
Rig 1        Rig 2

     Tracking Volume

Rig 3        Rig 4
```

Each rig independently captures stereo observations and streams them to the host system.

---

# Data Flow

Each stereo rig operates independently and produces synchronized observations.

```
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

The host system aggregates all incoming observations and reconstructs a **consistent spatial representation**.

---

# Advantages of Multi-Rig Systems

## Reduced Occlusions

Multiple viewpoints reduce the probability that an object becomes fully hidden.

Even if one rig loses sight of an object, another rig may still observe it.

---

## Increased Spatial Coverage

Multi-rig setups can cover larger volumes than a single stereo pair.

This enables applications such as:

* room-scale tracking
* multi-user environments
* robotics workspaces
* XR authoring setups

---

## Improved Geometric Stability

Observing the same object from multiple viewpoints improves triangulation stability.

This results in:

* lower noise
* improved depth precision
* more reliable pose estimation

---

## Higher Effective Temporal Sampling

When rigs operate with **phase-offset timing**, the effective temporal sampling rate can increase.

Example:

```
Rig 1 → frame at t0
Rig 2 → frame at t0 + Δt
Rig 3 → frame at t0 + 2Δt
```

This technique can increase the **effective temporal resolution** of motion tracking beyond the frame rate of a single camera.

---

# Synchronization

Multi-rig setups require precise timing coordination.

EdgeTrack uses hardware-level synchronization to ensure:

* predictable frame timing
* deterministic capture sequences
* consistent multi-view reconstruction

Synchronization can be controlled through dedicated timing hardware such as **TDMStrobe**.

---

# Networking

Rigs connect to the host system through **wired Ethernet**.

Ethernet provides several advantages compared to USB-based camera systems:

* stable bandwidth
* predictable latency
* better scalability
* easier distributed deployments

This allows rigs to be physically separated while still operating as part of a single synchronized system.

---

# Fusion on the Host

All rigs stream their observations to **CoreFusion**.

CoreFusion performs:

* cross-rig alignment
* multi-view triangulation
* outlier rejection
* temporal smoothing

The output is a **stable, unified 3D representation** of the scene.

---

# Scalability

The architecture is designed to scale from:

* **single-rig systems** (compact setups)
* **dual-rig systems** (improved robustness)
* **multi-rig installations** (large tracking volumes)

Additional rigs can be integrated without changing the core system architecture.

---

# Design Principle

EdgeTrack follows a **distributed perception model**.

Instead of building a single complex sensor, the system combines **multiple simple stereo rigs**.

This approach offers several benefits:

* lower hardware cost
* modular expansion
* increased redundancy
* improved reliability

It also aligns with the overall design philosophy of the EdgeTrack ecosystem:

**geometry-first, deterministic, and modular.**

---