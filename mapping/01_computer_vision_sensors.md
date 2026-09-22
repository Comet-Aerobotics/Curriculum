# Mapping Module 1: Computer Vision Concepts, Depth Cameras & Spatial AI

> **Target:** Master pinhole camera models, active vs. passive stereo disparity, OAK-D Pro hardware architecture, and Visual-Inertial Odometry (VIO) fundamentals.  
> **Key References:** [Luxonis DepthAI ROS 2 Guide](https://docs.luxonis.com/software-v3/depthai/ros) | [Luxonis OAK-D Pro Specifications](https://shop.luxonis.com/products/oak-d-pro)

---

## 1. Pinhole Camera Model & Intrinsics

The pinhole camera model maps 3D metric coordinates in the camera frame $[X, Y, Z]^T$ to 2D pixel coordinates $[u, v]^T$ via the intrinsic matrix $K$:

$$\begin{bmatrix} u \\ v \\ 1 \end{bmatrix} = \frac{1}{Z} \begin{bmatrix} f_x & 0 & c_x \\ 0 & f_y & c_y \\ 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} X \\ Y \\ Z \end{bmatrix}$$

* $f_x, f_y$: Focal lengths in pixels.
* $c_x, c_y$: Principal point (optical center) in pixels.

---

## 2. Stereo Vision & Depth Calculation

Stereo cameras compute depth by matching corresponding pixels between left and right rectified images:

$$Z = \frac{f \cdot B}{d}$$

* $Z$: Distance/depth along the optical axis (meters).
* $f$: Focal length (pixels).
* $B$: Baseline distance between the two stereo camera centers (meters).
* $d$: Disparity ($x_{\text{left}} - x_{\text{right}}$ in pixels).

### Active Stereo with IR Dot Projectors
In featureless environments (such as smooth regolith or dark lunar simulant), standard passive stereo fails to match features. The **Luxonis OAK-D Pro** uses an infrared dot projector to project a pseudo-random pattern onto surfaces, providing high-contrast texture for dense depth maps.

---

## 3. Visual-Inertial Odometry (VIO) & Spatial AI

The OAK-D Pro includes an onboard **BNO086 9-axis IMU** and an **Intel Movidius Myriad X / RVC2** VPU running stereo disparity, neural inference, and tracking on-chip at up to 60 FPS, offloading heavy computation from the host robot computer.

---

## 4. Verification & Practice Checklist
- [ ] Calculated pixel projection $[u, v]$ from given 3D points $[X, Y, Z]$ and camera matrix $K$.
- [ ] Calculated depth $Z$ from baseline $B$, focal length $f$, and measured disparity $d$.
- [ ] Explained the advantage of Active Stereo (IR dot projector) in featureless regolith.
