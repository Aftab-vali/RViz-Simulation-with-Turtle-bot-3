# 🤖 ROS 2 TurtleBot3 Indoor Simulation

A complete **ROS 2 Humble indoor mobile-robot simulation** using **TurtleBot3, Gazebo, SLAM, and RViz2**.

This project provides a ready-to-use environment for learning and experimenting with:

* ROS 2
* TurtleBot3
* Gazebo simulation
* LiDAR sensing
* Odometry
* TF transforms
* SLAM
* RViz2 visualization
* Robot teleoperation
* Autonomous navigation

The final setup allows you to see a simulated TurtleBot3 moving inside an indoor environment while its **LiDAR data, robot pose, map, and navigation information are visualized live in RViz2**.

---

## 🧠 System Architecture

```text
                    ┌───────────────────────────────┐
                    │           Gazebo              │
                    │                               │
                    │   🧱 Walls       🧱           │
                    │                               │
                    │          🤖                   │
                    │       TurtleBot3              │
                    │          ↻ LiDAR              │
                    │                               │
                    │   🧱              🧱           │
                    └──────────────┬────────────────┘
                                   │
                    /scan /odom /tf /cmd_vel
                                   │
                                   ▼
                         ┌─────────────────┐
                         │      ROS 2      │
                         │                 │
                         │   SLAM Toolbox  │
                         │      + Nav2     │
                         └────────┬────────┘
                                  │
                                  ▼
                           ┌─────────────┐
                           │    RViz2    │
                           │             │
                           │ 🗺 Map      │
                           │ 🔴 LiDAR    │
                           │ 🤖 Robot    │
                           │ 🟢 Path     │
                           │ TF Frames   │
                           └─────────────┘
```

---

# 📋 Requirements

## Operating System

This project is configured for:

```text
Ubuntu 22.04 LTS
```

If using Windows, **WSL2 + Ubuntu 22.04** can be used.

## ROS 2

```text
ROS 2 Humble
```

Check your installation:

```bash
echo $ROS_DISTRO
```

Expected:

```text
humble
```

Check Ubuntu:

```bash
lsb_release -a
```

Expected:

```text
Ubuntu 22.04
```

---

# 📁 Workspace Structure

The recommended ROS 2 workspace is:

```text
~/ros2_ws/
│
├── src/
│   ├── turtlebot3/
│   ├── turtlebot3_msgs/
│   └── turtlebot3_simulations/
│
├── build/
├── install/
└── log/
```

---

# 🚀 Installation

## 1. Create the ROS 2 workspace

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
```

---

# 2. Clone TurtleBot3 Packages

> **Important:** Because this project uses ROS 2 Humble, use the `humble` branches.

```bash
git clone -b humble https://github.com/ROBOTIS-GIT/turtlebot3.git

git clone -b humble https://github.com/ROBOTIS-GIT/turtlebot3_msgs.git

git clone -b humble https://github.com/ROBOTIS-GIT/turtlebot3_simulations.git
```

Verify the branch:

```bash
cd ~/ros2_ws/src/turtlebot3
git branch --show-current
```

Expected:

```text
humble
```

---

# 3. Source ROS 2 Humble

```bash
source /opt/ros/humble/setup.bash
```

You can verify:

```bash
echo $ROS_DISTRO
```

Expected:

```text
humble
```

---

# 4. Install Dependencies

Return to the workspace:

```bash
cd ~/ros2_ws
```

Run:

```bash
rosdep install --from-paths src --ignore-src -r -y
```

This automatically checks the packages in the workspace and installs required ROS dependencies.

---

# 5. Build the Workspace

Build everything:

```bash
colcon build --symlink-install
```

If the build succeeds, you should see something similar to:

```text
Summary: ...
packages finished
```

There should be **no failed packages**.

---

# 6. Source the Workspace

After building:

```bash
source ~/ros2_ws/install/setup.bash
```

For convenience, you can add it to your `.bashrc`:

```bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
echo "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc
```

Then reload:

```bash
source ~/.bashrc
```

---

# 🤖 TurtleBot3 Model

For this project we use the TurtleBot3 Burger.

Set the model:

```bash
export TURTLEBOT3_MODEL=burger
```

Verify:

```bash
echo $TURTLEBOT3_MODEL
```

Expected:

```text
burger
```

You can also make this permanent:

```bash
echo "export TURTLEBOT3_MODEL=burger" >> ~/.bashrc
```

Then:

```bash
source ~/.bashrc
```

---

# 🌎 Launch Gazebo Indoor Simulation

Start the TurtleBot3 simulation:

```bash
ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
```

Gazebo should open with an indoor environment containing:

```text
             GAZEBO
