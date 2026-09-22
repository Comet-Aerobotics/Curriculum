# Simulation Module 1: ROS 2 Launch Files, Mock Sensors & Mission Simulation

> **Target:** Master authoring Python ROS 2 launch files (`launch.py`), creating mock sensor publishers, simulating subsystem physics and timing, and testing end-to-end autonomous action workflows without physical robot hardware.  
> **Key References:** [ROS 2 Launch Tutorial](https://docs.ros.org/en/humble/Tutorials/Intermediate/Launch/Launch-Main.html) | [cometbot_control launch files](https://github.com/Comet-Aerobotics/CAN)

---

## 1. Why Simulation in Robotics?

In competition and production robotics, physical hardware access is often limited or dangerous. Writing simulation nodes and launch files enables:
1. **Parallel Development:** Software and autonomous state machines can be developed and validated before mechanical manufacturing is finished.
2. **Deterministic Automated Testing:** CI/CD pipelines can run automated unit and integration tests against synthetic sensor streams.
3. **Edge-Case Stress Testing:** Testing fault handling, safety timeouts, and action cancellations safely.

---

## 2. Writing Python Launch Files in ROS 2

ROS 2 launch files are written in Python, allowing dynamic logic, argument parsing, namespace configuration, and parameter assignment.

### a. Launch File Structure (`mission_sim.launch.py`)

```python
from launch import LaunchDescription
from launch_ros.actions import Node
from launch.actions import DeclareLaunchArgument
from launch.substitutions import LaunchConfiguration

def generate_launch_description():
    # 1. Declare Launch Arguments
    use_sim_time_arg = DeclareLaunchArgument(
        'use_sim_time',
        default_value='false',
        description='Use simulation (Gazebo/synthetic) clock if true'
    )

    # 2. Define Action Server Nodes
    excavator_server = Node(
        package='cometbot_control',
        executable='excavator_action_server',
        name='excavator_action_server',
        output='screen',
        parameters=[{'use_sim_time': LaunchConfiguration('use_sim_time')}]
    )

    depositor_server = Node(
        package='cometbot_control',
        executable='depositor_action_server',
        name='depositor_action_server',
        output='screen',
        parameters=[{'use_sim_time': LaunchConfiguration('use_sim_time')}]
    )

    # 3. Define Mock Sensor Node (Simulated Load Cell)
    load_sensor_sim = Node(
        package='cometbot_control',
        executable='load_sensor_sim',
        name='load_sensor_sim',
        output='screen'
    )

    # 4. Return LaunchDescription containing all nodes
    return LaunchDescription([
        use_sim_time_arg,
        excavator_server,
        depositor_server,
        load_sensor_sim
    ])
```

---

## 3. Mock Sensor Simulation (Simulated Load Cell)

A mock sensor node generates synthetic telemetry messages that mirror the physical sensor's message type and update frequency:

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import Float32

class MockLoadCell(Node):
    def __init__(self):
        super().__init__('mock_load_cell')
        self.publisher = self.create_publisher(Float32, '/load_sensor/weight', 10)
        self.timer = self.create_timer(0.1, self.timer_callback)  # 10 Hz
        self.current_weight = 0.0

    def timer_callback(self):
        msg = Float32()
        msg.data = self.current_weight
        self.publisher.publish(msg)

def main(args=None):
    rclpy.init(args=args)
    node = MockLoadCell()
    rclpy.spin(node)
```

---

## 4. Running and Inspecting the Simulation

1. Build and source your workspace:
   ```bash
   colcon build --symlink-install
   source install/setup.bash
   ```
2. Launch the full mission simulation:
   ```bash
   ros2 launch cometbot_control mission_sim.launch.py
   ```
3. Inspect active nodes in a second terminal:
   ```bash
   ros2 node list
   ros2 action list
   ros2 topic list
   ```

---

## 5. Verification & Checklist
- [ ] Created and executed a multi-node ROS 2 Python launch file.
- [ ] Verified simultaneous startup of Action Servers and simulated sensor streams.
- [ ] Confirmed node introspection via `ros2 node list` and `ros2 action list`.
