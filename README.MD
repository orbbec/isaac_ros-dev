
# NVSLAM and NVblox Setup Guide for AGX Orin with Orbbec Gemini Cameras

This guide covers the setup process for NVSLAM and NVblox environments on the NVIDIA Jetson AGX Orin platform. The setup is based on Ubuntu 22.04 with JetPack 6. This guide specifically addresses the use of Orbbec Gemini cameras, tested with the Gemini 335L and Gemini 336L models.

## 1. Platform-Specific Steps

### 1.1 CUDA Installation (if not already installed)

Check if CUDA is already installed:
```bash
nvcc --version
```

If not installed, run:
```bash
sudo apt-get install cuda
```

Configure CUDA environment variables:
```bash
echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
echo 'export CUDA_HOME=/usr/local/cuda' >> ~/.bashrc
source ~/.bashrc
```

Verify CUDA installation:
```bash
nvcc --version
```

## 2. Common Steps

### 2.1 ROS2 Humble Installation

Set up the APT repository using [NVIDIA's Isaac ROS build farm repo](https://nvidia-isaac-ros.github.io/getting_started/isaac_apt_repository.html). Please select the Isaac ROS 3.0 version.

Then install ROS2 Humble:
```bash
sudo apt-get update
sudo apt-get install ros-humble-desktop-full
```

### 2.2 Installing the Negotiated Package

```bash
rosdep init
sudo curl -o /etc/ros/rosdep/sources.list.d/nvidia-isaac.yaml https://isaac.download.nvidia.com/isaac-ros/extra_rosdeps.yaml
echo "yaml file:///etc/ros/rosdep/sources.list.d/nvidia-isaac.yaml" | sudo tee /etc/ros/rosdep/sources.list.d/00-nvidia-isaac.list
rosdep update
sudo apt-get install -y build-essential devscripts dh-make quilt fakeroot

mkdir -p ~/ros2_ws/src && cd ~/ros2_ws/src \
    && git clone https://github.com/osrf/negotiated && cd negotiated && git checkout master \
    && source /opt/ros/humble/setup.bash \
    && cd negotiated_interfaces && bloom-generate rosdebian && fakeroot debian/rules binary \
    && cd ../ && sudo apt-get install -y ./*.deb && rm ./*.deb \
    && cd negotiated && bloom-generate rosdebian && fakeroot debian/rules binary \
    && cd ../ && sudo apt-get install -y ./*.deb && rm ./*.deb
```

### 2.3 Additional Dependencies

```bash
sudo apt install libnvvpi3 ros-humble-isaac-ros-nvblox ros-humble-isaac-ros-visual-slam
```

## 3. Setting Up the Environment

### 3.1 Get Source Code and Initialize Submodules

```bash
git clone https://github.com/orbbec/isaac_ros-dev.git
cd isaac_ros-dev
git submodule update --init --recursive
```

### 3.2 Build the Workspace

```bash
colcon build
```

### 3.3 Prepare the Camera

```bash
cd src/OrbbecSDK_ROS2/orbbec_camera/scripts
sudo bash install_udev_rules.sh
```

## 4. Running Examples

Note: NVSLAM and NVblox are separate examples and cannot be run simultaneously. Choose one of the following examples to run.

### 4.1 Running NVSLAM Example

To run the NVSLAM example, open three terminal windows:

#### Terminal 1 - Launch Camera for NVSLAM

```bash
source install/setup.bash
ros2 launch isaac_orbbec_launch isaac_ros_visual_slam_orbbec.launch.py
```

#### Terminal 2 - Start RViz2 for NVSLAM

```bash
source install/setup.bash
rviz2 -d src/isaac_orbbec_launch/rviz/orbbec.rviz
```

### 4.2 Running NVblox Example

To run the NVblox example, open two terminal windows:

#### Terminal 1 - Launch NVblox Example

```bash
source install/setup.bash
ros2 launch isaac_orbbec_launch orbbec_dynamics_example.launch.py
```

## 5. Demo Video

For a demonstration of Orbbec Gemini 335L on NVIDIA Jetson AGX Orin with Isaac ROS Visual SLAM and Nvblox, you can watch this video:

[Orbbec Gemini 335L Nvblox Demo](https://youtu.be/jpbjC_9eQEI?si=UKw3BR0Gjs6QiKLb)
