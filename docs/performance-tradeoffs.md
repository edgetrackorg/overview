# Performance Trade-offs

This document explains the architectural trade-offs behind EdgeTrack. It covers hardware constraints, compute considerations, and why the system favors RAW stereo capture, deterministic timing, and ROI-focused processing instead of dense depth everywhere.

---

## Important: Don’t Confuse AI Depth with Physical Stereo

For NIR stereo tracking, **AI is not required**.

EdgeTrack is built on **deterministic, geometry-based NIR stereo vision** as the primary measurement system. Depth is computed from **physical baseline geometry, calibrated optics, and synchronized capture** — not from learned priors or dataset inference.

AI is **optional** and only used where it provides measurable benefit, such as:

* Stability monitoring
* Lightweight classification
* Left/right consistency checks
* Failure detection and recovery
* Semantic interpretation (gesture meaning, object type, intent)

The **core 3D reconstruction pipeline remains purely geometric and metric**.

### Why Multi-View Geometry Is Stronger Than AI Guessing

A single stereo rig can benefit from AI assistance in difficult cases.

However, the structurally stronger solution is **multi-view geometry**.

With **2–3 synchronized stereo rigs**, the system gains:

* Reduced occlusions
* Redundant triangulation
* Cross-validation between viewpoints
* Increased robustness without relying on learned priors

In many real-world setups, this reduces the need for AI almost entirely.

Geometry scales predictably.
Inference does not.

### Example: Apple’s Depth Pro

Apple’s Depth Pro is a powerful **monocular AI depth model**.

It is technically impressive — but it does not replace physical stereo measurement.

* AI depth infers structure from statistical patterns learned during training.
* Stereo depth measures disparity derived from real-world geometry and baseline separation.

Both approaches are valid — but they serve different purposes.

In EdgeTrack, AI models act as **assistive layers**, not as the measurement foundation.

### Physical Stereo vs Neural Stereo vs AI Depth

| Property                   | Classic Stereo (Geometry)       | Neural Stereo (AI-assisted matching) | Monocular AI Depth                |
| -------------------------- | ------------------------------- | ------------------------------------ | --------------------------------- |
| **Number of cameras**      | 2                               | 2                                    | 1                                 |
| **Requires baseline**      | Yes                             | Yes                                  | No                                |
| **Depth principle**        | Triangulation from disparity    | Learned disparity estimation         | Learned depth inference           |
| **Metric scale accuracy**  | True metric (after calibration) | True metric (after calibration)      | Often relative unless constrained |
| **Determinism**            | High (repeatable geometry)      | Medium (model-dependent)             | Low (probabilistic inference)     |
| **Training required**      | No                              | Yes                                  | Yes                               |
| **NPU/GPU requirement**    | No                              | Often beneficial                     | Required for real-time            |
| **Per-frame latency**      | Low, predictable                | Medium to high                       | Medium to high                    |
| **Multi-view consistency** | Naturally consistent            | Needs fusion logic                   | Needs full reconstruction logic   |
| **Texture-poor surfaces**  | Can struggle without pattern *  | Often improved                       | Often improved                    |
| **Gloss / reflections**    | Controlled with NIR + filtering | Model-dependent                      | Can hallucinate                   |
| **Occlusion handling**     | Solved via multi-view geometry  | Improved with fusion                 | Must infer hidden geometry        |
| **Interpretability**       | Physical measurement            | Hybrid                               | Statistical estimate              |

\* In controlled indoor environments with stable NIR illumination, minimal glare, and no direct sunlight, stereo performs very well. Typical failure modes mainly occur on very low-texture materials, strong specular reflections, and severe occlusions. In such cases, polarization can be used as an additional optical countermeasure.

EdgeTrack addresses these primarily through **optics and geometry**, not neural compensation.

The current prototype uses **diffuse (matte) NIR illumination** to generate a clean, homogeneous flood field. This improves stereo correspondence and reduces hotspots that destabilize matching.

On top of that, **MultiView coverage (2–3 rigs)** significantly increases robustness by:

* Reducing occlusions
* Increasing usable perspective diversity
* Improving triangulation reliability

Structured-light projection can improve extreme texture-poor scenes. However, it adds cost, optical complexity, synchronization constraints, and cross-talk risks — often without clear benefit for EdgeTrack’s primary use case.

