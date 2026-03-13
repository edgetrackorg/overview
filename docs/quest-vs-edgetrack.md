# Quest vs EdgeTrack – Tracking Architecture Comparison

This document briefly explains the architectural differences between consumer VR hand tracking (e.g. **Meta Quest**) and the **EdgeTrack stereo tracking concept**.

Both systems solve similar problems — tracking hands and gestures in 3D space — but they use **fundamentally different approaches**.

---

## Typical Quest Tracking Architecture

Consumer XR devices such as the Meta Quest rely primarily on **vision + machine learning pose estimation**.

Typical hardware components inside a headset include:

* 4 × wide-angle tracking cameras
* IR-sensitive image sensors
* optional IR illumination
* XR-class mobile SoC (e.g. Qualcomm XR series)
* IMU (gyro + accelerometer)

The processing pipeline typically works as follows:

```
Multi-camera capture
        ↓
ML hand pose estimation
        ↓
Sparse triangulation of keypoints
        ↓
Temporal filtering
```

The cameras observe the hands from multiple viewpoints.
A machine learning model detects **hand keypoints** (for example 21 finger joints).

Depth is estimated through a combination of:

* multi-view triangulation
* learned depth estimation
* temporal filtering

The result is a **sparse hand skeleton**, not a full 3D reconstruction.

### Advantages

* low power consumption
* optimized for mobile hardware
* robust against difficult lighting conditions
* works even with partial visibility

### Limitations

* depth is often **estimated rather than measured**
* precision is typically in the **centimeter range**
* occlusions may cause incorrect inference
* behavior is not fully deterministic
* higher latency and frequent keypoint jitter
* ML inference can become inconsistent under difficult conditions
* detailed finger articulation is often not captured with the accuracy required for demanding 3D authoring workflows

This approach is highly effective for **consumer XR interaction**, where gestures such as pinch, grab, or pointing are sufficient.

---

## EdgeTrack Architecture

EdgeTrack follows a different philosophy: **geometry-first stereo vision**.

Instead of relying primarily on AI inference, the system measures real spatial geometry using stereo cameras and controlled illumination.

A typical EdgeTrack module may consist of:

* 2 × stereo camera pairs (4 cameras total, matching the configuration used in the comparison above)
* NIR bandpass filters
* controlled NIR illumination panel
* embedded compute nodes (e.g. Raspberry Pi class hardware)
* host computer for fusion and processing

The pipeline is conceptually different:

```text
NIR illumination
        ↓
Synchronized stereo capture
        ↓
ROI-optimized dense stereo disparity
        ↓
Depth reconstruction
        ↓
Optional ML interpretation
```

Because depth is derived from **stereo disparity**, the system can produce a **dense depth map** in selected regions of interest rather than relying only on sparse keypoints. In practice, this means the system is not necessarily intended to run full-frame dense disparity at all times, but instead focuses compute resources where dense geometric measurement is most valuable.

This allows the reconstruction of hand surfaces, tools, or nearby objects while keeping the processing pipeline more efficient and better aligned with real-time constraints.

### Advantages

* true geometric depth measurement
* low latency and high precision through RAW sensor processing
* deterministic behavior
* stable spatial measurements
* suitable for precise interaction
* reproducible results

### Limitations

* higher compute requirements, mitigated by ROI-optimized processing
* more hardware components (typical for early prototypes)

---

## Hardware Example

A small experimental EdgeTrack prototype might contain:

```
4 × monochrome cameras
4 × NIR bandpass filters
2 × embedded compute boards
2 × aluminum camera housings
NIR illumination panel
```

Prototype systems can cost roughly **€600–1000** in materials due to small-volume components and development hardware.

However, this is typical for early hardware prototypes.

In larger production volumes, many components become significantly cheaper:

* camera modules
* filters
* embedded processors
* LED illumination systems
* custom PCBs

A realistic estimate for mass production could be roughly:

```
€200 - €300 for two stereo rigs (4 cameras total)
```

while maintaining the same architectural principles.


**By the Way:** Even **€1,000** is still relatively affordable by comparison. Industrial stereo camera systems or professional motion-tracking solutions such as **OptiTrack** often cost **well above €10,000**.

Thanks to platforms such as the **Raspberry Pi 5**, this prototype remains comparatively accessible. Without such low-cost embedded hardware, the system cost would increase significantly and move much closer to the price range of traditional industrial vision systems.

---

## Why the Approaches Differ

The design goals of both systems are different.

**Consumer XR headsets prioritize:**

* battery life
* compact hardware
* low heat
* simple gestures
* mass production

**EdgeTrack prioritizes:**

* geometric accuracy
* deterministic tracking
* controlled illumination
* stable spatial interaction

These goals lead to different engineering trade-offs.

---

## Summary

| Feature             | Quest-style tracking | EdgeTrack                |
| ------------------- | -------------------- | ------------------------ |
| Depth source        | ML estimation        | stereo disparity         |
| Output              | sparse hand skeleton | ROI + dense depth map    |
| Precision           | centimeter range     | millimeter potential     |
| Hardware complexity | low                  | moderate                 |
| Determinism         | limited              | high                     |
| Target use          | consumer XR          | professional interaction |

EdgeTrack is not intended to replace consumer XR tracking systems.
Instead, it explores an alternative architecture focused on **geometry-based interaction**, which may be valuable for applications such as:

* CAD and 3D authoring
* teleoperation
* industrial interaction
* research systems
* advanced gesture interfaces

---