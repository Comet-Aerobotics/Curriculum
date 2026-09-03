# Module 4: Computer Vision & Robotics Data Structures

> **Target:** Master extracting, processing, and converting core robotics perception data structures: Depth Maps, Point Clouds (`PointCloud2`), Nav2 Occupancy Grids, Luxonis OAK AprilTag detections, and TF2 coordinate frames.  
> **Key Goal:** Understand how each data structure is extracted from ROS 2 topics and the OAK camera, study the linked tutorials, complete the 2D-to-3D back-projection exercise, and verify results against expected outputs.

---

## 1. Core Perception Data Structures & How to Extract Them

Robotics perception pipelines transform raw camera sensor pixels into structured 3D spatial representations. Below is how each data structure is formatted and extracted in ROS 2 and Python:

```
                          +-----------------------------+
                          |   Luxonis OAK-D Pro / ROS 2 |
                          +-----------------------------+
                                         |
     +-------------------+---------------+-------------------+-------------------+
     |                   |                                   |                   |
     v                   v                                   v                   v
[Depth Map]       [PointCloud2]                      [OAK AprilTag]       [TF2 Frames]
(16UC1 / 32FC1)   (3D XYZ + RGB)                      (6-DoF Pose)     (Spatial Tree)
     |                   |                                   |                   |
     v                   v                                   v                   v
cv_bridge /         sensor_msgs_py /                    DepthAI Node /       tf2_ros /
Back-projection     Open3D Voxel Filter                 apriltag_msgs        TransformBuffer
     |                   |                                   |                   |
     +-------------------+-----------------------------------+-------------------+
                                         |
                                         v
                         [Nav2 2D/3D Occupancy Costmap]
                           (Free, Occupied, Unknown)
```

---

### a. Depth Maps (`sensor_msgs/msg/Image`)
* **Data Format:** A 2D array where each pixel $(u, v)$ stores distance $Z$ from the camera sensor plane, encoded as `16UC1` (unsigned 16-bit integer in millimeters) or `32FC1` (32-bit floating point in meters).
* **How to Extract in ROS 2 / Python:**
  Subscribe to `/camera/stereo/depth` and convert the ROS Image message into a NumPy 2D array using `cv_bridge`:
  ```python
  from cv_bridge import CvBridge
  import numpy as np

  bridge = CvBridge()
  depth_image = bridge.imgmsg_to_cv2(msg, desired_encoding="16UC1") # mm units
  depth_meters = depth_image.astype(np.float32) / 1000.0            # convert to meters
  ```
* **Back-Projecting $(u, v)$ Pixels to 3D Metric Coordinates $(X, Y, Z)$:**
  Using the camera intrinsics ($f_x, f_y, c_x, c_y$ from `/camera/rgb/camera_info`):
  $$X = \frac{(u - c_x) \cdot Z}{f_x}, \qquad Y = \frac{(v - c_y) \cdot Z}{f_y}, \qquad Z = Z$$

---

### b. Point Clouds (`sensor_msgs/msg/PointCloud2`)
* **Data Format:** A packed binary payload representing an unordered collection of 3D spatial points with geometry fields (`x`, `y`, `z`) and optional attributes (`rgb`, `intensity`).
* **How to Extract in ROS 2 / Python:**
  Subscribe to `/camera/depth/color/points` and parse the binary buffer into structured tuples or NumPy arrays using `sensor_msgs_py`:
  ```python
  import sensor_msgs_py.point_cloud2 as pc2
  import numpy as np

  # Generator yielding (x, y, z) coordinates, skipping invalid NaNs
  point_generator = pc2.read_points(cloud_msg, field_names=("x", "y", "z"), skip_nans=True)
  points_3d = np.array(list(point_generator)) # Shape: (N, 3)
  ```
* **Processing with Open3D:** Pass `points_3d` into Open3D to perform **Voxel Grid Downsampling** (reducing point density while preserving geometry) and **RANSAC Plane Segmentation** (detecting and filtering out the floor plane).

---

### c. Occupancy Grids (`nav_msgs/msg/OccupancyGrid`)
* **Data Format:** A 2D probabilistic top-down grid representing space for robot path planning. The grid cell values are:
  * `-1`: Unknown space (unexplored).
  * `0`: Free space (navigable).
  * `100`: Occupied obstacle (wall, object).
  * `1` to `99`: Cost inflation zones surrounding obstacles.
* **How to Extract in ROS 2 / Python:**
  Subscribe to `/local_costmap/costmap_raw` or `/global_costmap/costmap` published by Nav2:
  ```python
  import numpy as np

  resolution = msg.info.resolution          # meters per cell (e.g. 0.05m = 5cm)
  width = msg.info.width                    # number of cells wide
  height = msg.info.height                  # number of cells high
  origin_x = msg.info.origin.position.x     # world X coordinate of bottom-left cell (0, 0)
  origin_y = msg.info.origin.position.y     # world Y coordinate of bottom-left cell (0, 0)

  # Reshape 1D flattened array into 2D matrix (height, width)
  grid_2d = np.array(msg.data, dtype=np.int8).reshape((height, width))
  ```
