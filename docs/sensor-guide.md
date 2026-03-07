# 🎥 Recommended Sensor Options

for 3D Hand-Tracking Reconstruction

---

## 📷 Key sensor parameters

* **Type**

  * ✅ **Monochrome sensors (preferred)** – maximum sharpness, no Bayer debayering, better NIR sensitivity.
  * ❌ **RGB color** – for debugging/visualization only; not recommended for precise 3D reconstruction.
  * *Note:* An **RGB camera placed between two mono cameras** can still be useful as an auxiliary channel for **calibration support** and **texture/context cues** (without driving the core 3D reconstruction).

* **Shutter**

  * ✅ **Global shutter** – required for precise 3D, fast motion, and synchronized multi-view rigs.
  * ❌ **Rolling shutter** – causes distortions with motion and unsynchronized exposures.

* **Sync / Trigger I/O**

  * ✅ **Required.** The sensor **must** support external sync/trigger (input/output), driven by our MCU / TDM controller.
  * ❌ **No trigger support** – not acceptable for this project.
  * Goal: **frame-accurate synchronization** of all cameras in the rig.
  * *Nice to have:* a STROBE / exposure-active output that can be used as a hardware feedback signal (trigger accepted / exposure occurred).

* **Pixel format**

  * ✅ **RAW10** or **RAW12**.
  * ❌ **MJPEG** or full **YUV color formats** – only for preview/debug, **not** for the main 3D/measurement pipeline.
  * **RAW12** offers better quantization (more effective intensity/depth levels), at the cost of higher bandwidth and compute.
  * **Fallback:** **RAW8 / Y8** is acceptable for low-cost / low-bandwidth prototypes.
 
* **Spectral / optics**

  * ✅ **CMOS without IR-cut filter** – full NIR sensitivity.
  * ❌ **IR-cut filter** – not recommended; it blocks most NIR light and severely reduces signal at 850 
  * Use an **850 nm band-pass filter** with matching NIR illumination (LED/VCSEL).
  * Goal: stable contrast in NIR, robust against visible-light textures, colors, and ambient light.
  * Optional: If **specular reflections** become an issue, consider using a **polarizer** (only helps in certain geometries and may reduce overall signal).

* **Sensor size**

  * 🎯 Preferred: **1/4"** – good trade-off between cost, size, and lens availability.
  * Optional: **~1/2.5"** – more light, better SNR, more optical flexibility, but higher cost and larger form factor.

* **Frame rate**

  * 🎯 Target: **≥ 120 FPS** (for fast gestures, low latency, and better temporal fusion).
  * Acceptable for early development: **≥ 60 FPS**.

* **Resolution**

  * 🎯 Minimum: **≥ 1280 × 800**.
  * Higher resolutions (**≥ 2 MP**) only if:

    * your full pipeline can handle **RAW10/RAW12** at higher bandwidth, and
    * the additional compute load (stereo matching / pose estimation) is planned for.

* **Interface**

  * 🎯 **Lowest latency / best determinism: MIPI-CSI**

  * Alternatives (valid, but not my primary use case):

    * **USB3 Vision**
      → Good for single industrial cameras; cable length is limited without active extenders/repeaters.
    * **GigE Vision (≥ 2.5 GbE)**
      → Robust over longer distances; PoE possible; scales well for distributed/industrial rigs.
    * **SFP+**
      → Great for very high bandwidth and long runs, but typically requires custom hardware and is rarely fully off-the-shelf.
    * **CoaXPress**
      → Top-tier on bandwidth, latency, and cable length, but significantly more expensive and complex to integrate.

