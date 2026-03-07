# ROI Processing

ROI (Region of Interest) processing is a core design concept of the EdgeTrack architecture.
Instead of computing dense stereo depth across the entire image, the system focuses processing resources on **relevant regions only**.

This approach significantly reduces computational load while maintaining high tracking precision and deterministic performance.

ROI processing is particularly effective for **motion tracking, gesture interaction, and tool tracking**, where only a small part of the image contains useful information.

---

# Motivation

Traditional stereo pipelines often compute **dense disparity maps across the entire frame**.

While this is useful for general depth perception, it is often unnecessary for many real-time tracking applications.

Dense processing has several drawbacks:

* high computational cost
* increased latency
* unnecessary memory bandwidth usage
* reduced frame rate on embedded systems

ROI processing addresses these issues by limiting stereo computation to **small dynamic regions**.

---

# Basic Concept

The EdgeTrack pipeline detects and tracks relevant objects or features and performs stereo matching **only within these regions**.

Typical workflow:

```
RAW Stereo Frames
        ↓
Rectification
        ↓
ROI Detection / Tracking
        ↓
Stereo Matching (inside ROI only)
        ↓
3D Keypoints / Sparse Points
```

Instead of processing millions of pixels per frame, the system processes **only the pixels that matter**.

---

# Advantages of ROI Processing

## Reduced Compute Load

Processing a small ROI drastically reduces the number of pixels involved in stereo matching.

Example:

```
Full frame: 1280 × 800 = 1,024,000 pixels
ROI:        200 × 200 =   40,000 pixels
```

This can reduce compute requirements by **an order of magnitude**.

---

## Higher Frame Rates

By reducing computation, ROI processing enables higher frame rates on modest hardware.

For example, on Raspberry Pi platforms this makes it possible to maintain **stable high-FPS tracking**.

---

## Lower Latency

Processing fewer pixels reduces the total pipeline time.

This leads to:

* faster feedback
* lower interaction latency
* more responsive systems

---

## Deterministic Performance

Dense pipelines often produce unpredictable compute spikes depending on scene complexity.

ROI pipelines are more predictable because the **processing workload remains bounded**.

This improves real-time reliability.

---

# Dynamic ROI Tracking

ROIs are not fixed.

They move dynamically as tracked objects move in the scene.

The system maintains a **stable bounding region** around relevant features.

Typical sources for ROI updates include:

* motion tracking
* feature tracking
* keypoint detection
* previous frame prediction

---

# Sparse vs Dense Reconstruction

EdgeTrack supports two main stereo reconstruction strategies:

### Sparse / ROI Stereo

Used for tracking tasks.

Characteristics:

* small ROI regions
* targeted stereo matching
* keypoints or sparse point clouds

Advantages:

* fast
* efficient
* low latency

---

### Dense Stereo (Optional)

Used for reconstruction tasks.

Characteristics:

* full-frame disparity
* higher compute cost
* typically performed on host hardware

Dense processing may run in **CoreStereo** when required.

---

# Integration with the EdgeTrack Pipeline

ROI processing is typically executed on the edge device.

Example pipeline:

```
EdgeTrack (edge device)
    RAW capture
    Rectification
    ROI tracking
    Stereo inside ROI
        ↓
CoreFusion (host)
    multi-view fusion
    filtering
    pose estimation
```

The host system receives **compact motion information** instead of full-frame depth maps.

This significantly reduces network bandwidth and compute overhead.

---

# Interaction Workflows

ROI processing is especially suitable for **interaction systems**, where only hands or tools are relevant.

Examples:

* gesture recognition
* hand tracking
* tool tracking
* stylus / pen tracking
* object manipulation

In these cases, computing dense depth across the entire scene is unnecessary.

---

# Design Philosophy

EdgeTrack prioritizes **geometry-first reconstruction with efficient compute usage**.

ROI processing reflects this philosophy:

* focus compute where it matters
* avoid unnecessary processing
* maintain deterministic system behavior

Combined with RAW-first capture and deterministic timing, ROI-based stereo enables **high-performance tracking even on modest hardware platforms**.

---