* **Coordinate Conversion (World Metric $\leftrightarrow$ Grid Cell Index):**
  $$\text{cell}_x = \left\lfloor \frac{X_{\text{world}} - \text{origin}_x}{\text{resolution}} \right\rfloor, \qquad \text{cell}_y = \left\lfloor \frac{Y_{\text{world}} - \text{origin}_y}{\text{resolution}} \right\rfloor$$

---

### d. AprilTag Fiducials (Luxonis DepthAI AprilTag API)
* **Data Format:** High-contrast 2D barcodes (such as `tag36h11`) placed on landmarks, charging docks, or competition game elements. By matching known physical tag dimensions against 2D corner pixels and stereo depth, the camera computes the exact **6-DoF Rigid Body Pose** $[R \mid t]$ (3D position $x, y, z$ and 3D orientation quaternion $q_x, q_y, q_z, q_w$).
* **How to Extract with Luxonis OAK-D / DepthAI:**
  The OAK-D platform runs the `AprilTag` node directly in the pipeline, executing corner detection and Perspective-n-Point (PnP) pose estimation:
  ```python
  # DepthAI Python Pipeline Definition
  import depthai as dai

  pipeline = dai.Pipeline()
  mono_left = pipeline.create(dai.node.MonoCamera)
  apriltag = pipeline.create(dai.node.AprilTag)

  # Configure tag family and edge refinement
  apriltag.initialConfig.setFamily(dai.AprilTagConfig.Family.TAG_36H11)
  mono_left.out.link(apriltag.inputImage)
  ```
* **ROS 2 Output (`apriltag_msgs/msg/AprilTagDetectionArray` or TF2):**
  Publishes detected tag IDs, decision margins, center/corner pixel coordinates, and 3D pose relative to the camera optical frame.

---

### e. TF2 Coordinate Frame Trees (REP 103 / REP 105)
* **Conventions:**
  * **Robot Base Frame (`base_link`):** Standard robotics Cartesian frame ($+X$ Forward, $+Y$ Left, $+Z$ Up).
  * **Camera Optical Frame (`camera_color_optical_frame`):** Standard optical ray frame ($+Z$ Forward along optical axis, $+X$ Right, $+Y$ Down).
* **How to Query Transforms in ROS 2 / Python:**
  ```python
  import tf2_ros
  import rclpy

  tf_buffer = tf2_ros.Buffer()
  tf_listener = tf2_ros.TransformListener(tf_buffer, node)

  # Look up transformation from camera frame to robot base_link
  transform = tf_buffer.lookup_transform(
      target_frame='base_link',
      source_frame='camera_color_optical_frame',
      time=rclpy.time.Time()
  )
  ```

---

## 2. Web Tutorials & Documentation to Follow

Follow the official documentation and guides below:

