# 🎥 Recommended Sensor Options
for 3D Hand-Tracking Reconstruction

---

## 📍 Five options integrated with MotionCoder:

---

### 1) **Budget-Optimized: MVCore3D**

* **Stack:** **MMPose + Anipose**
* **MJPEG pipeline:** use **libjpeg-turbo** for fast decode.
* **Requirements:** **CPU:** ≥ **Ryzen 7 5700X / 7900** (or comparable Intel). **MJPEG** is CPU-heavy.
* **Budget:** from **€100** for **4× OV9281 (UVC)** + **TDMStrobe** — **best price/performance** (BOM in [MVCore3D](https://github.com/xtanai/mvcore3d) and [TDMStrobe](https://github.com/xtanai/tdmstrobe))
* **Note:** If camera costs exceed ~€200 (rare) or ~€800 (typical setups), **Leap Motion** can be a sensible alternative (see **Leap2Pose** below).
* **Not recommended:** **Standard RGB webcams**, **Meta Quest 3**, **iPhone 15 Pro Max** etc. (rolling shutter, auto exposure/gain, weak IR response) — fine for experiments, **unsuitable for precise hand tracking**. Also **depth solutions (LiDAR/ToF/active stereo, e.g., Intel RealSense, Kinect)** are often **unsuitable** for fine hand/finger tracking: **relatively expensive**, **lower/inconsistent FPS**, and **insufficient precision** for **parametric CAD commands**.
* **Pros:** very inexpensive, immediately available, **occlusion-robust (4 viewpoints)**
* **Cons:** **higher CPU load** (e.g., MJPEG decode), **more tuning** (exposure/sync), somewhat higher end-to-end latency, scaling limited by CPU

---

### 2) **Semi-Raw (Uncompressed YUV): MVYUV3D**

* **What it is:** Uncompressed, **pre-processed** (not sensor-RAW) YUV ingest with deterministic settings.
* **Stack:** **MMPose + Anipose** (no JPEG decode).
* **Input formats:** **YUY2/UYVY (4:2:2)**, optional **NV12 (4:2:0)** with **fixed exposure/gain** (auto off).
* **When to choose:** You want **lower CPU load** than MJPEG and **simpler setup** than RAW, and can spare **more USB bandwidth**.
* **Requirements:** UVC cams with YUV; stable lighting; soft-sync OK (HW sync recommended for best results).
* **Pros:** **Lower CPU** than MJPEG; predictable output; quick to bring up
* **Cons:** **More USB bandwidth**; still ISP-dependent (not RAW); less deterministic than HW-synced RAW

---


### 3) **High-Fidelity (Sensor RAW): MVRaw3D**

* **What it is:** **RAW10/12** (Bayer/Mono) ingest → **debayer/denoise** → triangulation; maximum control.
* **Stack:** **MMPose + Anipose**, **GPU-accelerated** image ops where available.
* **When to choose:** You need **lowest latency**, **highest fidelity**, tight **photometrics**, and **full determinism**.
* **Requirements:** Global-shutter cams, **HW trigger/sync**, fixed exposure; fast storage/GPU.
* **Pros:** **Latency↓**, **fidelity↑**, fully controllable pipeline; best for research/precision
* **Cons:** **Bandwidth↑**, more engineering (debayer/denoise), stricter lighting & calibration

---

### 4) **Comfort: Leap2Pose**

* **Stack:** **LeapC**
* **4× Leap Motion Controller** in a stable frame, properly aligned → **works very well** with MotionCoder
* **2× Leap** → **works**, with limitations (being optimized)
* **1× Leap** → **significantly limited**; robust, adapted gestures are provided
* **Price ballpark:** **~€250** per device (new version) or from **~€15** (older version, used, EU)
* **Note:** The **old version** typically reaches **up to ~60 cm**; the **new version** **up to ~110 cm**. With **4× sensors**, overlap and angles can **partially compensate** limitations.
* **Pros:** **low latency**, **plug-and-play**
* **Cons:** smaller working volume, **less scalable**, somewhat less occlusion-robust

---

### 5) **High-Performance: MVMono3D**

* **Stack:** (later)
* **Upgrade path** to true **multi-view** with **Mono8/RAW10**, **hardware trigger/sync**, and a **high-precision geometry pipeline** — **without** expensive **ToF sensors**.
* **AI-assisted geometry** to reduce **occlusions** and boost **quality**.

**Core building blocks**

* **Adaptive voxelization & downsampling**, **ROI sampling**, **3D keypoints**, **quantization/compression**
* **Robust multi-view triangulation** (weighting, outlier robustness), local **bundle adjustment**
* **AI-based refinement** of geometry and **reconstruction of occluded structures** — **bandwidth- and memory-friendly**

**In practice**

* **3D reprojection & skeleton stability:** very robust (less jitter/dropouts, low reprojection error).
* **Occlusion handling:** **multi-view + IR**, **adaptive weighting**, and **temporal models** close gaps seamlessly.
* **Hand geometry:** capture only the **essential structures** instead of millions of irrelevant points.
* **Animation:** **dynamically activate & track** only the **relevant regions**.

**Pros / Cons**

* **Pros:** **plug-and-play**, **high performance**, **very high occlusion robustness**, **high precision**, **very low latency**
* **Cons:** investment from **≈ €1,000** (cost-efficient for the results); **ToF** at comparable quality is often **> €10,000**

---

## 📊 **Coarse Assessment of Optimal Sensor**

*relative to my CAD/DCC use case*

| Sensor                        | Integration Module    |   Level % | Notes                                                                                                |
| ----------------------------- | --------------------- | --------: | ---------------------------------------------------------------------------------------------------- |
| 4× mono cams (global shutter) | **MVMono3D**          | **~100%** | Reference setup; highly scalable (more cams/MP), PTZ (zoom/focus), strong GPU, ample bandwidth.      |
| 4× mono cams (global shutter) | **MVRaw3D**           |  **~70%** | RAW (Bayer/Mono) pipeline; low latency, high fidelity; needs HW sync & careful debayer/denoise.      |
| 4× mono cams (global shutter) | **MVYUV3D**           |  **~60%** | Uncompressed YUV (YUY2/UYVY/NV12); less CPU than MJPEG, more USB; fix exposure/gain for determinism. |
| 4× Leap Motion Controller 2   | **Leap2Pose**         |  **~50%** | Solid baseline; high FPS/low latency; limited working volume & occlusion headroom.                   |
| 4× U20CAM-9281M               | **MVCore3D + TDM**    |  **~45%** | Low-budget MJPEG; available now; good with **TDMStrobe**; high CPU (decode).                         |
| 4× Leap Motion (Gen 1)        | **Leap2Pose**         |  **~42%** | Inexpensive; reliable range ~30–40 cm; more sensitive to occlusions.                                 |
| 2× Leap Motion Controller 2   | **Leap2Pose**         |  **~40%** | Good latency; stereo helps but occlusion gaps remain pose-dependent.                                 |
| 4× U20CAM-9281M               | **MVCore3D (no TDM)** |  **~30%** | IR always on → crosstalk/blooming; softer edges; less stable reconstruction.                         |
| 2× U20CAM-9281M               | **MVCore3D + TDM**    |  **~25%** | Stereo helps; without HW sync/MJPEG decode → fragile, less occlusion margin, higher CPU load.        |
| 1× Leap Motion Controller 2   | **Leap2Pose**         |  **~25%** | Very smooth but small volume; strongly pose/occlusion-dependent.                                     |
| 4× Kinect / RealSense         | —                     |  **~20%** | Limited scalability/precision; unsuitable for precise finger/tool gestures.                          |
| 4× high-quality RGB webcams   | **MediaPipe / YOLO**  |  **~10%** | Theoretically scalable; in practice blur/artifacts/latency for precise hands/tools.                  |
| 1× Kinect / RealSense         | —                     |   **~7%** | Prototype playground; not for precise hand/tool gestures.                                            |
| Myo armband / AIfES           | —                     |   **~5%** | Very limited suitability for this use case.                                                          |


*Note:* Percentages are **use-case estimates** (precision, robustness, latency, occlusion, tunability) — **guidance**, not lab measurements.

---

## 📷 Sensor Lists

Coming soon.
