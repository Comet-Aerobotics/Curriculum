# Module 5: ROS 2 Topics & Foxglove Studio Visualization

> **Target:** Understand ROS 2 topic-based pub/sub architecture and learn how Foxglove Studio connects to, subscribes to, and visualizes live robotics telemetry.  
> **Key Goal:** Master ROS 2 topic concepts and CLI tools (`ros2 topic`), configure Foxglove panels to visualize camera, 3D point cloud, and time-series topics, build a custom telemetry layout, and export the layout JSON.

---

## 1. What are ROS Topics?

In ROS 2, robots are built as a network of modular, independent processes called **Nodes** (e.g., camera driver node, motor controller node, perception node). Nodes communicate with each other using the **Publisher-Subscriber (Pub/Sub)** model over named channels called **Topics**.

```
  +----------------------+                     +---------------------+
  |   Camera Node        |                     |   Foxglove Studio   |
  |  (depthai_ros)       |                     |     (Subscriber)    |
  |                      |                     |                     |
  | Publishes:           |                     | Visualizes:         |
  |  - /camera/rgb/image |---> [/camera/rgb/image] ----> Image Panel |
  |  - /camera/points    |---> [/camera/points]    ----> 3D Panel    |
  +----------------------+                     +---------------------+
                                                          ^
  +----------------------+                                |
  |   Robot Odometry     |                                |
  |  (diff_drive_node)   |                                |
  |                      |                                |
  | Publishes:           |                                |
  |  - /odom             |---> [/odom] -------------------+--> Plot Panel
  +----------------------+
```

### a. Key Properties of Topics
* **Decoupled & Anonymous:** A publisher node broadcasts data without knowing which nodes (if any) are listening. A subscriber node receives data whenever it is published without needing to know the publisher's identity.
* **Many-to-Many:** Multiple nodes can publish to the same topic, and multiple nodes can subscribe simultaneously.
* **Strongly Typed Messages:** Every topic carries a strictly defined message structure (e.g., `sensor_msgs/msg/Image`, `geometry_msgs/msg/Twist`, `nav_msgs/msg/Odometry`).

### b. Essential ROS 2 Topic CLI Tools
Before visualizing data in a GUI, roboticists inspect topics directly in the terminal:
* **`ros2 topic list`:** Displays all currently active topics in the system.
* **`ros2 topic type <topic>`:** Shows the message type used by a topic (e.g., `sensor_msgs/msg/Image`).
* **`ros2 topic info <topic>`:** Displays publisher count, subscriber count, and message type.
* **`ros2 topic echo <topic>`:** Prints live messages to the terminal in real-time.
* **`ros2 topic hz <topic>`:** Measures the publishing rate (frequency) of a topic (e.g., $30\text{ Hz}$ for camera, $100\text{ Hz}$ for IMU).
* **`ros2 topic pub <topic> <type> "<data>"`:** Publishes a single message or continuous stream from the command line.

---

## 2. How Foxglove Studio Uses ROS Topics

**Foxglove Studio** is a modern robotics visualization and telemetry dashboard. Instead of creating ad-hoc GUIs, Foxglove acts as a rich subscriber interface that connects to your robot's topic graph (either live via `foxglove_bridge` / ROS 2 DDS, or during recorded playback).

### a. Panel-to-Topic Mapping
Foxglove uses specialized panels designed to render specific ROS message types:

| Panel Type | Target ROS Message Type | Example Topic Name | What it Displays |
| :--- | :--- | :--- | :--- |
| **Image Panel** | `sensor_msgs/msg/Image`, `CompressedImage` | `/camera/rgb/image_raw`, `/camera/stereo/depth` | Live camera video feeds, depth heatmaps, detection bounding boxes. |
| **3D Scene Panel** | `sensor_msgs/msg/PointCloud2`, `nav_msgs/msg/OccupancyGrid`, `tf2_msgs/msg/TFMessage` | `/camera/depth/color/points`, `/local_costmap/costmap`, `/tf` | Interactive 3D spatial world: laser scans, point clouds, coordinate axes, and costmaps. |
| **Plot Panel** | Any numerical field within any message | `/odom.twist.twist.linear.x`, `/imu/data.angular_velocity.z` | Real-time graphs showing velocity curves, sensor noise, error metrics, and battery state. |
| **Raw Messages Panel** | Any message type | `/robot_status`, `/diagnostics` | Interactive expandable tree showing raw fields (like a GUI `ros2 topic echo`). |
| **Teleop Panel** | Publishes `geometry_msgs/msg/Twist` | `/cmd_vel` | On-screen joystick/buttons to manually drive the robot from Foxglove. |

