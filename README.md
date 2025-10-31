# 🎥 Recommended Sensor Options

for 3D Hand-Tracking Reconstruction

**Short note — not recommended:** Standard **RGB webcams**, **Meta Quest 3**, **iPhone 15 Pro Max**, etc. (rolling shutter, auto exposure/gain, weak IR response) are fine for experiments but **unsuitable for precise hand tracking**. Likewise, **depth solutions** (LiDAR/ToF/active stereo — e.g., Intel RealSense, Kinect) are often **ill-suited** to fine hand/finger work: **relatively expensive**, **lower/variable FPS**, and **insufficient precision** for **parametric CAD commands**.

## 📍 Six recommended options integrated with MotionCoder:


### 1) **Best Use: Pi5Track3D**

* **What it is:** On-edge **RAW10 mono ingest** on Raspberry Pi 5 with **GPU/NEON-optimized preproc** (undistort/normalize); optional on-Pi keypoints; stream to host for triangulation.
* **When to choose:** You want **deterministic low-latency ingest** and to offload decode/preproc from the PC; great for **4–8 cams** at 60–120 FPS.
* **Requirements:** **Raspberry Pi 5 (4/8 GB)**, 2× MIPI-CSI (e.g., OV9281), NVMe or USB-to-2.5GbE; **HW trigger** recommended.
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

### 4) **High-Fidelity (Sensor RAW): MVRaw3D**

* **What it is:** **RAW10/12 (Bayer/Mono) ingest** → **debayer/denoise** → triangulation; maximum control.
* **Stack:** **MMPose + Anipose**, **GPU-accelerated** image ops where available.
* **When to choose:** You need **lowest latency**, **highest fidelity**, tight **photometrics**, and **full determinism**.
* **Requirements:** Global-shutter cams, **HW trigger/sync**, fixed exposure; fast storage/GPU.
* **Pros:** **Latency ↓**, **fidelity ↑**, fully controllable pipeline; best for research/precision.
* **Cons:** **Bandwidth ↑**, more engineering (debayer/denoise), stricter lighting & calibration.

---

### 5) **Comfort: Leap2Pose**

* **Stack:** **LeapC**
* **4× Leap Motion Controller** in a rigid frame, properly aligned → **works very well** with MotionCoder.
  **2× Leap** → **works** with limitations (being optimized).
  **1× Leap** → **limited**; robust, adapted gestures provided.
* **Price ballpark:** **~€250** per device (new) or from **~€15** (older, used, EU).
* **Note:** Older units typically reach **~60 cm**; newer **~110 cm**. With **4× sensors**, overlap/angles can **partially compensate** limitations.
* **Pros:** **Low latency**, **plug-and-play**.
* **Cons:** Smaller working volume, **less scalable**, no point cloud/face tracking, less occlusion-robust.

---

### 6) **High-Performance: MVMono3D**

* **What it is:** True **synchronized mono multi-view** (Mono8/RAW10), **HW trigger/sync**, high-precision geometry pipeline — **without** expensive ToF.
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

| Sensor                        |    Integration Module |   Level % | Notes                                                                                                  |
| ----------------------------- | --------------------: | --------: | ------------------------------------------------------------------------------------------------------ |
| 4× mono cams (global shutter) |    **MVMono3D + TDM** | **~100%** | Reference setup; highly scalable (more cams/MP), PTZ (zoom/focus), strong GPU, ample bandwidth.        |
| 4× mono cams (global shutter) |  **Pi5Track3D + TDM** |  **~70%** | Pi 5 capture/stream; good SNR/latency with GS+NIR; watch PCIe/USB and network headroom.                |
| 4× mono cams (global shutter) |     **MVRaw3D + TDM** |  **~70%** | RAW (Bayer/mono) pipeline; very low latency, high fidelity; needs HW sync and careful debayer/denoise. |
| 4× mono cams (global shutter) |     **MVYUV3D + TDM** |  **~60%** | Uncompressed YUV (YUY2/UYVY/NV12); lower CPU than MJPEG, higher bus load; fix exposure/gain.           |
| 4× Leap Motion Controller 2   |         **Leap2Pose** |  **~50%** | Solid baseline; high FPS/low latency; limited working volume and occlusion headroom.                   |
| 4× U20CAM-9281M               |    **MVCore3D + TDM** |  **~45%** | Low-budget MJPEG; available now; works with **TDM-strobe**; CPU load from decode.                      |
| 4× Leap Motion (Gen 1)        |         **Leap2Pose** |  **~42%** | Inexpensive; reliable range ~30–40 cm; more sensitive to occlusions.                                   |
| 2× Leap Motion Controller 2   |         **Leap2Pose** |  **~40%** | Good latency; stereo helps, but pose-dependent occlusion gaps remain.                                  |
| 4× U20CAM-9281M               | **MVCore3D (no TDM)** |  **~30%** | IR always on → crosstalk/blooming; softer edges; less stable reconstruction.                           |
| 2× U20CAM-9281M               |    **MVCore3D + TDM** |  **~25%** | Stereo helps; without HW sync/MJPEG decode → fragile, less occlusion margin, higher CPU load.          |
| 1× Leap Motion Controller 2   |         **Leap2Pose** |  **~25%** | Very smooth but small volume; strongly pose/occlusion-dependent.                                       |
| 4× Kinect / RealSense         |                     — |  **~20%** | Limited scalability/precision; unsuitable for precise finger/tool gestures.                            |
| 4× high-quality RGB webcams   |  **MediaPipe / YOLO** |  **~10%** | Theoretically scalable; in practice blur/artifacts/latency for precise hands/tools.                    |
| 1× Kinect / RealSense         |                     — |   **~7%** | Prototype playground; not for precise hand/tool gestures.                                              |
| Myo armband / AIfES           |                     — |   **~5%** | Very limited suitability for this use case.                                                            |

**Optimal path (target):**
**Pi5Track3D with 8× mono cams + small markers (wrist triangle + fingertips)** can deliver **>400%** of the reference baseline in **precision/robustness**, given proper **GS+NIR**, **TDM strobes**, and **tight calibration**.

*Notes:* Percentages are **use-case estimates** (precision, robustness, latency, occlusion tolerance, tunability) — **guidance**, not lab measurements.




