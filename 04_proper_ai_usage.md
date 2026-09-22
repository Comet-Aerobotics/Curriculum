# Module 4: How to Properly Use AI in Engineering

> **Target:** Learn how to properly use AI as a system architecture sparring partner and diagnostic tutor to unblock yourself—while ensuring all production code is understood and 100% owned by you.  
> **Core Principles:**
> 1. **System Architecture (Dialogue Encouraged):** Use multi-turn dialogue to brainstorm node hierarchies, state machines, and communication patterns (Topics vs. Services vs. Actions).
> 2. **Getting Unstuck & Debugging (The 2-Step Diagnostic Workflow):** Use AI to brainstorm hypotheses when stuck, investigate the symptoms yourself, and ask for targeted explanations/fixes.
> 3. **Never Apply Fixes Blindly:** AI-suggested fixes are completely fine to use, but **never run or copy-paste code without understanding *why* it works** and auditing potential side effects.
> 4. **Production Code vs. Helper Scripts:** Production robot code (ROS 2 nodes, controllers, firmware, actions) must be human-authored. AI may assist with one-off helper scripts (e.g. bash install scripts or setup utilities).
> 5. **100% Code Ownership:** You are 100% accountable for every line of code and script you commit.

---

## 1. Engineering Philosophy: Production Code vs. Helper Scripts

### a. Why Production Code Must Be Human-Authored
* **Skill Acquisition:** Onboarding builds your mental models and low-level debugging intuition. Copy-pasting unverified code for core algorithms bypasses understanding.
* **Fragility in Field Robotics:** AI lacks physical ground truth. It does not know when a motor driver is about to overheat or why a CAN bus dropped frames. Engineers who rely on AI to generate production code cannot troubleshoot hardware when the robot fails in the competition arena.

### b. Permissible Use: Assistance & Setup Scripts
It is completely acceptable to leverage AI for developer automation and tooling:
* **Reusable Setup Scripts:** Authoring `postCreate.sh`, docker environment bootstrap scripts, or dependency installers.
* **Data Parsing & Scratch Helpers:** Quick scripts to parse CSV telemetry, format Foxglove JSON layouts, or generate test inputs.

### c. The Golden Rule of Code Ownership
**You must own every line of code you commit—including scripts.**
* Never run or commit a script or code fix you do not understand.
* Saying *"the AI wrote this and that's why it broke"* is never acceptable in code review or post-mortems.

---

## 2. Effective AI Workflows: Architecture, Getting Unstuck & Debugging

```
[ Stuck / Problem Encountered ]
               │
               ▼
   Step 1: Brainstorm Hypotheses with AI  ("What are 4 common reasons for symptom X?")
               │
               ▼
   Step 2: Investigate with CLI / Logs    (Check topic frequency, QoS profiles, echo)
               │
               ▼
   Step 3: Targeted Fix & Understanding   ("I see symptom Y. Why does this happen, and how do I fix it?")
               │
               ▼
   Step 4: Audit & Apply                  (Understand every line before committing)
```

### a. 1. System Architecture: Back-and-Forth Dialogue 🔄
When brainstorming system design, have a multi-turn conversation:
* **Trade-Off Analysis:** Bouncing ideas on Topics vs. Services vs. Actions, message frequency, and computational overhead.
* **Interface Design:** Discussing custom ROS message definitions (`.msg`, `.srv`, `.action`) and parameters.

### b. 2. The 2-Step Diagnostic Workflow (Getting Unstuck & Fixing Bugs) 🎯
When an error occurs or a node silently fails:

1. **Phase 1 — Brainstorm Potential Causes:**
   * Instead of pasting a wall of broken code and asking AI to rewrite it, ask for diagnostic hypotheses:
   * *Example:* *"My ROS 2 subscriber callback is never being triggered even though the topic shows up in `ros2 topic list`. What are 4 possible reasons in ROS 2 Humble that could cause this?"*
2. **Phase 2 — Investigate & Verify Symptoms:**
   * Use ROS 2 CLI tools (`ros2 topic info -v`, `ros2 node info`, `ros2 doctor`) or logs to test the hypotheses.
