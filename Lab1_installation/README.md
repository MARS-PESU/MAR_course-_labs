# ROS Introduction & Setup

## 1. What is ROS?

The Robot Operating System (ROS) is not an operating system in the traditional sense (like Windows or Linux). Instead, it is a flexible middleware framework for writing robot software. It provides a collection of tools, libraries, and conventions that simplify the task of creating complex and robust robot behavior across a wide variety of robotic platforms.

### Key Features

- **Message Passing**: Anonymous "publish/subscribe" mechanism for data exchange
- **Hardware Abstraction**: Write code that works on different sensors/actuators
- **Package Management**: Easily share and reuse code within the global robotics community
- **Tooling**: High-quality visualization (RViz) and simulation (Gazebo) tools

## 2. ROS Versions & Lifecycle (EOL)

As of January 2026, the ROS ecosystem has fully transitioned from ROS 1 to ROS 2. All ROS 1 versions (like Noetic) have officially reached End-of-Life (EOL).

### Active & Future ROS 2 Distributions

| Distribution | Release Date | EOL Date | Target Ubuntu Version |
|---|---|---|---|
| Lyrical Luth | May 2026 (Upcoming) | May 2031 | Ubuntu 26.04 |
| Kilted Kaiju | May 2025 | Dec 2026 | Ubuntu 24.04 / 25.04 |
| Jazzy Jalisco | May 2024 | May 2029 (LTS) | Ubuntu 24.04 |
| Humble Hawksbill | May 2022 | May 2027 (LTS) | Ubuntu 22.04 |
| Iron Irwini | May 2023 | Dec 2024 (EOL) | Ubuntu 22.04 |

## 3. Installation

### For Humble (Ubuntu 22.04 LTS)

```bash
# Set locale
locale  # check for UTF-8

sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

locale  # verify settings

# Setup sources
sudo apt install software-properties-common
sudo add-apt-repository universe

sudo apt update && sudo apt install curl -y
export ROS_APT_SOURCE_VERSION=$(curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest | grep -F "tag_name" | awk -F\" '{print $4}')
curl -L -o /tmp/ros2-apt-source.deb "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo ${UBUNTU_CODENAME:-${VERSION_CODENAME}})_all.deb"
sudo dpkg -i /tmp/ros2-apt-source.deb

# Install ROS 2 Humble
sudo apt update
sudo apt upgrade
sudo apt install ros-humble-desktop

sudo apt update && sudo apt install ros-dev-tools

# Environment setup
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### For Jazzy (Ubuntu 24.04 LTS)

```bash
# Set locale
locale  # check for UTF-8

sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

locale  # verify settings

# Setup sources
sudo apt install software-properties-common
sudo add-apt-repository universe

sudo apt update && sudo apt install curl -y
export ROS_APT_SOURCE_VERSION=$(curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest | grep -F "tag_name" | awk -F\" '{print $4}')
curl -L -o /tmp/ros2-apt-source.deb "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo ${UBUNTU_CODENAME:-${VERSION_CODENAME}})_all.deb"
sudo dpkg -i /tmp/ros2-apt-source.deb

# Install ROS 2 Jazzy
sudo apt update
sudo apt upgrade
sudo apt install ros-jazzy-desktop

# Environment setup
echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### Verify Installation

```bash
# Check ROS 2 installation
ros2 --help

# Test with demo nodes
ros2 run demo_nodes_cpp talker
# In another terminal:
ros2 run demo_nodes_py listener
```
