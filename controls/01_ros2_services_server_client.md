# Controls Module 1: ROS 2 Services — Writing a Service Server & Client in Python

> **Target:** Master 1-to-1 synchronous/asynchronous Request-Response communication in ROS 2. Understand service interfaces (`.srv`), write a Service Server and Client in Python following official ROS 2 Humble specifications, and implement real-world robotics service triggers (e.g. `/zero_load_cell` and `/trigger_calibration`).  
> **Key References:** [ROS 2 Humble Service Tutorial (Python)](https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Service-And-Client.html) | [ROS 2 Services Concepts](https://docs.ros.org/en/humble/Concepts/About-ROS-2-Services.html)

---

## 1. What is a ROS 2 Service?

While **Topics** provide continuous one-way data streams (e.g., publishing camera images or drive velocity at 20 Hz), a **Service** provides a **1-to-1 Request/Response** communication pattern.

### a. When to Use a Service vs. a Topic vs. an Action
* **Topic (Stream):** High-frequency, continuous data where individual message loss is acceptable (e.g., `/cmd_vel`, `/imu/data`).
* **Service (Instantaneous RPC):** A discrete request that finishes almost immediately (< 1 second) and requires confirmation of completion or a return value (e.g., `/zero_load_cell`, `/reset_odometry`, `/set_camera_exposure`).
* **Action (Long-running Behavior):** Tasks that take seconds or minutes to complete, require continuous progress feedback, and may need to be cancelled mid-operation (e.g., `/excavate`, `/deposit`, `/navigate_to_pose`).

```
+-------------------+      Request Message (e.g., Trigger.Request)      +-------------------+
|  Service Client   | ------------------------------------------------> |  Service Server   |
|      (Node)       | <------------------------------------------------ |      (Node)       |
+-------------------+      Response Message (e.g., Trigger.Response)    +-------------------+
```

---

## 2. Defining a Service Interface (`.srv` File)

A service interface consists of a **Request** definition and a **Response** definition separated by `---`:

```text
# Request (Sent from Client to Server)
int64 a
int64 b
---
# Response (Returned from Server to Client)
int64 sum
```

Common standard services available out of the box in `std_srvs`:
* `std_srvs/srv/Trigger`: Empty request $\rightarrow$ `bool success`, `string message`.
* `std_srvs/srv/SetBool`: `bool data` $\rightarrow$ `bool success`, `string message`.

---

## 3. Official ROS 2 Tutorial: Writing a Python Service Server

Following the [official ROS 2 Humble Service Server Tutorial](https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Service-And-Client.html), here is how to create a Service Server node.

### a. Example: Subsystem Sensor Calibration Server (`calibration_service_server.py`)

```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from std_srvs.srv import Trigger

class CalibrationServiceServer(Node):
    def __init__(self):
        super().__init__('calibration_service_server')

        # Create the service: (ServiceType, 'service_name', callback_function)
        self.srv = self.create_service(
            Trigger, 
            'zero_load_cell', 
            self.zero_load_cell_callback
        )
        self.get_logger().info('Load Cell Zero/Calibration Service Server is ready.')

    def zero_load_cell_callback(self, request, response):
        """Processes the request and returns the response."""
        self.get_logger().info('Received request to tare/zero the load cell sensor.')

        # Perform tare/zero calibration logic
        try:
            # Simulated hardware tare routine
            self.get_logger().info('Applying tare offset...')
            response.success = True
            response.message = 'Load cell tare offset successfully reset to 0.0 kg.'
        except Exception as e:
            response.success = False
            response.message = f'Calibration failed: {str(e)}'

        self.get_logger().info(f'Sending response: success={response.success}, msg="{response.message}"')
        return response


def main(args=None):
    rclpy.init(args=args)
    node = CalibrationServiceServer()
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

---

## 4. Official ROS 2 Tutorial: Writing a Python Service Client

A Service Client sends requests asynchronously using `call_async()` to avoid blocking the ROS 2 executor event loop.

### a. Example: Asynchronous Service Client (`calibration_service_client.py`)

```python
#!/usr/bin/env python3
import sys
import rclpy
from rclpy.node import Node
from std_srvs.srv import Trigger

class CalibrationServiceClient(Node):
    def __init__(self):
        super().__init__('calibration_service_client')
        
        # Create client for the target service
        self.client = self.create_client(Trigger, 'zero_load_cell')

        # Wait until the service server is available
        while not self.client.wait_for_service(timeout_sec=1.0):
            self.get_logger().info('Waiting for "zero_load_cell" service server...')

    def send_request(self):
        """Prepares request and calls service asynchronously."""
        request = Trigger.Request()
        self.get_logger().info('Sending tare request to service server...')
        
        # call_async returns a Future object
        self.future = self.client.call_async(request)
        return self.future


def main(args=None):
    rclpy.init(args=args)
    client_node = CalibrationServiceClient()
    future = client_node.send_request()

    # Spin until the future completes
    rclpy.spin_until_future_complete(client_node, future)

    response = future.result()
    if response is not None:
        client_node.get_logger().info(
            f'Service Call Response -> Success: {response.success} | Message: "{response.message}"'
        )
    else:
        client_node.get_logger().error(f'Service call failed: {client_node.future.exception()}')

    client_node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

---

## 5. CLI Service Introspection & Manual Testing

ROS 2 provides built-in command-line tools to test services without writing custom client nodes:

1. **List all active services in the graph:**
   ```bash
   ros2 service list
   ```
2. **Find the message type of a service:**
   ```bash
   ros2 service type /zero_load_cell
   ```
3. **Inspect the request and response structure:**
   ```bash
   ros2 interface show std_srvs/srv/Trigger
   ```
4. **Call a service directly from the command line:**
   ```bash
   ros2 service call /zero_load_cell std_srvs/srv/Trigger "{}"
   ```

---

## 6. Hands-On Practice Scenario (Run inside Dev Container)

1. Open your Dev Container terminal in `/workspace/cometbot_ws`.
2. Ensure workspace dependencies are sourced:
   ```bash
   source /opt/ros/humble/setup.bash
   ```
3. In Terminal 1, run the Python service server:
   ```bash
   python3 /workspace/cometbot_ws/src/cometbot_control/cometbot_control/calibration_service_server.py
   ```
4. In Terminal 2, inspect the active service:
   ```bash
   ros2 service list | grep zero_load_cell
   ros2 service type /zero_load_cell
   ```
5. In Terminal 2, trigger the service via CLI:
   ```bash
   ros2 service call /zero_load_cell std_srvs/srv/Trigger "{}"
   ```
6. In Terminal 2, run the Python client script:
   ```bash
   python3 /workspace/cometbot_ws/src/cometbot_control/cometbot_control/calibration_service_client.py
   ```

---

## 7. Expected Outputs & Instructor Verification Checklist

### a. Expected Terminal Outputs

#### Check 1: CLI Service Call (`ros2 service call /zero_load_cell ...`)
```text
requester: making request: std_srvs.srv.Trigger_Request()

response:
std_srvs.srv.Trigger_Response(success=True, message='Load cell tare offset successfully reset to 0.0 kg.')
```

#### Check 2: Python Service Client Output
```text
[INFO] [calibration_service_client]: Waiting for "zero_load_cell" service server...
[INFO] [calibration_service_client]: Sending tare request to service server...
[INFO] [calibration_service_client]: Service Call Response -> Success: True | Message: "Load cell tare offset successfully reset to 0.0 kg."
```

### b. Completion Checklist for Instructors & Students
- [ ] Explained the differences between Topics, Services, and Actions.
- [ ] Understood `.srv` interface structure (Request vs. Response separated by `---`).
- [ ] Implemented a ROS 2 Python Service Server (`create_service`, callback, returning response).
- [ ] Implemented a ROS 2 Python Service Client (`create_client`, `wait_for_service`, `call_async`).
- [ ] Successfully tested services using `ros2 service list`, `ros2 service type`, and `ros2 service call`.
