# Dashboard Module 1: Foxglove Studio for Robotics Telemetry & Operator UIs

> **Target:** Master connecting Foxglove Studio to ROS 2 via the Foxglove WebSocket Bridge, configuring 3D PointCloud visualizers, numerical plots, and operator teleoperation dashboards, and exporting reproducible layout JSON configurations.  
> **Key References:** [Foxglove Studio Documentation](https://docs.foxglove.dev) | [Foxglove ROS 2 Bridge](https://github.com/foxglove/ros-foxglove-bridge)

---

## 1. Foxglove Studio Architecture

Foxglove Studio is a modern, extensible robotics visualization platform. It connects to the ROS 2 computational graph via a high-performance WebSocket bridge node (`foxglove_bridge`):

```
+--------------------------------------------------------+                      +-----------------------+
|                   ROS 2 Node Graph                     |                      |    Foxglove Studio    |
|                                                        |                      |    (Web / Desktop)    |
| /camera/depth/color/points  --> [ foxglove_bridge ]    | === ws://localhost:8765 ===>  - 3D Grid Panel   |
| /load_sensor/weight         --> [    (Node)       ]    |                      |  - Plot Panel         |
| /robot_status               -->                        |                      |  - Teleop Panel       |
+--------------------------------------------------------+                      +-----------------------+
```

---

## 2. Launching the Foxglove Bridge

1. In your Dev Container terminal, launch the Foxglove Bridge:
   ```bash
   ros2 run foxglove_bridge foxglove_bridge --ros-args -p port:=8765
   ```
2. Open Foxglove Studio (desktop app or browser at `https://studio.foxglove.dev`).
3. Click **"Open Connection"**, select **"Foxglove WebSocket"**, and connect to `ws://localhost:8765`.

---

## 3. Configuring Visualizer Panels

| Panel Type | Target ROS 2 Topic | Configuration Settings |
| :--- | :--- | :--- |
| **3D Panel** | `/camera/depth/color/points` | Set decay time to 0, point size to 2px, and color by `Z (Height)`. |
| **Plot Panel** | `/load_sensor/weight` | Set rolling time window to 30 seconds, min 0 kg, max 10 kg. |
| **Image Panel** | `/camera/color/image_raw` | Select raw image stream. |
| **Teleop Panel** | `/cmd_vel` | Configure interactive on-screen joystick. |

---

## 4. Exporting & Sharing Dashboard Layouts

To ensure all team members have consistent operator telemetry:
1. Click the layout menu in the top bar of Foxglove Studio.
2. Select **"Export Layout to File"** and save as `cometbot_operator_layout.json`.
3. Track the layout in Git under the `dashboard/` directory.

---

## 5. Verification & Checklist
- [ ] Launched `foxglove_bridge` and established a live WebSocket session.
- [ ] Subscribed Foxglove 3D panel to 3D point cloud topics and verified spatial rendering.
- [ ] Subscribed Plot panel to load sensor weight and plotted real-time telemetry curves.
- [ ] Exported and saved layout JSON file.