┌───────────────────────────────┐
│                               │
│    █████          █████       │
│                               │
│              🤖               │
│           TurtleBot3          │
│             ↻ LiDAR           │
│                               │
│   ███████            █████    │
│                               │
└───────────────────────────────┘
```

The robot exists inside the simulated environment and publishes ROS 2 sensor and motion data.

---

# 📡 ROS 2 Topics

The simulation produces several important ROS 2 topics.

Check all topics:

```bash
ros2 topic list
```

Important topics include:

```text
/cmd_vel
/odom
/scan
/tf
/tf_static
```

---

## 🔴 LiDAR

Check the LiDAR topic:

```bash
ros2 topic echo /scan
```

The `/scan` topic contains laser range measurements from the simulated LiDAR.

You can also check its type:

```bash
ros2 topic type /scan
```

Expected:

```text
sensor_msgs/msg/LaserScan
```

---

# 📍 Odometry

Check odometry:

```bash
ros2 topic echo /odom
```

Check its type:

```bash
ros2 topic type /odom
```

Expected:

```text
nav_msgs/msg/Odometry
```

Odometry provides information about the robot's estimated movement.

---

# 🔄 TF

ROS 2 uses TF to describe relationships between coordinate frames.

Check TF:

```bash
ros2 topic list | grep tf
```

Typical frames include:

```text
map
odom
base_footprint
base_link
laser
```

The relationship can be understood as:

```text
map
 │
 ▼
odom
 │
 ▼
base_footprint
 │
 ▼
base_link
 │
 ▼
laser
```

---

# 🎮 Move the Robot

You can control the robot directly using `/cmd_vel`.

## Move Forward Once

Use:

```bash
ros2 topic pub --once \
/cmd_vel \
geometry_msgs/msg/Twist \
"{linear: {x: 0.5}, angular: {z: 0.0}}"
```

The robot will receive one velocity command.

---

## Rotate Once

```bash
ros2 topic pub --once \
/cmd_vel \
geometry_msgs/msg/Twist \
"{linear: {x: 0.0}, angular: {z: 0.5}}"
```

---

# 🧭 Understanding `/cmd_vel`

The command uses the standard ROS coordinate convention.

```text
              +Y
               ↑
               │
               │
        +X ─── 🤖 ─── -X
               │
               ↓
              -Y
```

For a mobile robot:

```text
linear.x
```

controls forward/backward motion.

```text
linear.y
```

controls sideways motion for robots that support it.

```text
linear.z
```

controls vertical movement.

For TurtleBot3, the important value is:

```text
linear.x
```

Rotation is controlled using:

```text
angular.z
```

Therefore:

```yaml
linear:
  x: 0.5
```

means move forward.

And:

```yaml
angular:
  z: 0.5
```

means rotate around the vertical axis.

---

# 🖥️ Launch RViz2

Keep Gazebo running.

Open a **second WSL terminal**.

Source ROS:

```bash
source /opt/ros/humble/setup.bash
```

Source your workspace:

```bash
source ~/ros2_ws/install/setup.bash
```

Set the robot model:

```bash
export TURTLEBOT3_MODEL=burger
```

Then launch RViz:

```bash
ros2 launch turtlebot3_bringup rviz2.launch.py
```

RViz2 should open.

---

# 👁️ RViz2 Visualization

RViz2 can visualize the live ROS 2 data coming from Gazebo.

The final visualization can contain:

```text
┌─────────────────────────────────────────┐
│                 RViz2                   │
│                                         │
│       · · · · · · · · ·                │
│     ·                    ·              │
│    ·        🟢           ·              │
│    ·         🤖          ·              │
│    ·        ↻↗           ·              │
│     ·                    ·              │
│       · · · · · · · · ·                │
│                                         │
│  Map      LiDAR      TF      Robot      │
└─────────────────────────────────────────┘
```

---

# 🗺️ SLAM

To create a map of the indoor environment while the robot moves, use **SLAM Toolbox**.

Install it if necessary:

```bash
sudo apt update
sudo apt install ros-humble-slam-toolbox
```

Then launch SLAM:

```bash
ros2 launch slam_toolbox online_async_launch.py
```

Depending on the exact TurtleBot3 configuration, you may need to provide the appropriate SLAM parameters.

The data flow is:

```text
                Gazebo
                   │
                   │
                /scan
                   │
                   ▼
             SLAM Toolbox
                   │
                   ▼
             Occupancy Map
                   │
                   ▼
                 RViz
