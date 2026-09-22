# Module 2: Essential Linux Commands & Terminal Mastery

> **Target:** Master daily command-line navigation, directory/file management, text streams, pipes, redirection, and system documentation (`man`).  
> **Key Goal:** Complete the online interactive tutorials (Terminal Tutor, Command Line for Beginners, Text-Fu), finish the hands-on practice scenario on your VM / Dev Container, and verify the expected terminal outputs.

---

## 1. Web Tutorials to Follow

Open your VM browser or workstation browser and complete the following online tutorials:

### a. Tutorial 1: Interactive Terminal Training (Terminal Tutor)
* 🌐 **Interactive Practice Course:** [Terminal Tutor (https://www.terminaltutor.com/)](https://www.terminaltutor.com/)
* **What you will practice:**
  * Fundamental directory navigation (`pwd`, `cd`).
  * Directory listing (`ls`).
  * Understanding command syntax, arguments, and option flags.

### b. Tutorial 2: Command Line for Beginners (Ubuntu Official)
* 🌐 **Follow this guide:** [Ubuntu: Command Line for Beginners](https://ubuntu.com/tutorials/command-line-for-beginners)
* **What you will practice:**
  * Navigating directories and relative vs. absolute paths.
  * Creating, moving, copying, and deleting files & directories (`mkdir -p`, `touch`, `cp -r`, `mv`, `rm -rf`).
  * Viewing and scrolling file contents (`cat`, `less`, `head`, `tail`).

### c. Tutorial 3: Text Manipulation, Redirection & Pipes (Linux Journey)
* 🌐 **Follow this guide:** [Linux Journey: Text-Fu](https://linuxjourney.com/lesson/text-fu)
* **What you will practice:**
  * Standard streams (`stdin`, `stdout`, `stderr`).
  * Redirection operators (`>` to create/overwrite, `>>` to append).
  * Chaining commands together using pipes (`|`).
  * Pattern searching and text filtering with `grep`.

### d. Tutorial 4: Built-in Manuals & Help (`man`)
Instead of searching online for every command flag, Linux provides built-in offline manual pages for virtually all system utilities:
* **Opening a Manual:** Run `man <command>` (e.g. `man ls`, `man grep`, `man cat`).
* **Navigating within `man`:**
  * `j` / `Down Arrow` — Scroll down one line.
  * `k` / `Up Arrow` — Scroll up one line.
  * `Space` — Scroll down one full page.
  * `/pattern` then `Enter` — Search forward for a specific keyword (press `n` for next match, `N` for previous match).
  * `q` — Quit and return to the terminal prompt.
### e. Tutorial 5: Package Management in Linux (System `apt` vs. Python `pip` & `venv`)

Package management in Linux allows you to install, update, and manage dependencies without manual `.exe` installers. In robotics, you will work across two primary package managers: **System Packages (`apt`)** and **Python Packages (`pip` & `venv`)**.

#### 1. Linux System Packages (`apt`)
`apt` (Advanced Package Tool) is the Debian/Ubuntu package manager. It downloads pre-compiled `.deb` binaries and resolves system-wide dependencies into `/usr/bin`, `/usr/lib`, and `/opt/ros/`.
* **Update Package Index (`sudo apt update`):** Refreshes your machine's local list of available packages and version numbers from remote repositories listed in `/etc/apt/sources.list`. *(Always run this before installing new software)*.
* **Install Packages (`sudo apt install -y <package>`):** Downloads and installs the package and all required system dependencies.
  ```bash
  sudo apt update && sudo apt install -y htop tree net-tools
  ```
* **Search for Packages (`apt search <keyword>`):** Searches repository indexes for matching software.
  ```bash
  apt search ros-humble-joy
  ```
* **Remove Packages (`sudo apt remove <package>`):** Uninstalls a package while preserving configuration files.
* **ROS 2 Package Naming Convention:** ROS 2 packages distributed via `apt` always follow the format `ros-<distro>-<package-name-with-hyphens>` (e.g. `ros-humble-joy`, `ros-humble-cv-bridge`, `ros-humble-foxglove-bridge`).

#### 2. Python Package Management (`pip` vs. Virtual Environments)
Python packages provide libraries and scripts (`numpy`, `opencv-python`, `scipy`, `matplotlib`) distributed via PyPI (Python Package Index).

* **System Python vs. Pip Packages:**
  * System-wide Python tools can be installed via `apt` (e.g. `sudo apt install python3-pip python3-numpy`), placing files in `/usr/lib/python3/dist-packages/`.
  * Pip (`pip install <package>`) installs Python libraries directly from PyPI into `/usr/local/lib/` or `~/.local/lib/`.
* **Managing Requirements Files:**
  ```bash
  # Install a list of exact dependencies for a robotics node
  pip install -r requirements.txt
  ```
* **Python Virtual Environments (`venv`):**
  A virtual environment is an isolated directory tree containing its own Python interpreter, `pip` binary, and `site-packages` directory. It prevents package version collisions between different projects.
  1. **Create an Environment:**
     ```bash
     python3 -m venv ~/practice_ws/test_env
     ```
     *(Use `--system-site-packages` if the environment needs access to ROS 2 packages in `/opt/ros/humble`).*
  2. **Activate the Environment:**
     ```bash
     source ~/practice_ws/test_env/bin/activate
     ```
     *(Your terminal prompt will change to show `(test_env)` at the beginning).*
  3. **Inspect the Active Environment:**
     ```bash
     which python3   # Points to ~/practice_ws/test_env/bin/python3
     which pip       # Points to ~/practice_ws/test_env/bin/pip
     ```
  4. **Install Packages inside the Virtual Environment:**
     ```bash
     pip install numpy
     ```
  5. **Deactivate and Return to System Shell:**
     ```bash
     deactivate
     ```
  *(Real-world Robotics Example: VS Code PlatformIO uses an isolated virtualenv located in `~/.platformio/penv` to cross-compile embedded micro-ROS firmware without conflicting with your system Python).*

---

## 2. Hands-On Practice Scenario (Run on your VM / Dev Container)

After completing the web tutorials above, execute the following practice task in your terminal:

### a. Instructions
1. Create the directory tree `~/practice_ws/logs` using `mkdir -p`.
2. Generate a system report log at `~/practice_ws/logs/telemetry.log` using command output redirection:
   - Redirect the current timestamp (`date`) into `~/practice_ws/logs/telemetry.log` using `>`.
   - Append kernel and OS information (`uname -a`) to the log using `>>`.
   - Extract and append the total memory line from `/proc/meminfo` using `grep MemTotal /proc/meminfo >> ~/practice_ws/logs/telemetry.log`.
3. Practice chaining commands with pipes (`|`):
   - List files in `/etc` and pipe to grep to find release files: `ls -la /etc | grep release`.
   - Count the number of system processes currently running by piping `ps aux` to `wc -l`.
4. Use `man grep` to look up the options for case-insensitive search (`-i`) and counting matches (`-c`).
5. Practice package management and Python isolation:
   - Search for the ROS joy package with `apt search ros-humble-joy`.
   - Create a Python virtual environment at `~/practice_ws/test_env` using `python3 -m venv ~/practice_ws/test_env`.
   - Activate it (`source ~/practice_ws/test_env/bin/activate`), verify the interpreter path with `which python3`, install NumPy (`pip install numpy`), and test importing it:
     ```bash
     python3 -c "import numpy as np; print('Isolated NumPy version:', np.__version__)"
     ```
   - Deactivate the environment with `deactivate`.

---

## 3. Expected Outputs & Instructor Verification Checklist

Use the expected outputs below to verify your practice task was completed successfully.

### a. Expected Terminal Outputs

#### Check 1: Verify Telemetry Log Creation (`cat ~/practice_ws/logs/telemetry.log`)
* **Expected Output:**
  ```text
  Thu Sep  3 10:45:00 UTC 2026
  Linux <hostname> 5.15.0-xx-generic ...
  MemTotal:        xxxxxx kB
  ```

#### Check 2: Verify Piped Output & Grep (`grep -i "memtotal" ~/practice_ws/logs/telemetry.log`)
* **Expected Output:**
  ```text
  MemTotal:        xxxxxx kB
  ```

#### Check 3: Verify Man Page Navigation (`man grep | grep -m 1 "\-i,"`)
* **Expected Output:**
  ```text
  -i, --ignore-case
  ```

#### Check 4: Verify Python Virtual Environment & NumPy Import
* **Expected Output inside `(test_env)`:**
  ```text
  (test_env) user@host:~/practice_ws$ which python3
  /home/<user>/practice_ws/test_env/bin/python3
  (test_env) user@host:~/practice_ws$ python3 -c "import numpy as np; print('Isolated NumPy version:', np.__version__)"
  Isolated NumPy version: 2.x.x
  ```

### b. Completion Checklist for Instructors & Students
- [ ] Completed Terminal Tutor interactive lessons (`pwd`, `cd`, `ls`, command arguments).
- [ ] Completed Ubuntu Command Line for Beginners tutorial (`mkdir`, `cp`, `mv`, `rm`, `cat`, `less`).
- [ ] Completed Linux Journey Text-Fu tutorial (standard streams, `>` / `>>` redirection, `|` pipes, `grep`).
- [ ] Demonstrated reading and searching manual pages using `man` and `/search` syntax.
- [ ] Successfully generated telemetry log via redirection and processed output using pipes and `grep`.
- [ ] Demonstrated Linux package searching (`apt search`) and understanding of `sudo apt update && sudo apt install`.
- [ ] Created, activated, and tested an isolated Python virtual environment (`venv`) with `pip install numpy`.


