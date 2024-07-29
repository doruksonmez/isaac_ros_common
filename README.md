# Isaac ROS Common

Dockerfiles and scripts for development using the Isaac ROS suite.

## Overview

The [Isaac ROS Common](https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_common)
repository contains a number of scripts and Dockerfiles to help
streamline development and testing with the Isaac ROS suite.

<div align="center"><a class="reference internal image-reference" href="https://media.githubusercontent.com/media/NVIDIA-ISAAC-ROS/.github/main/resources/isaac_ros_docs/repositories_and_packages/isaac_ros_common/isaac_ros_common_tools.png/"><img alt="Isaac ROS DevOps tools" src="https://media.githubusercontent.com/media/NVIDIA-ISAAC-ROS/.github/main/resources/isaac_ros_docs/repositories_and_packages/isaac_ros_common/isaac_ros_common_tools.png/" width="auto"/></a></div>

Docker containers allow you to quickly set up a sensitive set of frameworks
and dependencies to ensure a smooth experience with Isaac ROS packages.
The Dockerfiles for x86_64 are based on the version 23.10 image from [Deep Learning
Frameworks Containers](https://docs.nvidia.com/deeplearning/frameworks/support-matrix/index.html).
On Jetson platforms, JetPack manages all of these dependencies for you.

Use of Docker images enables CI|CD systems to scale with DevOps work and
run automated testing in cloud native platforms on Kubernetes.

For solutions to known issues, see the [Troubleshooting](https://nvidia-isaac-ros.github.io/troubleshooting/index.html) section.

---

## Documentation

Please visit the [Isaac ROS Documentation](https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_common/index.html) to learn how to use this repository.

---

## Latest

Update 2024-05-30: Update to be compatible with JetPack 6.0

## Installation
```
cd ${ISAAC_ROS_WS}/src && \
   git clone https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_common.git
```
```
sudo apt-get install -y curl tar
```
```
NGC_ORG="nvidia"
NGC_TEAM="isaac"
NGC_RESOURCE="isaac_ros_assets"
NGC_VERSION="isaac_ros_bi3d"
NGC_FILENAME="quickstart.tar.gz"

REQ_URL="https://api.ngc.nvidia.com/v2/resources/$NGC_ORG/$NGC_TEAM/$NGC_RESOURCE/versions/$NGC_VERSION/files/$NGC_FILENAME"

mkdir -p ${ISAAC_ROS_WS}/isaac_ros_assets/${NGC_VERSION} && \
    curl -LO --request GET "${REQ_URL}" && \
    tar -xf ${NGC_FILENAME} -C ${ISAAC_ROS_WS}/isaac_ros_assets/${NGC_VERSION} && \
    rm ${NGC_FILENAME}
```
```
mkdir -p ${ISAAC_ROS_WS}/isaac_ros_assets/models/bi3d_proximity_segmentation && \
cd ${ISAAC_ROS_WS}/isaac_ros_assets/models/bi3d_proximity_segmentation && \
wget 'https://api.ngc.nvidia.com/v2/models/nvidia/isaac/bi3d_proximity_segmentation/versions/2.0.0/files/featnet.onnx' &&
wget 'https://api.ngc.nvidia.com/v2/models/nvidia/isaac/bi3d_proximity_segmentation/versions/2.0.0/files/segnet.onnx'
```
```
cd ${ISAAC_ROS_WS}/src/isaac_ros_common && \
./scripts/run_dev.sh
```
```
sudo apt-get install -y ros-humble-isaac-ros-bi3d
```
```
/usr/src/tensorrt/bin/trtexec --saveEngine=${ISAAC_ROS_WS}/isaac_ros_assets/models/bi3d_proximity_segmentation/featnet.plan \
--onnx=${ISAAC_ROS_WS}/isaac_ros_assets/models/bi3d_proximity_segmentation/featnet.onnx --int8 &&
/usr/src/tensorrt/bin/trtexec --saveEngine=${ISAAC_ROS_WS}/isaac_ros_assets/models/bi3d_proximity_segmentation/segnet.plan \
--onnx=${ISAAC_ROS_WS}/isaac_ros_assets/models/bi3d_proximity_segmentation/segnet.onnx --int8
```
```
sudo apt-get install -y ros-humble-isaac-ros-examples
```
```
ros2 launch isaac_ros_examples isaac_ros_examples.launch.py \
launch_fragments:=bi3d \
interface_specs_file:=${ISAAC_ROS_WS}/isaac_ros_assets/isaac_ros_bi3d/rosbag_quickstart_interface_specs.json \
featnet_engine_file_path:=${ISAAC_ROS_WS}/isaac_ros_assets/models/bi3d_proximity_segmentation/featnet.plan \
segnet_engine_file_path:=${ISAAC_ROS_WS}/isaac_ros_assets/models/bi3d_proximity_segmentation/segnet.plan \
max_disparity_values:=10
```
Second terminal:
```
xhost +
docker exec -ti  bash
```
```
ros2 bag play --loop ${ISAAC_ROS_WS}/isaac_ros_assets/isaac_ros_bi3d/bi3dnode_rosbag
```
Three new terminals:
#1 for disparity calculation and output image:
```
ros2 run isaac_ros_bi3d isaac_ros_bi3d_visualizer.py --max_disparity_value 30
```
#2 for raw camera input from right eye:
```
ros2 run image_view image_view --ros-args -r image:=right/image_rect
```
#3 for raw camera input from left eye:
```
ros2 run image_view image_view --ros-args -r image:=left/image_rect
```
