# 🎥 Recommended Sensor Options

for 3D Hand-Tracking Reconstruction

**Short note — not recommended:** Standard **RGB webcams**, **Meta Quest 3**, **iPhone 15 Pro Max**, etc. (rolling shutter, auto exposure/gain, weak IR response) are fine for experiments but **unsuitable for precise hand tracking**. Likewise, **depth solutions** (LiDAR/ToF/active stereo — e.g., Intel RealSense, Kinect) are often **ill-suited** to fine hand/finger work: **relatively expensive**, **lower/variable FPS**, and **insufficient precision** for **parametric CAD commands**.

---

## 📷 Key sensor parameters

* **Type**

  * ✅ **Monochrome sensors (preferred)** – maximum sharpness, no Bayer debayering, better NIR sensitivity.
  * ❌ **RGB color** – for debugging/visualization only; not recommended for precise 3D reconstruction.

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

* **Spectral / optics**

  * ✅ **CMOS without IR-cut filter** – full NIR sensitivity.
  * ❌ **IR-cut filter** – not recommended; it blocks most NIR light and severely reduces signal at 850 
  * Use an **850 nm band-pass filter** with matching NIR illumination (LED/VCSEL).
  * Goal: stable contrast in NIR, robust against visible-light textures, colors, and ambient light.

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

---


## 📍 Five recommended options integrated with MotionCoder:


### 1) **Best Use: Pi5Track3D**

* **What it is:** On-edge **RAW10 mono ingest** on Raspberry Pi 5 with **GPU/NEON-optimized preproc** (undistort/normalize); optional on-Pi keypoints; stream to host for triangulation.
* **When to choose:** You want **deterministic low-latency ingest** and to offload decode/preproc from the PC; great for **4–8 cams** at 60–120 FPS.
* **Requirements:** **Raspberry Pi 5 (4/8 GB)**, 2× MIPI-CSI (e.g., OV9281), NVMe; **HW trigger** recommended.
* **Pros:** **Low latency**, predictable timing, **CPU offload** on host, scalable.
* **Cons:** Needs Pi-side integration (cabling, sync, enclosure); RAW pipeline setup is more involved than UVC.

---

### 2) **Budget-Optimized: MVCore3D**

* **Stack:** **MMPose + Anipose**
* **MJPEG pipeline:** Use **libjpeg-turbo** for fast decode.
* **Requirements:** **CPU** ≥ **Ryzen 7 5700X / 7900** (or comparable Intel). **MJPEG** is CPU-heavy.
* **Budget:** from **~€100** for **4× OV9281 (UVC)** + **TDMStrobe** — **best price/performance**.
* **Pros:** Very inexpensive, immediately available, **occlusion-robust (4 viewpoints)**.
* **Cons:** **Higher CPU load** (MJPEG decode), **more tuning** (exposure/sync), somewhat higher end-to-end latency; CPU limits scaling.

---

### 3) **Semi-Raw (Uncompressed YUV): MVYUV3D**

* **What it is:** Uncompressed, **pre-processed** (not sensor-RAW) YUV ingest with deterministic settings.
* **Stack:** **MMPose + Anipose** (no JPEG decode).
* **Input formats:** **YUY2/UYVY (4:2:2)**, optional **NV12 (4:2:0)**; **fixed exposure/gain** (auto off).
* **When to choose:** You want **lower CPU** than MJPEG and **simpler** than RAW, and can spare **more USB bandwidth**.
* **Requirements:** UVC cams with YUV; stable lighting; soft-sync OK (HW sync recommended for best results).
* **Pros:** **Lower CPU** than MJPEG; predictable output; quick to bring up.
* **Cons:** **More USB bandwidth**; still ISP-dependent (not RAW); less deterministic than HW-synced RAW.

---

### 4) **High-Fidelity (Sensor RAW): MVRAW3D**

* **What it is:** **RAW10/12 (Bayer/Mono) ingest** → **debayer/denoise** → triangulation; maximum control.
* **Stack:** **MMPose + Anipose**, **GPU-accelerated** image ops where available.
* **When to choose:** You need **lowest latency**, **highest fidelity**, tight **photometrics**, and **full determinism**.
* **Requirements:** Global-shutter cams, **HW trigger/sync**, fixed exposure; fast storage/GPU.
* **Pros:** **Latency ↓**, **fidelity ↑**, fully controllable pipeline; best for research/precision.
* **Cons:** **Bandwidth ↑**, more engineering (debayer/denoise), stricter lighting & calibration.

---

### 5) **High-Performance: MVMono3D**

