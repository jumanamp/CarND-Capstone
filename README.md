
Project: Integrate the subsystems for autonomous driving using ROS and run the full system software on a Self Driving Car.
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

## Driftomaniac Team Members

This is our final project implementation of the Udacity Self-Driving Car Nanodegree: Programming a Real Self-Driving Car.

**Team Member 1** : Owen Hua, owen.hua930531@hotmail.com

**Team Member 2** :  Jumana Mundichipparakkal, jumanamp91@gmail.com

## Overview
In this project, the goal is to write a fully functional system software that implements the core functionality of the autonomous vehicle system, including traffic light detection, control, and waypoint following. System is integrated by writing ROS nodes for each sub system which then talks to each other by publish subscribe topics. End goal is to test the software on the simulator as well as Udacity Carla car.

The following is a system architecture diagram showing the ROS nodes and topics used in the project.
We will cover each node and topics briefly along with our implementation description below.

<p align="center">
<img width="700" height="500" src="readme/sys_archi.png">
</p>
<p align="center">
<em>Figure 1: Autonomous Driving System Architecture</em>
</p>

Autonomous driving system has three major subsystems:
* Perception Subsystem: Process the input data from sensors to create awareness of the environment.
* Planning Subsystem: From the localized information of the car's position and environment details like traffic signals, plan the path for movement and generate trajectory.
* Control Subsystem: Produce actuation commands for the car to move in the trajectory as planned.

---
### The Project
System Integration project aims to integrate the ROS nodes that form parts of the various subsystems of the autonomous drive which are:
* Waypoint Updater Node: Part of path planning subsystem that updates the waypoints for the car to move and input to the control subsystem
* Traffic Light Detector Node: Takes care of traffic signal detection which is part of the perception subsystem.
* DBW Node: Drive by Wire Node controls the actuation of the car and enable the software to talk to the hardware.

### Project Solution

System Integration for the project is implemented by taking a simpler approach of three major steps as detailed below.


#### 1. Waypoint Updater Node

Waypoint updater node intakes the sensor data and position data from perception subsystems and generate the final waypoints as shown in Figure 2.

<p align="center">
<img width="400" height="150" src="readme/waypoint-updater-ros-graph.png">
</p>
<p align="center">
<em>Figure 2: Sensor Fusion based Prediction Flow chart</em>
</p>

For generating the final waypoints, it also intakes the input from the traffic light detector node to know which light is on now and take action accordingly. System has two states, one for moving, and one for stopping. When there is no traffic lights nearby, or if the light is green, car is accelerating forward. when car is close to a traffic light and light is red, car begins to stop. For this, planned waypoints to the traffic light are decelerated on each waypoint so that last waypoint before the traffic light is zero.

#### 2. Traffic Light Detector Node

This node intakes the vehicle's location and the (x, y) coordinates for traffic lights to find the nearest visible traffic light ahead of the vehicle. We used the get_closest_waypoint method to find the closest waypoints to the vehicle and lights. It also uses the camera image data to classify the color of the traffic light.

<p align="center">
<img width="400" height="150" src="readme/tl-detector-ros-graph.png">
</p>
<p align="center">
<em>Figure 3: Sensor Fusion based Prediction Flow chart</em>
</p>

We started with training deep learning classifier to classify the entire image as containing either a red light, yellow light, green light, or no light. We used a SSD(Single Shot MultiBox Detector) model initially, which was designed for object detection in real-time. The SSD object detection involves extracting feature maps and then applying convolution filters to detect objects.

As training requires a lot of data as well as is time consuming to get to a high level accuracy, we decided to go for frozen models based on SSD from another classmate who had trained in past. Link to the model is attached here:
https://github.com/xfqbuaa/Cargo-CarND-Capstone/tree/master/ros/src/tl_detector/model

#### 3. DBW Node

This node intakes the final waypoints published by the planning module and use various controllers to provide appropriate throttle, brake, and steering commands. Steering is controlled by yaw controller and throttle and brake are controlled by the PID controller.


<p align="center">
<img width="400" height="150" src="readme/dbw-node-ros-graph.png">
</p>
<p align="center">
<em>Figure 4: Sensor Fusion based Prediction Flow chart</em>
</p>


---

### Run instructions
Please use **one** of the two installation options, either native **or** docker installation.
## Native Installation

* Be sure that your workstation is running Ubuntu 16.04 Xenial Xerus or Ubuntu 14.04 Trusty Tahir. [Ubuntu downloads can be found here](https://www.ubuntu.com/download/desktop).
* If using a Virtual Machine to install Ubuntu, use the following configuration as minimum:
  * 2 CPU
  * 2 GB system memory
  * 25 GB of free hard drive space

  The Udacity provided virtual machine has ROS and Dataspeed DBW already installed, so you can skip the next two steps if you are using this.

* Follow these instructions to install ROS
  * [ROS Kinetic](http://wiki.ros.org/kinetic/Installation/Ubuntu) if you have Ubuntu 16.04.
  * [ROS Indigo](http://wiki.ros.org/indigo/Installation/Ubuntu) if you have Ubuntu 14.04.
* [Dataspeed DBW](https://bitbucket.org/DataspeedInc/dbw_mkz_ros)
  * Use this option to install the SDK on a workstation that already has ROS installed: [One Line SDK Install (binary)](https://bitbucket.org/DataspeedInc/dbw_mkz_ros/src/81e63fcc335d7b64139d7482017d6a97b405e250/ROS_SETUP.md?fileviewer=file-view-default)
* Download the [Udacity Simulator](https://github.com/udacity/CarND-Capstone/releases).

## Docker Installation
[Install Docker](https://docs.docker.com/engine/installation/)

Build the docker container
```bash
docker build . -t capstone
```

Run the docker file
```bash
docker run -p 4567:4567 -v $PWD:/capstone -v /tmp/log:/root/.ros/ --rm -it capstone
```

## Port Forwarding
To set up port forwarding, please refer to the [instructions from term 2](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/0949fca6-b379-42af-a919-ee50aa304e6a/lessons/f758c44c-5e40-4e01-93b5-1a82aa4e044f/concepts/16cf4a78-4fc7-49e1-8621-3450ca938b77)

## Usage

1. Clone the project repository
```bash
git clone https://github.com/udacity/CarND-Capstone.git
```

2. Install python dependencies
```bash
cd CarND-Capstone
pip install -r requirements.txt
```
3. Make and run styx
```bash
cd ros
catkin_make
source devel/setup.sh
roslaunch launch/styx.launch
```
4. Run the simulator

## Real world testing
1. Download [training bag](https://s3-us-west-1.amazonaws.com/udacity-selfdrivingcar/traffic_light_bag_file.zip) that was recorded on the Udacity self-driving car.
2. Unzip the file
```bash
unzip traffic_light_bag_file.zip
```
3. Play the bag file
```bash
rosbag play -l traffic_light_bag_file/traffic_light_training.bag
```
4. Launch your project in site mode
```bash
cd CarND-Capstone/ros
roslaunch launch/site.launch
```
5. Confirm that traffic light detection works on real life images
