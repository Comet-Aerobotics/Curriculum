# Module 3: Computer Vision Concepts, Depth Cameras & Spatial AI

> **Target:** Master geometric camera models, active stereo depth sensing, Luxonis OAK-D Pro hardware architecture, ROS 2 integration via `depthai_ros`, and their role in Visual SLAM.  
> **Key Goal:** Study the Luxonis DepthAI ROS documentation, understand active stereo perception and SLAM mechanics, complete the depth perception math exercise, and verify results against expected outputs.

---

## 1. What is a Depth Camera? (OAK-D Pro Architecture)

Traditional RGB cameras only capture 2D color intensity ($u, v$). A **Depth Camera (RGB-D)** captures both 2D color and per-pixel distance metrics ($Z$), producing rich 3D spatial representations.

```
       +-------------------------------------------------------------------+
       |                       Luxonis OAK-D Pro                           |
       |                                                                   |
       |   [Left Mono]        [IR Projector & LED]        [Right Mono]     |
       |     (Global)           (Active Texture)            (Global)       |
       |        |                       |                      |           |
       |        +------------+          |          +-----------+           |
       |                     |          |          |                       |
       |                     v          v          v                       |
       |                +-------------------------------+                  |
       |   [4K RGB] --->|   Onboard RVC2 Vision Core    |<--- [9-DoF IMU]  |
       |                |  - Stereo Hardware Depth      |                  |
       |                |  - Edge Neural Inference      |                  |
       |                +-------------------------------+                  |
       |                                |                                  |
       |                                v (USB-C / Ethernet)               |
       |                         ROS 2 Workspace                           |
       |            Topics: /rgb, /stereo/depth, /points, /imu             |
       +-------------------------------------------------------------------+
```

### a. Active Stereo Vision vs. Passive Stereo
* **Passive Stereo Limitation:** Standard stereo cameras rely entirely on natural environmental texture (edges, corners, patterns) to match pixels between left and right images. On smooth, featureless surfaces (plain white walls, tiled floors, blank doors), passive stereo cannot find matches and produces blank "holes" in the depth map.
* **Active Stereo (OAK-D Pro Solution):** The OAK-D Pro integrates an **Infrared (IR) Laser Dot Projector** that projects thousands of invisible, structured IR dots onto surfaces. The left and right global-shutter monochrome sensors observe this artificial texture, enabling reliable, dense disparity calculation even on completely blank surfaces.
* **Night Vision Illumination:** An onboard **IR Flood Illumination LED** illuminates dark environments, allowing the camera to operate in low-light or total zero-lux darkness.

### b. Onboard Edge Compute (Robotics Vision Core - RVC2)
Unlike webcams that send raw frames to the host computer, the OAK-D Pro processes computer vision tasks on-chip:
* **Hardware Stereo Engine:** Performs epipolar rectification, subpixel interpolation, and disparity-to-depth conversion in real-time at 30+ FPS with zero host CPU usage.
* **Edge Neural Network Acceleration:** Runs object detection (YOLOv8, MobileNet), feature tracking, and semantic segmentation directly on-device.

### c. The Role of Depth Cameras in Visual SLAM
**Simultaneous Localization and Mapping (SLAM)** is the computational problem of constructing a map of an unknown environment while simultaneously keeping track of the robot's location within it.

The OAK-D Pro provides the essential sensor trifecta for **Visual-Inertial SLAM (VIO/SLAM)** (e.g., RTAB-Map, ORB-SLAM3, Cartographer):
1. **RGB Image Stream (`sensor_msgs/msg/Image`):** Tracks visual landmarks and identifies loop closures when revisiting locations.
2. **Dense Depth Map & Point Cloud (`sensor_msgs/msg/PointCloud2`):** Supplies metric 3D point measurements ($X, Y, Z$) to construct 3D obstacle maps and 2D Nav2 costmaps.
3. **Synchronized 9-DoF IMU (`sensor_msgs/msg/Imu`):** Provides high-rate ($100\text{--}400\text{ Hz}$) angular velocity (gyroscope) and linear acceleration (accelerometer) measurements, preventing localization loss during fast robot turns or motion blur.

---

## 2. Geometric Camera Models & Stereo Mathematics

### a. Pinhole Model & Intrinsics Matrix ($K$)
A pinhole camera maps a 3D point in the camera frame $\mathbf{P} = [X, Y, Z]^T$ into 2D image coordinates $(u, v)$:

$$\begin{bmatrix} u \\ v \\ 1 \end{bmatrix} = \frac{1}{Z} \mathbf{K} \begin{bmatrix} X \\ Y \\ Z \end{bmatrix} = \frac{1}{Z} \begin{bmatrix} f_x & 0 & c_x \\ 0 & f_y & c_y \\ 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} X \\ Y \\ Z \end{bmatrix}$$

* **$f_x, f_y$ (Focal Lengths in pixels):** Determine scale and magnification along horizontal/vertical sensor axes ($f = \frac{F}{\text{pixel size}}$).
* **$c_x, c_y$ (Principal Point in pixels):** The optical center of the sensor where the optical axis intersects the image plane (typically near image center).

Projection equations:
$$u = \frac{f_x \cdot X}{Z} + c_x, \qquad v = \frac{f_y \cdot Y}{Z} + c_y$$

