# Module 1: Introduction to Linux & Development Environment Setup

> **Target:** Understand Linux architecture, contrast it with Windows/macOS, and configure your robotics development environment using Docker and VS Code Dev Containers.  
> **Key Goal:** Set up Docker and VS Code Dev Containers, build micro-ROS and ROS 2 test packages, and verify your environment with the completion checklist.

---

## 1. What is Linux?
### a. Kernel vs. Distribution (Distro)
* **i. The Linux Kernel:** The low-level software engine managing CPU scheduling, RAM allocation, peripheral hardware drivers, and system calls.
* **ii. Linux Distributions (Distros):** Complete operating systems bundling the kernel with GNU utilities, package managers, and desktop graphical environments.
  * 1. *Ubuntu LTS (Long-Term Support):* Our robotics standard (Ubuntu 22.04 LTS), offering 5 years of stable upstream package support and tier-1 ROS 2 compatibility.
  * 2. *Debian:* The upstream, rock-solid base upon which Ubuntu is built.
* **iii. Why Robotics Runs on Linux:**
  * 1. Native ROS 2 & Robotics Ecosystem: Primary tier-1 operating system for ROS 2, bridging high-level autonomy with microcontrollers and embedded firmware via PlatformIO and micro-ROS.
  * 2. Direct hardware interface access (`/dev/ttyUSB*`, `/dev/can*`, `/dev/video*`).
  * 3. Headless deployment on onboard compute (NVIDIA Jetson, Raspberry Pi, x86 mini-PCs).

---

