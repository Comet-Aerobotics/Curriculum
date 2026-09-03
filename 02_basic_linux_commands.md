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
* **Quick Flag Summary:** Most CLI tools also support the `--help` flag for a quick options summary (e.g. `colcon build --help` or `grep --help`).

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

### b. Completion Checklist for Instructors & Students
- [ ] Completed Terminal Tutor interactive lessons (`pwd`, `cd`, `ls`, command arguments).
- [ ] Completed Ubuntu Command Line for Beginners tutorial (`mkdir`, `cp`, `mv`, `rm`, `cat`, `less`).
- [ ] Completed Linux Journey Text-Fu tutorial (standard streams, `>` / `>>` redirection, `|` pipes, `grep`).
- [ ] Demonstrated reading and searching manual pages using `man` and `/search` syntax.
- [ ] Successfully generated telemetry log via redirection and processed output using pipes and `grep`.


