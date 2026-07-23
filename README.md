# Task 5 — Robot Description: TF & Gazebo Simulation

https://github.com/user-attachments/assets/e400e55c-3601-4718-88c4-3aaaa49804c9
## 1. Project Overview

This project continues from the previous URDF/Xacro assignment (Task 4). It builds on the existing `robot_description` package by adding everything needed to:

- Publish the robot model with `robot_state_publisher` and visualize joint states with `joint_state_publisher_gui`.
- Verify and visualize a fully connected TF tree in RViz2.
- Simulate the robot in Gazebo using plugins for differential drive, odometry, LiDAR, and an RGB camera.
- Drive the robot using `/cmd_vel` (keyboard, Gazebo teleop panel, or terminal).
- Visualize LiDAR and camera data in RViz2 while the robot runs in Gazebo.

The robot is a two-wheeled differential-drive robot (`two_wheel_pyramid_robot`) equipped with a LiDAR sensor and an RGB camera, spawned into the TurtleBot3 house world.

---

## 2. Package Structure

```
task4-robot_description/
└── robot_description/
    ├── urdf/
    │   ├── robot.urdf.xacro
    │   └── robot.gazebo.xacro
    ├── meshes/
    │   ├── lidar.STL
    │   └── zed.stl
    ├── config/
    │   └── gz_bridge.yaml
    ├── launch/
    │   ├── display.launch.py
    │   └── gazebo.launch.py
    ├── rviz/
    │   └── robot_view.rviz
    ├── package.xml
    ├── CMakeLists.txt
    └── README.md
```

> Note: `build/`, `install/`, and `log/` directories are **not** included in this repository.

---

## 3. Linux Commands Used

```bash
cd ../..
source ~/.bashrc

git init
git remote add origin https://github.com/h-git-m/task5-robot_desccription.git
git add .
git commit -m "Add ROS2 package: robot_description"
git branch -M main
git push -u origin main
```

---

## 4. ROS 2 / Gazebo Commands Used

**Build and source the package:**
```bash
colcon build --packages-select robot_description
source install/setup.bash
```

**Launch the robot model for TF verification (no Gazebo):**
```bash
ros2 launch robot_description display.launch.py
```

**Launch the robot in Gazebo:**
```bash
ros2 launch robot_description gazebo.launch.py
```

**Inspect available Gazebo topics:**
```bash
gz topic -l
```
Expected output includes:
```
/camera/camera_info
/camera/image_raw
/clock
/cmd_vel
/gazebo/resource_paths
/model/two_wheel_pyramid_robot/odometry
/model/two_wheel_pyramid_robot/odometry_with_covariance
/model/two_wheel_pyramid_robot/tf
/odom
/scan
/scan/points
/sensors/marker
/stats
/tf
/world/default/clock
/world/default/dynamic_pose/info
/world/default/model/two_wheel_pyramid_robot/joint_state
/world/default/pose/info
/world/default/scene/deletion
/world/default/scene/info
/world/default/state
/world/default/stats
/model/two_wheel_pyramid_robot/enable
/world/default/light_config
/world/default/material_color
```

**Move the robot from the terminal:**
```bash
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.2}, angular: {z: 0.0}}"
```

**Inspect the TF tree:**
```bash
ros2 run tf2_ros tf2_echo base_link left_wheel_link
ros2 run tf2_tools view_frames
ros2 topic echo /tf
```

**Example `/tf` output (odom → base_footprint):**
```
header:
  stamp:
    sec: 539
    nanosec: 660000000
  frame_id: odom
  child_frame_id: base_footprint
pose:
  pose:
    position:
      x: -1.999999830877909
      y: 0.5003031284798056
      z: 0.0
    orientation:
      x: -0.029791270833412157
      y: 0.029814987887105193
      z: 0.70619727051582
      w: 0.7067594794521174
  covariance: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, ...]
```

---

## 5. How to Launch RViz

1. From your workspace root, source the install space:
   ```bash
   cd ~/your_workspace
   source install/setup.bash
   ```
2. Launch RViz2:
   ```bash
   rviz2
   ```
3. Inside the RViz menu bar, select **File → Open Config**, then locate and open `robot_view.rviz` (inside `robot_description/rviz/`).

This preconfigured view will let you visualize:
- The robot model
- The TF tree
- LiDAR readings
- Camera readings

**If you don't use the saved config**, add each display manually and set the topics as follows, and set the **Fixed Frame** to `odom`:

| Display     | Topic                  |
|-------------|-------------------------|
| RobotModel  | `/robot_description`    |
| TF          | (auto, shows full tree) |
| LaserScan   | `/scan`                 |
| Image       | `/camera/image_raw`     |

Alternatively, launch the model-only view directly with:
```bash
ros2 launch robot_description display.launch.py
```

---

