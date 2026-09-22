# Perception, SLAM & Mapping Subcurriculum

> **Track Overview:** Learn how robots perceive the physical world using stereo depth cameras, spatial AI, 3D point clouds, 2D occupancy grids, and coordinate frame transformations.

---

## Track Modules

### 1. [Mapping Module 1: Computer Vision Concepts, Depth Cameras & Spatial AI](01_computer_vision_sensors.md)
* **Camera Geometry:** Pinhole camera matrix $K$ and focal length/principal point calibration.
* **Stereo Depth:** Triangulation mathematics ($Z = \frac{f \cdot B}{d}$) and active infrared illumination.
* **Spatial AI Hardware:** OAK-D Pro VPU edge processing and Visual-Inertial Odometry (VIO).

### 2. [Mapping Module 2: Robotics & Computer Vision Data Structures](02_spatial_data_structures.md)
* **Image Conversion:** Bridging ROS images to NumPy arrays with `cv_bridge`.
* **Point Clouds:** Processing and filtering 3D point clouds with `sensor_msgs_py`.
* **Occupancy Grids:** Nav2 2D costmap indexing and coordinate mathematics.
* **TF2 Coordinate Frames:** Querying rigid body transforms across robot kinematic links.
