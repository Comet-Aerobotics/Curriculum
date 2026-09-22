# Perception, Mapping & Localization Subcurriculum

> **Track Overview:** Learn how to extract 6-DoF poses from AprilTag visual markers, fuse multi-sensor streams using Extended Kalman Filters (EKF), understand camera intrinsics, and test perception nodes against NVIDIA Isaac Sim synthetic sensor streams.

---

## Track Modules

### 1. [Mapping Module 1: AprilTag 6-DoF Pose Estimation & Kalman Filter Sensor Fusion](01_apriltags_and_sensor_fusion.md)
* **6-DoF Pose Intuition:** Physical interpretation of camera optical frames ($+X$ right, $+Y$ down, $+Z$ depth forward) and 3D rotations (roll, pitch, yaw).
* **Relative Target Geometry:** Calculating metric distance ($d = \sqrt{X^2 + Z^2}$) and relative bearing angles ($\theta = \text{atan2}(X, Z)$).
* **Extended Kalman Filtering (EKF):** Using `robot_localization` (`ekf_node`) to fuse high-rate drifting measurements (wheel odometry + IMU) with low-rate zero-drift visual landmarks (AprilTags).
* **Measurement Covariance:** Understanding confidence matrices in state estimation.

### 2. [Mapping Module 2: Camera Intrinsics & Isaac Sim Perception Interface](02_camera_intrinsics_and_isaac_sim_interface.md)
* **Camera Intrinsics Matrix ($K$):** How focal lengths ($f_x, f_y$) and principal points ($c_x, c_y$) map 2D image pixels to 3D metric rays.
* **Inspecting Camera Info:** Querying live `sensor_msgs/msg/CameraInfo` streams.
* **Isaac Sim Interface Contract:** High-level input/output topic contract (`/camera/color/image_raw`, `/camera_info`, `/depth`, `/points`, `/imu/data`, `/cmd_vel`) to test perception pipelines in simulation without hardware.
