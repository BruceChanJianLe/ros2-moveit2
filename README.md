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

3. Install Moveit2 dependencies
```bash
sudo apt install -y \
  build-essential \
  cmake \
  git \
  python3-colcon-common-extensions \
  python3-flake8 \
  python3-rosdep \
  python3-setuptools \
  python3-vcstool \
  wget
sudo apt update
sudo apt dist-upgrade
rosdep update
```

4. Clone Moveit2

Please pay special attention to some of the external pkg repos.
For example, hpp-fcl and ruckig both has third-parties dependencies which will need to be handled (submodule) manually.
```bash
cd ros2_ws/humble_ws/src
git clone https://github.com/moveit/moveit2.git -b humble
for repo in moveit2/moveit2.repos $(f="moveit2/moveit2.repos"; test -r $f && echo $f); do vcs import < "$repo"; done
# rosdep install -r --from-paths . --ignore-src --rosdistro $ROS_DISTRO -y # this won't work
mkdir external_pkgs
cd external_pkgs
git clone https://github.com/BruceChanJianLe/ros2-moveit2.git
vcs import src < src/linux-vcstool/test_ws.repos
```

5. Update cmakes and extra errands

Based on this repo, please copy over the files to replace in moveit2 directory inside of `humble_ws/src/moveit2`.

6. Build it

```bash
cd ros2_ws/humble_ws
colcon build --symlink-install --cmake-args -DPYBIND11_PYTHON_VERSION=3
```