### b. Live Connection vs. Recorded Data
* **Live Connection (`foxglove_bridge`):** A lightweight ROS 2 node (`foxglove_bridge`) running on the robot opens a high-speed WebSocket server. Foxglove connects to this WebSocket to subscribe to live topics with low latency over Wi-Fi/Ethernet.
* **Playback Mode:** Foxglove can load recorded robotics sessions (such as `.mcap` topic log files) to scrub backwards and forwards along a unified timeline, synchronizing all panels at once.

---

## 3. Web Tutorials to Follow

Follow the official guides and video tutorials below:

### a. Tutorial 1: Understanding ROS 2 Topics (ROS 2 Official)
* 🌐 **Follow this guide:** [ROS 2: Understanding Topics](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Topics/Understanding-ROS2-Topics.html)
* **What you will learn:**
  * Running publisher and subscriber nodes (`turtlesim`, `teleop_turtle`).
  * Using `ros2 topic list`, `ros2 topic echo`, and `ros2 topic info` to introspect message flow.

### b. Tutorial 2: Getting Started with Foxglove Studio
* 🌐 **Watch the Video Walkthrough:** [Foxglove Studio: Getting Started Video Tutorial](https://www.youtube.com/watch?v=wX5y-P5p58M)
* 🌐 **Follow the Official Guide:** [Foxglove Studio Documentation](https://docs.foxglove.dev/docs/studio/)
* **What you will learn:**
  * Opening Foxglove Studio in browser ([app.foxglove.dev](https://app.foxglove.dev)) or desktop app.
  * Adding panels (Image, 3D, Plot) and configuring their topic subscriptions.
  * Arranging, snapping, and exporting dashboard layouts.

### c. Tutorial 3: Live Robot Telemetry Setup
* 🌐 **Setup Guide:** [Foxglove ROS 2 WebSocket Bridge Documentation](https://github.com/foxglove/ros-foxglove-bridge)
* **What you will learn:**
  * Launching `foxglove_bridge` in ROS 2.
  * Connecting Foxglove Studio to the robot's IP address.

---

## 4. Hands-On Practical Lab (Run on your VM or Browser)

Build a 3-panel telemetry dashboard that visualizes camera, 3D point cloud, and time-series topics:

### a. Step-by-Step Instructions
1. Open [app.foxglove.dev](https://app.foxglove.dev) in your browser (or open the desktop app).
2. Click **Open file from URL** and load a sample robotics dataset:  
   `https://raw.githubusercontent.com/foxglove/mcap/main/testdata/mcap/demo.mcap`  
   *(or connect to a live `foxglove_bridge` if your robot/simulation is running).*
3. Create a clean 3-panel telemetry dashboard:
   * **Top Left — Image Panel:** Subscribe to the primary camera color topic to verify video feed.
   * **Top Right — 3D Scene Panel:** Enable the Point Cloud layer and Coordinate Frame (TF) layer.
   * **Bottom — Plot Panel:** Select a numerical topic field (e.g. forward velocity, angular speed, or position) to plot its time-series curve.
4. Scrub along the timeline and confirm all three panels update in perfect synchronization.
5. In the top navigation bar, click **Layout** $\rightarrow$ **Export layout to file...** and save your configuration as `my_telemetry_layout.json`.

---

## 5. Expected Outputs & Instructor Verification Checklist

### a. Expected Configuration File (`my_telemetry_layout.json`)

Opening your exported layout file should produce valid JSON configuring the panel-to-topic subscriptions:
```json
{
  "configById": {
    "3D!xxxx": { ... },
    "ImageView!xxxx": { ... },
    "Plot!xxxx": { ... }
  },
  "globalVariables": {},
  "layout": {
    "direction": "row",
    "first": "ImageView!xxxx",
    "second": { ... }
  }
}
```

### b. Completion Checklist for Instructors & Students
- [ ] Understands the Pub/Sub architecture and role of topics in ROS 2.
- [ ] Demonstrated running `ros2 topic list`, `ros2 topic echo`, and `ros2 topic info`.
- [ ] Explains which Foxglove panel corresponds to `sensor_msgs/Image`, `sensor_msgs/PointCloud2`, and numerical time-series fields.
- [ ] Configured a 3-panel Foxglove dashboard (Image, 3D Scene, Plot).
- [ ] Exported `my_telemetry_layout.json` and verified valid layout structure.
