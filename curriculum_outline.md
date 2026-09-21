# Robotics & Autonomy Onboarding Curriculum

> **Purpose:** Comprehensive, tutorial-driven onboarding curriculum for new team members to learn essential Linux, Computer Vision, Telemetry, AI, and Git engineering skills.  
> **Approach:** Students follow curated official documentation and interactive web courses on their VM, complete hands-on checkpoints, and verify their results against expected outputs.  
> **Key References:** [ROS 2 Humble Tutorials](https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Cpp-Publisher-And-Subscriber.html) | [micro-ROS Documentation](https://micro.vulcanexus.org/docs/tutorials/core/first_application_linux/) | [Foxglove Studio Docs](https://docs.foxglove.dev)

---

## Curriculum Overview & Learning Roadmap

```
                                +------------------------------------------+
                                |  Module 1: Intro to Linux & Architecture |
                                +------------------------------------------+
                                                     |
                                                     v
                                +------------------------------------------+
                                |  Module 2: Basic & Essential Linux CLI   |
                                +------------------------------------------+
                                                     |
                                                     v
                                +------------------------------------------+
                                |  Module 7: Git & GitHub Team Workflows   |
                                +------------------------------------------+
                                                     |
                                                     v
                                +------------------------------------------+
                                |   Module 6: How to Properly Use AI       |
                                +------------------------------------------+
                                                     |
                         +---------------------------+---------------------------+
                         |                                                       |
                         v                                                       v
+--------------------------------------------------+   +--------------------------------------------------+
| Module 3: Computer Vision Concepts & Sensors     |   | Module 5: Foxglove Studio for Robotics Telemetry |
+--------------------------------------------------+   +--------------------------------------------------+
                         |                                                       |
                         +---------------------------+---------------------------+
                                                     |
                                                     v
                                +------------------------------------------+
                                | Module 4: Robotics & CV Data Structures  |
                                +------------------------------------------+
                                                     |
                                                     v
                                +------------------------------------------+
                                | Final Capstone: End-to-End Integration   |
                                +------------------------------------------+
```

---

# Module Directory & Tutorial Roadmap

### 1. [Module 1: Intro to Linux & Development Environment Setup](01_intro_to_linux.md)
* **Core Concepts:** Linux architecture, kernel vs. distribution, ROS 2 & embedded ecosystem, filesystem hierarchy (`/`), permissions model (`sudo`), and Docker containerization.
* **Hands-on Environment Setup:**
  * Docker Desktop (WSL2 backend), Git, and VS Code Dev Containers installation.
  * Cloning `https://github.com/Comet-Aerobotics/CAN` and opening the project in the automated ROS 2 Humble / micro-ROS Dev Container.
  * In-container test builds for micro-ROS (`/workspace/microROS_test` via `pio run`) and ROS 2 workspace (`/workspace/cometbot_ws` via `colcon build`).
* **Verification Checkpoint:** Dev Container status badge / `docker ps`, micro-ROS build success (`pio run`), and ROS 2 teleop publisher execution.

---

### 2. [Module 2: Essential Linux Commands & Terminal Mastery](02_basic_linux_commands.md)
* **Core Concepts:** Filesystem navigation (`pwd`, `cd`, `ls`), directory/file operations (`mkdir`, `cp`, `mv`, `rm`), standard streams & redirection (`>`, `>>`), pipes (`|`), `grep`, and system documentation (`man`).
* **Web Tutorials to Follow:**
  * [Terminal Tutor (https://www.terminaltutor.com/)](https://www.terminaltutor.com/)
  * [Ubuntu: Command Line for Beginners](https://ubuntu.com/tutorials/command-line-for-beginners)
  * [Linux Journey: Text-Fu](https://linuxjourney.com/lesson/text-fu)
* **Verification Checkpoint:** Generating telemetry log with redirection, piping output to `grep`, and searching manual pages using `man`.

---

### 3. [Module 3: Computer Vision Concepts, Depth Cameras & Spatial AI](03_intro_to_computer_vision.md)
* **Core Concepts:** Pinhole camera model ($K$), stereo triangulation ($Z = \frac{f \cdot b}{d}$), Active vs. Passive Stereo (IR dot projector), OAK-D Pro hardware edge compute (RVC2), and Visual-Inertial SLAM (RGB-D + IMU).
* **Web Tutorials to Follow:**
  * [Luxonis DepthAI ROS 2 Guide](https://docs.luxonis.com/software-v3/depthai/ros)
  * [Luxonis OAK-D Pro Hardware Specifications](https://shop.luxonis.com/products/oak-d-pro)
* **Verification Checkpoint:** Calculating 3D-to-2D pinhole projection and OAK-D Pro stereo disparity-to-depth metrics.

---

### 4. [Module 4: Computer Vision & Robotics Data Structures](04_computer_vision_data_structures.md)
* **Core Concepts:** Extracting Depth Maps (`cv_bridge`), parsing Point Clouds (`sensor_msgs_py` / Open3D), indexing Nav2 Costmaps (`OccupancyGrid`), Luxonis DepthAI AprilTag 6-DoF pose detection, and querying TF2 coordinate frames.
* **Web Tutorials to Follow:**
  * [Open3D: Point Cloud Processing & Downsampling](http://www.open3d.org/docs/release/tutorial/geometry/pointcloud.html)
  * [Nav2 Concepts: Costmaps & Occupancy Grids](https://navigation.ros.org/concepts/index.html)
  * [Luxonis DepthAI AprilTag Documentation](https://docs.luxonis.com/)
  * [ROS 2: Introduction to TF2 & Coordinate Frames](https://docs.ros.org/en/humble/Tutorials/Intermediate/Tf2/Introduction-To-Tf2.html)
* **Verification Checkpoint:** Back-projecting 2D depth pixels into 3D metric coordinates and calculating 2D occupancy grid indices.

---

### 5. [Module 5: ROS 2 Topics & Foxglove Studio Visualization](05_intro_to_foxglove.md)
* **Core Concepts:** ROS 2 Publisher-Subscriber architecture, topic typing and CLI introspection (`ros2 topic list`, `echo`, `hz`), and Foxglove Studio panel-to-topic subscriptions (Image, 3D PointCloud, and Numerical Plot panels).
* **Web Tutorials to Follow:**
  * [ROS 2: Understanding Topics Tutorial](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Topics/Understanding-ROS2-Topics.html)
  * [Foxglove Studio Getting Started Video](https://www.youtube.com/watch?v=wX5y-P5p58M) & [Documentation](https://docs.foxglove.dev/docs/studio/)
  * [Foxglove ROS 2 WebSocket Bridge Guide](https://github.com/foxglove/ros-foxglove-bridge)
* **Verification Checkpoint:** Connecting Foxglove to topic streams, building a 3-panel synchronized dashboard, and exporting the layout JSON.

---

### 6. [Module 6: How to Properly Use AI in Engineering](06_proper_ai_usage.md)
* **Core Concepts:** Collaborative back-and-forth dialogue for system design and architecture; targeted single-prompt explanations for debugging (NOT having AI write code); reflecting on failures (*Why is it breaking? What do you expect of it?*), question refinement, investigating via docs/logs, and restarting fresh for new errors.
* **Web Tutorials & Reference Guides:**
  * [Anthropic Prompt Engineering Interactive Guide](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview)
  * [OpenAI Prompt Engineering Strategies](https://platform.openai.com/docs/guides/prompt-engineering)
  * [ROS 2 Humble Architecture & Concepts](https://docs.ros.org/en/humble/Concepts.html)
  * [micro-ROS First Application on Linux](https://micro.vulcanexus.org/docs/tutorials/core/first_application_linux/)
* **Verification Checkpoint:** Multi-turn architectural trade-off discussion with AI, human-authored node implementation, and single-prompt/reflection debugging practice.

---

### 7. [Module 7: Git & GitHub Collaboration Workflows](07_intro_to_github.md)
* **Core Concepts:** Git vs. GitHub, 3 local states, feature branching off `newdepositorbranch`, Conventional Commits scoped to robot subsystems (`feat(depositor):`, `feat(excavator):`, `feat(microros):`), PR lifecycle, and `.gitignore` hygiene.
* **Web Tutorials to Follow:**
  * [GitHub Skills: Introduction to GitHub (Interactive Course)](https://skills.github.com/)
  * [Learn Git Branching (Interactive Sandbox Game)](https://learngitbranching.js.org/)
  * [Conventional Commits 1.0.0 Specification](https://www.conventionalcommits.org/en/v1.0.0/)
  * [GitHub Docs: Resolving Merge Conflicts](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts/resolving-a-merge-conflict-using-the-command-line)
* **Verification Checkpoint:** Creating a feature branch, committing with Conventional Commits, and verifying commit graph linearity.

---

# Final Capstone: Cometbot End-to-End Integration Challenge

> **Goal:** Test complete end-to-end robotics mastery by connecting all 7 modules across the Comet-Aerobotics Lunabotics competition stack.

### 1. The Capstone Progression
* **a. Step 1 (Git & Dev Container):** In the `microros` Dev Container, checkout a new feature branch `feature/onboarding-<name>` from `newdepositorbranch` in `~/CAN/cometbot_ws`.
* **b. Step 2 (micro-ROS & CAN Firmware):** Build the Teensy/ESP32 firmware in `/root/CAN/microROS_test` via `pio run` to verify CAN motor controller communications.
* **c. Step 3 (Subsystem & Action Orchestration):** Build `cometbot_ws` (`colcon build`) and launch the simulated excavation/deposition action pipeline (`ros2 launch cometbot_control mission_sim.launch.py`).
* **d. Step 4 (Foxglove Telemetry Dashboard):** Connect Foxglove Studio to live topic streams (`/camera/depth/color/points`, `/load_sensor/weight`, `/robot_status`), verify synchronized playback, and export your layout configuration JSON.
* **e. Step 5 (PR Submission & Verification):** Push your branch to `https://github.com/Comet-Aerobotics/CAN`, open a Pull Request adhering to Conventional Commits, and document terminal verification outputs.
