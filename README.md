# carla_ros_bridge_docker

## Introduction

This is a Dockerfile to use [CARLA ROS bridge](https://github.com/carla-simulator/ros-bridge) on Docker container.

![](img/carla_ad_demo_with_scenario.png)

Current [carla-simulator/ros-bridge](https://github.com/carla-simulator/ros-bridge) supports to CARLA 0.9.12. So, I used CARLA 0.9.12 in this Dockerfile. And, base image of this Docker file is `nvidia/cudagl:11.4.2-devel-ubuntu20.04`.

> **Note**: The Debian package is available for both Ubuntu 18.04 and Ubuntu 20.04, however the officially supported platform is Ubuntu 18.04.

## Requirements

* NVIDIA graphics driver
* Docker
* nvidia-docker2

## Preparation

### Build Docker image

```shell
docker build -t carla-ros-bridge:0.9.12-noetic-ubuntu20.04 --build-arg CARLA_VERSION=0.9.12 --build-arg GID=$(id -g) --build-arg UID=$(id -u) -f Dockerfile.noetic .
```

#### On Windows behind Corporate Proxy

The trickiest part is to make `rosdep init` and `rosdep update` work, since they don't accept any argument for proxy. After many trials and errors with `docker build` options like `--network` and `--add-host`, even resorting to desperate measure of setting up a local HTTP server for pre-downloaded files from [20-default.list](https://raw.github.com/ros/rosdistro/master/rosdep/sources.list.d/20-default.list), I finally arrived at a solution, [Px proxy server](https://github.com/genotrance/px). Here's my [px.ini](https://github.com/genotrance/px/blob/master/px.ini) settings:

```
[proxy]
server = <your corporate proxy>
port = 3128
hostonly = 1
username = <your domain>\<your username>
...
```

Start your Px server, then build the docker image:

```shell
docker build -t carla-ros-bridge:0.9.12-noetic-ubuntu20.04 --build-arg CARLA_VERSION=0.9.12 --build-arg GID=$(id -g) --build-arg UID=$(id -u) --build-arg http_proxy=http://<your ip>:3128 --build-arg https_proxy=http://<your ip>:3128 -f Dockerfile.noetic .
```

### Create Docker container

```shell
./launch_container.sh
```

## Usage

### CARLA ROS bridge

#### 1. Launch CARLA Simulator

Please launch CARLA Simulator by the following command.

```shell
/opt/carla-simulator/CarlaUE4.sh -windowed -ResX=160 -ResY=120
```

#### 2. Set the configuration of CARLA Simulator

```shell
cd /opt/carla-simulator/PythonAPI
python util/config.py -m Town03 --fps 10
```

#### 3. Launch CARLA ROS bridge

```shell
roslaunch carla_ros_bridge carla_ros_bridge_with_example_ego_vehicle.launch vehicle_filter:='vehicle.toyota.prius*'
```

### CARLA AD Demo

#### 1. Launch CARLA Simulator

Please launch CARLA Simulator by the following command.

```shell
/opt/carla-simulator/CarlaUE4.sh -windowed -ResX=160 -ResY=120
```

#### 2. Set the configuration of CARLA Simulator

```shell
cd /opt/carla-simulator/PythonAPI
python util/config.py -m Town01 --fps 10
```

#### 3. Launch CARLA AD Demo

```shell
roslaunch carla_ad_demo carla_ad_demo_with_scenario.launch vehicle_filter:='vehicle.toyota.prius*'
```

And, please push `Execute` button from RViz panel.
