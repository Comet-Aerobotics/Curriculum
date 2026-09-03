# Module 7: Git & GitHub Collaboration Workflows

> **Target:** Master Git version control, branching strategies, Conventional Commits, Pull Request workflows, and merge conflict resolution applied directly to the **Comet-Aerobotics/CAN** repository.  
> **Key Goal:** Complete the interactive Git courses, execute a feature-branch commit workflow using Conventional Commits, and verify your commit history.

---

## 1. Core Git & GitHub Concepts Overview

In the Comet-Aerobotics robotics software team, Git is our local version control engine and GitHub is our collaboration hub for code reviews, automated CI testing, and tracking issues across the robot stack.

```
+---------------------------------------------------------------------------------------+
|                                    GitHub Remote                                      |
|                     https://github.com/Comet-Aerobotics/CAN                          |
|                                                                                       |
|   main (Protected Production) <--- [Pull Request] <--- newdepositorbranch (Feature)   |
+------------------------------------------^--------------------------------------------+
                                           | git push origin <branch>
+------------------------------------------v--------------------------------------------+
|                             Local Dev Container / VM                                  |
|                                                                                       |
|   Working Directory  ---[ git add ]---> Staging Area ---[ git commit ]---> Local Git |
|  (/root/CAN/...)                         (Index)                             History  |
+---------------------------------------------------------------------------------------+
```

### a. The 3 Local States + Remote
1. **Working Directory:** The local files you are actively modifying in VS Code (e.g. `cometbot_ws/src/cometbot_control/` or `microROS_test/`).
2. **Staging Area (`git add`):** A buffer tracking which modified files will be packaged into the next commit snapshot.
3. **Local Repository (`git commit`):** Permanent historical commits stored locally in your `.git` folder.
4. **Remote Repository (`git push`):** The upstream GitHub server (`origin`) accessible by all team members.

### b. Branching Strategy on `Comet-Aerobotics/CAN`
* **`main` Branch:** Stable, competition-ready codebase. Direct commits to `main` are strictly prohibited.
* **Feature Branches (`newdepositorbranch`, `feature/<subsystem>-<name>`):** All active development (e.g., depositor action server, CAN sniffer upgrades, AprilTag docking) occurs in isolated feature branches branched off `main` or subsystem integration branches.
* **Pull Requests (PRs):** Changes are merged back via GitHub PRs only after peer review, code review comments are addressed, and integration checks pass.

### c. Conventional Commits Standard
To maintain clean, readable commit histories, all commits must follow the [Conventional Commits](https://www.conventionalcommits.org/) format:

$$\text{<type>}(\text{<scope>}): \text{<concise description in imperative mood>}$$

**Common Subsystem Scopes in `Comet-Aerobotics/CAN`:**
* `feat(depositor): add action server and hopper tilt feedback loop`
* `feat(excavator): implement load sensor threshold trigger for dig cycle`
* `feat(microros): add SPARK MAX periodic CAN frame sniffer`
* `fix(control): resolve teleop joystick deadband calculation`
* `docs(orchestration): update action orchestration FSM diagram`
* `refactor(can): standardize CAN Device Interface message headers`

### d. `.gitignore` Hygiene in Robotics
Robotics workspaces generate large build folders and binary logs that **must never be committed to Git**:
* **Ignored Directories:** `cometbot_ws/build/`, `cometbot_ws/install/`, `cometbot_ws/log/`, `microROS_test/.pio/`, `.vscode/`
* **Ignored Files:** `.mcap`, `.bag`, `.tar.gz` container images, `.pyc` Python bytecode, and firmware `.hex`/`.bin` artifacts.

---

## 2. Interactive Web Tutorials to Follow

Open your web browser and complete the following interactive tutorials:

### a. Tutorial 1: Introduction to GitHub (Interactive GitHub Course)
* 🌐 **Follow this interactive course:** [GitHub Skills: Introduction to GitHub](https://skills.github.com/)
* **What you will practice:**
  * Creating branches, making commits, and opening Pull Requests.
  * Reviewing diffs, adding review comments, and merging PRs.

### b. Tutorial 2: Visual Git Branching & Merging (Interactive Sandbox)
* 🌐 **Play this interactive game:** [Learn Git Branching (https://learngitbranching.js.org/)](https://learngitbranching.js.org/)
* **Complete the first 4 levels of "Main":**
  1. Introduction to Git Commits (`git commit`)
  2. Branching in Git (`git branch`, `git checkout` / `git switch`)
  3. Merging in Git (`git merge`)
  4. Git Rebase (`git rebase`)

### c. Tutorial 3: Conventional Commits & Conflict Resolution
* 🌐 **Specification Guide:** [Conventional Commits 1.0.0 Standard](https://www.conventionalcommits.org/en/v1.0.0/)
* 🌐 **Conflict Resolution Guide:** [GitHub Docs: Resolving Merge Conflicts](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts/resolving-a-merge-conflict-using-the-command-line)

---

## 3. Hands-On Practical Lab (Run on your Dev Container / VM)

In this lab, you will configure your local Git identity, create a feature branch within your repository workspace, and commit changes using Conventional Commits.

### a. Step-by-Step Instructions
1. **Configure Git Identity:**  
   In your container terminal, set your name and email (matching your GitHub account):
   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "your.email@example.com"
   git config --global init.defaultBranch main
   ```
2. **Navigate to Repository Workspace:**
   ```bash
   cd ~/CAN/cometbot_ws/src/cometbot_control
   ```
3. **Check Branch & Status:**
   ```bash
   git status
   git branch
   ```
4. **Create a Feature Branch:**
   ```bash
   git checkout -b feature/telemetry-docs
   ```
5. **Stage & Commit Changes with Conventional Commits:**
   Create or edit a documentation file (e.g. `TELEMETRY_NOTES.md`), stage it, and commit:
   ```bash
   git add TELEMETRY_NOTES.md
   git commit -m "docs(telemetry): add Foxglove topic streaming guidelines"
   ```
6. **Inspect the Commit Graph:**
   ```bash
   git log --oneline --graph -n 5
   ```

---

## 4. Expected Outputs & Instructor Verification Checklist

### a. Expected Terminal Outputs

#### Check 1: Verify Git Configuration (`git config --list`)
* **Expected Output:**
  ```text
  user.name=<Your Name>
  user.email=<Your Email>
  init.defaultbranch=main
  ```

#### Check 2: Verify Commit History Graph (`git log --oneline --graph -n 3`)
* **Expected Output:**
  ```text
  * xxxxxxx (HEAD -> feature/telemetry-docs) docs(telemetry): add Foxglove topic streaming guidelines
  * xxxxxxx (origin/newdepositorbranch, newdepositorbranch) ...
  ```

### b. Completion Checklist for Instructors & Students
- [ ] Completed GitHub Skills: Introduction to GitHub course.
- [ ] Completed first 4 levels of Learn Git Branching interactive simulator.
- [ ] Local Git identity configured correctly (`user.name`, `user.email`).
- [ ] Created feature branch (`feature/<name>`) without modifying `main` directly.
- [ ] Commits strictly adhere to Conventional Commits with appropriate subsystem scopes (`feat(depositor):`, `feat(excavator):`, `docs(telemetry):`).
- [ ] Explains why build artifacts (`build/`, `install/`, `.pio/`) must remain in `.gitignore`.
- [ ] Understands the Pull Request review lifecycle on `https://github.com/Comet-Aerobotics/CAN`.
