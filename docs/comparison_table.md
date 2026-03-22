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
| **Deterministic timing layer**                |           🔴           |             🔴            |     🟡 (industrial sync support)    |             🔴             |             🟢             |              🔴              |         🔴        |                 🟢                 |
| **Linux-based edge OS on device**             |           🔴           |             🔴            |                  🔴                 |             🔴             |             🔴             |              🔴              |         🔴        |                 🟢                 |
| **Open-source core**                          |           🔴           |             🔴            |                  🔴                 |             🔴             |             🔴             |              🔴              |         🔴        |                 🟢                 |
| **Typical depth range (manufacturer stated)** |       ~0.3–20 m*       |         ~0.4–6 m*         |              ~0.3–10 m*             |          ~0.1–1 m*         |         ~0.2–20 m*         |          ~0.2–10 m*          |    ~0.15–10 m*    |             0.1–10 m**             |

---

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

## OAK (Luxonis)

OAK is a strong company, and in many ways its approach is similar to EdgeTrack.

However, there are still some important differences:

| Topic           | OAK (Luxonis)                          | EdgeTrack                                                                 |
| --------------- | -------------------------------------- | ------------------------------------------------------------------------- |
| Overall concept | Strong integrated stereo/AI platform   | Geometry-first tracking architecture with optional AI                     |
| Hardware model  | Partly platform-locked                 | More hardware-agnostic and adaptable across many PC and ARM-based systems |
| Flexibility     | Good, but within the Luxonis ecosystem | More open to custom architecture                                          |
| Focus           | Ready-made device pipeline             | Direct control over timing, RAW data, and ROI-based processing            |

**One of EdgeTrack’s main advantages is direct RAW access close to memory, especially for zero-copy processing. This allows a more efficient low-latency pipeline. In contrast, with OAK, custom host-side processing is more dependent on transferring data out of the device pipeline first, which can be less efficient for strict RAW-first architectures.**

> Note: By the way, although Luxonis often presents DepthAI as its overall platform, their documentation suggests that the main and more broadly supported depth pipeline is still the classical StereoDepth approach rather than a neural one. A neural alternative called NeuralDepth also exists, but it is more limited in hardware support and operating range, which suggests practical trade-offs compared with the classical stereo pipeline.