For **close-range hand interaction (0.5–0.8 m)**, natural micro-texture of skin and clothing combined with controlled NIR lighting is typically sufficient. Structured light is therefore unnecessary in most scenarios.

### Architectural Positioning

EdgeTrack follows a strict hierarchy:

1. **Primary layer:** Deterministic NIR stereo geometry
2. **Secondary layer (optional):** AI-based refinement or validation
3. **Semantic layer:** Gesture interpretation and high-level understanding

This avoids a common market confusion:

> AI depth estimation is not equivalent to physical stereo measurement.

Stereo **measures**.
Neural stereo **optimizes matching**.
Monocular AI **estimates**.

When combined properly, they complement each other — but they are not interchangeable.

---

## What is the “best” stereo camera?

Many people ask: *What is the “best” stereo camera — and is higher resolution automatically better?*

This question cannot be answered by pixel count alone. In stereo vision, the primary goal is not “as many details as possible,” but **stable and reliable geometry**. The decisive parameters are:

* **Baseline**
* **Field of View (FOV)**
* **Disparity range**
* **Frame rate (FPS)**
* **Available compute budget**

Equally important is the **targeted optimization of the processing pipeline**.

To make these relationships more tangible, I created a simple HTML-based stereo calculator. It visualizes the relationship between baseline, working distance, disparity, and compute budget through clear diagrams. The default parameters are chosen to be practical and representative of the typical EdgeTrack use case.

