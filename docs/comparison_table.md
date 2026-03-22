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

# Data RAW stream

This section compares several ways to stream **RAW image data** from a camera or edge device to a host system.
The goal is to evaluate which transport path is more suitable for **low latency**, **low jitter**, and **high-performance stereo processing**.

In this context, the comparison focuses on the transport stage only — not on stereo reconstruction itself.

## General observation

A **direct industrial-camera-to-host path** usually provides the **lowest latency** and **lowest jitter**, because it avoids the extra buffering, packetization, and OS scheduling stage introduced by an intermediate ARM node. By contrast, an ARM-based capture node can offer more flexibility and custom control, especially when using a lean V4L2-based stack instead of a heavier camera framework. 

## Approximate latency and jitter comparison

> These values are **practical engineering estimates**, not fixed vendor guarantees.
> Real results depend on sensor mode, buffering, driver behavior, packet size, host load, and network configuration.

| Path                                            | Approx. additional transport / pipeline latency |        Approx. jitter | Positioning                                                               |
| ----------------------------------------------- | ----------------------------------------------: | --------------------: | ------------------------------------------------------------------------- |
| **CoaXPress industrial camera → host**          |                             **sub-ms to ~2 ms** |          **very low** | Best for high-speed, deterministic machine vision                         |
| **Industrial camera GigE → host**               |                                     **~1–5 ms** |   **low to moderate** | Strong balance of cable length, standardization, and host-side processing |
| **Industrial camera USB3 → host**               |                                     **~1–4 ms** |   **low to moderate** | Very fast and direct, but shorter cable range and host dependency         |
| **ARM node with libcamera → host**              |                                    **~5–15 ms** |          **moderate** | Flexible and open, but adds software and buffering overhead               |
| **ARM node with custom lean V4L2 stack → host** |                                    **~3–10 ms** | **moderate to lower** | Better than libcamera-based streaming when optimized for minimal overhead |

### Why these numbers differ

* **CoaXPress** is designed for very high throughput and strong real-time behavior. Adimec reports about **3.4 µs latency** with about **4 ns jitter** for triggering over CoaXPress, and Euresys describes CoaXPress trigger latency as **under 0.5 µs**. These figures refer to the trigger path rather than full frame transport, but they illustrate why CoaXPress is generally considered the most deterministic option. ([Adimec][2])
* **GigE** and **USB3** camera pipelines are still direct host paths, but Basler notes that for both interfaces the **transmission start delay can vary between frames** and depends on when the host calls for data transmission. ([Basler Produktdokumentation][1])
* **libcamera** on Raspberry Pi includes a pipeline handler plus image-processing algorithms such as auto exposure, auto white balance, and lens-shading correction, which makes it convenient but can add complexity and overhead compared with a minimal capture path. ([Raspberry Pi][3])
* A **custom V4L2 streaming path** can reduce overhead because V4L2 streaming with `mmap` exchanges **buffer pointers instead of copying image data**, which is more suitable for low-latency RAW capture. ([Linux-Kernel-Archive][4])

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