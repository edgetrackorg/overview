# Design Principles

The EdgeTrack architecture is built around a set of core design principles that guide both hardware and software development.

These principles prioritize **determinism, transparency, and geometric accuracy** over convenience features often found in consumer vision systems.

Rather than treating the vision pipeline as a black box, EdgeTrack exposes the full system architecture so that timing behavior, reconstruction quality, and performance trade-offs remain visible and controllable.

---

# Geometry-First Reconstruction

EdgeTrack follows a **geometry-first approach**.

The system relies primarily on **physical stereo geometry** rather than depth inferred purely from machine learning.

Stereo matching provides:

* metric depth estimation
* physically interpretable reconstruction
* stable multi-view triangulation

AI methods may assist the pipeline, but they are not the primary source of spatial information.

This ensures that reconstruction remains **predictable and explainable**.

---

# RAW-First Capture

The pipeline begins with **direct RAW sensor data**.

Instead of relying on ISP-processed images or compressed video streams, EdgeTrack preserves the original sensor output.

Advantages include:

* predictable pixel statistics
* controllable preprocessing
* reproducible reconstruction

RAW-first capture also avoids hidden processing stages that could introduce latency or artifacts.

---

# Deterministic Timing

Precise timing control is essential for stereo reconstruction.

EdgeTrack therefore uses:

* global shutter sensors
* hardware synchronization
* deterministic capture scheduling

This ensures that frames across stereo pairs and across multiple rigs remain time-consistent.

Stable timing improves:

* stereo correspondence
* multi-view fusion
* motion tracking accuracy

Timing is treated as a **core architectural component**.

---

# Modular System Architecture

EdgeTrack is designed as a modular system composed of independent layers.

Typical layers include:

* edge capture (EdgeTrack)
* optional edge AI assistance (EdgeSense)
* host fusion (CoreFusion)
* interaction layers (MotionCoder)

Each layer has a clearly defined responsibility and can evolve independently.

This modular structure allows the system to adapt to different applications without redesigning the entire pipeline.

---

# Distributed Multi-Rig Systems

Instead of relying on a single complex sensor, EdgeTrack supports **distributed stereo rigs**.

Multiple rigs observe the same environment from different viewpoints.

This improves:

* spatial coverage
* robustness against occlusion
* triangulation accuracy

Host-side fusion combines observations into a unified spatial model.

---

# Efficient Processing

EdgeTrack prioritizes **efficient computation**.

Instead of performing dense stereo reconstruction across entire frames, the system focuses processing resources on relevant regions.

ROI-based processing allows:

* higher frame rates
* lower latency
* predictable compute budgets

Dense reconstruction remains possible when required but is not mandatory.

---

# Open and Transparent Pipeline

A central goal of EdgeTrack is to maintain a **transparent and inspectable processing pipeline**.

The architecture avoids opaque vendor SDKs and hidden firmware behavior whenever possible.

Developers should be able to understand:

* how frames are captured
* how timing is controlled
* how depth is reconstructed
* how motion signals are generated

This transparency makes the system easier to debug, extend, and reproduce.

---

# Scalable Architecture

EdgeTrack is designed to scale from small setups to larger installations.

Possible configurations include:

* single stereo rigs
* dual-rig systems
* multi-rig environments

The distributed architecture allows systems to expand gradually while maintaining consistent processing principles.

---

# Design Philosophy

The EdgeTrack ecosystem prioritizes:

* deterministic behavior
* geometric accuracy
* modular architecture
* open system design

By combining **RAW-first capture, precise timing control, distributed fusion, and efficient ROI processing**, the system provides a robust foundation for high-precision spatial tracking and interaction systems.

This philosophy enables EdgeTrack to remain flexible, transparent, and scalable across research, industrial, and creative applications.
