# 🎥 Recommended Sensor Options
for 3D Hand-Tracking Reconstruction

## 📍 Three options integrated with MotionCoder:

### 1) **Budget-Optimized: MVCore3D or MVRaw3D**

* **Stack:** **MMPose + Anipose**

  * **MJPEG pipeline:** use **libjpeg-turbo** for fast decoding.
  * **RAW (YUY2/Mono8):** **no** libjpeg-turbo required.

* **Budget:** from **~€100** for **4× OV9281 (UVC)** + **TDMStrobe** — excellent **price/performance**.
  BOMs: **[MVCore3D](https://github.com/xtanai/mvcore3d)**, **[TDMStrobe](https://github.com/xtanai/tdmstrobe)**

* **Requirements:**

  * **CPU:** at least **Ryzen 7 5700X / 7900** (or comparable Intel). **MJPEG** decoding is CPU-heavy.
  * **RAW:** lower CPU load but **higher USB bandwidth** demand.
  * **GPU (recommended):** **RTX 3060+** for pose inference.


* **Stack:** **MMPose + Anipose + libjpeg-turbo** (if Raw, nothing libjpeg-turbo=). 
* **Budget:** from **€100** for **4× OV9281 (UVC)** + **TDMStrobe** — **best price/performance** (BOM in [MVCore3D](https://github.com/xtanai/mvcore3d) and [TDMStrobe](https://github.com/xtanai/tdmstrobe))
* **Requirements:** **strong CPU** (at least **Ryzen 7 5700X / 7900** or equivalent Intel) (if Raw, nothing strong CPU)
* **Cameras:** Alternatively **USB3 global-shutter** with **trigger/strobe**; **recommendations & trade-offs** are listed in the **camera list** in the repo
* **Note:** If camera costs exceed ~€200 (rare) or ~€800 (typical setups), **Leap Motion** can be a sensible alternative (see **Leap2Pose** below).
* **Not recommended:** **Standard RGB webcams**, **Meta Quest 3**, **iPhone 15 Pro Max** etc. (rolling shutter, auto exposure/gain, weak IR response) — fine for experiments, **unsuitable for precise hand tracking**. Also **depth solutions (LiDAR/ToF/active stereo, e.g., Intel RealSense, Kinect)** are often **unsuitable** for fine hand/finger tracking: **relatively expensive**, **lower/inconsistent FPS**, and **insufficient precision** for **parametric CAD commands**.

**Pros / Cons**

* **Pros:** very inexpensive, immediately available, **occlusion-robust (4 viewpoints)**
* **Cons:** **higher CPU load** (e.g., MJPEG decode), **more tuning** (exposure/sync), somewhat higher end-to-end latency, scaling limited by CPU

---

### 2) **Comfort: Leap2Pose**

* **Stack:** **LeapC**
* **4× Leap Motion Controller** in a stable frame, properly aligned → **works very well** with MotionCoder
* **2× Leap** → **works**, with limitations (being optimized)
* **1× Leap** → **significantly limited**; robust, adapted gestures are provided
* **Price ballpark:** **~€250** per device (new version) or from **~€15** (older version, used, EU)
* **Note:** The **old version** typically reaches **up to ~60 cm**; the **new version** **up to ~110 cm**. With **4× sensors**, overlap and angles can **partially compensate** limitations.

**Pros / Cons**

* **Pros:** **low latency**, **plug-and-play**
* **Cons:** smaller working volume, **less scalable**, somewhat less occlusion-robust

---

### 3) **High-Performance: MVMono3D**

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

| Sensor                        | Integration Module    |   Level % | Notes                                                                                                   |
| ----------------------------- | --------------------- | --------: | ------------------------------------------------------------------------------------------------------- |
| 4× mono cams (global shutter) | **MVMono3D**          | **~100%** | Reference level; highly scalable (more cams/MP, PTZ with zoom/focus, strong GPU, sufficient bandwidth). |
| 4× mono cams (global shutter) | **MVRaw3D**           |  **~75%** | Best             |
| 4× Leap Motion Controller 2   | **Leap2Pose**         |  **~50%** | Good baseline for hand tracking; slightly limited scalability.                                          |
| 4× U20CAM-9281M               | **MVCore3D + TDM**    |  **~45%** | Low-budget, immediately available; high CPU demand.                                                     |
| 4× Leap Motion (Gen 1)        | **Leap2Pose**         |  **~42%** | Inexpensive, but reliable working distance only ~30–40 cm.                                              |
| 2× Leap Motion Controller 2   | **Leap2Pose**         |  **~40%** | Good latency; occlusion gaps depending on pose.                                                         |
| 4× U20CAM-9281M               | **MVCore3D (no TDM)** |  **~30%** | IR always on → crosstalk/blooming; softer edges, less stable reconstruction.                            |
| 2× U20CAM-9281M               | **MVCore3D + TDM**    |  **~25%** | Stereo helps, but without HW sync/MJPEG decode: more fragile, less occlusion headroom, higher CPU load. |
| 1× Leap Motion Controller 2   | **Leap2Pose**         |  **~25%** | Very smooth, but strongly pose/occlusion-dependent; small volume.                                       |
| 4× Kinect / RealSense         | —                     |  **~20%** | Limited scalability, lower precision; not suitable for precise finger/tool gestures.                    |
| 4× high-quality RGB webcams   | **MediaPipe / YOLO**  |  **~10%** | Theoretically scalable; in practice blur/artifacts/latency for precise hands/tools.                     |
| 1× Kinect / RealSense         | —                     |   **~7%** | Prototype playground; not for precise hand/tool gestures.                                               |
| Myo armband / AIfES           | —                     |   **~5%** | Very limited suitability for this use case.                                                             |

*Note:* Percentages are **use-case estimates** (precision, robustness, latency, occlusion, tunability) — **guidance**, not lab measurements.

---

## 📷 Sensor Lists

Coming soon.
