# Controls & Autonomy Subcurriculum

> **Track Overview:** Learn the fundamentals of robot controls, teleoperation signal conditioning, and long-running behavioral state orchestration in ROS 2.

---

## Track Modules

### 1. [Controls Module 1: ROS 2 Actions — Writing an Action Server & Client in Python](01_ros2_actions_server_client.md)
* **Topics vs. Services vs. Actions:** Communication paradigm tradeoffs and architecture.
* **Official ROS 2 Humble Action Tutorial Walkthrough:** Implementing `ActionServer`, `ActionClient`, `execute_callback`, `goal_response_callback`, `feedback_callback`, and multithreaded executors.
* **Cometbot Subsystem Actions:** Writing and testing the `/excavate` and `/deposit` action pipeline with live weight/depth feedback.
* **CLI Introspection:** Sending goals and inspecting live progress using `ros2 action send_goal --feedback`.

### 2. [Controls Module 2: Teleoperation, Joystick Mapping & Actuator Control](02_teleop_and_subsystems.md)
* **Gamepad Interfacing:** Interfacing with Linux gamepads via `joy_node`.
* **Signal Conditioning:** Implementing mathematical deadbands ($\sqrt{v_x^2 + \omega_z^2}$) and polynomial sensitivity scaling ($v^3$).
* **Subsystem Control:** Managing multi-publisher actuator setpoints (`Float32`) with hardware limit clamping.
