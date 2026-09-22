# Controls & Autonomy Subcurriculum

> **Track Overview:** Learn the fundamentals of robot controls, 1-to-1 Services, long-running Action state machines, and teleoperation signal conditioning in ROS 2.

---

## Track Modules

### 1. [Controls Module 1: ROS 2 Services — Writing a Service Server & Client in Python](01_ros2_services_server_client.md)
* **Topics vs. Services vs. Actions:** Communication paradigm decision matrix.
* **Official ROS 2 Humble Service Tutorial Walkthrough:** Implementing `create_service`, callback handling, `create_client`, and `call_async()`.
* **Subsystem Calibration Triggers:** Practical implementation of sensor tare and calibration services (`/zero_load_cell`).
* **CLI Service Introspection:** Inspecting and triggering services with `ros2 service list`, `type`, and `call`.

### 2. [Controls Module 2: ROS 2 Actions — Writing an Action Server & Client in Python](02_ros2_actions_server_client.md)
* **Goal, Feedback, and Result Lifecycle:** Understanding asynchronous execution, feedback streaming, and cancellation handling.
* **Official ROS 2 Humble Action Tutorial Walkthrough:** Implementing `ActionServer`, `ActionClient`, `execute_callback`, `goal_callback`, `cancel_callback`, and `MultiThreadedExecutor`.
* **Cometbot Subsystem Actions:** Writing and testing the `/excavate` and `/deposit` action pipeline with live weight/depth feedback.
* **CLI Introspection:** Sending goals and inspecting live progress using `ros2 action send_goal --feedback`.

### 3. [Controls Module 3: Teleoperation, Joystick Mapping & Actuator Control](03_teleop_and_subsystems.md)
* **Gamepad Interfacing:** Interfacing with Linux gamepads via `joy_node`.
* **Signal Conditioning:** Implementing mathematical deadbands ($\sqrt{v_x^2 + \omega_z^2}$) and polynomial sensitivity scaling ($v^3$).
* **Subsystem Control:** Managing multi-publisher actuator setpoints (`Float32`) with hardware limit clamping.
