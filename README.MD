# NVSLAM, NVblox and Foundationpose Setup Guide for AGX Orin with Orbbec Gemini Cameras

This guide covers the setup process for NVSLAM and NVblox environments on the NVIDIA Jetson AGX Orin platform. The setup is based on Ubuntu 22.04 with JetPack 6. This guide specifically addresses the use of Orbbec Gemini cameras, tested with the Gemini 335L, Gemini 335,  Gemini 336L and Gemini 336 models.

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

Set up the APT repository using [NVIDIA&#39;s Isaac ROS build farm repo](https://nvidia-isaac-ros.github.io/getting_started/isaac_apt_repository.html). Please select the Isaac ROS 3.0 version.

Then install ROS2 Humble:

```bash
sudo apt-get update
sudo apt-get install ros-humble-desktop-full python3-rosdep
```

### 2.2 Installing the Negotiated Package

```bash
sudo rosdep init
sudo curl -o /etc/ros/rosdep/sources.list.d/nvidia-isaac.yaml https://isaac.download.nvidia.com/isaac-ros/extra_rosdeps.yaml
echo "yaml file:///etc/ros/rosdep/sources.list.d/nvidia-isaac.yaml" | sudo tee /etc/ros/rosdep/sources.list.d/00-nvidia-isaac.list
sudo apt-get install -y build-essential devscripts dh-make quilt fakeroot python3-bloom
```

Edit `/etc/ros/rosdep/sources.list.d/nvidia-isaac.yaml`, replace `focal` with `jammy`, then run:

```bash
rosdep update
```

```bash
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
sudo apt install vpi3-dev libnvvpi3 ros-humble-isaac-ros-nvblox ros-humble-isaac-ros-visual-slam
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

### 3.4 Increase USB Memory buffer

```bash
echo 128 | sudo tee /sys/module/usbcore/parameters/usbfs_memory_mb
```

### 3.5 Setup DDS configuration

Please refer to the [Fast DDS Configuration](https://github.com/orbbec/OrbbecSDK_ROS2/blob/main/docs/fastdds_tuning.md) for more information.

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

To run the NVblox example, open one terminal windows:

#### Terminal 1 - Launch NVblox Example

```bash
source install/setup.bash
ros2 launch isaac_orbbec_launch orbbec_dynamics_example.launch.py
```

### 4.3 Running Foundationpose Example

Please refer to the following link for more information: [ISAAC ROS FoundationPose](https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_pose_estimation/isaac_ros_foundationpose/index.html).

To run the Foundationpose example, open one terminal windows:

#### Terminal 1 - Build and Run Foundationpose Example

```bash
colcon build --cmake-clean-first --packages-select isaac_ros_examples isaac_ros_orbbec isaac_ros_foundationpose orbbec_camera orbbec_camera_msgs orbbec_description
source install/setup.bash
ros2 launch isaac_ros_examples isaac_ros_examples.launch.py launch_fragments:=realsense_mono_rect_depth,foundationpose mesh_file_path:=${ISAAC_ROS_WS}/isaac_ros_assets/isaac_ros_foundationpose/Mac_and_cheese_0_1/Mac_and_cheese_0_1.obj texture_path:=${ISAAC_ROS_WS}/isaac_ros_assets/isaac_ros_foundationpose/Mac_and_cheese_0_1/materials/textures/baked_mesh_tex0.png score_engine_file_path:=${ISAAC_ROS_WS}/isaac_ros_assets/models/foundationpose/score_trt_engine.plan refine_engine_file_path:=${ISAAC_ROS_WS}/isaac_ros_assets/models/foundationpose/refine_trt_engine.plan rt_detr_engine_file_path:=${ISAAC_ROS_WS}/isaac_ros_assets/models/synthetica_detr/sdetr_grasp.plan
```

#### Terminal 2 - Start RViz2 for Foundationpose

```bash
source install/setup.bash
rviz2 -d  $(ros2 pkg prefix isaac_ros_foundationpose --share)/rviz/foundationpose_orbbec.rviz
```

## 5. Demo Video

For a demonstration of Orbbec Gemini 335L on NVIDIA Jetson AGX Orin with Isaac ROS Visual SLAM, Nvblox and Foundationpose, you can watch this video:

[Orbbec Gemini 335L Nvblox Demo](https://youtu.be/jpbjC_9eQEI?si=UKw3BR0Gjs6QiKLb)
