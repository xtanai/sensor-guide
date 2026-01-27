# 🎥 Recommended Sensor Options

for 3D Hand-Tracking Reconstruction

**Short note — not recommended:** Standard **RGB webcams**, **Meta Quest 3**, **iPhone 15 Pro Max**, etc. (rolling shutter, auto exposure/gain, weak IR response) are fine for experiments but **unsuitable for precise hand tracking**. Likewise, **depth solutions** (LiDAR/ToF/active stereo — e.g., Intel RealSense, Kinect) are often **ill-suited** to fine hand/finger work: **relatively expensive**, **lower/variable FPS**, and **insufficient precision** for **parametric CAD commands**.

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

* **Pixel format**

  * ✅ **RAW10** or **RAW12** – preferred (see rating table above).
  * ❌ **MJPEG** or full **YUV color formats** – only for preview/debug, **not** for the main 3D/measurement pipeline.
  * **RAW12** offers better quantization (more effective intensity/depth levels), at the cost of higher bandwidth and compute.
  * **Fallback:** **RAW8 / Y8** is acceptable for low-cost / low-bandwidth prototypes.
 
* **Spectral / optics**

  * ✅ **CMOS without IR-cut filter** – full NIR sensitivity.
  * ❌ **IR-cut filter** – not recommended; it blocks most NIR light and severely reduces signal at 850 
  * Use an **850 nm band-pass filter** with matching NIR illumination (LED/VCSEL).
  * Goal: stable contrast in NIR, robust against visible-light textures, colors, and ambient light.

* **Sensor size**

  * 🎯 Preferred: **1/4"** – good trade-off between cost, size, and lens availability.
  * Optional: **1/3"** or **1/2"** – more light, better SNR, more optical flexibility, but higher cost and larger form factor.

* **Frame rate**

  * 🎯 Target: **≥ 120 FPS** (for fast gestures, low latency, and better temporal fusion).
  * Acceptable for early development: **≥ 60 FPS**.

* **Resolution**

  * 🎯 Minimum: **≥ 1280 × 800**.
  * Higher resolutions (**≥ 2 MP**) only if:

    * your full pipeline can handle **RAW10/RAW12** at higher bandwidth, and
    * the additional compute load (stereo matching / pose estimation) is planned for.

* **Interface priority**

  1. **MIPI-CSI** (RPi 5 compatible)
     → Short flex cables, compute module close to the sensors; TDM/multiplexer for multi-camera setups if needed.
  2. **USB3 Vision**
     → Good for single industrial cameras; cable length limited without active extenders/repeaters.
  3. **GigE Vision (≥ 2.5 GbE)**
     → Robust over longer distances; PoE possible; scales well for industrial rigs.
  4. **SFP+**
     → Ideal for very high bandwidth and long distances, but requires custom hardware and is rarely “off-the-shelf”.
  5. **CoaXPress**
     → Technically top tier (bandwidth, latency, cable length), but significantly more expensive and complex.

* **Field of View (FOV)**

  * 🎯 Target range: **60–120°**, depending on:

    * rig geometry,
    * working distance,
    * desired measurement volume (e.g., hand vs. full upper body).
  * For detailed formulas and examples, see: **[Vision Geometry Rules](https://github.com/xtanai/geo_rules)**.

---

## ⭐ Pixel format – preference

| Format                     | Rating | Comment                                                                                  |
| -------------------------- | ------ | ---------------------------------------------------------------------------------------- |
| **MJPEG / JPEG**           | ★☆☆☆☆☆ | Only for preview/debug. Strong artifacts, variable bitrate, poor for precise 3D.         |
| **YUV / YUYV / NV12**      | ★★☆☆☆☆ | OK if you only use the **Y (luma)** channel. Extra bandwidth wasted on color info.       |
| **RAW8 / Y8 (8-bit mono)** | ★★★☆☆☆ | Solid baseline. Lower dynamic range, but good enough with proper NIR illumination.       |
| **RAW10**                  | ★★★★☆☆ | Very good: higher dynamic range, finer quantization, still manageable bandwidth.         |
| **RAW12**                  | ★★★★★★ | Ideal for high precision: maximum dynamic range and depth resolution, highest bandwidth. |




