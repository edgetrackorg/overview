# Performance Trade-offs

This document explains the architectural trade-offs behind EdgeTrack. It covers hardware constraints, compute considerations, and why the system favors RAW stereo capture, deterministic timing, and ROI-focused processing instead of dense depth everywhere.

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

The calculator is available here: [Disparity Calc](https://xcnc.eu/sub/old/disparity/)

> Note: The tool focuses on **geometric and computational factors only**. Optical effects such as NIR bandpass filtering, polarization, sensor behavior, and contrast characteristics are not included, even though they also strongly influence reconstruction stability.

If the compute requirements exceed the capabilities of the **Raspberry Pi 5**, there are essentially two options:

1. Use more powerful hardware (which can easily cost several thousand euros depending on FPS and pipeline complexity).
2. Optimize the pipeline so that it runs efficiently on the Pi.

EdgeTrack deliberately follows the second path. The Raspberry Pi 5 offers a **very strong price-performance window** when geometry is used correctly and processing focuses only on the data that truly matters — for example **ROI-based processing instead of dense full-frame disparity**.

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

## VPU vs CPU vs GPU vs FPGA

Many stereo cameras use a **VPU**, **ASIC**, or fixed-function depth engine to compute disparity directly on-device. This can be highly efficient for producing dense depth output with low power consumption.

However, such approaches often come with reduced flexibility, limited pipeline visibility, and less control over the underlying processing stages.

By contrast, host-side and programmable architectures allow much deeper control over the stereo pipeline.

The **Raspberry Pi 5** and similar ARM-based systems represent a different trade-off. They are not the most efficient option for **full-frame dense disparity**, but they provide **full pipeline control** and make it easier to experiment with custom stereo strategies.

EdgeTrack leverages this flexibility through:

* **ROI-based matching**
* **adjustable disparity ranges**
* **custom RAW preprocessing**
* **host-side fusion and reconstruction**
* **hardware-level synchronization across multiple rigs**

This can significantly reduce compute cost in practice while improving determinism, transparency, and reproducibility.

**FPGA-based processing** represents another important category. Compared with general-purpose CPUs and GPUs, FPGAs can provide **very low latency**, **high determinism**, and efficient streaming-style processing. They are especially attractive for fixed, high-speed stereo pipelines once the processing architecture has stabilized. The trade-off is significantly higher development complexity and reduced iteration speed compared with software-based CPU pipelines.

| Feature                   | CPU (ARM)       | CPU (High-End Host) | GPU (Host-Side)   | VPU / ASIC (On-Device)                    | FPGA                          |
| ------------------------- | --------------- | ------------------- | ----------------- | ----------------------------------------- | ----------------------------- |
| Dense depth efficiency    | ⚠️ Medium       | 🚀 Very high        | 🚀 Very high      | ✅ High                                    | 🚀 Very high                  |
| 720p @ 30 FPS             | ⚠️ Borderline   | ✅ Stable            | ✅ Stable          | ✅ Stable                                  | ✅ Stable                      |
| 120 FPS dense             | ❌ Not practical | ⚠️ Possible         | ⚠️ Possible       | ❌ Rare                                    | ✅ Strong potential            |
| ROI processing            | ✅ Excellent     | ✅ Excellent         | ✅ Strong          | ⚠️ Limited                                | ✅ Excellent                   |
| Pipeline control          | ✅ Full          | ✅ Full              | ✅ High            | ⚠️ Limited                                | ✅ Very high                   |
| Multi-rig synchronization | ✅ Ideal         | ⚠️ Complex          | ⚠️ Host-dependent | ⚠️ Limited                                | ✅ Excellent                   |
| Determinism               | ✅ Good          | ✅ Good              | ⚠️ Moderate       | ✅ Good                                    | 🚀 Excellent                  |
| Development speed         | ✅ Fast          | ✅ Fast              | ⚠️ Medium         | ✅ Easy to use, limited to vendor pipeline | ❌ Slow / complex              |
| Flexibility               | ✅ Very high     | ✅ Very high         | ✅ High            | ⚠️ Limited                                | ✅ High, but hardware-specific |

### Summary

* **VPU / ASIC** → ideal for simple, low-power on-device dense depth
* **GPU** → strong acceleration for large-scale host-side processing
* **CPU (ARM)** → best for flexible edge-side control, synchronization, and custom RAW pipelines
* **CPU (high-end host)** → ideal for deterministic, controllable, host-side stereo processing and multi-rig systems
* **FPGA** → best suited for low-latency, highly deterministic, high-speed stereo pipelines once the architecture is stable

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