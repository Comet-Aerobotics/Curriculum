# Mapping Module 2: Camera Intrinsics & Isaac Sim Perception Interface

> **Target:** Understand the Camera Intrinsics Matrix ($K$), how to retrieve camera parameters, and master the high-level ROS 2 interface contract between NVIDIA Isaac Sim and your perception/autonomy stack.  
> **Key References:** [ROS 2 sensor_msgs/CameraInfo](https://docs.ros2.org/latest/api/sensor_msgs/msg/CameraInfo.html) | [NVIDIA Isaac Sim ROS 2 Bridge](https://docs.isaacsim.omniverse.nvidia.com/)

---

## 1. What is the Camera Intrinsics Matrix ($K$)?

A raw digital image is just a 2D grid of pixels ($u, v$). Without **Camera Intrinsics**, a robot cannot tell whether an object in an image is a small marker $1\text{ meter}$ away or a gigantic billboard $100\text{ meters}$ away.

The **Intrinsics Matrix ($K$)** describes the optical properties inside the camera lens:

$$K = \begin{bmatrix} f_x & 0 & c_x \\ 0 & f_y & c_y \\ 0 & 0 & 1 \end{bmatrix}$$

* **$f_x, f_y$ (Focal Lengths in pixels):** How strongly the lens magnifies and bends incoming light rays.
* **$c_x, c_y$ (Principal Point in pixels):** The exact pixel coordinate where the optical center of the lens aligns with the digital sensor (typically near the image center, e.g. $(640, 360)$ for a $1280\times720$ camera).

### How to Get the Intrinsics Matrix in ROS 2
You do not need to compute $K$ by hand:
1. **In Physical Hardware (e.g. OAK-D Pro):** Factory calibration is stored in internal EEPROM and published automatically by the camera driver on `/camera/color/camera_info`.
2. **In Simulation (Isaac Sim):** Isaac Sim calculates $K$ based on the simulated focal length and sensor aperture, publishing it continuously on `/camera/color/camera_info`.

To inspect the live intrinsics of any active camera:
```bash
ros2 topic echo /camera/color/camera_info --once
```

---

## 2. High-Level Perception Interface with Isaac Sim

When testing perception or mapping algorithms in simulation, the Mapping team does **not** need to build physics engines or 3D meshes. You only need to know the **ROS 2 Topic Interface Contract**:

```
+-----------------------------------------------------------------------------------+
|                              NVIDIA Isaac Sim Engine                              |
|                                                                                   |
|           [Simulated RGB Camera]      [Simulated Depth / Lidar]    [Simulated IMU] |
|                      |                            |                       |       |
+----------------------|----------------------------|-----------------------|-------+
                       |                            |                       |
            ROS 2 Communication Bridge (Domain ID 0)                        |
                       |                            |                       |
                       v                            v                       v
+-----------------------------------------------------------------------------------+
|                        Your ROS 2 Perception Stack (Dev Container)                |
|                                                                                   |
|  - AprilTag Detector (/camera/color/image_raw + /camera/color/camera_info)        |
|  - Point Cloud Obstacle Filter (/camera/depth/color/points)                       |
|  - EKF Sensor Fusion Node (/imu/data + /odom + /apriltag_pose)                    |
+-----------------------------------------------------------------------------------+
```

---

## 3. The Isaac Sim Topic Contract (Inputs & Outputs)

### a. Outputs from Isaac Sim (Perception Ingestion)
The simulation generates synthetic sensor streams that match real physical sensor messages:

| ROS 2 Topic | Message Type | Purpose in Mapping & Perception |
| :--- | :--- | :--- |
| `/camera/color/image_raw` | `sensor_msgs/msg/Image` | Color RGB image stream for AprilTag detection and object recognition. |
| `/camera/color/camera_info` | `sensor_msgs/msg/CameraInfo` | $3\times3$ $K$ matrix, distortion coefficients, and resolution. |
| `/camera/depth/image_rect_raw` | `sensor_msgs/msg/Image` | 32-bit floating point metric depth map ($Z$ in meters). |
| `/camera/depth/color/points` | `sensor_msgs/msg/PointCloud2` | 3D $(X, Y, Z, \text{RGB})$ point cloud of the virtual environment. |
| `/imu/data` | `sensor_msgs/msg/Imu` | High-rate simulated angular velocity and linear acceleration. |
| `/odom` | `nav_msgs/msg/Odometry` | Simulated wheel ground-truth velocity and dead-reckoning. |

### b. Inputs to Isaac Sim (Control Loopback)
Commands published by your controls nodes that drive the simulated robot:
* `/cmd_vel` (`geometry_msgs/msg/Twist`): Linear ($v_x$) and angular ($\omega_z$) drive commands.
* `/depositor/position_setpoint` & `/excavator/velocity_setpoint` (`std_msgs/msg/Float32`): Subsystem joint actuator positions.

---

## 4. How the Mapping Team Verifies Simulation Feeds

To verify that synthetic sensor feeds are ready for your perception algorithms:

1. **Verify Topic Availability:**
   ```bash
   ros2 topic list
   ```
2. **Verify Camera Stream Frequency (Expect 20–30 Hz):**
   ```bash
   ros2 topic hz /camera/color/image_raw
   ```
3. **Run your AprilTag detector against the simulated stream:**
   ```bash
   ros2 run apriltag_ros apriltag_node --ros-args \
     -r image_rect:=/camera/color/image_raw \
     -r camera_info:=/camera/color/camera_info
   ```

Because ROS 2 abstracts hardware behind standard message interfaces, **your perception code does not change at all between Isaac Sim and the real robot.**

---

## 5. Verification Checklist
- [ ] Explained the role of the Intrinsics Matrix ($K$) in converting 2D pixel coordinates to 3D metric space.
- [ ] Queried and inspected a live `sensor_msgs/msg/CameraInfo` topic.
- [ ] Identified the key perception output topics from Isaac Sim (`/image_raw`, `/camera_info`, `/points`, `/imu/data`).
