# Comparison of Available Hardware on the Market

> **Note:** This comparison focuses on architectural design principles and system integration models.
> Feature availability varies by firmware version, SDK access level, and configuration.
> Specifications are summarized at a high level and may differ by exact model variant.

---

| Feature / Focus                               |         ZED 2i         |   RealSense (e.g., D455)  |          Bumblebee X 5GigE          | Leap Motion (Controller 2) |          OptiTrack         |  Basler Stereo (stereo ace)  | Orbbec (Gemini 2) |              EdgeTrack             |
| --------------------------------------------- | :--------------------: | :-----------------------: | :---------------------------------: | :------------------------: | :------------------------: | :--------------------------: | :---------------: | :--------------------------------: |
| **Primary use case**                          |  Robotics / XR / Depth |       Depth sensing       |       Industrial stereo depth       |        Hand tracking       |     Marker-based MoCap     |    Industrial stereo depth   |   Depth sensing   | Editor authoring / pro interaction |
| **Typical interface**                         |           USB          |            USB            |           5GigE (Ethernet)          |             USB            |   Ethernet (system-based)  | GigE / USB (model dependent) |        USB        |        Ethernet (multi-rig)        |
| **Stereo depth**                              |           🟢           |             🟢            |                  🟢                 |           🔴***            |             🔴             |              🟢              |         🟢        |                 🟢                 |
| **On-device depth compute**                   |  🔴 (host GPU typical) | 🟢 (dedicated depth ASIC) |   🟢 (on-board stereo processing)   |             🔴             |             🔴             |  🟢 (camera-based disparity) |  🟢 (custom ASIC) |     🟢 (ROI-based edge compute)    |
| **AI/VPU-style accelerator**                  |           🔴           |  🔴 (depth ASIC ≠ AI VPU) |                  🔴                 |             🔴             |             🔴             |              🔴              |         🔴        |    Optional (platform-dependent)   |
| **FPGA-based stereo pipeline**                |           🔴           |             🔴            |  🟢 (industrial hardware pipeline)  |             🔴             |             🔴             |              🔴              |         🔴        |                 🔴                 |
| **Open RAW sensor access**                    | 🟡 (limited SDK modes) | 🟡 (not typical workflow) | 🟢 (12-bit rectified stereo option) |             🔴             |             🟡             |              🟢              |       🔴/🟡       |                 🟢                 |
| **Native multi-device fusion**                |           🔴           |             🔴            |                  🔴                 |             🔴             | 🟢 (system-level software) |              🔴              |         🔴        |                 🟢                 |
| **Deterministic timing layer**                |           🔴           |             🔴            |     🟡 (industrial sync support)    |             🔴             |             🟢             |              🔴              |         🔴        |                 🟢      --           |
| **Linux-based edge OS on device**             |           🔴           |             🔴            |                  🔴                 |             🔴             |             🔴             |              🔴              |         🔴        |                 🟢                 |
| **Open-source core**                          |           🔴           |             🔴            |                  🔴                 |             🔴             |             🔴             |              🔴              |         🔴        |                 🟢                 |
| **Typical depth range (manufacturer stated)** |       ~0.3–20 m*       |         ~0.4–6 m*         |              ~0.3–10 m*             |          ~0.1–1 m*         |         ~0.2–20 m*         |          ~0.2–10 m*          |    ~0.15–10 m*    |             0.1–10 m**             |

## Footnotes

\* Manufacturer-stated operational ranges under ideal conditions.
Actual performance depends on lighting, surface texture, calibration, and environmental conditions.

\** EdgeTrack is optimized for **high-precision operation in the near field (≤ ~1.2 m)** using homogeneous NIR flood illumination.
Extended ranges up to ~10 m are configuration-dependent and may require:

* VCSEL dot-pattern projection (active stereo assist)
* Increased baseline
* Higher-power NIR flood illumination
* Environment-dependent neural stereo refinement

Depth precision and usable range depend on baseline geometry, optics, illumination design, and processing strategy.
For transparency, geometric performance relationships are documented separately via disparity-based calculation tools.

\*** Leap Motion uses two cameras, but it is not a conventional stereo disparity system.

---

# OAK (Luxonis)

OAK is a strong company, and in many ways its approach is similar to EdgeTrack.

However, there are still some important differences:

| Topic           | OAK (Luxonis)                          | EdgeTrack                                                                 |
| --------------- | -------------------------------------- | ------------------------------------------------------------------------- |
| Overall concept | Strong integrated stereo/AI platform*  | Geometry-first tracking architecture with optional AI                     |
| Hardware model  | Partly platform-locked                 | More hardware-agnostic and adaptable across many PC and ARM-based systems |
| Flexibility     | Good, but within the Luxonis ecosystem | More open to custom architecture                                          |
| Focus           | Ready-made device pipeline             | Direct control over timing, RAW data, and ROI-based processing            |

**One of EdgeTrack’s main advantages is direct RAW access close to memory, especially for zero-copy processing. This allows a more efficient low-latency pipeline. In contrast, with OAK, custom host-side processing is more dependent on transferring data out of the device pipeline first, which can be less efficient for strict RAW-first architectures.**