```

As the robot moves around the environment, SLAM builds the map from the simulated LiDAR data.

---

# 🧭 Navigation

After generating a map, ROS 2 Nav2 can be used for autonomous navigation.

Install Nav2:

```bash
sudo apt install ros-humble-navigation2 ros-humble-nav2-bringup
```

The overall architecture becomes:

```text
                     Gazebo
                        │
                ┌───────┴────────┐
                │                │
              LiDAR           Odometry
                │                │
                └───────┬────────┘
                        │
                        ▼
                  SLAM Toolbox
                        │
                        ▼
                     Map
                        │
                        ▼
                      Nav2
                        │
               ┌────────┴────────┐
               │                 │
         Global Planner     Local Planner
               │                 │
               └────────┬────────┘
                        │
                        ▼
                    /cmd_vel
                        │
                        ▼
                    TurtleBot3
```

This allows the robot to navigate toward goals instead of being manually controlled.

---

# 🔬 Useful ROS 2 Commands

## List Nodes

```bash
ros2 node list
```

## List Topics

```bash
ros2 topic list
```

## Check Topic Information

```bash
ros2 topic info /scan
```

## Check Topic Frequency

```bash
ros2 topic hz /scan
```

## Check TF

```bash
ros2 run tf2_tools view_frames
```

## Inspect a Topic

```bash
ros2 topic echo /scan
```

## Check Running Nodes

```bash
ros2 node list
```

---

# 🧪 Useful Test

After Gazebo starts, check:

```bash
ros2 topic list
```

You should see topics related to:

```text
/cmd_vel
/odom
/scan
/tf
/tf_static
```

Then check LiDAR:

```bash
ros2 topic hz /scan
```

If the sensor is working, you should see messages being published continuously.

---

# 🏗️ Complete Data Flow

The complete simulation can be understood as:

```text
┌──────────────────────────────────────────┐
│                  Gazebo                  │
│                                          │
│       Indoor Environment                 │
│                                          │
│             🤖 TurtleBot3                │
│                ↻ LiDAR                   │
└───────────────────┬──────────────────────┘
                    │
                    │ ROS 2 Topics
                    │
        ┌───────────┼────────────┐
        ▼           ▼            ▼
      /scan       /odom         /tf
        │           │            │
        └───────────┼────────────┘
                    │
                    ▼
             ┌─────────────┐
             │ SLAM Toolbox│
             └──────┬──────┘
                    │
                    ▼
               🗺 Occupancy
                  Map
                    │
                    ▼
             ┌─────────────┐
             │    Nav2     │
             └──────┬──────┘
                    │
                    ▼
                 /cmd_vel
                    │
                    ▼
                🤖 Robot
                    │
                    ▼
               ┌─────────┐
               │  RViz2  │
               └─────────┘
```

---

# 📊 Gazebo vs RViz2

It is important to understand that Gazebo and RViz2 have different purposes.

| Tool             | Purpose                                      |
| ---------------- | -------------------------------------------- |
| **Gazebo**       | Simulates the physical robot and environment |
| **ROS 2**        | Connects robot software and sensors          |
| **LiDAR**        | Produces simulated distance measurements     |
| **SLAM Toolbox** | Builds a map                                 |
| **Nav2**         | Plans and executes navigation                |
| **RViz2**        | Visualizes robot data                        |

In simple terms:

```text
Gazebo = "What is physically happening?"

