# bno055_hardware_interface

A ROS 2 hardware interface plugin for exposing data from a Bosch BNO055 IMU over I2C to the `ros2_control` framework. The package wraps the [`imu_bno055` driver](https://github.com/dheera/ros-imu-bno055) and exports orientation, angular velocity, and linear acceleration state interfaces for use in controllers or fused localization nodes.

## Features
- Implements `hardware_interface::SensorInterface` to publish IMU state interfaces (orientation quaternion, angular velocity, linear acceleration).
- Uses the `imu_bno055::BNO055I2CDriver` to communicate with the sensor over I2C.
- Packaged as a `pluginlib` plugin for loading through `ros2_control` hardware abstraction.

## Package layout
- `include/bno055_hardware_interface/bno055_hardware.hpp` – public hardware interface definition and configuration parameters (I2C device path, address, frame ID).
- `src/bno055_hardware.cpp` – lifecycle hooks, state interface export, and read loop that fetches data from the driver and populates the state interfaces.
- `bno055_hardware_interface.xml` – plugin registration for `pluginlib`/`ros2_control`.
- `CMakeLists.txt` and `package.xml` – ROS 2 build configuration and dependencies.

## Dependencies
- ROS 2 (tested with the build settings in the repository) with `ros2_control` packages providing `hardware_interface` and `pluginlib`.
- `imu_bno055` driver library (pulled in via `find_package`).
- I2C development headers (`libi2c-dev`).

## Building
1. Clone the repository into a ROS 2 workspace `src` directory alongside the `imu_bno055` driver.
2. Source your ROS 2 installation (`source /opt/ros/<distro>/setup.bash`).
3. Build with `colcon`:
   ```bash
   colcon build --packages-select bno055_hardware_interface
   ```
4. Source the workspace (`. install/setup.bash`).

## Configuring with ros2_control
Add the hardware interface to your `ros2_control` configuration (URDF/xacro or YAML). The plugin class name is `bno055_hardware_interface/BNO055HardwareInterface` as registered in `bno055_hardware_interface.xml`.

Example URDF snippet:
```xml
<ros2_control name="bno055_imu" type="system">
  <hardware>
    <plugin>bno055_hardware_interface/BNO055HardwareInterface</plugin>
    <param name="device">/dev/i2c-1</param>
    <param name="address">40</param>
    <param name="frame_id">imu_link</param>
  </hardware>
  <sensor name="imu" type="imu_sensor">
    <state_interface name="orientation.x" />
    <state_interface name="orientation.y" />
    <state_interface name="orientation.z" />
    <state_interface name="orientation.w" />
    <state_interface name="angular_velocity.x" />
    <state_interface name="angular_velocity.y" />
    <state_interface name="angular_velocity.z" />
    <state_interface name="linear_acceleration.x" />
    <state_interface name="linear_acceleration.y" />
    <state_interface name="linear_acceleration.z" />
  </sensor>
</ros2_control>
```

### Hardware parameters
The interface reads configuration from hardware parameters defined in the `ros2_control` hardware block:
- `device`: I2C device path (default: `/dev/i2c-1`).
- `address`: I2C address as an integer (default: `40`, i.e., `0x28`).
- `frame_id`: Frame name to associate with IMU data (default: `imu_link`).

## Runtime behavior
- On activation, the interface initializes the I2C driver, readying the IMU for reads.
- Each call to `read()` polls the sensor, logging a warning if an exception is thrown, and updates orientation, linear acceleration (scaled from centi-m/s² to m/s² by dividing by 100), and angular velocity (scaled from raw units by dividing by 900).
- `export_state_interfaces()` exposes the nine IMU state channels expected by `ros2_control` sensors.

## Usage notes
- Ensure the BNO055 is wired for I2C and visible at the configured address (commonly `0x28` or `0x29`).
- The interface is read-only; no command interfaces are exported.
- For downstream consumers (e.g., `robot_localization`), use the state interfaces to publish a `sensor_msgs/Imu` message within your controller or a custom node.

## License and maintenance
Update the `package.xml` license and maintainer fields as appropriate for your distribution.
