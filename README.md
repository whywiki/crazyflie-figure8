<div align="center">

# Crazyflie Figure-8

[![ROS2](https://img.shields.io/badge/ROS_2-Humble-22314E?style=for-the-badge&labelColor=2D2D2D&logo=ros&logoColor=white)](https://docs.ros.org/en/humble/index.html)
&nbsp;&nbsp;
[![Gazebo](https://img.shields.io/badge/Gazebo-Harmonic-FF7043?style=for-the-badge&labelColor=2D2D2D&logo=gazebo&logoColor=white)](https://gazebosim.org/docs/harmonic/getstarted/)
&nbsp;&nbsp;
[![Python](https://img.shields.io/badge/Python-3.10-4CAF50?style=for-the-badge&labelColor=2D2D2D&logo=python&logoColor=white)](https://python.org)
&nbsp;&nbsp;
[![Docker](https://img.shields.io/badge/Docker-Ubuntu-2496ED?style=for-the-badge&labelColor=2D2D2D&logo=docker&logoColor=white)](https://docs.docker.com/)

A Crazyflie quadrotor flying in Gazebo through ROS 2. The drone takes off, hovers, then follows a figure-8 trajectory using cascaded PD position and attitude controllers.

![Drone Simulation](assets/drone.gif)

</div>

## What I built

My work lives in [`my_controller/`](my_controller):

- **`drone_controller.py`**: a ROS 2 node that closes the loop on the Gazebo drone.
  - **Position controller (PD):** turns position and velocity error into collective thrust and desired roll/pitch angles, with tilt limited to 25°.
  - **Attitude controller (PD):** turns angle error and angular rate into body torques.
  - **Trajectory:** ramps the altitude up for takeoff, then switches to a figure-8 once above 0.85 m, using velocity and acceleration feedforward on the curved path.
  - **Thrust tilt compensation:** keeps the drone from losing altitude when it leans.
  - Converts body-frame velocity to world frame and quaternions to Euler angles, and publishes `/pose` and `/path` for visualisation.
- **`drone_sim.py`**: a standalone Python simulation of the Crazyflie's physics, using Newton–Euler rigid-body equations with inertia values from the datasheet, including the gyroscopic term. It flies a circle with the same PD controllers.

## Stack

- ROS 2 Humble
- Gazebo Harmonic
- Python, NumPy
- Docker (Ubuntu, WSL2 on Windows)

## Setup

Open the project in VS Code and reopen in container. Then:

```bash
cd /home/developer/ros2_ws
source ./setup.sh <your_unique_id>
./build.sh
source install/setup.bash
```

## Running

In every new terminal, first run:

```bash
source install/setup.bash
```

```bash
# Terminal 1 - simulation
ros2 launch ros_gz_crazyflie_bringup crazyflie_simulation.launch.py

# Terminal 2 - mixer
ros2 run cf_control mixer

# Terminal 3 - controller
cd /home/developer/ros2_ws/src/my_controller
python3 drone_controller.py
```

The controller publishes collective thrust and torques to `/cf_control/control_command`, and the mixer converts them into individual motor commands.

To run the standalone Python simulation instead (no Gazebo needed):

```bash
cd /home/developer/ros2_ws/src/my_controller
python3 drone_sim.py
```

## Credits

The development container, Gazebo world and Crazyflie model, ROS–Gazebo bridge and the C++ mixer (`cf_control`) come from the [AGH Vision Lab course template](https://github.com/vision-agh/cf-control). The controllers and the Python simulation in `my_controller/` are my own.
