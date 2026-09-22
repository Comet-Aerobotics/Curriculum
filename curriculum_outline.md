# Robotics & Autonomy Onboarding Curriculum

> **Purpose:** Comprehensive, tutorial-driven onboarding curriculum for new team members to learn essential Linux, Git, AI collaboration, and specialized robotics engineering domains (Controls, Mapping, Simulation, and Telemetry Dashboards).  
> **Approach:** Students complete core foundation modules, progress through specialized track subcurriculums in their Dev Container, and verify their results against expected terminal outputs before completing the Capstone challenge.  
> **Key References:** [ROS 2 Humble Documentation](https://docs.ros.org/en/humble/) | [ROS 2 Action Tutorial (Python)](https://docs.ros.org/en/humble/Tutorials/Intermediate/Writing-an-Action-Server-Client/Py.html) | [Foxglove Studio Docs](https://docs.foxglove.dev) | [micro-ROS Documentation](https://micro.vulcanexus.org/)

---

## Curriculum Overview & Learning Roadmap

```
                                +------------------------------------------+
                                |  Core Module 1: Intro to Linux & Setup   |
                                +------------------------------------------+
                                                     |
                                                     v
                                +------------------------------------------+
                                |  Core Module 2: CLI & Package Management |
                                +------------------------------------------+
                                                     |
                                                     v
                                +------------------------------------------+
                                |  Core Module 7: Git & GitHub Workflows   |
                                +------------------------------------------+
                                                     |
                                                     v
                                +------------------------------------------+
                                |  Core Module 6: Proper AI Engineering    |
                                +------------------------------------------+
                                                     |
         +---------------------------+---------------+---------------------------+
         |                           |                               |           |
         v                           v                               v           v
+------------------+       +-------------------+           +------------------+ +------------------+
|     Controls     |       |      Mapping      |           |    Simulation    | |    Dashboard     |
|  Subcurriculum   |       |   Subcurriculum   |           |  Subcurriculum   | |  Subcurriculum   |
| (Actions/Teleop) |       | (Sensors/Clouds)  |           | (Launch/Mocks)   | | (Foxglove/UIs)   |
+------------------+       +-------------------+           +------------------+ +------------------+
         |                           |                               |           |
         +---------------------------+---------------+---------------------------+
                                                     |
                                                     v
                                +------------------------------------------+
                                |  Final Capstone: End-to-End Integration  |
                                +------------------------------------------+
```

---

# 1. Core Foundation Modules

### 1. [Module 1: Intro to Linux & Development Environment Setup](01_intro_to_linux.md)
* **Core Concepts:** Linux architecture, kernel vs. distribution, ROS 2 & embedded ecosystem, filesystem hierarchy (`/`), permissions model (`sudo`), and Docker containerization.
* **Hands-on Environment Setup:** Docker Desktop (WSL2 backend), Git, and VS Code Dev Containers installation for `https://github.com/Comet-Aerobotics/CAN`.
* **Verification Checkpoint:** Active Dev Container status, micro-ROS firmware build (`pio run`), and ROS 2 joy test.

### 2. [Module 2: Essential Linux Commands & Package Management](02_basic_linux_commands.md)
* **Core Concepts:** Navigation (`pwd`, `cd`, `ls`), file ops (`mkdir`, `cp`, `mv`, `rm`), streams & redirection (`>`, `>>`), pipes (`|`), `grep`, manuals (`man`), and package management (System `apt` vs. Python `pip` & `venv`).
* **Verification Checkpoint:** Telemetry log creation, pipe-to-grep filtering, man page search, and isolated Python virtual environment (`venv`) with NumPy.

### 3. [Module 6: How to Properly Use AI in Engineering](06_proper_ai_usage.md)
* **Core Concepts:** Collaborative architecture dialogue, targeted debugging prompts, root-cause reflection, and docs/log investigation.
* **Verification Checkpoint:** Multi-turn architectural trade-off discussion, human-authored node implementation, and structured reflection debugging.

### 4. [Module 7: Git & GitHub Collaboration Workflows](07_intro_to_github.md)
* **Core Concepts:** Git vs. GitHub, 3 local states, feature branching off `newdepositorbranch`, Conventional Commits scoped to robot subsystems (`feat(depositor):`, `feat(excavator):`, `feat(microros):`), PR lifecycle, and `.gitignore` hygiene.
* **Verification Checkpoint:** Creating a feature branch, committing with Conventional Commits, and verifying commit graph linearity.

---

# 2. Specialized Domain Subcurriculums

### 🕹️ [Controls & Autonomy Track](controls/README.md)
* **[Controls Module 1: ROS 2 Actions — Writing an Action Server & Client in Python](controls/01_ros2_actions_server_client.md)**  
  * Deep dive into the [official ROS 2 Action Tutorial](https://docs.ros.org/en/humble/Tutorials/Intermediate/Writing-an-Action-Server-Client/Py.html).
  * Topics vs. Services vs. Actions decision matrix.
  * Writing Action Servers and Clients in Python with goal validation, live feedback loops, cancellation, and multithreading.
  * Hands-on implementation of the `/excavate` and `/deposit` competition action pipeline.
* **[Controls Module 2: Teleoperation, Joystick Mapping & Actuator Control](controls/02_teleop_and_subsystems.md)**  
  * Gamepad interfacing via `joy_node`, mathematical deadbands ($\sqrt{v_x^2 + \omega_z^2}$), exponential power scaling ($v^3$), and clamped multi-publisher actuator outputs.

---

### 🗺️ [Perception, SLAM & Mapping Track](mapping/README.md)
* **[Mapping Module 1: Computer Vision Concepts, Depth Cameras & Spatial AI](mapping/01_computer_vision_sensors.md)**  
  * Pinhole camera intrinsic matrix $K$, active vs. passive stereo triangulation ($Z = \frac{f \cdot B}{d}$), IR dot projection, and OAK-D Pro hardware edge compute.
* **[Mapping Module 2: Robotics & Computer Vision Data Structures](mapping/02_spatial_data_structures.md)**  
  * Extracting depth matrices with `cv_bridge`, 3D PointClouds (`sensor_msgs/msg/PointCloud2`), 2D Occupancy Grids (`nav_msgs/msg/OccupancyGrid`), AprilTag 6-DoF detection, and TF2 coordinate trees.

---

### 🚀 [Robotics Simulation & Testing Track](simulation/README.md)
* **[Simulation Module 1: ROS 2 Launch Files, Mock Sensors & Mission Simulation](simulation/01_launch_files_and_mission_sim.md)**  
  * Writing modular ROS 2 Python launch files (`launch.py`), declaring launch arguments, instantiating mock sensor nodes (simulated load cells), and running end-to-end mission simulations (`mission_sim.launch.py`).

---

### 📊 [Telemetry & Operator UI Track](dashboard/README.md)
* **[Dashboard Module 1: Foxglove Studio for Robotics Telemetry & Operator UIs](dashboard/01_foxglove_studio_telemetry.md)**  
  * Real-time WebSocket streaming with `foxglove_bridge`, configuring 3D PointCloud visualizers, time-series actuator plots, interactive teleop panels, and exporting version-controlled layout JSON files.

---

# Final Capstone: Cometbot End-to-End Integration Challenge

> **Goal:** Connect all core foundation modules and domain tracks across the Comet-Aerobotics Lunabotics competition stack.

### The Capstone Progression
1. **Step 1 (Git & Dev Container):** In the Dev Container, checkout a new feature branch `feature/onboarding-<name>` from `newdepositorbranch`.
2. **Step 2 (micro-ROS & CAN Firmware):** Build the microcontroller firmware in `/workspace/microROS_test` via `pio run` to verify embedded CAN motor controller communications.
3. **Step 3 (Subsystem & Action Orchestration):** Build `cometbot_ws` (`colcon build`) and launch the simulated excavation/deposition action pipeline (`ros2 launch cometbot_control mission_sim.launch.py`).
4. **Step 4 (Foxglove Telemetry Dashboard):** Connect Foxglove Studio to live topic streams (`/camera/depth/color/points`, `/load_sensor/weight`, `/robot_status`), verify synchronized playback, and export your layout configuration JSON.
5. **Step 5 (PR Submission & Verification):** Push your branch to `https://github.com/Comet-Aerobotics/CAN`, open a Pull Request adhering to Conventional Commits, and document terminal verification outputs.