RViz2  = "What does ROS think is happening?"
```

---

# 📁 Recommended Repository Structure

A clean repository can eventually look like:

```text
ros2-turtlebot3-indoor-simulation/
│
├── README.md
│
├── worlds/
│   └── indoor_world.world
│
├── maps/
│   └── indoor_map.yaml
│
├── config/
│   ├── slam.yaml
│   └── nav2_params.yaml
│
├── launch/
│   ├── simulation.launch.py
│   ├── slam.launch.py
│   └── navigation.launch.py
│
├── rviz/
│   └── indoor_robot.rviz
│
└── scripts/
    └── robot_commands.sh
```

---

# 🎯 Project Goal

The final objective of this project is to create a complete indoor robotics simulation where:

```text
             👤 User
                │
                ▼
             RViz2
                │
                │ Goal
                ▼
              Nav2
                │
                ▼
           Path Planning
                │
                ▼
             /cmd_vel
                │
                ▼
          🤖 TurtleBot3
                │
       ┌────────┴────────┐
       ▼                 ▼
     LiDAR            Odometry
       │                 │
       └────────┬────────┘
                ▼
           SLAM Toolbox
                │
                ▼
             🗺 Map
                │
                └──────────► RViz2
```

The result is a complete **ROS 2 indoor autonomous mobile robotics environment** suitable for learning, demonstrations, workshops, and further development.

---

# 🛠️ Troubleshooting

## `gz_math_vendor` not found

If you see:

```text
Could not find a package configuration file provided by
"gz_math_vendor"
```

check your TurtleBot3 branch:

```bash
cd ~/ros2_ws/src/turtlebot3
git branch --show-current
```

For this project using ROS 2 Humble, it should be:

```text
humble
```

If it says:

```text
jazzy
```

you have cloned the wrong branch.

Remove the incompatible repositories:

```bash
cd ~/ros2_ws/src

rm -rf turtlebot3
rm -rf turtlebot3_msgs
rm -rf turtlebot3_simulations
```

Then clone the Humble versions:

```bash
git clone -b humble https://github.com/ROBOTIS-GIT/turtlebot3.git
git clone -b humble https://github.com/ROBOTIS-GIT/turtlebot3_msgs.git
git clone -b humble https://github.com/ROBOTIS-GIT/turtlebot3_simulations.git
```

Reinstall dependencies:

```bash
cd ~/ros2_ws

rosdep install --from-paths src --ignore-src -r -y
```

Then rebuild:

```bash
colcon build --symlink-install
```

---

# 🚀 Quick Start

Once everything is installed, the normal workflow is:

### Terminal 1 — Gazebo

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
export TURTLEBOT3_MODEL=burger

ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
```

### Terminal 2 — RViz2

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
export TURTLEBOT3_MODEL=burger

ros2 launch turtlebot3_bringup rviz2.launch.py
```

### Terminal 3 — Robot Control

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash

ros2 topic pub --once \
/cmd_vel \
geometry_msgs/msg/Twist \
"{linear: {x: 0.5}, angular: {z: 0.0}}"
```

---

# 📚 Technologies Used

* **ROS 2 Humble**
* **Ubuntu 22.04**
* **WSL2**
* **TurtleBot3**
* **Gazebo**
* **RViz2**
* **SLAM Toolbox**
* **Nav2**
* **LiDAR**
* **TF2**
* **ROS 2 Topics**
* **ROS 2 Nodes**

---

# 📌 Learning Path

Recommended progression:

```text
01. ROS 2 Basics
       ↓
02. Nodes & Topics
       ↓
03. TurtleBot3
       ↓
04. Gazebo
       ↓
05. LiDAR
       ↓
06. TF2
       ↓
07. RViz2
       ↓
08. SLAM
       ↓
09. Mapping
       ↓
10. Nav2
       ↓
11. Autonomous Navigation
       ↓
12. Real Robot Deployment
```

---

# ⭐ Project Outcome

By completing this project, you will have a working simulated mobile robot capable of:

* Moving inside an indoor environment
* Publishing LiDAR data
* Publishing odometry
* Broadcasting TF transforms
* Building an indoor map
* Visualizing sensor data in RViz2
* Planning navigation paths
* Executing autonomous navigation

This provides a practical foundation for **mobile robotics, warehouse robotics, AMRs, AGVs, logistics automation, SLAM, and ROS 2 development**.
