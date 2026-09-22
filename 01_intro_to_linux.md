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
* **iv. Package Management (`apt` vs. `pip`):** Deterministic, command-line dependency resolution instead of manual `.exe` or `.dmg` installers (covered in hands-on depth in [Module 2](02_basic_linux_commands.md#e-tutorial-5-package-management-in-linux-system-apt-vs-python-pip--venv)).

---

## 3. Robotics Development Environment Setup (VS Code & Dev Containers)

Follow the step-by-step instructions below to configure Docker, Git, VS Code, and launch the team's ROS 2 Humble / micro-ROS Dev Container from the `CAN` repository.

### a. Step 1: Install Docker Desktop / Engine
Select the guide for your host operating system:
* 🪟 **Windows Setup:**  
  Follow the [Docker Desktop for Windows Installation Guide](https://docs.docker.com/desktop/setup/install/windows-install/).
  * If you do not have WSL installed, open an **Administrator PowerShell** and run:
    ```powershell
    wsl --install
    ```
  * *Troubleshooting:* If this fails, search for **"Turn Windows features on or off"** in the Windows Start menu, ensure **"Virtual Machine Platform"** and **"Windows Subsystem for Linux"** are checked, restart your machine, and re-run the Docker Desktop installer selecting the **WSL2 backend**.
  * Ensure Docker Desktop is started and running before proceeding.
* 🍎 **macOS Setup:**  
  Follow the [Docker Desktop for Mac Installation Guide](https://docs.docker.com/desktop/setup/install/mac-install/).
* 🐧 **Linux Setup:**  
  Follow the [Docker Desktop for Linux Installation Guide](https://docs.docker.com/desktop/setup/install/linux/) or [Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/).

---

### b. Step 2: Install Git & Clone the Repository
If you do not have Git installed on your host system:
* 🪟 **Windows:** Download and install [Git for Windows](https://git-scm.com/download/win) (or run `winget install --id Git.Git -e --source winget` in PowerShell).
* 🍎 **macOS:** Run `xcode-select --install` in Terminal, or install via Homebrew with `brew install git`.
* 🐧 **Linux (Ubuntu/Debian):** Run `sudo apt update && sudo apt install -y git`.

Once Git is installed, open your terminal (PowerShell, macOS Terminal, or Linux Bash) in your preferred projects folder and clone the repository:
```bash
git clone https://github.com/Comet-Aerobotics/CAN.git
```

---

### c. Step 3: Install VS Code & Dev Containers Extension
1. 📥 **Install VS Code:** Download and install [Visual Studio Code](https://code.visualstudio.com/download) on your host operating system.
2. 🔌 **Install the Dev Containers Extension:**  
   In VS Code, open the Extensions view (`Ctrl + Shift + X` / `Cmd + Shift + X`) and install:
   * **Dev Containers** (`ms-vscode-remote.remote-containers`)  
   *(Optionally, install the **Remote Development** extension pack `ms-vscode-remote.vscode-remote-extensionpack`).*

---

### d. Step 4: Open the `CAN` Repository in the Dev Container
The cloned `CAN` repository includes a predefined `.devcontainer` configuration (`Dockerfile`, `devcontainer.json`, and `postCreate.sh`) that automatically provisions ROS 2 Humble, micro-ROS setup tools, PlatformIO, CMake, Python, and the necessary VS Code extensions inside an isolated Linux container.

1. **Open `CAN` Folder in VS Code:**  
   Launch VS Code, click **File > Open Folder...**, and select the cloned `CAN` repository folder.
2. **Reopen in Container:**  
   * When the folder opens, VS Code will display a notification in the bottom right corner:  
     > *"Folder contains a Dev Container configuration file. Reopen in Container to develop in a container."*  
     Click **"Reopen in Container"**.
   * *Alternative / Manual Trigger:* Press `F1` (or `Ctrl + Shift + P` / `Cmd + Shift + P`), type **`Dev Containers: Reopen in Container`**, and press `Enter`.
3. **Container Build & Initialization (First-Time Setup: ~15 minutes):**  
   * VS Code will build the Docker container image based on `ros:humble` (pre-installing `ros-humble-joy`, `teleop-twist-joy`, `numpy`, and developer tools).
   * The workspace will be mounted to `/workspace` inside the container.
   * VS Code automatically runs `.devcontainer/postCreate.sh`, which sources ROS 2 Humble in `~/.bashrc`, clones `micro_ros_setup` into `~/microros_ws`, and updates `rosdep`.
   * VS Code automatically installs all required extensions in the container (C/C++, CMake Tools, Python, and PlatformIO IDE).

> [!NOTE]
> **Hardware & Peripheral Access:**  
> The `.devcontainer/devcontainer.json` configuration includes `--privileged` and maps `/dev/ttyUSB0`, `/dev/ttyACM0`, and `/dev/bus/usb` so connected microcontrollers, CAN adapters, and sensors can be accessed directly from within the container.

---

### e. Step 5: Working in the Container
* **Status Bar Indicator:** Once connected, the bottom-left corner of VS Code will display `Dev Container: CAN - ROS2 Humble Dev Container`.
* **Integrated Terminal:** Open a terminal in VS Code (`Ctrl + ~` / `Ctrl + ` ` ` or via **Terminal > New Terminal**).
  * You will be logged in as user `vscode` in `/workspace`.
  * ROS 2 Humble and micro-ROS environment variables are automatically sourced via `~/.bashrc`.

---

### f. Step 6: Test Builds & Verification

#### 1. micro-ROS Embedded Firmware Build Test
> [!NOTE]
> **First-Time Build Time Expectation (~25–45 minutes on laptops):**  
> On the very first run, PlatformIO downloads the ESP32 toolchain, pulls ~50 micro-ROS embedded packages (`rcutils`, `rmw_microxrcedds`, `micro_ros_msgs`, `sensor_msgs`, `geometry_msgs`, `rcl`, `rclc`, etc.), and cross-compiles the entire embedded ROS 2 stack for the microcontroller. **The build is NOT frozen** while displaying `Building micro-ROS library`. Once this initial build finishes, all libraries are permanently cached, and subsequent builds will take only **~5–10 seconds**.

> [!TIP]
> **How to Monitor Live micro-ROS Compilation Progress:**  
> If the terminal appears stuck on `Building micro-ROS library`, you can open a second integrated terminal tab (`Ctrl + Shift + \``) inside the container and check active progress:
> 1. **Check which packages have finished building:**
>    ```bash
>    ls /workspace/microROS_test/.pio/libdeps/esp32dev/micro_ros_platformio/build/mcu/build
>    ```
> 2. **Check active compiler processes & CPU activity:**
>    ```bash
>    top
>    ```
>    *(Look for active `cmake`, `colcon`, and `xtensa-esp32-elf-gcc` compiler jobs).*
> 3. **Run with Verbose Output:**
>    ```bash
>    pio run -v
>    ```

1. In the VS Code integrated terminal, navigate to the `microROS_test` directory:
   ```bash
   cd /workspace/microROS_test
   ```
2. Build the firmware target using PlatformIO:
   ```bash
   pio run
   ```
   *(Alternatively, click the **PlatformIO Build** checkmark $\checkmark$ in the VS Code status bar).*
3. Confirm the build finishes with `[SUCCESS]`.

#### 2. ROS 2 Python Workspace Build Test
1. In the VS Code integrated terminal, navigate to the `cometbot_ws` workspace:
   ```bash
   cd /workspace/cometbot_ws
   ```
2. Build the workspace packages and source the local overlay:
   ```bash
   colcon build
   source install/setup.bash
   ```
3. Test running the pre-installed ROS 2 joy node:
   ```bash
   ros2 run joy joy_node
   ```
   *(Press `Ctrl + C` to stop the node).*
4. Test running the teleop publisher node:
   ```bash
   ros2 run cometbot_control teleop_publisher
   ```
   *(Press `Ctrl + C` to stop the node).*

---

## 4. Expected Outputs & Instructor Verification Checklist

### a. Verification Checks

#### Check 1: Verify Active Dev Container in VS Code
* **Status Indicator:** The VS Code bottom-left status badge displays:
  ```text
  >< Dev Container: CAN - ROS2 Humble Dev Container
  ```
* **Host Docker Verification (`docker ps` on host terminal):**
  ```text
  CONTAINER ID   IMAGE                                 COMMAND                  STATUS         NAMES
  <id>           vsc-can-<hash>-uid                    "sleep infinity"         Up X minutes   <container-name>
  ```

#### Check 2: Verify PlatformIO Embedded Build (`pio run`)
* **Expected Output in `/workspace/microROS_test`:**
  ```text
  ========================= [SUCCESS] Took X.XX seconds =========================
  Environment    Status    Duration
  esp32dev       SUCCESS   00:00:XX
  ```

#### Check 3: Verify ROS 2 Colcon Build (`colcon build`)
* **Expected Output in `/workspace/cometbot_ws`:**
  ```text
  Starting >>> cometbot_control
  Finished <<< cometbot_control [X.XXs]
  Summary: 1 package finished [X.XXs]
  ```

### b. Completion Checklist for Instructors & Students
- [ ] Docker Desktop / Engine installed and running with WSL2 (Windows) or native engine (Linux/macOS).
- [ ] Git installed on the host OS and `https://github.com/Comet-Aerobotics/CAN` cloned.
- [ ] VS Code installed with the **Dev Containers** extension (`ms-vscode-remote.remote-containers`).
- [ ] `CAN` repository opened via Dev Containers (`Dev Container: CAN - ROS2 Humble Dev Container`).
- [ ] Dev Container initialized with automated `postCreate.sh` setup (ROS 2 Humble + `~/microros_ws`).
- [ ] Successfully compiled micro-ROS embedded target in `/workspace/microROS_test` (`pio run`).
- [ ] Successfully built ROS 2 workspace in `/workspace/cometbot_ws` (`colcon build`) and verified `teleop_publisher`.

