# Module 6: How to Properly Use AI in Engineering

> **Target:** Learn how to properly use AI for system design dialogue and targeted debugging—**without** having AI write code for you.  
> **Core Principles:**
> 1. **System Architecture:** Back-and-forth collaborative dialogue is encouraged to explore trade-offs and system design.
> 2. **Debugging (Single-Prompt & Reflection):** Use one prompt per error. If it doesn't fix it or reveals a new error, **reflect and refine your question**: *Why is it breaking? What do you expect of it?* Consult docs/logs, and restart fresh for new errors.
> 3. **Implementation:** All code must be human-authored and fully understood by you.

---

## 1. Engineering Philosophy: Why AI Shouldn't Write Your Code
### a. Building Real Engineering Intuition
* **i. Skill Acquisition:** The goal of onboarding is to build your mental models, low-level debugging intuition, and command of the robotics stack. Copy-pasting AI-generated code bypasses understanding.
* **ii. Fragility in Robotics:** AI lacks physical ground truth. It does not know when a motor driver is about to overheat or why a real sensor dropped packets. Engineers who rely on AI to generate code are unable to troubleshoot hardware when the robot fails in the field.
* **iii. Code Ownership & PR Accountability:** In code reviews, you are 100% accountable for every line of code submitted. Saying *"the AI wrote that"* is never acceptable.

---

## 2. When to Go Back-and-Forth vs. When to Use One Prompt

### a. 1. System Architecture & Design: Back-and-Forth Dialogue Encouraged 🔄
When brainstorming system architecture, you **can and should** have a multi-turn conversation with AI:
* **Explaining Alternatives:** Bouncing ideas on node hierarchies, communication paradigms, and state machine structures.
* **Trade-Off Analysis:** Comparing message passing overhead, latency implications, and modularity.
* **Refining Interfaces:** Discussing custom ROS message definitions, parameter structures, or data flows.

### b. 2. Debugging: The Single-Prompt & Reflection Rule 🎯
When debugging broken code or compiler errors, **do not** engage in long back-and-forth loops where AI blindly guesses fixes:

