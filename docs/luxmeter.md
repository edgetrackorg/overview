# No Luxmeter Required – LED Power Test Using RAW10

Goal: **quickly verify** whether your IR LED illumination is **too weak / sufficient / excessive** using only the camera image (RAW10) — without a dedicated lux meter.

This method is intended for typical NIR setups (e.g. **850 nm + bandpass filter**, **90° matte/diffuse optics**, **stereo / tracking systems**).

---

## Basic Idea

Use a **fixed reference setup** (recommended: **1.0 m distance**) and measure the **RAW10 grayscale value** on a **white matte paper** target.

This gives you a reproducible reference.
You can later scale to other distances mathematically instead of “guessing” how many LEDs you need.

RAW10 range: **0…1023**

---

## Why Not “Walk Back Until the Image Turns Black”?

“Black” is subjective and depends heavily on:

- Auto Exposure / Auto Gain
- ISP processing / gamma / tone mapping
- Room reflections (walls reflect IR)
- Display brightness

For meaningful comparison you need:

**Fixed camera settings + defined target + ROI mean value**

---

## Reference Target

- **White matte paper** (standard printer paper works well)
- Positioned **as perpendicular as possible** to the camera
- Optional: add **gray and black surfaces** for contrast / worst-case testing

---

## Lock Camera Settings (Mandatory)

To ensure reproducibility:

- **Disable Auto Exposure**
- **Disable Auto Gain**
- **Fix exposure time**, e.g. **2.0 ms**
  *(Important: set this in the camera control stack, e.g. libcamera or your own module.)*
- **Fix gain** (moderate, not near maximum)
- Use **RAW / linear output** (no gamma, no ISP tone mapping)
- **Define a black reference level**: measure a dark/black target and aim for **≤ 30 / 1023 (RAW10)** in the ROI
  *(This helps detect excessive ambient IR, flare, or internal reflections. Note: the exact number is a rule of thumb.)*
- **Warm up the system for at least 10 minutes** before taking measurements (LED output and sensor behavior drift with temperature)
- **Repeat the test at different temperatures** (or after thermal soak) to verify worst-case performance

Without fixed exposure and gain, the camera will compensate brightness automatically, making LED evaluation impossible.

---

## Recommended RAW10 Target Levels (Rule of Thumb)

All values below assume:
- **1.0 m reference distance**
- **white matte paper** as the target
- exposure/gain fixed (no auto)

> Measure the **ROI mean** (e.g. a 100×100 pixel region), not the brightest pixel.

### Quick rating

- **~100 / 1023** → *not optimal* (bare minimum, low SNR / low margin)
- **~200 / 1023** → *weak* (works in controlled cases, little reserve)
- **~500 / 1023** → **great** (high SNR, good margin for typical use)
- **~1000 / 1023** → *too bright / near clipping* (often overpowered, risk of saturation)

### Practical interpretation (example use case up to ~1.2 m)

- If you get **~500 at 1.0 m**, you typically have solid margin for a **~1.2 m** working distance.
- If you get **~1000 at 1.0 m**, and you only need **up to ~1.2 m**, your system is likely **overpowered** (you can dim down, reduce LED current, or reduce LED count).

> Note: These are **rules of thumb**. Dark targets, corner falloff, diffuser losses, bandpass transmission, LED temperature, and sensor differences can shift the “best” value.
