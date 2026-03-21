# EdgeTrack on FPGA

This is a possible future direction for EdgeTrack and is not part of the current main implementation. The idea behind an FPGA-based version is to achieve significantly lower latency, stronger determinism, and more reliable real-time behavior, especially for high-speed stereo processing and motion-critical interaction.

At the moment, EdgeTrack runs on ARM-based hardware such as the Raspberry Pi 5, which is already a practical and capable solution. It is flexible, accessible, and sufficient for many geometry-first applications. In this form, EdgeTrack can already support a wide range of experiments and real-world use cases.

An FPGA-based variant would not aim to replace the current architecture, but to specialize it further. Its main purpose would be to reduce latency, minimize jitter, and offload time-critical stereo and trigger-related tasks directly onto dedicated hardware. This would be particularly relevant for applications where responsiveness and deterministic timing are more important than architectural simplicity or low development effort.

In short, the current ARM-based EdgeTrack remains the main usable platform today, while an FPGA-based version is a future performance-oriented path focused on ultra-low-latency operation.

### 📝 Note: EdgeTrack on ARM (e.g., Pi 5) vs. FPGA-based EdgeTrack

**EdgeTrack on ARM should not be seen as an inferior version of FPGA-based EdgeTrack.**
When properly configured, it already provides a practical, capable, and geometry-first solution for many real-world applications.

**FPGA-based EdgeTrack** is a possible future direction focused on **lower latency**, **higher determinism**, and **stronger real-time performance**. It is intended as a specialized hardware path, not as a general replacement for the ARM-based version.

At present, **the ARM-based implementation is the main directly usable EdgeTrack solution**.
The **FPGA-based variant** remains a possible future internal development path, depending on how ARM platforms evolve in performance. See the comparison table below.

### Architecture Comparison (Estimated)

| Variant                                            | Estimated End-to-End Latency |             Jitter | 120 FPS Feasible?                                                  | Compute Headroom           | Development Effort | Strength                                                           | Weakness                                                               |
| -------------------------------------------------- | ---------------------------: | -----------------: | ------------------------------------------------------------------ | -------------------------- | ------------------ | ------------------------------------------------------------------ | ---------------------------------------------------------------------- |
| **2× Camera + ARM (e.g., Pi 5)**                   |                 **12–30 ms** |     medium to high | limited                                                            | low to medium              | low                | Fastest to start, highly flexible                                  | CPU can become a bottleneck                                            |
| **2× Camera + FPGA (on-device)**                   |                  **3–10 ms** |           very low | yes, most reliable                                                 | high (streaming / ROI)     | very high          | Best responsiveness, highly deterministic                          | Highest development complexity                                         |
| **2× Camera + ARM → Host processing (CoreStereo)** |                  **8–20 ms** |             medium | yes, better than ARM-only                                          | high (host-side)           | medium             | Best practical compromise                                          | Network transport and synchronization add complexity                   |
| **2× Camera + Future higher-performance ARM**      |                  **8–20 ms** | potentially medium | possibly, depending on future platform performance                 | potentially medium to high | low to medium      | Could combine simpler architecture with improved local performance | Not yet proven and highly platform-dependent                           |
| **2× USB Cameras → Host processing**               |               **50–200+ ms** |     medium to high | possible, but highly dependent on camera, driver, and USB pipeline | high (host-side)           | low to medium      | Very easy to prototype with commodity hardware                     | Higher latency, weaker determinism, and additional USB/driver overhead |

> ⚠️ Note: These values are **not measured benchmarks**, but **realistic engineering estimates** based on system architecture, data flow, and typical performance characteristics.