* **Step 1 — The Single Prompt:** Use **one single prompt** to understand what a specific compiler error, assertion, or stack trace means conceptually.
* **Step 2 — Reflect & Refine (If it doesn't fix it or reveals a new error):**  
  Do not immediately type *"it still didn't work"* or paste the next error blindly into the chat. Step back and reflect:
  * **Why is it breaking?** What underlying assumption in your code or environment failed? (e.g., wrong data type, unhandled null values, missing frame transformation in TF2, wrong units).
  * **What do you expect of it?** Clearly define what the input is, what transformation should occur, and what the expected output state is.
  * **Refine your question:** Formulate a much more precise question anchored in your reflection and the system's actual constraints, rather than asking for code.
* **Step 3 — Investigate & Search:** Cross-reference your hypothesis with the codebase, debug logs, Google / StackOverflow, or official documentation.
* **Step 4 — Start Fresh for New Errors:** If you fix the root cause and encounter a completely new error downstream, start a clean conversation/prompt with your refined understanding to prevent context pollution and hallucination drift.
* **Why?** Multi-turn debugging with AI creates a "hallucination spiral" where the model blindly edits code until it accidentally compiles, leaving behind subtle runtime and logic bugs.

---

## 3. Good Prompts vs. Bad Prompts

### ❌ Bad Prompts (Asking AI to write your code or do your work)
> *"Write a Python script for a ROS 2 node that subscribes to camera images and publishes bounding boxes."*  
> *(Why it's bad: You learn nothing, copy-paste unverified code, and cannot maintain it).*

> *"Here is my buggy code and my compiler error. Fix it for me and give me the new code."*  
> *(Why it's bad: Creates dependency on AI and bypasses root-cause understanding).*

---

### ✅ Good Engineering Prompts

#### 1. System Architecture Prompt (Back-and-Forth Discussion):
> *"I am designing a robot docking system in ROS 2 Humble. I need to decide between using a ROS 2 Service vs. a ROS 2 Action Server for initiating and monitoring the docking maneuver. Can you compare the architectural trade-offs regarding feedback, preemption, and non-blocking execution?"*
> 
> *(Follow-up turn in dialogue)*: *"Given that our docking camera publishes at 30 Hz, how should the Action Server handle intermediate visual alignment feedback without overloading the network?"*

#### 2. Targeted Debugging Prompt (Single-Prompt Rule):
> *"I received this C++ compiler error when building my ROS 2 node:  
> `error: no matching member function for call to 'create_subscription'`  
> Can you explain what causes this type mismatch without writing the solution for me?"*

#### 3. Refined Reflection Prompt (When Initial Prompt Doesn't Resolve / Shows New Error):
> *"My ROS 2 node is receiving `sensor_msgs/msg/Image` on `/camera/stereo/depth`, but my OpenCV conversion fails with `cv_bridge: image encoding '16UC1' cannot be converted to 'bgr8'`.  
> - **Why it is breaking:** I am attempting a color conversion on a single-channel 16-bit millimeter depth map.  
> - **What I expect:** I expect to access raw metric depth values in millimeters, not color image pixels.  
> What is the standard ROS 2 approach to extract raw numeric values from a 16UC1 depth image in Python?"*

#### 4. Conceptual Clarification Prompt:
> *"Can you explain intuitively why camera optical frames use +Z forward, +X right, +Y down, while mobile robot body frames use +X forward, +Y left, +Z up according to REP 103?"*

---

## 4. Web References to Consult Before / Alongside AI

Always anchor your technical understanding in official documentation:
* 🌐 **ROS 2 Concepts Overview:** [ROS 2 Humble Architecture & Concepts](https://docs.ros.org/en/humble/Concepts.html)
* 🌐 **micro-ROS Architecture:** [micro-ROS Core Architecture](https://micro.vulcanexus.org/docs/tutorials/core/first_application_linux/)
* 🌐 **Prompt Engineering Guide:** [Anthropic Prompt Engineering Best Practices](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview)

---

## 5. Hands-On Practice Lab (Run on your VM or Browser)

In this lab, you will practice having an architectural discussion with AI, writing the node yourself, and using the single-prompt and reflection workflow to understand a simulated compiler/runtime error.

### a. Step 1: System Design Dialogue with AI
1. Open your AI tool and paste the **System Architecture Prompt** from Section 3.
2. Have a 2-3 turn dialogue exploring the architectural trade-offs between **Topics**, **Services**, and **Actions**.
3. Conclude the discussion with a clear mental model of when to choose each communication pattern.

### b. Step 2: Write Your Own Node (Human-Authored)
Based on your architectural understanding, author your own script `~/practice_ws/heartbeat_node.py`:
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

### c. Step 3: Run & Test
```bash
python3 ~/practice_ws/heartbeat_node.py
```
*(Press `Ctrl+C` after 3 ticks to verify clean shutdown).*

---

## 6. Expected Outputs & Instructor Verification Checklist

### a. Expected Terminal Outputs

```text
[INFO] [1725321600.100000000] [heartbeat_node]: Heartbeat Node started.
[INFO] [1725321601.100000000] [heartbeat_node]: Published: "System Active - Tick 0"
[INFO] [1725321602.100000000] [heartbeat_node]: Published: "System Active - Tick 1"
[INFO] [1725321603.100000000] [heartbeat_node]: Published: "System Active - Tick 2"
^C
```

### b. Completion Checklist for Instructors & Students
- [ ] Student understands the strict policy: **Do NOT use AI to write your code**.
- [ ] Student understands when to dialogue (system design & architecture) vs. when to use the **Single-Prompt & Reflection Rule** (debugging).
- [ ] Student successfully conducted an architectural trade-off discussion (Topics vs. Services vs. Actions).
- [ ] When debugging, student uses single prompts to explain errors, reflects on *why it broke* and *what was expected*, refines questions, and searches official docs rather than relying on multi-turn code-guessing loops.
- [ ] Student authored, reviewed, and ran `heartbeat_node.py` independently.
- [ ] Student confirms no API keys, private passwords, or proprietary code were shared with AI models.
