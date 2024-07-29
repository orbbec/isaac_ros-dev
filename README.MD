# Setting Up the AGX Orin NVblox Environment

## 1. Installing Dependencies

### CUDA Installation
Check for the presence of CUDA using `nvcc --version`. If not installed, proceed with the following command:
```bash
sudo apt-get install cuda
```

#### Adding CUDA Environment Variables
Configure your system's environment variables to ensure proper operation of CUDA:
```bash
echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
echo 'export CUDA_HOME=/usr/local/cuda' >> ~/.bashrc
source ~/.bashrc
```

#### Verifying CUDA Installation
To verify CUDA installation:
```bash
nvcc --version
```

### ROS2 Humble Installation
Set up the APT repository using the guidelines provided on [NVIDIA's Isaac ROS build farm CDN](https://nvidia-isaac-ros.github.io/getting_started/isaac_ros_buildfarm_cdn.html).
```bash
sudo apt-get update
sudo apt-get install ros-humble-desktop-full
```

## 2. Installing the Negotiated Package
Install ROS dependencies and the negotiated package:
```bash
rosdep init
curl -o /etc/ros/rosdep/sources.list.d/nvidia-isaac.yaml https://isaac.download.nvidia.com/isaac-ros/extra_rosdeps.yaml
echo "yaml file:///etc/ros/rosdep/sources.list.d/nvidia-isaac.yaml" | tee /etc/ros/rosdep/sources.list.d/00-nvidia-isaac.list
rosdep update
sudo apt-get install -y build-essential devscripts dh-make quilt fakeroot 
mkdir -p ~/ros2_ws/src && cd ~/ros2_ws/src \
    && git clone https://github.com/osrf/negotiated && cd negotiated && git checkout master \
    && source ~/ros2_ws/setup.bash \
    && cd negotiated_interfaces && bloom-generate rosdebian && fakeroot debian/rules binary \
    && cd ../ && apt-get install -y ./*.deb && rm ./*.deb \
    && cd negotiated && bloom-generate rosdebian && fakeroot debian/rules binary \
    && cd ../ && apt-get install -y ./*.deb && rm ./*.deb
```

## 3. Additional Dependencies
Further required libraries and ROS packages:
```bash
sudo apt install libnvvpi2 ros-humble-isaac-ros-nvblox ros-humble-isaac-ros-visual-slam
```

Certainly! Here's a more tactful and user-friendly way to convey the instructions, ensuring clarity while maintaining a polite tone:

---

## Ready to Launch NVblox Examples

Assuming all necessary software has been compiled and installed correctly, you should now be able to run the NVblox examples directly. 

### Preparing Your Environment
It is likely that you have already placed the relevant packages in the home directory of your AGX Orin. For the next steps, please ensure that you are operating within the `~/isaac_ros-dev` directory on your system. This will help in keeping all commands organized and functioning as expected.

Feel free to proceed with launching the applications as detailed in the setup instructions. If you encounter any issues, verify that you are in the correct directory and that all installation steps have been properly completed.

### Preparing the Camera
Execute the following in the terminal to set up the camera:
```bash 
cd src/OrbbecSDK_ROS2/orbbec_camera/scripts
sudo bash install_udev_rules.sh
```

### Launching the Camera
Open two terminal windows and proceed as follows:

#### Window 1 - Camera Launch
```bash
source install/setup.bash
ros2 launch orbbec_camera gemini_330_series.launch.py laser_on_off_mode:=1
```
Note: If you encounter a "Resource busy" error, please retry after a few seconds.

### Running NVblox Examples
```bash
source install/setup.bash
ros2 launch nvblox_examples_bringup orbbec_dynamics_example.launch.py
```

## NVSLAM Setup Instructions
Set up and run NVSLAM using three terminal windows:

### Window 1 - Launch Camera for NVSLAM
```bash
source install/setup.bash
ros2 launch orbbec_camera gemini_330_series.launch.py enable_laser:=false
```
Repeat the attempt if a "Resource busy" error occurs.

### Window 2 - Execute Visual SLAM
```bash
source install/setup.bash
ros2 launch isaac_ros_visual_slam isaac_ros_visual_slam_orbbec.launch.py
```

### Window 3 - Start RViz2
```bash
source install/setup.bash
rviz2 -d src/isaac_ros_visual_slam/isaac_ros_visual_slam/rviz/default.cfg.rviz
```