* **Field of View (FOV)**

  * 🎯 Target range: **60–120°**, depending on:

    * rig geometry,
    * working distance,
    * desired measurement volume (e.g., hand vs. full upper body).
  * For detailed formulas and examples, see: **[Vision Geometry Rules](https://github.com/xtanai/geo_rules)**.

---

## 📍 Recommended sensors (available modules)

Below are practical, readily available **global-shutter** camera modules that work well for NIR / tracking / stereo on Raspberry Pi (CSI-2 MIPI).

> **Lens note:** Very wide lenses (e.g. **130°**) are usually **not recommended** for stereo/tracking because they reduce per-pixel energy (darker image), increase distortion, and make illumination harder.  
> For most setups, **~70–80° FoV** is a good starting point (depending on working distance and baseline).

### ⚡ Up to ~120 FPS (1 MP class)

**OV9281 (1 MP) mono global shutter**
- MyBotShop (ArduCAM OV9281, 130° lens)  
  [Link](https://www.mybotshop.de/ArduCAM-1MP-OV9281-Mono-Global-Shutter-130deg-M12)
  **Note:** The included **130° lens is not recommended** → swap to **~70–80°** FoV if possible.
- Inno-Maker OV9281 module  
   [Link](https://www.inno-maker.com/product/cam-mipiov9281/)
- Inno-Maker OV9281 RAW v2  
   [Link](https://www.inno-maker.com/product/cam-mipi9281raw-v2/)

**Why OV9281?**
- Very common in stereo / tracking projects
- Good mode support and high frame rates (depending on ROI / sensor modes)
- Mono + no IR-cut options available on many modules

### 🎞️ Up to ~60 FPS (higher resolution)

**OV2311 (2 MP) mono global shutter**
- ArduCAM OV2311 for Raspberry Pi  
  [Link](https://www.arducam.com/arducam-ov2311-mipi-2mp-mono-global-shutter-camera-module-for-raspberry-pi.html)

**IMX296 (1.58 MP) global shutter**
- ArduCAM IMX296 module  
  [Link](https://www.arducam.com/1-58mp-imx296-mono-global-shutter-camera-module-with-m12-lens-for-raspberry-pi.html)
- Inno-Maker IMX296 RAW + trigger variant  
  [Link](https://www.inno-maker.com/product/cam-mipi296raw-trigger/)
  
**Why these?**
- Higher resolution compared to OV9281
- Larger optical format than typical 1/4" sensors (better light collection per frame)
- Useful when you need more spatial detail and can accept a lower maximum frame rate

### 🧩 Other / stereo kits (caution)

**ArduCAM dual OV9281 wide-angle stereo kit**
-  [Link](https://www.arducam.com/arducam-1mp2-wide-angle-stereo-camera-for-raspberry-pi-jetson-nano-and-xavier-nx-dual-ov9281-monochrome-global-shutter-camera-module.html)

⚠️ **Note:** This kit is often **not ideal** for synchronized IR strobe / trigger use-cases, because it does **not provide a proper external trigger/strobe interface** (depends on variant).  
For time-multiplexed strobing (TDM) you typically want modules with **trigger input** or a well-defined sync mechanism.

### 💡 General advice
- Prefer **mono + no IR-cut** for 850 nm NIR illumination.
- Prefer camera modules with **trigger/sync support** if you plan IR strobes (especially TDM).
- Plan lens FoV together with your illumination: **wide FoV = more LED power needed**.


---

## 🧭 Mono Sensors on the Market (MIPI CSI-2)

For **mono global-shutter stereo with hardware trigger** on Raspberry Pi (MIPI CSI-2), there are only **a few** affordable and widely available options. The most accessible and practical modules today are typically based on **OV9281**, **Sony IMX296LLR**, and **onsemi AR0145**. These sensors are common in the Raspberry Pi ecosystem and work well for early development, prototyping, and validation.


### Current Prototype

The current prototype uses OV9281-, IMX296LLR, AR0145-based modules, currently configured with a **limited CSI-2 lane setup**. This is ideal for development and validation, but it does **not fully utilize** the Raspberry Pi 5’s available CSI bandwidth and leaves less headroom for higher-throughput capture modes.

### Long-Term Goal: Higher-Bandwidth RAW Sensors (Transparent Capture Path)

For the next generation, I’m targeting a **RAW-first sensor path** with **transparent, deterministic control**—meaning no opaque or mandatory processing stages in the capture chain. The goal is to take full advantage of **4-lane CSI-2 on Raspberry Pi 5**, increasing throughput and leaving more headroom for advanced modes and future upgrades.

Today’s sensors and modes are still bounded by the Raspberry Pi 5 CPU budget, so the current design stays realistic and avoids “overbuilding” features that the Pi 5 cannot efficiently run. At the same time, the pipeline is structured to be future-ready: as soon as a faster platform (e.g., a future Pi 6) becomes available, the same RAW-first architecture can scale to higher-end sensors and more demanding modes without needing a major redesign.

Securing the right sensor/module partner is feasible—but it depends on **validated demand**.

### Open, Repeatable Platform (MotionCoder-first)

This is not meant to be closed or exclusive. The platform should remain **repeatable and widely adoptable**, because the long-term value sits in the software layer—**MotionCoder**—which benefits most from a broadly available, standardized capture backend.

### Feasibility & Manufacturing Reality

From a system perspective, this direction is realistic: a RAW-first camera path can keep the electronics BOM lean, while Linux-based edge platforms provide strong, direct control over capture, timing, and the imaging pipeline.

The main challenge is not “extra chips” - it’s **reliable module sourcing**, **mechanical integration**, **QC**, and **calibration**. That’s exactly what demand validation helps unlock.

Based on current assumptions, I estimate the sensor/module cost can be kept below **~€100 per unit**, but this depends heavily on volume. In many cases, a **MIPI CSI-2** camera path can be **significantly cheaper than typical USB webcams**, because the electronics BOM is often leaner (fewer bridge/ISP components and a more direct sensor-to-SoC connection). Achieving stable pricing and supply, however, usually requires a meaningful production run (on the order of **~1,000 units**).

### Vote / Demand Signal (Call to Action)

If you want **higher-end mono sensors** (more bandwidth, better optics support, stronger long-term supply), please leave a **vote / comment**. I’m collecting demand signals to move forward with supplier negotiations, final sourcing, and production commitments.

Once demand is validated, I can confidently push this to production—so you can **swap camera modules in EdgeTrack** and keep using the same high-end capture backend as sensors evolve.

---




