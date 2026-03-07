# Timing Architecture

Precise timing is a fundamental requirement for reliable stereo reconstruction and multi-rig fusion. EdgeTrack therefore uses a **deterministic timing architecture** that synchronizes cameras, illumination, and capture pipelines at the hardware level.

Instead of relying on software-triggered frame capture, the system uses **explicit timing control** to ensure predictable frame relationships across stereo pairs and across multiple rigs.

This approach reduces jitter, improves stereo correspondence, and enables stable multi-view geometry.

---

# Why Timing Matters

Stereo vision relies on comparing images captured at the **same moment in time**.

If frames are not synchronized precisely, several problems occur:

* motion mismatch between left and right images
* reduced stereo matching accuracy
* unstable disparity estimation
* inconsistent multi-view triangulation

These effects become more pronounced at higher frame rates and when tracking fast motion.

For multi-rig systems, timing errors also degrade **cross-rig fusion** and temporal filtering.

---

# Deterministic Capture

EdgeTrack prioritizes **deterministic capture timing**.

Each stereo pair operates using:

* synchronized global-shutter sensors
* controlled exposure timing
* deterministic frame scheduling

This ensures that both cameras in a stereo pair observe the scene at the same instant.

---

# Hardware Synchronization

To guarantee stable timing behavior, EdgeTrack relies on **hardware-level synchronization** rather than software triggers.

Hardware synchronization enables:

* microsecond-level timing precision
* consistent exposure across cameras
* predictable frame ordering

This synchronization can be implemented using dedicated timing hardware such as **TDMStrobe**.

---

# Time Division Multiplexing (TDM)

In multi-rig setups, simultaneous illumination from multiple systems can cause interference.

EdgeTrack addresses this using **Time Division Multiplexing (TDM)**.

Each rig operates in its own **timing slot**, meaning:

* illumination pulses do not overlap
* sensors capture clean frames without cross-rig interference

Example timing schedule:

```
Rig 1 → capture window A
Rig 2 → capture window B
Rig 3 → capture window C
Rig 4 → capture window D
```

This allows multiple rigs to operate in the same environment without interfering with each other.

---

# Phase-Offset Capture

Another technique used in multi-rig environments is **phase-offset capture**.

Instead of all rigs capturing frames simultaneously, each rig can be shifted slightly in time.

Example:

```
Rig 1 → frame at t0
Rig 2 → frame at t0 + Δt
Rig 3 → frame at t0 + 2Δt
Rig 4 → frame at t0 + 3Δt
```

This technique can increase the **effective temporal sampling rate** of the overall system.

For example, four rigs operating at 120 FPS with phase offsets can produce an effective temporal sampling of up to **480 samples per second**.

---

# Illumination Timing

EdgeTrack often uses **NIR illumination** to improve stereo matching stability.

To maximize efficiency and reduce motion blur, illumination is typically synchronized with camera exposure.

Short pulsed illumination allows:

* reduced motion blur
* higher effective brightness
* improved contrast in NIR bandpass setups

Precise timing ensures that illumination occurs **only during sensor exposure**.

---

# Timing in the Processing Pipeline

Deterministic timing is preserved throughout the processing pipeline.

The capture sequence typically follows:

```
Exposure trigger
      ↓
Sensor capture
      ↓
RAW frame acquisition
      ↓
Optional edge processing
      ↓
Transmission to host
      ↓
Multi-rig fusion
```

Maintaining consistent timing across these stages improves temporal filtering and motion stability.

---

# Benefits of Deterministic Timing

The timing architecture provides several advantages:

* stable stereo correspondence
* reliable multi-rig fusion
* reduced temporal jitter
* predictable latency
* improved reconstruction quality

These properties are essential for **high-precision motion tracking and spatial interaction systems**.

---

# Design Philosophy

EdgeTrack treats **timing as a first-class system component**.

Instead of relying on loosely synchronized sensors, the architecture uses explicit timing control to ensure deterministic behavior across the entire capture pipeline.

This approach supports:

* reproducible tracking results
* scalable multi-rig systems
* predictable real-time performance

It is a key element of the overall **geometry-first and deterministic design philosophy** of the EdgeTrack ecosystem.

---