## 2. Linux vs. Windows vs. macOS
### a. Architecture & Conceptual Differences
* **i. Single Unified Root (`/`):** All drives and peripherals mount within one directory hierarchy (no `C:\` or `D:\` drives).
* **ii. "Everything is a File":** Devices, serial ports, and live telemetry appear as file paths (`/dev/`, `/proc/`, `/sys/`).
* **iii. Security & Permissions:** Strict separation between unprivileged users and superuser administrative tasks via `sudo`.
* **iv. Package Management (`apt`):** Deterministic, command-line dependency resolution instead of manual `.exe` or `.dmg` installers.

---

## 3. Robotics Development Environment Setup (Docker & Dev Containers)

Follow the step-by-step instructions below to configure Docker, VS Code, and the team's micro-ROS / ROS 2 Dev Container environment.

### a. Step 1: Install Docker Desktop / Engine
Select the guide for your host operating system:
* 🪟 **Windows Setup:**  
  Follow the [Docker Desktop for Windows Installation Guide](https://docs.docker.com/desktop/setup/install/windows-install/).
  * If you do not have WSL installed, open an **Administrator PowerShell** and run:
    ```powershell
    wsl --install
    ```
  * *Troubleshooting:* If this fails, search for **"Turn Windows features on or off"** in the Windows Start menu, ensure **"Virtual Machine Platform"** and **"Windows Subsystem for Linux"** are checked, restart your machine, and re-run the installer selecting the **WSL2 backend**.
* 🍎 **macOS Setup:**  
  Follow the [Docker Desktop for Mac Installation Guide](https://docs.docker.com/desktop/setup/install/mac-install/).
* 🐧 **Linux Setup:**  
  Follow the [Docker Desktop for Linux Installation Guide](https://docs.docker.com/desktop/setup/install/linux/) or [Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/).

### b. Step 2: Download & Run the Docker Container
1. 📥 **Download Container Archive:**  
   [Download the Docker container (`microros.tar.gz`)](https://drive.google.com/open?id=1zUqxaGNR6tewxhOx2clXEHCjqa0ZG-T5)
2. **Load and Run the Container:**  
   Open a terminal in the folder containing `microros.tar.gz` and execute:
   ```bash
   docker load -i microros.tar.gz
   docker run -it --net=host -v /dev:/dev --privileged --name microros basicuros
   ```
   > [!NOTE]
   > `--net=host` allows ROS 2 DDS discovery across your local network.  
   > `-v /dev:/dev --privileged` maps connected microcontrollers (e.g. `/dev/ttyUSB0`, CAN adapters) into the container.

3. **Re-opening the Container on Daily Work:**  
   Once the container is created, get an interactive shell session anytime by running:
   ```bash
   docker start -i microros bash
   ```
   *(Or click the terminal icon within Docker Desktop).*
   * 💡 **Windows 1-Click Terminal Shortcut:**  
     In Windows Terminal Settings (`Ctrl + ,`), add a new profile with the command line:
     ```powershell
     %SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe docker start -i microros bash
     ```

### c. Step 3: Install VS Code & Dev Container Extensions
1. 📥 **Install VS Code:** Download and install [Visual Studio Code](https://code.visualstudio.com/download).
2. 🔌 **Install Remote / Dev Containers Extension Pack:**  
   In VS Code, open the Extensions tab (`Ctrl + Shift + X`) and install:
   * `ms-vscode-remote.vscode-remote-extensionpack` (Remote Development)
   * `ms-vscode-remote.remote-containers` (Dev Containers)

### d. Step 4: Connect VS Code to the Running Container
1. Ensure the `microros` Docker container is running (`docker start -i microros bash`).
2. In VS Code, click the **Remote Status Bar icon** in the bottom-left corner (or press `F1` / `Ctrl + Shift + P`).
3. Select **"Dev Containers: Attach to Running Container..."** (or **"Attach to Running Container..."**) and select **`microros`**.
4. A new VS Code window will open attached directly inside the container environment.
5. In the attached window, go to **File > Open Folder...** and open either:
   * `/root/CAN/microROS_test/` (Embedded micro-ROS firmware repository)
   * `~/CAN/cometbot_ws` (ROS 2 robot workspace)
6. *If prompted:* Allow CMake to **"scan for kits"**, open the parent git repository, and choose the provided `CMakeLists.txt`. Run `git pull` if necessary to fetch the latest upstream changes.

### e. Step 5: Install Required Extensions Inside the Container
Inside the container-attached VS Code window, open Extensions (`Ctrl + Shift + X`) and verify/install:
* `ms-vscode.cpptools-extension-pack` (C/C++ Extension Pack)
* `platformio.platformio-ide` (PlatformIO IDE for embedded builds)
* `nonanonno.vscode-ros2` (ROS 2 tooling & syntax support)

### f. Step 6: Test Builds & Verification

#### 1. micro-ROS Build Test
1. Press `Ctrl + Shift + P` and execute `CMake: Configure`.
2. Open `src/main.cpp`.
3. Click the **PlatformIO Build** checkmark ($\checkmark$) in the status bar at the bottom (or run `pio run` in the terminal).
4. Confirm the build completes with `[SUCCESS]`.

#### 2. ROS 2 Python Workspace Build Test
1. In VS Code, open `~/CAN/cometbot_ws`.
2. Open an integrated terminal (`Ctrl + ~`) and build the workspace:
   ```bash
   colcon build
   source install/setup.bash
   ```
3. Test running a ROS 2 joy node:
   ```bash
   ros2 run joy joy_node
   ```
   *(Press `Ctrl + C` to stop the node).*
4. Test running the teleop publisher node:
   ```bash
   ros2 run cometbot_control teleop_publisher
   ```

---

## 4. Expected Outputs & Instructor Verification Checklist

### a. Verification Checks

#### Check 1: Verify Running Docker Container (`docker ps`)
* **Expected Output on Host Machine:**
  ```text
  CONTAINER ID   IMAGE        COMMAND   CREATED         STATUS         PORTS   NAMES
  <id>           basicuros    "bash"    5 minutes ago   Up 5 minutes           microros
  ```

#### Check 2: Verify PlatformIO Embedded Build (`pio run`)
* **Expected Output in `/root/CAN/microROS_test`:**
  ```text
  ========================= [SUCCESS] Took X.XX seconds =========================
  Environment    Status    Duration
  teensy40       SUCCESS   00:00:XX
  ```

#### Check 3: Verify ROS 2 Colcon Build (`colcon build`)
* **Expected Output in `~/CAN/cometbot_ws`:**
  ```text
  Starting >>> cometbot_control
  Finished <<< cometbot_control [X.XXs]
  Summary: 1 package finished [X.XXs]
  ```

### b. Completion Checklist for Instructors & Students
- [ ] Docker installed and configured with WSL2 / native backend.
- [ ] `microros.tar.gz` image loaded and container created with `--net=host` and `/dev` permissions.
- [ ] VS Code Dev Containers attached to the running `microros` container.
- [ ] In-container extensions installed (`C/C++`, `PlatformIO`, `ROS 2`).
- [ ] Successfully compiled micro-ROS embedded target (`pio run` / CMake).
- [ ] Successfully built ROS 2 workspace (`colcon build`) and ran `teleop_publisher`.

