# Mapping Module 1: AprilTag 6-DoF Pose Estimation & Kalman Filter Sensor Fusion

> **Target:** Master extracting 6-DoF poses ($X, Y, Z, \text{yaw}, \text{pitch}, \text{roll}$) from AprilTag fiducial markers, understanding camera optical frames, and fusing visual landmarks with IMU and wheel odometry using Extended Kalman Filters (EKF) in ROS 2.  
> **Key References:** [apriltag_ros Documentation](https://github.com/AprilRobotics/apriltag_ros) | [robot_localization EKF Guide](https://docs.nav2.org/setup_guides/odom/setup_odom.html)

---

## 1. What is an AprilTag & How Does Detection Work?

AprilTags are high-contrast 2D fiducial barcodes (similar to QR codes) designed specifically for robust, low-latency robotic perception. When an AprilTag detector node processes an image, it identifies the four tag corners in pixel space, compares them against the known real-world physical tag dimensions (e.g. $16\text{ cm}$), and computes the **6-Degree-of-Freedom (6-DoF) Rigid Transform** from the camera optical frame to the tag.

```
       +-----------------------------------------------------------+
       |                  Camera Optical Frame                     |
       |                                                           |
       |                    +Z (Forward into scene / Depth)        |
       |                   /                                       |
       |                  /                                        |
       |                 +-----> +X (Right)                        |
       |                 |                                         |
       |                 v                                         |
       |                +Y (Down)                                  |
       +-----------------------------------------------------------+
                                    |
                    6-DoF Transformation Matrix (T_c_t)
                                    |
                                    v
       +-----------------------------------------------------------+
       |                  AprilTag Target Frame                    |
       |       Position: [X, Y, Z]    Orientation: [Roll, Pitch, Yaw] |
       +-----------------------------------------------------------+
```

---

## 2. Demystifying 6-DoF Pose: $[X, Y, Z]$ and $[\text{Roll}, \text{Pitch}, \text{Yaw}]$

When an AprilTag is detected, ROS 2 outputs a `geometry_msgs/msg/PoseStamped` or `TransformStamped` relative to the camera optical center. Understanding what each value represents is critical for spatial reasoning:

### a. 3D Translations ($X, Y, Z$)
* **$X$ (Lateral / Horizontal Offset):** Distance left or right of the camera's optical axis.
  * $X > 0$: Tag is to the **right** of center.
  * $X < 0$: Tag is to the **left** of center.
  * $X = 0$: Tag is directly centered horizontally.
* **$Y$ (Vertical / Elevation Offset):** Distance above or below the camera lens.
  * $Y > 0$: Tag is **below** the camera level.
  * $Y < 0$: Tag is **above** the camera level.
* **$Z$ (Depth / Axial Distance):** True metric distance **straight forward** along the camera's optical axis.
  * $Z = 1.50\text{ m}$ means the tag is exactly $1.5$ meters in front of the lens plane.

### b. 3D Rotations ($\text{Roll}, \text{Pitch}, \text{Yaw}$)
* **$\text{Yaw}$ ($\psi$ - Heading / Bearing Angle):** Rotation around the vertical axis.
  * Represents the relative angle needed to turn the robot's base to look directly at the tag: $\theta = \text{atan2}(X, Z)$.
* **$\text{Pitch}$ ($\theta$ - Elevation Tilt):** Rotation around the horizontal axis.
  * Indicates if the tag plane is tilted backward or forward relative to the camera lens.
* **$\text{Roll}$ ($\phi$ - In-Plane Tilt):** Rotation around the optical $Z$-axis.
  * Indicates if the tag is mounted crooked or tilted sideways.

---

## 3. Sensor Fusion in ROS 2 with Extended Kalman Filters (EKF)

In a competition field, relying on a single sensor causes autonomy to fail:
* **Wheel Odometry (`/odom`):** High frequency ($50\text{ Hz}$), but suffers from cumulative slip on loose lunar regolith.
* **IMU (`/imu/data`):** High frequency ($100\text{--}200\text{ Hz}$), tracks rotation perfectly, but double integration of linear acceleration drifts exponentially.
* **AprilTags (`/apriltag_pose`):** Low frequency ($10\text{--}15\text{ Hz}$), susceptible to visual occlusions, but provides **zero-drift ground-truth absolute landmarks**.

### a. How the EKF Node Works (`robot_localization`)

ROS 2 provides the industry-standard `robot_localization` package (`ekf_node`). The EKF runs a continuous two-phase mathematical cycle:
1. **Prediction Step (High Rate):** Propagates state estimates forward using wheel encoders and IMU accelerometers/gyros.
2. **Correction / Update Step (When Available):** Whenever an AprilTag is detected, the EKF calculates the residual difference between expected and observed tag pose, pulling the robot's estimated position back to ground truth.

```
       +-----------------------+     Prediction (50 Hz)
       | Wheel Odom + IMU Data | ------------------------+
       +-----------------------+                         |
                                                         v
                                              +---------------------+      Fused /odometry/filtered
                                              |      ekf_node       | --------------------------->
                                              | (robot_localization)|      TF: odom -> base_link
                                              +---------------------+
                                                         ^
       +-----------------------+                         |
       | AprilTag Visual Poses | ------------------------+
       +-----------------------+      Correction (10 Hz)
```

### b. Covariance: Quantifying Measurement Uncertainty

In ROS 2 message definitions (`PoseWithCovarianceStamped` and `TwistWithCovarianceStamped`), a **$6\times6$ Covariance Matrix** tells the Kalman Filter how much to trust each sensor:
* **Small Covariance ($10^{-4}$):** High confidence (e.g., AprilTag $X, Z$ position when tag is close).
* **Large Covariance ($10^3$):** High uncertainty (e.g., Wheel odometry $X, Y$ during wheel slip in loose dirt).

---

## 4. Hands-On Verification & Checklist
- [ ] Explained the physical meaning of camera optical coordinates ($+X$ right, $+Y$ down, $+Z$ forward).
- [ ] Calculated relative target distance $d = \sqrt{X^2 + Z^2}$ and horizontal bearing $\theta = \text{atan2}(X, Z)$.
- [ ] Understood why Extended Kalman Filters fuse high-rate drifting sensors (wheel odom + IMU) with low-rate absolute visual markers (AprilTags).
