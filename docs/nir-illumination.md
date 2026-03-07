# NIR Illumination

Near-Infrared (NIR) illumination plays an important role in the EdgeTrack system.
It improves stereo matching stability by creating controlled lighting conditions that are independent of visible ambient light.

EdgeTrack primarily uses **850 nm or 940 nm infrared illumination**, combined with optical filtering and synchronized exposure timing.

This allows stereo cameras to operate reliably even when visible lighting conditions are inconsistent.

---

# Why NIR Illumination

Stereo vision relies on **image texture and contrast** to match corresponding points between the left and right camera images.

In many environments, visible light conditions can change rapidly due to:

* ambient lighting variations
* shadows
* reflections
* background clutter

Controlled NIR illumination allows the system to create a **stable and predictable visual environment** for stereo reconstruction.

Advantages include:

* improved surface contrast
* reduced dependency on ambient lighting
* consistent reconstruction quality
* better repeatability

---

# Typical Wavelengths

EdgeTrack systems commonly use two NIR wavelength ranges:

| Wavelength | Characteristics                                                          |
| ---------- | ------------------------------------------------------------------------ |
| **850 nm** | Higher sensor sensitivity, stronger signal, slight visible red glow      |
| **940 nm** | Completely invisible to the human eye, slightly lower sensor sensitivity |

Both wavelengths can be used depending on application requirements.

---

# Global Shutter and Illumination Timing

EdgeTrack systems typically use **global shutter sensors**.

Global shutter capture allows the entire frame to be exposed simultaneously, which is essential for precise stereo matching.

When combined with **synchronized NIR illumination pulses**, the system can:

* maximize signal strength
* reduce motion blur
* minimize interference from ambient light

Illumination is typically synchronized with the camera exposure window.

---

# Optical Filtering

Optical filters help isolate the NIR signal and suppress unwanted visible light.

Typical filter components include:

* **NIR bandpass filters** (e.g. 850 nm bandpass)
* **IR-pass filters**
* **polarization filters**

Filtering improves the signal-to-noise ratio for stereo matching.

---

# Polarization

In some environments, strong specular reflections can reduce stereo matching stability.

Polarization filters can help mitigate this problem.

Using cross-polarized illumination and camera filters can reduce:

* glare
* reflective hotspots
* unstable disparity results

Polarization is optional but useful for difficult surfaces such as:

* glossy plastics
* polished metal
* glass-like materials

---

# Controlled Indoor Environments

In controlled indoor environments, stereo systems with NIR illumination perform extremely well.

Ideal conditions include:

* no direct sunlight
* stable NIR illumination
* minimal specular reflections
* moderate surface texture

Typical failure modes may still occur with:

* very low-texture materials
* strong reflections
* severe occlusions

These limitations are inherent to stereo vision systems.

---

# Multi-Rig Illumination

In multi-rig systems, multiple NIR emitters may operate in the same environment.

To prevent interference between rigs, EdgeTrack supports **timing coordination using TDM (Time Division Multiplexing)**.

Each rig can emit NIR illumination in a separate time slot.

Example:

```
Rig 1 → illumination slot A
Rig 2 → illumination slot B
Rig 3 → illumination slot C
Rig 4 → illumination slot D
```

This ensures that cameras receive illumination only from their own system.

---

# Hardware Implementation

Typical illumination hardware includes:

* high-power NIR LEDs
* LED driver circuits
* current-controlled pulse drivers
* optional VCSEL emitters

Illumination systems may be integrated directly into the camera housing or deployed as separate light modules.

Proper thermal management is important when operating high-power emitters.

---

# Design Philosophy

EdgeTrack treats illumination as a **controlled system component**, not merely an environmental factor.

By combining:

* synchronized NIR illumination
* RAW image capture
* deterministic timing
* optical filtering

the system achieves stable and reproducible stereo reconstruction.

This approach improves tracking reliability and reduces dependence on unpredictable ambient lighting conditions.

---