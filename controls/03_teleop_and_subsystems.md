# Controls Module 3: Teleoperation, Joystick Mapping & Actuator Control

> **Target:** Master manual and semi-autonomous robot teleoperation, joystick axes and button mapping, mathematical deadband and power curve filtering, and multi-publisher actuator control.  
> **Key References:** [ROS 2 joy package documentation](https://index.ros.org/p/joy/) | [geometry_msgs/msg/Twist](https://docs.ros2.org/latest/api/geometry_msgs/msg/Twist.html)

---

## 1. Teleoperation Architecture

The teleoperation subsystem translates human input from a gamepad or joystick into velocity setpoints (`geometry_msgs/msg/Twist`) and subsystem actuator commands (`Float32`).

```
+----------------+      sensor_msgs/Joy      +--------------------+      geometry_msgs/Twist      +------------------+
| Gamepad / Joy  | ------------------------> |  teleop_publisher  | ----------------------------> | /cmd_vel (Drive) |
| (Xbox/Logitech)|       /joy topic          |       (Node)       | ----------------------------> | /depositor/...   |
+----------------+                           +--------------------+      std_msgs/Float32         | /excavator/...   |
                                                                                                  +------------------+
```

---

## 2. Joystick Signal Conditioning & Mathematics

Raw joystick potentiometers often produce slight hardware drift around zero (center position) and linear scaling can feel overly sensitive at low speeds.

### a. Deadband Filtering
To prevent the robot from creeping when the thumbstick is untouched:
$$\text{input\_magnitude} = \sqrt{v_x^2 + \omega_z^2}$$
If $\text{input\_magnitude} < \text{deadzone}$, set output to $0.0$.

### b. Exponential Power Curve Scaling
Applying a polynomial or exponential curve provides fine-grained precision at low speeds while retaining full maximum speed at full stick deflection:
$$v_{\text{scaled}} = \text{sign}(v) \cdot |v|^{\text{power}}$$
*(Where $\text{power} = 3$ gives smooth cubic response).*

---

## 3. Subsystem Actuator Clamping & State Tracking

For excavator arms and depositor dump mechanisms, discrete button presses (e.g. A/B buttons, D-Pad, or Bumpers) adjust setpoints in fixed steps, bounded by safety limits:

```python
def _clamp(self, value: float, minimum: float, maximum: float) -> float:
    return max(min(value, maximum), minimum)
```

---

## 4. Hands-On Verification & Checklist

1. Launch `ros2 run joy joy_node`.
2. Launch `ros2 run cometbot_control teleop_publisher`.
3. In a separate terminal, echo the output topics:
   ```bash
   ros2 topic echo /cmd_vel
   ros2 topic echo /depositor/position_setpoint
   ros2 topic echo /excavator/velocity_setpoint
   ```
4. Move the joystick axes and press buttons to verify smooth scaling and clamped bounds.

### Completion Checklist
- [ ] Configured ROS 2 `joy_node` for Linux gamepad input (`/dev/input/js0`).
- [ ] Implemented deadzone thresholding and cubic power curve scaling.
- [ ] Verified multi-topic publishing across `/cmd_vel`, `/depositor/*`, and `/excavator/*`.
