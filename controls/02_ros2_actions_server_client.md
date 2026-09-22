# Controls Module 2: ROS 2 Actions — Writing an Action Server & Client in Python

> **Target:** Master ROS 2 Actions for long-running, preemptible robotic behaviors. Understand the action communication lifecycle (Goal, Feedback, Result), implement an Action Server and Action Client in Python following official ROS 2 Humble specifications, and apply actions to autonomous competition subsystems (`Excavate` & `Deposit`).  
> **Prerequisites:** [Controls Module 1: ROS 2 Services](01_ros2_services_server_client.md)  
> **Key References:** [ROS 2 Humble Action Tutorial (Python)](https://docs.ros.org/en/humble/Tutorials/Intermediate/Writing-an-Action-Server-Client/Py.html) | [ROS 2 Action Design Article](https://design.ros2.org/articles/actions.html)

---

## 1. Action Fundamentals & Communication Paradigms

In [Controls Module 1](01_ros2_services_server_client.md), you mastered discrete 1-to-1 **Services**. While services are ideal for instantaneous triggers, **Actions** are designed for tasks that take seconds or minutes to complete and require continuous progress updates.

### a. Topics vs. Services vs. Actions Comparison

| Paradigm | Pattern | Blocking / Async | Feedback During Execution? | Preemptible / Cancellable? | Ideal Robotics Use Case |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Topics** | Publish / Subscribe (1-to-Many) | Asynchronous Streaming | No (continuous stream) | No | Sensor telemetry (`/camera/image`, `/imu/data`), joystick drive velocity (`/cmd_vel`). |
| **Services** | Request / Response (1-to-1) | Synchronous / Async RPC | No | No | Instantaneous state queries or quick toggles (`/reset_encoders`, `/zero_load_cell`, `/set_camera_exposure`). |
| **Actions** | Goal / Feedback / Result (1-to-1) | Asynchronous Task Pipeline | **Yes (live stream)** | **Yes (cancel at any time)** | Long-running behaviors taking seconds to minutes: `/excavate`, `/deposit`, `/navigate_to_pose`. |

### b. The ROS 2 Action Architecture (Goal, Feedback, Result)

An action is composed of three underlying services and one topic:
1. **Goal Service:** The Action Client sends a goal request to the Action Server. The server accepts or rejects the goal.
2. **Feedback Topic:** While executing the task, the server periodically publishes real-time progress updates back to the client.
3. **Result Service:** Once the task completes, fails, or is cancelled, the server returns the final outcome to the client.
4. **Cancel Service:** The client can request to abort or cancel the active goal at any point during execution.

```
+---------------+                                     +---------------+
|               |  ---- (1) Send Goal (Service) ----> |               |
|               |  <--- (2) Goal Accepted/Rejected -- |               |
| Action Client |                                     | Action Server |
|    (Node)     |  <--- (3) Live Feedback (Topic) --- |    (Node)     |
|               |                                     |               |
|               |  <--- (4) Final Result (Service) -- |               |
+---------------+                                     +---------------+
       |                                                      ^
       +----------- (5) Request Cancel (Service) ------------+
```

---

## 2. Defining an Action Interface (`.action` File)

Action interfaces are defined in `.action` files within the `action/` directory of a ROS 2 interface package (such as `cometbot_msgs`).

An `.action` file consists of three parts separated by `---`:
```text
# 1. Goal Request (Sent from Client to Server)
float32 target_depth_cm
float32 duration_sec
---
# 2. Final Result (Returned from Server to Client when finished)
bool success
float32 final_weight_kg
string message
---
# 3. Live Feedback (Streamed periodically from Server to Client while running)
float32 current_depth_cm
float32 current_weight_kg
float32 percent_complete
```

---

## 3. Official ROS 2 Tutorial: Writing an Action Server in Python

Following the [official ROS 2 Humble Python Action Server Tutorial](https://docs.ros.org/en/humble/Tutorials/Intermediate/Writing-an-Action-Server-Client/Py.html), here is how to construct a robust Python Action Server using `rclpy.action.ActionServer`.

### a. Complete Action Server Implementation (`excavator_action_server.py`)

```python
#!/usr/bin/env python3
import time
import rclpy
from rclpy.action import ActionServer, CancelResponse, GoalResponse
from rclpy.callback_groups import ReentrantCallbackGroup
from rclpy.executors import MultiThreadedExecutor
from rclpy.node import Node

# Import the custom action definition
from cometbot_msgs.action import Excavate

class ExcavatorActionServer(Node):
    def __init__(self):
        super().__init__('excavator_action_server')

        # Use a ReentrantCallbackGroup to allow concurrent feedback and cancel handling
        self._callback_group = ReentrantCallbackGroup()

        # Initialize the ActionServer
        self._action_server = ActionServer(
            self,
            Excavate,
            'excavate',
            execute_callback=self.execute_callback,
            goal_callback=self.goal_callback,
            cancel_callback=self.cancel_callback,
            callback_group=self._callback_group
        )

        self.get_logger().info('Excavator Action Server initialized and ready for goals.')

    def goal_callback(self, goal_request):
        """Validates incoming goal parameters before accepting."""
        self.get_logger().info(f'Received goal request: Target Depth = {goal_request.target_depth_cm} cm')
        if goal_request.target_depth_cm < 0.0 or goal_request.target_depth_cm > 50.0:
            self.get_logger().warn('Goal rejected: Target depth out of safe range (0-50 cm).')
            return GoalResponse.REJECT
        return GoalResponse.ACCEPT

    def cancel_callback(self, goal_handle):
        """Handles client cancellation requests."""
        self.get_logger().info('Received cancellation request for active excavation.')
        return CancelResponse.ACCEPT

    def execute_callback(self, goal_handle):
        """Executes the long-running task and publishes live feedback."""
        self.get_logger().info('Executing excavation goal...')

        feedback_msg = Excavate.Feedback()
        result = Excavate.Result()

        target_depth = goal_handle.request.target_depth_cm
        duration = goal_handle.request.duration_sec if goal_handle.request.duration_sec > 0 else 5.0
        steps = int(duration * 10)  # 10 Hz feedback loop

        simulated_weight = 0.0
        simulated_depth = 0.0

        for i in range(1, steps + 1):
            # Check if the client requested cancellation
            if goal_handle.is_cancel_requested:
                goal_handle.canceled()
                self.get_logger().info('Excavation goal cancelled successfully.')
                result.success = False
                result.final_weight_kg = simulated_weight
                result.message = 'Excavation cancelled by client.'
                return result

            # Simulate actuator movement and weight gain
            simulated_depth = (i / steps) * target_depth
            simulated_weight += 0.2  # gaining 200g per step

            # Publish live feedback
            feedback_msg.current_depth_cm = simulated_depth
            feedback_msg.current_weight_kg = simulated_weight
            feedback_msg.percent_complete = (i / steps) * 100.0
            goal_handle.publish_feedback(feedback_msg)

            self.get_logger().info(
                f'Feedback: Depth={simulated_depth:.1f}cm, Weight={simulated_weight:.2f}kg, Progress={feedback_msg.percent_complete:.0f}%'
            )
            time.sleep(0.1)

        # Mark goal as succeeded and return result
        goal_handle.succeed()
        result.success = True
        result.final_weight_kg = simulated_weight
        result.message = 'Excavation completed successfully.'
        self.get_logger().info('Excavation goal reached [SUCCEEDED].')
        return result


def main(args=None):
    rclpy.init(args=args)
    server = ExcavatorActionServer()
    executor = MultiThreadedExecutor()
    try:
        rclpy.spin(server, executor=executor)
    except KeyboardInterrupt:
        pass
    finally:
        server.destroy_node()
        rclpy.shutdown()

if __name__ == '__main__':
    main()
```

---

## 4. Official ROS 2 Tutorial: Writing an Action Client in Python

An Action Client sends goal requests, receives continuous feedback via callbacks, and processes the final result asynchronously.

### a. Complete Action Client Implementation (`action_client_example.py`)

```python
#!/usr/bin/env python3
import rclpy
from rclpy.action import ActionClient
from rclpy.node import Node

from cometbot_msgs.action import Excavate

class ExcavatorActionClient(Node):
    def __init__(self):
        super().__init__('excavator_action_client')
        self._action_client = ActionClient(self, Excavate, 'excavate')

    def send_goal(self, target_depth_cm: float, duration_sec: float):
        """Waits for server, prepares goal, and registers callbacks."""
        self.get_logger().info('Waiting for excavation action server...')
        self._action_client.wait_for_server()

        goal_msg = Excavate.Goal()
        goal_msg.target_depth_cm = target_depth_cm
        goal_msg.duration_sec = duration_sec

        self.get_logger().info(f'Sending goal: Target Depth = {target_depth_cm} cm, Duration = {duration_sec} s')

        # Send goal asynchronously and register feedback callback
        self._send_goal_future = self._action_client.send_goal_async(
            goal_msg, 
            feedback_callback=self.feedback_callback
        )
        self._send_goal_future.add_done_callback(self.goal_response_callback)

    def goal_response_callback(self, future):
        """Handles server response (accepted or rejected)."""
        goal_handle = future.result()
        if not goal_handle.accepted:
            self.get_logger().error('Goal was rejected by server.')
            return

        self.get_logger().info('Goal accepted by server. Waiting for result...')
        self._get_result_future = goal_handle.get_result_async()
        self._get_result_future.add_done_callback(self.get_result_callback)

    def feedback_callback(self, feedback_msg):
        """Processes continuous feedback streamed from the server."""
        feedback = feedback_msg.feedback
        self.get_logger().info(
            f'[CLIENT FEEDBACK] Progress: {feedback.percent_complete:.1f}% | Depth: {feedback.current_depth_cm:.1f} cm | Weight: {feedback.current_weight_kg:.2f} kg'
        )

    def get_result_callback(self, future):
        """Processes final result upon task completion."""
        result = future.result().result
        status = future.result().status
        self.get_logger().info(
            f'[RESULT] Status Code: {status} | Success: {result.success} | Total Weight: {result.final_weight_kg:.2f} kg | Msg: {result.message}'
        )
        rclpy.shutdown()


def main(args=None):
    rclpy.init(args=args)
    client = ExcavatorActionClient()
    client.send_goal(target_depth_cm=20.0, duration_sec=4.0)
    rclpy.spin(client)

if __name__ == '__main__':
    main()
```

---

## 5. CLI Action Introspection & Manual Testing

1. **List all available actions:**
   ```bash
   ros2 action list
   ```
2. **Inspect an action's type, clients, and servers:**
   ```bash
   ros2 action info /excavate
   ```
3. **Show the action interface specification:**
   ```bash
   ros2 interface show cometbot_msgs/action/Excavate
   ```
4. **Manually send an action goal with live feedback stream:**
   ```bash
   ros2 action send_goal /excavate cometbot_msgs/action/Excavate "{target_depth_cm: 15.0, duration_sec: 5.0}" --feedback
   ```

---

## 6. Hands-On Practice Scenario (Run inside Dev Container)

1. Open your Dev Container terminal in `/workspace/cometbot_ws`.
2. Build the workspace to compile all message and action IDL bindings:
   ```bash
   colcon build --symlink-install
   source install/setup.bash
   ```
3. In Terminal 1, launch the Excavator Action Server:
   ```bash
   ros2 run cometbot_control excavator_action_server
   ```
4. In Terminal 2, trigger an action goal using the ROS 2 CLI:
   ```bash
   ros2 action send_goal /excavate cometbot_msgs/action/Excavate "{target_depth_cm: 25.0, duration_sec: 5.0}" --feedback
   ```
5. In Terminal 2, test the Python Action Client:
   ```bash
   ros2 run cometbot_control action_client_example
   ```

---

## 7. Expected Outputs & Instructor Verification Checklist

### a. Expected Terminal Outputs

#### Check 1: Action Goal with Feedback (`ros2 action send_goal /excavate ... --feedback`)
```text
Waiting for an action server to become available...
Sending goal:
     target_depth_cm: 25.0
     duration_sec: 5.0

Goal accepted with ID: <hash>

Feedback:
    current_depth_cm: 2.5
    current_weight_kg: 0.2
    percent_complete: 10.0

Feedback:
    current_depth_cm: 12.5
    current_weight_kg: 1.0
    percent_complete: 50.0

Feedback:
    current_depth_cm: 25.0
    current_weight_kg: 2.0
    percent_complete: 100.0

Result:
    success: true
    final_weight_kg: 2.0
    message: Excavation completed successfully.

Goal finished with status: SUCCEEDED
```

### b. Completion Checklist for Instructors & Students
- [ ] Explained the differences between Topics, Services, and Actions.
- [ ] Understood `.action` interface structure (Goal, Result, Feedback).
- [ ] Implemented and launched an Action Server in Python (`ActionServer`, `execute_callback`, `publish_feedback`).
- [ ] Implemented and launched an Action Client in Python (`ActionClient`, `send_goal_async`, `feedback_callback`).
- [ ] Successfully interacted with actions using ROS 2 CLI tools (`ros2 action list`, `ros2 action send_goal --feedback`).