### b. Stereo Triangulation & Disparity
Two parallel cameras separated horizontally by a known distance **baseline ($b$)** capture the same 3D point at different horizontal pixel locations ($u_L$ on left image, $u_R$ on right image).

* **Disparity ($d$):** The pixel offset between corresponding points:
  $$d = u_L - u_R$$
* **Metric Depth ($Z$):** By similar triangles, depth is inversely proportional to disparity:
  $$Z = \frac{f \cdot b}{d}$$
  *(where $f$ is the focal length in pixels, $b$ is the baseline in meters, and $d$ is disparity in pixels).*

> [!TIP]
> **Depth Resolution Insight:** Disparity is inversely proportional to distance. At near range, a small change in depth causes a large change in disparity (high accuracy). At far range, large depth differences result in tiny sub-pixel disparity changes.

---

## 3. Web Tutorials & Official Documentation

Follow the official Luxonis documentation and guides below:

### a. Tutorial 1: Luxonis DepthAI ROS 2 Guide
* 🌐 **Official Documentation:** [Luxonis DepthAI ROS 2 Guide (https://docs.luxonis.com/software-v3/depthai/ros)](https://docs.luxonis.com/software-v3/depthai/ros)
* **What you will learn:**
  * Architecture of the `depthai_ros_driver` node and ROS 2 launch files.
  * Standard ROS 2 perception topics published:
    * `/camera/rgb/image_raw` & `/camera/rgb/camera_info` (Color image & $K$ intrinsics)
    * `/camera/stereo/depth` (16-bit uint millimeter depth map)
    * `/camera/depth/color/points` (`sensor_msgs/msg/PointCloud2` for 3D spatial mapping)
    * `/camera/imu/data` (`sensor_msgs/msg/Imu` for visual odometry & filtering)
  * Configuring camera parameters (resolution, auto-calibration, exposure, and IR dot projector intensity).

### b. Tutorial 2: Luxonis OAK-D Pro Hardware Architecture
* 🌐 **Hardware Reference:** [Luxonis OAK-D Pro Product Specifications](https://shop.luxonis.com/products/oak-d-pro)
* **What you will learn:**
  * Active infrared dot projector functionality and power levels.
  * Global shutter mono cameras vs. rolling shutter RGB sensor.
  * Integrated IMU specifications (BNO086 / BMI270) and host synchronization.

---

## 4. Hands-On Verification Exercise (Run on your VM / Workstation)

Apply the pinhole projection and stereo depth formulas to verify perception computations using OAK-D Pro sensor parameters.

### a. Problem Statement
1. **Pinhole Camera Projection:**  
   An OAK-D Pro RGB camera has calibrated intrinsics $f_x = 800.0\text{ px}$, $f_y = 800.0\text{ px}$, optical center $(c_x = 640.0\text{ px}, c_y = 360.0\text{ px})$ at a $1280 \times 720$ resolution. An obstacle is detected in 3D camera space at $(X = 0.6\text{ m}, Y = -0.3\text{ m}, Z = 3.0\text{ m})$.  
   **Calculate the 2D projected pixel coordinate $(u, v)$ on the image plane.**

2. **Stereo Disparity to Metric Depth:**  
   The OAK-D Pro stereo baseline is $b = 7.5\text{ cm} = 0.075\text{ m}$ with mono focal length $f = 800.0\text{ px}$. The stereo matching block engine detects a matching feature point with a disparity of $d = 20.0\text{ pixels}$.  
   **Calculate the true metric depth $Z$ in meters.**

---

## 5. Expected Outputs & Instructor Verification Checklist

### a. Expected Calculation Outputs

#### Check 1: Pinhole 3D-to-2D Projection
$$u = \frac{f_x \cdot X}{Z} + c_x = \frac{800.0 \cdot 0.6}{3.0} + 640.0 = 160.0 + 640.0 = \mathbf{800.0\text{ px}}$$
$$v = \frac{f_y \cdot Y}{Z} + c_y = \frac{800.0 \cdot (-0.3)}{3.0} + 360.0 = -80.0 + 360.0 = \mathbf{280.0\text{ px}}$$
* **Expected Result:** Projected pixel coordinate is **$(u = 800.0\text{ px}, v = 280.0\text{ px})$**.

#### Check 2: Stereo Disparity to Depth
$$Z = \frac{f \cdot b}{d} = \frac{800.0 \cdot 0.075}{20.0} = \frac{60.0}{20.0} = \mathbf{3.000\text{ meters}}$$
* **Expected Result:** Metric depth $Z$ is **$3.000\text{ meters}$**.

### b. Completion Checklist for Instructors & Students
- [ ] Studied [Luxonis DepthAI ROS 2 Documentation](https://docs.luxonis.com/software-v3/depthai/ros) and understands topic conventions.
- [ ] Understands why Active Stereo (IR dot projector) is critical for feature matching on blank, textureless surfaces.
- [ ] Can explain how RGB-D depth maps, PointCloud2, and high-rate IMU streams enable Visual SLAM and Nav2 obstacle avoidance.
- [ ] Hand-calculated or scripted solutions to Check 1 and Check 2 with exact matches to expected outputs.
- [ ] Understands the role of on-device RVC2 edge acceleration in reducing host CPU load.