### a. Tutorial 1: Point Clouds & Voxel Filtering (Open3D)
* 🌐 **Follow this tutorial:** [Open3D: Point Cloud Processing & Downsampling](http://www.open3d.org/docs/release/tutorial/geometry/pointcloud.html)
* **What you will learn:**
  * Loading and converting $N \times 3$ NumPy point arrays into Open3D point cloud objects.
  * Applying `voxel_down_sample(voxel_size)` to compress dense point clouds for real-time robotics planning.
  * Applying RANSAC plane segmentation (`segment_plane`) to isolate ground planes from obstacle geometry.

### b. Tutorial 2: Nav2 Costmaps & Occupancy Grids (ROS 2)
* 🌐 **Follow this tutorial:** [Nav2 Concepts: Costmaps & Occupancy Grids](https://navigation.ros.org/concepts/index.html)
* **What you will learn:**
  * How Nav2 costmap 2D layers (Static Layer, Obstacle Layer via PointCloud2 raycasting, Inflation Layer) combine into a single costmap.
  * Understanding lethal obstacle costs ($254$), inscribed costs ($253$), and free space ($0$).

### c. Tutorial 3: Luxonis DepthAI AprilTag Marker Detection
* 🌐 **Follow this guide:** [Luxonis DepthAI AprilTag Documentation](https://docs.luxonis.com/)
* **What you will learn:**
  * Initializing the `AprilTag` node on DepthAI.
  * Tuning detection parameters: `quadDecimate`, `quadSigma`, and `maxHammingDistance`.
  * Extracting 6-DoF tag poses for robot docking and visual localization.

### d. Tutorial 4: Coordinate Frames & TF2 (ROS 2 Official)
* 🌐 **Follow this tutorial:** [ROS 2: Introduction to TF2 & Coordinate Frames](https://docs.ros.org/en/humble/Tutorials/Intermediate/Tf2/Introduction-To-Tf2.html)
* **What you will learn:**
  * The hierarchy of coordinate frames: `map` $\rightarrow$ `odom` $\rightarrow$ `base_link` $\rightarrow$ `camera_link` $\rightarrow$ `camera_optical_frame`.
  * Transforming 3D points and vectors between sensor frames and the world map.

---

## 3. Hands-On Practical Lab (Run on your VM / Workstation)

Apply the back-projection equations and coordinate transformations to extract 3D metric points from 2D depth map pixels.

### a. Problem Statement
An OAK-D Pro camera with calibrated intrinsics $f_x = 600.0\text{ px}$, $f_y = 600.0\text{ px}$, $c_x = 320.0\text{ px}$, $c_y = 240.0\text{ px}$ captures a depth frame:
1. **Target Point 1:** Pixel $(u_1 = 440\text{ px}, v_1 = 120\text{ px})$ has a measured depth of $Z_1 = 2.40\text{ meters}$.  
   **Calculate the 3D coordinates $(X_1, Y_1, Z_1)$ in the camera optical frame.**
2. **Target Point 2:** Pixel $(u_2 = 200\text{ px}, v_2 = 360\text{ px})$ has a measured depth of $Z_2 = 1.80\text{ meters}$.  
   **Calculate the 3D coordinates $(X_2, Y_2, Z_2)$ in the camera optical frame.**
3. **Occupancy Grid Mapping:**  
   A Nav2 costmap has a resolution of $0.05\text{ m/cell}$ ($5\text{ cm}$) and origin at $(X_{\text{origin}} = -10.0\text{ m}, Y_{\text{origin}} = -10.0\text{ m})$. An obstacle is detected at world coordinates $(X_{\text{obs}} = 2.50\text{ m}, Y_{\text{obs}} = 4.20\text{ m})$.  
   **Calculate the 2D grid cell indices $(\text{cell}_x, \text{cell}_y)$.**

---

## 4. Expected Outputs & Instructor Verification Checklist

### a. Expected Calculation Outputs

#### Check 1: 3D Back-Projection for Point 1
$$X_1 = \frac{(u_1 - c_x) \cdot Z_1}{f_x} = \frac{(440 - 320) \cdot 2.40}{600.0} = \frac{120 \cdot 2.40}{600.0} = \mathbf{+0.480\text{ m}}$$
$$Y_1 = \frac{(v_1 - c_y) \cdot Z_1}{f_y} = \frac{(120 - 240) \cdot 2.40}{600.0} = \frac{-120 \cdot 2.40}{600.0} = \mathbf{-0.480\text{ m}}$$
$$Z_1 = \mathbf{2.400\text{ m}}$$
* **Expected Result (Point 1):** **$(X = +0.480\text{ m}, Y = -0.480\text{ m}, Z = 2.400\text{ m})$**

#### Check 2: 3D Back-Projection for Point 2
$$X_2 = \frac{(u_2 - c_x) \cdot Z_2}{f_x} = \frac{(200 - 320) \cdot 1.80}{600.0} = \frac{-120 \cdot 1.80}{600.0} = \mathbf{-0.360\text{ m}}$$
$$Y_2 = \frac{(v_2 - c_y) \cdot Z_2}{f_y} = \frac{(360 - 240) \cdot 1.80}{600.0} = \frac{120 \cdot 1.80}{600.0} = \mathbf{+0.360\text{ m}}$$
$$Z_2 = \mathbf{1.800\text{ m}}$$
* **Expected Result (Point 2):** **$(X = -0.360\text{ m}, Y = +0.360\text{ m}, Z = 1.800\text{ m})$**

#### Check 3: Occupancy Grid Index Calculation
$$\text{cell}_x = \left\lfloor \frac{2.50 - (-10.0)}{0.05} \right\rfloor = \left\lfloor \frac{12.50}{0.05} \right\rfloor = \mathbf{250}$$
$$\text{cell}_y = \left\lfloor \frac{4.20 - (-10.0)}{0.05} \right\rfloor = \left\lfloor \frac{14.20}{0.05} \right\rfloor = \mathbf{284}$$
* **Expected Result (Grid Index):** **$(\text{cell}_x = 250, \text{cell}_y = 284)$**

### b. Completion Checklist for Instructors & Students
- [ ] Understands how to extract and decode `16UC1` depth map images using `cv_bridge`.
- [ ] Understands how to parse `sensor_msgs/PointCloud2` binary buffers into NumPy / Open3D arrays using `sensor_msgs_py`.
- [ ] Explains how Nav2 costmaps convert 3D obstacles and 2D laser scans into occupancy grid layers (`-1`, `0`, `100`).
- [ ] Understands how Luxonis DepthAI AprilTag node outputs 6-DoF rigid-body poses ($[R \mid t]$) from visual markers.
- [ ] Hand-calculated or scripted solutions to Check 1, Check 2, and Check 3 with exact matches to expected outputs.
- [ ] Can articulate coordinate conventions between `base_link` ($+X$ fwd) and `camera_optical_frame` ($+Z$ fwd).