3. **Phase 3 — Request a Targeted Explanation & Fix:**
   * Once you identify the specific symptom, ask AI how to solve it:
   * *Example:* *"I checked with `ros2 topic info -v` and found a QoS mismatch: the publisher is using `Best Effort` reliability while my subscriber is using `Reliable`. How do I configure my subscriber in Python to match this QoS profile?"*
4. **Phase 4 — Audit Before Applying:**
   * Review the suggested fix. Make sure you understand every argument before applying it to your node.

---

## 3. Good Prompts vs. Bad Prompts

### ❌ Bad Prompts (Blind Code Generation & Passive Guessing)
> *"Here is my broken 200-line script and an error. Fix it for me and give me the new code."*  
> *(Why it's bad: Creates dependency on AI, invites hallucinated logic bugs, and bypasses root-cause understanding).*

---

### ✅ Good Engineering Prompts

#### 1. Step 1 Diagnostic Prompt (Brainstorming Hypotheses):
> *"My ROS 2 node is running and `ros2 topic echo /cmd_vel` shows messages being published, but my subscriber node's callback is never being triggered. What are 4 possible reasons in ROS 2 Humble that could cause a subscriber callback to never fire, and how can I inspect each one using CLI tools?"*

#### 2. Step 2 Targeted Fix Prompt (Informed by Symptoms):
> *"I inspected the topic with `ros2 topic info -v` and confirmed the publisher is using `Best Effort` reliability, but my subscriber is default `Reliable`. Can you show me how to import `QoSProfile` and configure `ReliabilityPolicy.BEST_EFFORT` on a Python subscriber?"*

#### 3. System Architecture Prompt (Multi-turn Discussion):
> *"I am designing a robot docking system in ROS 2 Humble. I need to decide between using a ROS 2 Service vs. a ROS 2 Action Server for initiating and monitoring the docking maneuver. Can you compare the architectural trade-offs regarding feedback, preemption, and non-blocking execution?"*

#### 4. Compiler Error Conceptual Clarification:
> *"I received this C++ error: `error: no matching member function for call to 'create_subscription'`. What causes this signature mismatch conceptually in ROS 2 Humble?"*

---

## 4. Hands-On Practice Lab (Run in Terminal)

Practice an architectural discussion with AI, write a basic node yourself, and verify clean execution.

### a. Step 1: System Design Dialogue with AI
1. Ask AI the **System Architecture Prompt** from Section 3 to explore trade-offs between **Topics**, **Services**, and **Actions**.
2. Conclude with a clear understanding of why continuous sensor feeds use topics, while discrete commands use services/actions.

### b. Step 2: Author Your Own Heartbeat Node
Author your own script `~/practice_ws/heartbeat_node.py`:
```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class HeartbeatNode(Node):
    def __init__(self):
        super().__init__('heartbeat_node')
        self.pub = self.create_publisher(String, '/subteam/heartbeat', 10)
        self.timer = self.create_timer(1.0, self.on_timer)
        self.counter = 0
        self.get_logger().info('Heartbeat Node started.')

    def on_timer(self):
        msg = String()
        msg.data = f'System Active - Tick {self.counter}'
        self.pub.publish(msg)
        self.get_logger().info(f'Published: "{msg.data}"')
        self.counter += 1

def main():
    rclpy.init()
    node = HeartbeatNode()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### c. Step 3: Run & Verify
```bash
python3 ~/practice_ws/heartbeat_node.py
```
*(Press `Ctrl+C` after 3 ticks to verify clean shutdown).*

---

## 5. Instructor Verification Checklist

- [ ] Student understands the policy: **Production robot code must be human-authored; AI may assist with one-off helper/setup scripts**.
- [ ] Student acknowledges the **100% Code Ownership Rule**: You must review, audit, and understand every line of code you commit (including scripts and suggested fixes).
- [ ] Student demonstrates the **2-Step Diagnostic Workflow**:
  1. Uses AI to brainstorm diagnostic hypotheses to get unstuck.
  2. Investigates symptoms with CLI tools/logs and asks for targeted fixes.
  3. Audits and understands fixes before applying them (never applies code blindly).
- [ ] Student authored, reviewed, and ran `heartbeat_node.py` independently.
- [ ] Student confirms no API keys, private passwords, or proprietary code were shared with AI models.
