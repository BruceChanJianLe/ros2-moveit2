# ROS2 Moveit2

This repository has notes about moveit2 in ROS2.

## Compiling from Source

OS: Ubuntu 20.04
ROS2 Version: Humble (from source)
Moveit2 Version: Humble (from source)

1. Install ROS2 dependencies
```bash
sudo apt update && sudo apt install -y \
  python3-flake8-docstrings \
  python3-pip \
  python3-pytest-cov \
  ros-dev-tools

python3 -m pip install -U \
   flake8-blind-except \
   flake8-builtins \
   flake8-class-newline \
   flake8-comprehensions \
   flake8-deprecated \
   flake8-import-order \
   flake8-quotes \
   "pytest>=5.3" \
   pytest-repeat \
   pytest-rerunfailures
```

2. Clone Humble and build it
```bash
mkdir ros2_ws/humble_ws/src -p
vcs import --input https://raw.githubusercontent.com/ros2/ros2/humble/ros2.repos ros2_ws/humble_ws/src
sudo rosdep init
rosdep update
rosdep install --from-paths ros2_ws/humble_ws/src --ignore-src -y --skip-keys "fastcdr rti-connext-dds-6.0.1 urdfdom_headers"
cd ros2_ws/humble_ws
colcon build --symlink-install --cmake-args -DPYBIND11_PYTHON_VERSION=3
```