> \* Although Luxonis often presents DepthAI as its overall platform, their documentation suggests that the main and more broadly supported depth pipeline is still the classical StereoDepth approach rather than a neural one. A neural alternative called NeuralDepth also exists, but it is more limited in hardware support and operating range, which suggests practical trade-offs compared with the classical stereo pipeline.

---

# RAW Data Stream

This section compares several ways to stream **RAW image data** from a camera or edge device to a host system.
The goal is to evaluate which transport path is more suitable for **low latency**, **low jitter**, and **high-performance stereo processing**.

In this context, the comparison focuses on the transport stage only — not on stereo reconstruction itself.

## General observation

A **direct industrial-camera-to-host path** usually provides the **lowest latency** and **lowest jitter**, because it avoids the extra buffering, packetization, and OS scheduling stage introduced by an intermediate ARM node. By contrast, an ARM-based capture node can offer more flexibility and custom control, especially when using a lean V4L2-based stack instead of a heavier camera framework. 

## Approximate Latency and Jitter Comparison

> These values are **practical engineering estimates**, not fixed vendor guarantees.
> Real-world results depend on sensor mode, buffering, driver behavior, packet size, host load, and network configuration.

| Path                                                         | Approx. additional transport / pipeline latency |        Approx. jitter | Positioning                                                                                                                                                    |
| ------------------------------------------------------------ | ----------------------------------------------: | --------------------: | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **CoaXPress industrial camera → host**                       |                             **sub-ms to ~2 ms** |          **very low** | Best for high-speed, deterministic machine vision                                                                                                              |
| **Industrial camera GigE → host**                            |                                     **~2–5 ms** |   **low to moderate** | Strong balance of cable length, standardization, and host-side processing                                                                                      |
| **Industrial camera USB3 → host**                            |                                     **~1–4 ms** |   **low to moderate** | Very fast and direct, but limited by shorter cable length and stronger host dependency.                                                                        |
| **ARM node with libcamera → host**                           |                                    **~5–15 ms** |          **moderate** | Flexible and open, but adds software and buffering overhead                                                                                                    |
| **ARM node with custom lean V4L2 stack → host**              |                                    **~3–10 ms** | **moderate to lower** | Reduced overhead compared to libcamera when optimized                                                                                                          |
| **ARM node with RTLinux + lean V4L2 stack → host**           |                                     **~2–8 ms** |   **low to moderate** | Further optimized for reduced scheduling delay and improved determinism                                                                                        |
| **ARM node with RTLinux + lean V4L2 stack → on device**      |                                 **~0.2–1.5 ms** |          **very low** | Reference only; direct MIPI CSI-2 capture and processing on the same device, with only minimal internal transport overhead and no external interface transport |
| **Consumer USB camera → host**                               |                                  **~50–300 ms** |            **higher** | Reference only; useful as a consumer baseline, but not suitable for deterministic low-latency vision pipelines                                                 |

---

## Why These Numbers Differ

* **CoaXPress** is designed for very high throughput and strong real-time behavior. Reported trigger latencies in the microsecond range illustrate its highly deterministic nature, although these values refer to trigger paths rather than full frame transport.

* **GigE** and **USB3** camera pipelines are still direct host paths. However, transmission timing can vary depending on host-side scheduling and when data transfer is initiated, introducing some jitter.

* **libcamera** on ARM platforms (e.g., Raspberry Pi) includes a full pipeline with image processing (auto exposure, white balance, lens correction). While convenient, this adds complexity and additional latency compared to minimal RAW capture paths.

* A **custom V4L2 streaming implementation** can significantly reduce overhead. Using `mmap` allows buffer sharing without copying image data, which is beneficial for low-latency RAW streaming.

* A **RTLinux** can further reduce scheduling delays and jitter by improving task prioritization, interrupt handling, and CPU isolation. However, it does not eliminate the fundamental overhead of the additional ARM processing stage.

---

## Advantages and disadvantages

### Industrial camera → host

**Pros**

* Lower latency
* Lower jitter
* More deterministic transport path
* Fewer software stages between sensor and host
* Better suited for high-speed and timing-critical systems

**Cons**

* Usually more expensive
* Often more vendor-specific
* Less flexible for custom edge-side preprocessing
* Integration may depend more on vendor SDKs and interface standards

### ARM node → host

**Pros**

* More flexible and open architecture
* Easier to integrate custom logic, synchronization, or preprocessing
* Good fit for distributed stereo or multi-rig systems
* Can be lower cost than industrial camera systems

**Cons**

* Higher latency
* Higher jitter
* Additional buffering and packetization stage
* More dependent on software-stack quality and OS scheduling behavior

---

## Practical conclusion

If the main goal is **lowest latency and lowest jitter**, a **direct industrial-camera-to-host** connection is usually the stronger option.

If the main goal is **architectural flexibility, open control, and custom edge-side logic**, an **ARM-to-host streaming path** remains very attractive — especially when implemented with a **lean V4L2-based library** rather than a heavier framework.

A good summary is:

> **Industrial cameras are generally faster and more deterministic, while ARM-based stream nodes are generally more flexible and easier to adapt to custom multi-rig architectures.**

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