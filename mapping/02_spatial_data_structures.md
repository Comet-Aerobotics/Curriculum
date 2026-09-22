# Mapping Module 2: Robotics & Computer Vision Data Structures

> **Target:** Master extracting depth data with `cv_bridge`, parsing 3D PointClouds (`sensor_msgs/msg/PointCloud2`), indexing 2D Occupancy Grids (`nav_msgs/msg/OccupancyGrid`), AprilTag 6-DoF pose detection, and querying TF2 coordinate transforms.  
> **Key References:** [ROS 2 cv_bridge](https://github.com/ros-perception/vision_opencv) | [Nav2 Costmaps & Occupancy Grids](https://navigation.ros.org/concepts/index.html) | [ROS 2 TF2 Tutorial](https://docs.ros.org/en/humble/Tutorials/Intermediate/Tf2/Introduction-To-Tf2.html)

---

## 1. Depth Map Extraction with `cv_bridge`

`cv_bridge` converts ROS 2 Image messages (`sensor_msgs/msg/Image`) into OpenCV NumPy matrices:

```python
from cv_bridge import CvBridge
import numpy as np

bridge = CvBridge()
# Convert 16-bit millimeter depth image to float metric array
depth_image_mm = bridge.imgmsg_to_cv2(msg, desired_encoding='16UC1')
depth_meters = depth_image_mm.astype(np.float32) / 1000.0
```

---

## 2. 3D Point Clouds (`sensor_msgs/msg/PointCloud2`)

Point clouds represent unstructured 3D environments as arrays of $(X, Y, Z, \text{RGB}, \text{Intensity})$ points:

```python
from sensor_msgs_py import point_cloud2

# Generator yields (x, y, z) tuples in the sensor coordinate frame
for point in point_cloud2.read_points(cloud_msg, field_names=['x', 'y', 'z'], skip_nans=True):
    x, y, z = point
```

---

## 3. 2D Occupancy Grids & Nav2 Costmaps (`nav_msgs/msg/OccupancyGrid`)

An `OccupancyGrid` flattens 3D obstacles into a 2D row-major array representing cell probability of occupancy:
* `-1`: Unknown space
* `0`: Free space
* `100`: Fully occupied obstacle

### Coordinate Conversion:
$$\text{grid\_x} = \lfloor \frac{x_{\text{world}} - x_{\text{origin}}}{\text{resolution}} \rfloor, \quad \text{grid\_y} = \lfloor \frac{y_{\text{world}} - y_{\text{origin}}}{\text{resolution}} \rfloor$$
$$\text{index} = \text{grid\_y} \times \text{width} + \text{grid\_x}$$

---

## 4. Coordinate Transformations with TF2

Robots consist of many moving frames (`map`, `odom`, `base_link`, `camera_link`, `excavator_link`). TF2 tracks these frames through time:

```python
from tf2_ros import Buffer, TransformListener

tf_buffer = Buffer()
tf_listener = TransformListener(tf_buffer, node)

# Query transform from camera_link to base_link at current time
transform = tf_buffer.lookup_transform('base_link', 'camera_link', rclpy.time.Time())
```

---

## 5. Verification & Checklist
- [ ] Converted `sensor_msgs/msg/Image` into OpenCV depth matrix and extracted metric distance.
- [ ] Converted world coordinates $(x, y)$ into 1D Occupancy Grid flat array indices.
- [ ] Looked up relative coordinate frames using `tf2_ros`.