The calculator is available here: [Disparity Calc](https://edgetrack.org/disparity-calc)

> Note: The tool focuses on **geometric and computational factors only**. Optical effects such as NIR bandpass filtering, polarization, sensor behavior, and contrast characteristics are not included, even though they also strongly influence reconstruction stability.

If the compute requirements exceed the capabilities of the **Raspberry Pi 5**, there are essentially two options:

1. Use more powerful hardware (which can easily cost several thousand euros depending on FPS and pipeline complexity).
2. Optimize the pipeline so that it runs efficiently on the Pi.

EdgeTrack deliberately follows the second path. The Raspberry Pi 5 offers a **very strong price-performance window** when geometry is used correctly and processing focuses only on the data that truly matters — for example **ROI-based processing instead of dense full-frame disparity**.

In practice there is often **no real “middle class”** between a Raspberry Pi and workstation-level hardware. Systems tend to jump directly from low-cost embedded hardware to expensive high-performance compute platforms.

---

## The Importance of Direct RAW Access

A key advantage of the Raspberry Pi platform is **direct access to RAW sensor data**.

This enables:

* reproducible pipelines
* custom preprocessing
* precise timing control
* deterministic multi-camera synchronization

Instead of computing a **dense depth map over the entire image**, computational load can be reduced significantly by **targeted ROI processing**:

* fewer pixels processed
* reduced disparity search range
* fewer intermediate buffers

This approach allows **stable high FPS** without wasting unnecessary resources.

For many tracking applications, a full-frame dense reconstruction is simply not required.

Without direct RAW access, developers quickly become dependent on:

* industrial cameras
* proprietary SDK pipelines
* ISP-controlled processing

These often limit **control, timing accuracy, and reproducibility**.

---

## Limits of Consumer Stereo Cameras

Many commercially available stereo or depth cameras provide **ISP-processed image streams**, such as:

* demosaiced RGB
* tonemapped images
* noise-reduced frames
* compressed streams (MJPEG, H.264, H.265)

These systems are typically designed for **ease of integration**, not deterministic stereo reconstruction.

Typical practical performance:

* **15–60 FPS stable operation**
* higher FPS often tied to **reduced resolution**
* ROI or binning modes
* internal pipeline constraints

In real-world deployments, high advertised frame rates (e.g. **120 FPS**) are often difficult to maintain when:

* depth computation is enabled
* multiple sensors run in parallel
* streaming bandwidth becomes a bottleneck

A **fully controlled RAW-first pipeline with strict frame synchronization and stable 120 FPS** is therefore uncommon in the consumer/prosumer segment.

Systems requiring this level of determinism typically rely on industrial interfaces such as:

* **CoaXPress**
* **GigE Vision**
* **Camera Link**

—or on a custom deterministic edge pipeline such as EdgeTrack.

Another important limitation:

Without RAW access, even high-quality optics (e.g. NIR bandpass filters or polarization) lose effectiveness, because ISP processing can modify the image before stereo matching.

This introduces artifacts that significantly reduce reconstruction accuracy.

---

## Quick Comparison

| Typical stacks (“Other”)            | **EdgeTrack**                       |
| ----------------------------------- | ----------------------------------- |
| ISP-processed or compressed streams | **RAW-first capture pipeline**      |
| Single-rig focus                    | **Multi-rig capable architecture**  |
| Limited practical FPS               | **Scales with hardware and tuning** |
| USB timing jitter                   | **Ethernet + hardware sync**        |
| Dense depth everywhere              | **ROI-first processing**            |
| AI-inferred depth                   | **Metric stereo geometry**          |
| Vendor lock-in                      | **Open architecture**               |

Additional comparison tables are available in [Comparison Tables](https://github.com/edgetrackorg/overview/blob/main/docs/comparison_table.md).

---

## VPU vs CPU (Stereo Disparity)

Many stereo cameras use a **VPU or ASIC** to compute disparity on-device. This can be extremely efficient for dense depth output.

However, it comes with limitations in flexibility and control.

The Raspberry Pi 5 represents a different trade-off. It is not as efficient for **dense full-frame disparity**, but it allows **complete pipeline control**.

EdgeTrack leverages this flexibility through:

* ROI-based matching
* adjustable disparity ranges
* custom RAW preprocessing

This can dramatically reduce compute cost while improving determinism and reproducibility.

| Feature                | CPU (Pi 5)       | CPU (Threadripper) | GPU (Host-side) | VPU (On-device) |
| ---------------------- | ---------------- | ------------------ | ---------------- | --------------- |
| Dense depth efficiency | ⚠️ Medium        | 🚀 Very High       | 🚀 Very High     | ✅ High         |
| 720p @ 30 FPS          | ⚠️ Borderline    | ✅ Stable          | ✅ Stable        | ✅ Stable       |
| 120 FPS dense          | ❌ Not practical | ⚠️ Possible        | ⚠️ Possible      | ❌ Rare         |
| ROI processing         | ✅ Excellent     | ✅ Excellent       | ✅ Strong        | ⚠️ Limited      |
| Pipeline control       | ✅ Full          | ✅ Full            | ✅ High          | ⚠️ Limited      |
| Multi-rig sync         | ✅ Ideal         | ⚠️ Complex         | ⚠️ Host-dependent| ⚠️ Limited      |

**Summary**

* **VPU** → ideal for simple on-device dense depth
* **GPU** → strong acceleration for large-scale host processing
* **CPU + RAW pipeline** → ideal for deterministic, controllable multi-rig stereo systems

---

## Do We Use Jetson?

Yes, potentially — but not as the primary baseline.

Jetson provides significant compute performance, but it also moves the system toward a higher-cost, higher-power, and more GPU-centric architecture. EdgeTrack is deliberately designed around deterministic capture, strict timing control, and a RAW-first pipeline on affordable hardware.

In many existing Jetson-based camera workflows, H.264/H.265 transport is commonly used for convenience and bandwidth reduction. By contrast, fully controlled RAW-first pipelines are often less mature or less straightforward in the context of deterministic stereo capture and multi-rig synchronization.

Jetson support may be introduced later for more demanding processing modes, but it is not the baseline target for the first stable release.

---

## Why RAW Stereo Instead of H.265

Many stereo cameras rely on **H.264/H.265 compression**.

While suitable for visualization and preview streams, compression introduces:

* lossy artifacts
* temporal smoothing
* unpredictable frame timing

These effects reduce stereo reconstruction accuracy.

Using **RAW10 sensor data** preserves:

* linear pixel intensities
* precise timing
* deterministic reconstruction

RAW capture also enables precise control of:

* exposure
* gain
* synchronization

When combined with **NIR illumination**, RAW stereo becomes significantly more robust in challenging lighting conditions.

For this reason EdgeTrack deliberately avoids **H.265 pipelines** and focuses on **RAW-first stereo capture**.

---

### Pixel Format Comparison

| Format        | Rating       | Comment                        |
| ------------- | ------------ | ------------------------------ |
| MJPEG / H.265 | ⭐☆☆☆☆       | Preview/debug only             |
| YUV / YUYV    | ⭐⭐☆☆☆      | Only useful if using Y channel |
| RAW8          | ⭐⭐⭐☆☆     | Good baseline                  |
| RAW10         | ⭐⭐⭐⭐☆    | Excellent balance              |
| RAW12         | ⭐⭐⭐⭐⭐   | Highest precision              |

---

## SLAM Considerations

For SLAM applications, strict hard real-time performance is often not required.

Instead of maximizing FPS, systems can operate at:

* lower frame rates
* higher latency
* larger compute budgets

With **multi-view fusion (CoreFusion)**, geometric redundancy increases. This improves numerical conditioning and overall SLAM robustness.

The Raspberry Pi 5 is well suited for:

* experimental research setups
* flexible SLAM pipelines
* hybrid CPU/GPU processing

### Alternative Processing Strategies

Example configuration:

**1280×800 @ 30 FPS (RAW10)**
Two cameras stream directly from the Raspberry Pi to a host system where disparity computation is performed.

Alternative setup:

**1280×800 @ 120 FPS (RAW10)**
Two cameras connected to a **Radxa ROCK 5B (2.5GbE)**.

While 4 GB RAM can be sufficient for streaming, using **two boards (one per camera)** provides better bandwidth margin and system stability.

Future Raspberry Pi generations may significantly increase compute and network performance, enabling higher on-device disparity rates.

---

## LiDAR / ToF Trade-offs

LiDAR and Time-of-Flight sensors are often used for 3D perception but introduce trade-offs for **precision interaction workflows**.

Key limitations include:

* lower spatial detail at close range
* temporal noise and depth flutter
* emitter interference in multi-sensor setups
* limited deterministic control

High-quality industrial LiDAR/ToF systems can mitigate these issues but often require:

* higher cost
* proprietary SDKs
* constrained deployment environments

EdgeTrack therefore focuses on:

* **synchronized global-shutter stereo**
* **controlled NIR illumination**
* **deterministic timing**

This approach offers a scalable and transparent foundation for precise 3D interaction systems.

---


## Interface: USB vs. Ethernet vs. WLAN

USB, Ethernet, and WLAN are all commonly used to connect cameras and tracking devices, but they differ significantly in terms of determinism, scalability, and reliability.

**USB** is widely available and easy to set up, making it well suited for single-device configurations, prototyping, and consumer peripherals. However, USB is typically host-driven and shared across multiple devices on the same controller. As a result, bandwidth contention, variable latency, and timing jitter can occur, especially in multi-camera setups or under high system load. While acceptable for many use cases, these characteristics can limit predictability in time-critical pipelines.

**Ethernet** is designed for distributed and scalable systems. Each device operates independently on the network and commuates using explicit packetization, buffering, and timestamps. This enables more predictable latency, cleaner synchronization across multiple devices, and stable performance over longer cable distances. Ethernet also supports structured topologies using switches, VLANs, and Power-over-Ethernet (PoE), making it well suited for multi-rig and multi-room setups. For professional capture and deterministic processing pipelines, Ethernet is often the preferred transport layer.

**WLAN (Wi-Fi)** provides flexibility and mobility by removing physical cables, which can be advantageous in portable or rapidly reconfigurable environments. However, wireless links are inherently subject to interference, variable airtime, and changing network conditions. These factors can introduce fluctuating latency, packet loss, and jitter, which complicate synchronization and reproducibility. While modern Wi-Fi standards offer high peak bandwidth, sustained real-time performance is harder to guarantee.

In practice, USB is convenient for simple setups, Ethernet offers the most control and scalability for deterministic systems, and WLAN trades predictability for mobility and ease of deployment.

> **Note:** For tight timing, direct NIC connections are preferred. Switches usually add small latency, but can add variability under congestion; use QoS/VLAN/PTP if deterministic timing is required.

---

### ⚙️ Quick Engineering Comparison — What is the best interface for deterministic vision?

When designing a machine-vision or stereo system, the choice of sensor interface has a strong impact on latency, control, and system complexity.
Below is a simplified engineering comparison:

| Interface              | Additional Chips / Infra | RAW Access | Latency     | Determinism |
| ---------------------- | ------------------------ | ---------- | ----------- | ---------- |
| **MIPI CSI-2**         | very few                 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **USB2 (typical UVC)** | medium                   | ⭐☆☆☆☆     | ⭐⭐☆☆☆     | ⭐☆☆☆☆     |
| **USB3 (typical UVC)** | medium                   | ⭐⭐☆☆☆    | ⭐⭐⭐☆☆   | ⭐⭐☆☆☆     |
| **GigE / GigE Vision** | many                     | ⭐⭐⭐⭐☆  | ⭐⭐⭐☆☆   | ⭐⭐⭐⭐☆  |
| **CoaXPress**          | heavy (framegrabber)     | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

#### Summary

**MIPI CSI-2** is typically the best choice when deterministic timing, minimal latency, and direct RAW sensor access are required. The sensor is connected almost directly to the SoC, which reduces hidden processing stages and keeps the pipeline transparent.

**USB cameras** usually include additional ISP and bridge chips. They are convenient and plug-and-play, but often introduce internal processing and buffering that reduce determinism.

**GigE cameras** are powerful for industrial networking and long cable distances, but typically require more intermediate logic (FPGA/ASIC, packetization, buffering), which increases system complexity.

**CoaXPress** is a high-end industrial interface designed for very high bandwidth and deterministic transmission. It typically requires a dedicated frame grabber card on the host side and specialized hardware inside the camera. While it offers excellent throughput, low latency, and strong determinism, it significantly increases system cost, hardware complexity, and power requirements compared to embedded MIPI-based designs.

For edge-processing architectures focused on precise timing and reproducible results, **MIPI CSI-2 provides the most transparent and controllable capture path**.

---

### Latency Comparison (Typical, No Aggressive Tuning)

| # | Class                                                         | Typical latency (no tuning)                                          | Jitter / determinism                            | Main latency contributors                                                                       | Practical notes                                                                                                                                                                                                                                     |
| - | ------------------------------------------------------------- | -------------------------------------------------------------------- | ----------------------------------------------- | ----------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1 | **High-end USB webcam (UVC, often ISP + MJPEG/H.264)**        | **50–200 ms** (often “noticeably laggy”)                             | **poor–medium**, highly variable                | Camera ISP + **multi-frame buffering** + (optional) codec/decode + host color/format conversion | Often “beautified” internally (AE/NR/scaling) and buffered for smooth video. Usually not suitable for low-latency VR/tracking workloads.                                                                                                            |
| 2 | **Industrial USB (USB3 Vision, often uncompressed/RAW/mono)** | **10–30 ms**                                                         | **medium**                                      | Sensor (exposure/readout) + USB transfer + host queue/copy                                      | Much better than webcams. **Bottlenecks:** USB bandwidth per host controller, cable/EMI constraints, and multi-camera setups saturate quickly. Multi-view can heavily load the host (DMA + CPU/GPU processing).                                     |
| 3 | **Industrial GigE (GigE Vision, UDP)**                        | **15–40 ms**                                                         | **medium**, jitter possible                     | Packetization + NIC/kernel/queueing + (optional) switch/network effects                         | Robust and flexible (long cables, PoE options). Without tuning you can see **jitter/spikes** (network contention, driver scheduling, interrupts). Multi-camera works, but host + network design becomes critical.                                   |
| 4 | **CoaXPress (frame grabber)**                                 | **5–20 ms**                                                          | **good–very good**                              | Sensor dominates; transport is almost “invisible”                                               | Highly deterministic with very fast triggering. **Downside:** **expensive** (cameras + frame grabber + cables) and a heavier industrial ecosystem. Multi-camera scales well, but cost/integration is high.                                          |
| 5 | **EdgeTrack (edge compute + results over Ethernet only)**     | **5–15 ms (keypoints/ROI)** / **10–30 ms (dense depth/point cloud)** | **very good**, deterministic (HW trigger + MCU) | Sensor + edge compute time (minimal transport overhead)                                         | You “spend” latency on **edge compute**, but save a lot elsewhere: no video codec, no RAW transport, and far less host load. Multi-view scales better because the host fuses **small result streams** (CoreFusion) instead of handling full frames. |

#### Why #5 often “feels” best in real systems

* For **#1–#4**, the host typically has to **move frames + decode + convert formats + run vision**, which adds latency, jitter, and CPU/GPU load.
* For **#5**, the expensive work happens **before transport**, and you transmit only **keypoints/ROI/point clouds**, leading to lower, more predictable latency and a simpler host pipeline.