* **What it is:** True **synchronized mono multi-view** with RAW12, **HW trigger/sync**, high-precision geometry pipeline — **without** expensive ToF.
* **AI-assisted geometry** to reduce **occlusions** and boost quality.

**Core building blocks**

* **Adaptive voxelization & downsampling**, ROI sampling, **3D keypoints**, quantization/compression
* **Robust multi-view triangulation** (weights, outlier rejection), local **bundle adjustment**
* **AI-based refinement** and **inference of occluded structures** — bandwidth- and memory-friendly

**In practice**

* **3D reprojection & skeleton stability:** very robust (low jitter/dropouts, low reprojection error)
* **Occlusion handling:** **multi-view + IR**, adaptive weighting, temporal models close gaps
* **Hand geometry:** capture **only essentials** (not millions of irrelevant points)
* **Animation:** **dynamically track** only the **relevant regions**

**Pros / Cons**

* **Pros:** **Plug-and-play**, **high performance**, **ROI point cloud**, **very high occlusion robustness**, **high precision**, **very low latency**
* **Cons:** Investment from **≈ €2,000** (cost-effective for results); comparable ToF quality is often **> €10,000**

---

**Note:** Choose the module that fits your **budget and latency/precision needs**. See **Recommended Sensors** for BOM examples and scaling tips.




## 📊 **Coarse Assessment of Optimal Sensor**

*relative to my CAD/DCC use case*

| Sensor                        |    Integration Module       |   Level % | Notes                                                                                                  |
| ----------------------------- | --------------------------: | --------: | ------------------------------------------------------------------------------------------------------ |
| 4× mono cams (global shutter) |    **MVMono3D + TDMStrobe** | **~100%** | Reference setup; highly scalable (more cams/MP), PTZ (zoom/focus), strong GPU, ample bandwidth.        |
| 4× mono cams (global shutter) |  **Pi5Track3D + TDMStrobe** |  **~70%** | Pi 5 capture/stream; good SNR/latency with GS+NIR; watch PCIe/USB and network headroom.                |
| 4× mono cams (global shutter) |     **MVRAW3D + TDMStrobe** |  **~70%** | RAW (Bayer/mono) pipeline; very low latency, high fidelity; needs HW sync and careful debayer/denoise. |
| 4× mono cams (global shutter) |     **MVYUV3D + TDMStrobe** |  **~60%** | Uncompressed YUV (YUY2/UYVY/NV12); lower CPU than MJPEG, higher bus load; fix exposure/gain.           |
| 4× Leap Motion Controller 2   |         **Leap2Pose**       |  **~50%** | High FPS/low latency; limited working volume and occlusion headroom.                                   |
| 4× U20CAM-9281M               |    **MVCore3D + TDMStrobe** |  **~45%** | Low-budget MJPEG; available now; works with **TDMStrobe**; CPU load from decode.                       |
| 4× Leap Motion Controller 1   |         **Leap2Pose**       |  **~40%** | Inexpensive; reliable range ~30–40 cm; more sensitive to occlusions.                                   |
| 2× Leap Motion Controller 2   |         **Leap2Pose**       |  **~40%** | Good latency; stereo helps, but pose-dependent occlusion gaps remain.                                  |
| 4× U20CAM-9281M               | **MVCore3D (no TDMStrobe)** |  **~30%** | IR always on → crosstalk/blooming; softer edges; less stable reconstruction.                           |
| 2× U20CAM-9281M               |    **MVCore3D + TDMStrobe** |  **~25%** | Stereo helps; without HW sync/MJPEG decode → fragile, less occlusion margin, higher CPU load.          |
| 1× Leap Motion Controller 2   |         **Leap2Pose**       |  **~25%** | Very smooth but small volume; strongly pose/occlusion-dependent.                                       |
| 4× Kinect / RealSense         |                     —       |  **~20%** | Limited scalability/precision; unsuitable for precise finger/tool gestures.                            |
| 4× high-quality RGB webcams   |  **MediaPipe / YOLO**       |  **~10%** | Theoretically scalable; in practice blur/artifacts/latency for precise hands/tools.                    |
| 1× Kinect / RealSense         |                     —       |   **~7%** | Prototype playground; not for precise hand/tool gestures.                                              |
| Myo armband / AIfES           |                     —       |   **~5%** | Very limited suitability for this use case.                                                            |

**Optimal path (target):**
**Pi5Track3D with 8× mono cams + small markers (wrist triangle + fingertips)** can deliver **>400%** of the reference baseline in **precision/robustness**, given proper **GS+NIR**, **TDM strobes**, and **tight calibration**.

*Notes:* Percentages are **use-case estimates** (precision, robustness, latency, occlusion tolerance, tunability) — **guidance**, not lab measurements.




