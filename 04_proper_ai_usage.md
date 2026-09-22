# Module 4: How to Properly Use AI in Engineering

> **Target:** Learn how to properly use AI for system design dialogue, targeted debugging, and helper scripting—while ensuring all production code is human-authored and 100% owned by you.  
> **Core Principles:**
> 1. **System Architecture:** Back-and-forth collaborative dialogue is encouraged to explore trade-offs, system design, and communication patterns.
> 2. **Using AI to Get Unstuck:** When hit with an obscure bug or roadblock, do not give up or wait for help passively. Use AI as a diagnostic tutor to brainstorm potential causes and learn unfamiliar concepts—then investigate and write the fix yourself.
> 3. **Production Code vs. Helper Scripts:** All production robot code (ROS 2 nodes, controllers, firmware, state machines) must be human-authored. AI may be used to assist with one-off helper or developer scripts (such as reusable bash install scripts or setup utilities).
> 4. **100% Code Ownership (Even for Scripts):** You are 100% accountable for every line of code you commit to the repository. You must read, audit, understand, and own all code—including AI-assisted scripts.
> 5. **Debugging (Single-Prompt & Reflection):** Use one prompt per error to understand root causes. If it doesn't resolve it, **reflect and refine your question**: *Why is it breaking? What do you expect of it?* Consult docs/logs, and restart fresh for new errors.

---

## 1. Engineering Philosophy: Production Code vs. Helper Scripts

### a. Building Real Engineering Intuition for Production Code
* **i. Skill Acquisition:** The goal of onboarding is to build your mental models, low-level debugging intuition, and command of the robotics stack. Copy-pasting AI-generated code for core robot algorithms bypasses understanding.
* **ii. Fragility in Robotics:** AI lacks physical ground truth. It does not know when a motor driver is about to overheat or why a real CAN bus dropped frames. Engineers who rely on AI to generate production code cannot troubleshoot hardware when the robot fails in the field.
* **iii. Production Code Must Be Human-Authored:** All ROS 2 nodes, action servers, sensor pipelines, and competition state machines must be authored directly by you.

### b. Permissible Use: One-Off Assistance & Developer Scripts
It is completely acceptable to leverage AI for non-production tooling and developer automation:
* **Reusable Installation & Setup Scripts:** Authoring `postCreate.sh`, docker environment bootstrap scripts, or apt/pip dependency installers.
* **Data Parsing & Scratch Helpers:** Quick Python scripts to parse CSV logs, format Foxglove layout JSONs, or generate test inputs.

### c. The Golden Rule of Code Ownership
**You must own every line of code you commit—including scripts.**
* Never run or commit a script you do not understand.
* Saying *"the AI wrote this script and that's why it broke the environment"* is never acceptable in code review or post-mortems.
* You are responsible for security, correctness, idempotency, and error handling for all committed code.

---

## 2. When to Go Back-and-Forth vs. When to Use Targeted Prompts

### a. 1. System Architecture & Design: Back-and-Forth Dialogue Encouraged 🔄
When brainstorming system architecture, you **can and should** have a multi-turn conversation with AI:
* **Explaining Alternatives:** Bouncing ideas on node hierarchies, communication paradigms, and state machine structures.
* **Trade-Off Analysis:** Comparing message passing overhead, latency implications, and modularity.
* **Refining Interfaces:** Discussing custom ROS message definitions, parameter structures, or data flows.

### b. 2. Getting Unstuck: AI as a Diagnostic Brainstorming Partner 💡
When you encounter a silent failure, mysterious behavior, or obscure error where you don't even know what keywords to search for:
* **Don't Give Up or Wait Passively:** Use AI to give you initial traction. Ask for *potential hypotheses* and *diagnostic avenues to investigate*.
* **Ask for Possibilities, Not Fixes:** Ask *"What are 4 common reasons a ROS 2 subscriber callback might never trigger even though the topic exists?"* (e.g. QoS profile mismatch, executor starvation, namespace mismatch, unhandled exceptions).
* **Learn the Underlying Concepts:** When the AI mentions a term you don't know (such as "QoS Reliability Policy" or "Callback Group Deadlock"), look up that topic in the official docs to learn how it works.
* **Self-Reliance:** This transforms AI into a personalized tutor that helps you unblock yourself while keeping you in full control of your understanding.

### c. 3. Debugging: The Single-Prompt & Reflection Rule 🎯
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

#### 1. Getting Unstuck Prompt (Brainstorming Hypotheses):
> *"My ROS 2 node is running and `ros2 topic echo /cmd_vel` shows messages being published, but my subscriber node's callback is never being triggered. What are 4 possible reasons in ROS 2 Humble that could cause a subscriber callback to never fire, and how can I inspect each one using CLI tools?"*

#### 2. System Architecture Prompt (Back-and-Forth Discussion):
> *"I am designing a robot docking system in ROS 2 Humble. I need to decide between using a ROS 2 Service vs. a ROS 2 Action Server for initiating and monitoring the docking maneuver. Can you compare the architectural trade-offs regarding feedback, preemption, and non-blocking execution?"*
> 
> *(Follow-up turn in dialogue)*: *"Given that our docking camera publishes at 30 Hz, how should the Action Server handle intermediate visual alignment feedback without overloading the network?"*

#### 3. Targeted Debugging Prompt (Single-Prompt Rule):
> *"I received this C++ compiler error when building my ROS 2 node:  
> `error: no matching member function for call to 'create_subscription'`  
> Can you explain what causes this type mismatch without writing the solution for me?"*

#### 4. Refined Reflection Prompt (When Initial Prompt Doesn't Resolve / Shows New Error):
> *"My ROS 2 node is receiving `sensor_msgs/msg/Image` on `/camera/stereo/depth`, but my OpenCV conversion fails with `cv_bridge: image encoding '16UC1' cannot be converted to 'bgr8'`.  
> - **Why it is breaking:** I am attempting a color conversion on a single-channel 16-bit millimeter depth map.  
> - **What I expect:** I expect to access raw metric depth values in millimeters, not color image pixels.  
> What is the standard ROS 2 approach to extract raw numeric values from a 16UC1 depth image in Python?"*

#### 5. Conceptual Clarification Prompt:
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
- [ ] Student understands the policy: **Production robot code must be human-authored; AI may assist with one-off helper/setup scripts**.
- [ ] Student acknowledges the **100% Code Ownership Rule**: You must review, audit, and understand every line of code you commit (including scripts).
- [ ] Student knows how to use AI as a diagnostic partner to get unstuck by asking for hypotheses and learning unfamiliar concepts, rather than giving up or seeking copy-paste code.
- [ ] Student understands when to dialogue (system design & architecture) vs. when to use the **Single-Prompt & Reflection Rule** (debugging).
- [ ] Student successfully conducted an architectural trade-off discussion (Topics vs. Services vs. Actions).
- [ ] When debugging, student uses single prompts to explain errors, reflects on *why it broke* and *what was expected*, refines questions, and searches official docs rather than relying on multi-turn code-guessing loops.
- [ ] Student authored, reviewed, and ran `heartbeat_node.py` independently.
- [ ] Student confirms no API keys, private passwords, or proprietary code were shared with AI models.
