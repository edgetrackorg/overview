# Deterministic Pipeline

A deterministic processing pipeline is a key design principle of the EdgeTrack architecture.

Instead of relying on opaque camera firmware, compressed video streams, or unpredictable processing stages, the system maintains **explicit control over every step of the capture and reconstruction pipeline**.

This allows developers to reason about timing, latency, and reconstruction quality in a transparent and reproducible way.

---

# What “Deterministic” Means

In this context, deterministic means that:

* each stage of the pipeline behaves predictably
* frame timing remains stable and well-defined
* processing steps are explicitly controlled
* system latency can be measured and reproduced

The pipeline avoids hidden or automatic processing steps that could introduce variability.

Examples of such hidden stages include:

* ISP auto-processing
* video compression pipelines
* internal device buffering
* undocumented firmware filters

Removing these uncertainties improves **reproducibility and system stability**.

---

# RAW-First Capture

The deterministic pipeline begins with **direct access to RAW sensor data**.

Instead of relying on processed RGB streams, EdgeTrack uses RAW formats such as:

* RAW8
* RAW10
* RAW12

RAW capture preserves the original linear intensity values produced by the image sensor.

This provides several advantages:

* predictable stereo matching behavior
* full control over preprocessing
* consistent pixel statistics across frames

RAW capture is therefore the foundation of a deterministic pipeline.

---

# Pipeline Overview

The typical EdgeTrack processing pipeline follows a clearly defined sequence:

```text
Sensor Exposure
      ↓
RAW Frame Capture
      ↓
Rectification
      ↓
ROI Tracking
      ↓
Stereo Matching (ROI)
      ↓
3D Feature Extraction
      ↓
Host Fusion (CoreFusion)
      ↓
Interaction / Application Layers
```

Each stage has a specific purpose and can be independently inspected or modified.

---

# Timing Control

Deterministic pipelines require stable frame timing.

EdgeTrack achieves this through:

* hardware synchronization
* controlled exposure timing
* global shutter sensors

These mechanisms ensure that all frames entering the pipeline are **time-consistent**.

Precise timing is particularly important for:

* stereo correspondence
* multi-rig fusion
* motion tracking

The system therefore treats **timing as part of the pipeline itself**.

---

# Edge Processing

Initial processing steps typically occur on the edge device.

Typical tasks include:

* frame acquisition
* rectification
* ROI tracking
* optional stereo matching

Processing on the edge device reduces bandwidth and allows early filtering of irrelevant data.

Only relevant motion information or sparse 3D observations are transmitted to the host system.

---

# Host-Side Processing

More complex operations are performed on the host.

CoreFusion aggregates observations from multiple rigs and produces a unified spatial model.

Typical host tasks include:

* multi-view triangulation
* cross-rig alignment
* outlier rejection
* temporal filtering

The result is a **stable, time-consistent representation of the tracked scene**.

---

# Avoiding Hidden Processing

Many camera systems rely heavily on internal processing pipelines.

These may include:

* automatic exposure
* dynamic noise reduction
* tone mapping
* compression

While useful for consumer applications, such stages introduce variability and reduce transparency.

The deterministic pipeline intentionally avoids these mechanisms whenever possible.

Instead, processing is performed in **explicit, controllable software stages**.

---

# Benefits

A deterministic pipeline provides several advantages:

* predictable latency
* reproducible system behavior
* easier debugging
* improved scientific reproducibility
* reliable multi-rig fusion

These properties are particularly important for:

* robotics
* spatial interaction
* motion capture
* research systems

---

# Design Philosophy

EdgeTrack treats the capture pipeline as an **explicit engineering system**, not a black box.

By combining:

* RAW-first capture
* hardware timing control
* ROI processing
* distributed fusion

the system achieves a deterministic architecture suitable for precise spatial tracking.

This approach reflects the broader EdgeTrack philosophy of **geometry-first reconstruction, modular system design, and transparent processing pipelines**.

---