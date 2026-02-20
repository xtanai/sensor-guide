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
  * *Nice to have:* a STROBE / exposure-active output that can be used as a hardware feedback signal (trigger accepted / exposure occurred).

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

## ⭐ Pixel format – preference

| Format                     |   Rating   | Comment                                                                                  |
| -------------------------- | ---------- | ---------------------------------------------------------------------------------------- |
| **MJPEG / JPEG**           | ⭐☆☆☆☆     | Only for preview/debug. Strong artifacts, variable bitrate, poor for precise 3D.         |
| **YUV / YUYV / NV12**      | ⭐⭐☆☆☆    | OK if you only use the **Y (luma)** channel. Extra bandwidth wasted on color info.       |
| **RAW8 / Y8 (8-bit mono)** | ⭐⭐⭐☆☆   | Solid baseline. Lower dynamic range, but good enough with proper NIR illumination.       |
| **RAW10**                  | ⭐⭐⭐⭐☆  | Very good: higher dynamic range, finer quantization, still manageable bandwidth.         |
| **RAW12**                  | ⭐⭐⭐⭐⭐ | Ideal for high precision: maximum dynamic range and depth resolution, highest bandwidth. |

---

## ⚙️ Quick Engineering Comparison — What is the best interface for deterministic vision?

When designing a machine-vision or stereo system, the choice of sensor interface has a strong impact on latency, control, and system complexity.
Below is a simplified engineering comparison:

| Interface              | Additional Chips | RAW Access |    Latency   |  Determinism  |
| ---------------------- | ---------------- | ---------- | ------------ | ------------- |
| **MIPI CSI-2**         | very few         | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐  | ⭐⭐⭐⭐⭐    |
| **USB**                | medium           | ⭐⭐☆☆☆    | ⭐⭐⭐☆☆    | ⭐⭐☆☆☆       |
| **GigE / GigE Vision** | many             | ⭐⭐⭐☆☆   | ⭐⭐⭐☆☆    | ⭐⭐⭐⭐☆     |

### Summary

**MIPI CSI-2** is typically the best choice when deterministic timing, minimal latency, and direct RAW sensor access are required. The sensor is connected almost directly to the SoC, which reduces hidden processing stages and keeps the pipeline transparent.

**USB cameras** usually include additional ISP and bridge chips. They are convenient and plug-and-play, but often introduce internal processing and buffering that reduce determinism.

**GigE cameras** are powerful for industrial networking and long cable distances, but typically require more intermediate logic (FPGA/ASIC, packetization, buffering), which increases system complexity.

For edge-processing architectures focused on precise timing and reproducible results, **MIPI CSI-2 provides the most transparent and controllable capture path**.

---

## Mono Sensors on the Market (MIPI CSI-2)

For **mono global-shutter stereo with hardware trigger** on Raspberry Pi (MIPI CSI-2), there are only **a few** affordable and widely available options. The most accessible and practical modules today are typically based on **OV9281**, **Sony IMX296LLR**, and **onsemi AR0145**. These sensors are common in the Raspberry Pi ecosystem and work well for early development, prototyping, and validation.


### Current Prototype

The current prototype uses OV9281-, IMX296LLR, AR0145-based modules, currently configured with a **limited CSI-2 lane setup**. This is ideal for development and validation, but it does **not fully utilize** the Raspberry Pi 5’s available CSI bandwidth and leaves less headroom for higher-throughput capture modes.

---

### Long-Term Goal: Higher-Bandwidth RAW Sensors (Transparent Capture Path)

For the next generation, I’m targeting a **RAW-first sensor path** with **transparent, deterministic control**—meaning no opaque or mandatory processing stages in the capture chain. The goal is to take full advantage of **4-lane CSI-2 on Raspberry Pi 5**, increasing throughput and leaving more headroom for advanced modes and future upgrades.

Securing the right sensor/module partner is feasible—but it depends on **validated demand**.

---

### Open, Repeatable Platform (MotionCoder-first)

This is not meant to be closed or exclusive. The platform should remain **repeatable and widely adoptable**, because the long-term value sits in the software layer—**MotionCoder**—which benefits most from a broadly available, standardized capture backend.

---

### Feasibility & Manufacturing Reality

From a system perspective, this direction is realistic: a RAW-first camera path can keep the electronics BOM lean, while Linux-based edge platforms provide strong, direct control over capture, timing, and the imaging pipeline.

The main challenge is not “extra chips” - it’s **reliable module sourcing**, **mechanical integration**, **QC**, and **calibration**. That’s exactly what demand validation helps unlock.

Based on current assumptions, I estimate sensor/module cost can be kept below **~€100 per unit**, but this depends heavily on volume. Reaching stable pricing and supply typically requires a meaningful production run (on the order of **~1,000 units**).

---

### Vote / Demand Signal (Call to Action)

If you want **higher-end mono sensors** (more bandwidth, better optics support, stronger long-term supply), please leave a **vote / comment**. I’m collecting demand signals to move forward with supplier negotiations, final sourcing, and production commitments.

Once demand is validated, I can confidently push this to production—so you can **swap camera modules in EdgeTrack** and keep using the same high-end capture backend as sensors evolve.

---