## 6. How to Launch Gazebo

After creating the Gazebo launch file (`gazebo.launch.py`) and its related files — the bridge config (`gz_bridge.yaml`), the xacro description (`robot.urdf.xacro`), and the Gazebo plugins (`robot.gazebo.xacro`) — build and source the package:

```bash
colcon build --packages-select robot_description
source install/setup.bash
```

Then run:
```bash
ros2 launch robot_description gazebo.launch.py
```

This launch file will:
- Start Gazebo (server, with the correct `GZ_SIM_RESOURCE_PATH` set for meshes/models).
- Load the TurtleBot3 house world.
- Start `robot_state_publisher` with `use_sim_time: True`.
- Create the ROS–Gazebo bridge (`ros_gz_bridge`) using `gz_bridge.yaml`.
- Spawn the robot into the simulation from the `/robot_description` topic.

---

## 7. Expected Topics

Once Gazebo and the bridge are running, the following ROS 2 topics should be available:

| Topic                  | Type                              | Direction |
|-------------------------|-----------------------------------|-----------|
| `/clock`                | `rosgraph_msgs/msg/Clock`         | GZ → ROS  |
| `/cmd_vel`               | `geometry_msgs/msg/Twist`         | ROS → GZ  |
| `/odom`                  | `nav_msgs/msg/Odometry`           | GZ → ROS  |
| `/tf`                    | `tf2_msgs/msg/TFMessage`          | GZ → ROS  |
| `/joint_states`          | `sensor_msgs/msg/JointState`      | GZ → ROS  |
| `/scan`                  | `sensor_msgs/msg/LaserScan`       | GZ → ROS  |
| `/camera/image_raw`      | `sensor_msgs/msg/Image`           | GZ → ROS  |
| `/camera/camera_info`    | `sensor_msgs/msg/CameraInfo`      | GZ → ROS  |

Verify with:
```bash
ros2 topic list
ros2 topic echo /odom
ros2 topic echo /scan
```

---

## 8. How to Move the Robot

The robot can be driven in three ways:

**1. Keyboard teleoperation:**
```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```
Uses standard WASD-style keys to publish `/cmd_vel`.

**2. Gazebo's built-in Teleop panel** — open the panel from the Gazebo GUI and use the on-screen controls to send velocity commands directly.

**3. Terminal command:**
```bash
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.2}, angular: {z: 0.0}}"
```
- `linear.x` controls forward/backward speed (positive = forward, negative = backward).
- `angular.z` controls rotation (positive = turn left/counter-clockwise, negative = turn right/clockwise).
- Values are capped by the DiffDrive plugin limits (`max_linear_velocity`, `max_angular_velocity`) defined in `robot.gazebo.xacro`.

---

## 9. TF Tree Explanation

The TF tree describes how all coordinate frames of the robot relate to one another, from the world down to each sensor and wheel.

```
odom
 └── base_footprint
      └── base_link
            └── middle_step_link
                └── camera_link
                  └── camera_optical_link
                └──Top_step_link
                    └──lidar_link
            ├──left_wheel_link
            └──right_wheel_link
              
             
```

- **`odom`** — the fixed world-referenced frame published by the Gazebo `OdometryPublisher` / `DiffDrive` plugins. Represents the robot's estimated position over time.
- **`base_footprint`** — the robot's 2D ground-projected reference frame; the `robot_base_frame` used by the DiffDrive and Odometry plugins.
- **`base_link`** — the main body frame of the robot, fixed to `base_footprint` via a static joint.
- **`left_wheel_link` / `right_wheel_link`** — connected to `base_link` via continuous joints; their transforms are updated live from `/joint_states`, which is bridged from Gazebo's `JointStatePublisher` plugin.
- **`lidar_link`** — fixed joint from `Top_step_link`; used as the LiDAR sensor's frame (`gz_frame_id`) for `/scan` data.
- **`camera_link` / `camera_optical_link`** — fixed joints from `middle_step_link`; used by the RGB camera plugin for `/camera/image_raw` and `/camera/camera_info`.

A fully connected tree (no missing/disconnected frames) is required for RViz2 to correctly transform sensor data (LiDAR, camera) and the robot model into the `odom` frame. This was verified using:
```bash
ros2 run tf2_tools view_frames
ros2 run tf2_ros tf2_echo base_link left_wheel_link
```

---

## 10. Screenshots

- Robot in RViz: `![Robot in RViz](screenshots/robot_lidar_rviz.png)`
- TF tree: `![TF Tree](screenshots/robot_lidar_TF_tree_rviz_gazebo.png)`
- Robot in Gazebo: `![Robot in Gazebo](screenshots/robot_gazebo.png)`
- LiDAR visualization and Camera visualization: `![LiDAR](screenshots/robot_lidar_camera_rviz_gazebo.png)`
