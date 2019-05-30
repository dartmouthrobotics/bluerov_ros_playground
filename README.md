# BlueRov-ROS-playground
Scripts to help BlueRov integration with ROS.
What is possible ?
- Video streaming capture with opencv
- Read and write over mavlink protocol with MAVROS
- Joystick interaction
- Gazebo simulation

<p align="center">
  <img src="doc/imgs/bluerov2_gazebo_underwater.png">
</p>

## Getting Started

### Requirements ###
- git
- [ros-\*-desktop-full](http://wiki.ros.org/ROS/Installation)
  - kinetic or newer
- [mavros](http://wiki.ros.org/mavros)
- python
  - opencv_python
  - numpy
  - [gi, gobject](https://wiki.ubuntu.com/Novacut/GStreamer1.0)
  - PyYAML
  - pymavlink
  - mavproxy
- [freebuoyancy_gazebo](https://github.com/bluerobotics/freebuoyancy_gazebo)
- [ardupilot_gazebo](https://github.com/patrickelectric/ardupilot_gazebo)
- [sitl_gazebo](https://github.com/PX4/sitl_gazebo)
- [ardupilot](https://github.com/ArduPilot/ardupilot)


### Installation ###
The installation is in two steps:
 1. Installing all the Gazebo dependencies to make the Gazebo simulation working.
 2. Installing ROS related packages. 

#### Gazebo related ####
Assuming that ROS with Gazebo is installed, Gazebo needs to be updated from the OSRF:
```
sudo sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-latest.list'
wget http://packages.osrfoundation.org/gazebo.key -O - | sudo apt-key add -
sudo apt-get update
sudo apt-get install gazebo7
# For developers that work on top of Gazebo, one extra package
sudo apt-get install libgazebo7-dev
```
Then follow the installation instruction for the two Gazebo plugins mentioned in step 2. and 3.
[Running ArduSub SITL with Gazebo](https://www.ardusub.com/developers/sitl.html)

In addition, install the gstreamer plugin for Gazebo from [sitl_gazebo](https://github.com/PX4/sitl_gazebo), where instructions are reported. To ensure that the proper `Makefile` is generated, run

`$ cmake -DBUILD_GSTREAMER_PLUGIN=ON -DBUILD_ROS_INTERFACE=ON ..`

Clone also [ardupilot](https://github.com/ArduPilot/ardupilot) and use the proper version of ArduSub, e.g., beta.
```
git clone https://github.com/ArduPilot/ardupilot.git
cd ardupilot
git fetch && git fetch --tags
git checkout ArduSub-beta
git submodule update --init --recursive
```


#### ROS package ####
 1. Go to your ROS package source directory:
    - `$ cd ros_workspace_path/src`
 2. Clone this project.
    - `$ git clone https://github.com/patrickelectric/bluerov_ros_playground`
 3. Go back to your ROS workspace:
    - `$ cd ../`
 4. Build and install it:
    - `$ catkin_make --pkg bluerov_ros_playground`
    - if using ROS from source:
        - `$./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release --pkg bluerov_ros_playground`
 5. Reload your ROS env.
    - bash: `$ source devel/setup.sh`
    - zsh: `$ source devel/setup.sh`

## Running ##

- BlueRov2 node

    For more information check [here](src/bridge/README.md).

- Launch user example

    This example will start user example, reading data from mavlink, interacting with joystick over rc commands and showing ROV video stream.
    - `roslaunch bluerov_ros_playground user_mav.launch`

    To run QGC parallel with user_mav, it's necessary to run user_mav first. If it's necessary to change the input, the parameter `joy_dev` can be used to set the one used.

- Visualize video stream

    Shows video stream using opencv and gstreamer
    - `roslaunch bluerov_ros_playground video.launch`

    To capture video stream in user_mav/video and QGC at same time, it's necessary to modifie [gstreamer options](http://192.168.2.2:2770/camera), changing `! udpsink host=192.168.2.1 port=5600` to `! multiudpsink clients=192.168.2.1:5600,192.168.2.1:5601` and add the udp_port parameter when calling roslaunch (`video_udp_port:=5601`).

- Gazebo

    This example allow SITL communication with Gazebo, right now the only interaction that happen is the thruster control using [thruster pwm fitting](https://colab.research.google.com/notebook#fileId=1CEDW9ONTJ8Aik-HVsqck8Y_EcHYLg0zK).
    - Run SITL and start gazebo.launch
    - `roslaunch bluerov_ros_playground gazebo.launch`

    Another way of doing is (every line corresponds to a different terminal):
    - Start Gazebo (exporting properly all the paths -- note that the main folder is assumed to be the home directory)
      ```
      cd ~/catkin_ws/src/bluerov_ros_playground
      source gazebo.sh
      export GAZEBO_MODEL_PATH=~/ardupilot_gazebo/models:${GAZEBO_MODEL_PATH}
      export GAZEBO_RESOURCE_PATH=~/ardupilot_gazebo/worlds:${GAZEBO_RESOURCE_PATH}
      export SITL_GAZEBO_PATH=~/sitl_gazebo
      export GAZEBO_PLUGIN_PATH=${GAZEBO_PLUGIN_PATH}:~/sitl_gazebo/build
      gazebo worlds/underwater.world --verbose
      ```
    - Run the ArduPilot SITL
      ```
      cd ~/ardupilot/Tools/autotest 
      python sim_vehicle.py -v ArduSub -f gazebo-bluerov2 -L RATBeach --out=udp:0.0.0.0:14550 --map
      ```
    - Run the ROS package: `$ roslaunch bluerov_bringup bluerov_bringup.launch record:=0`

- Gazebo Teleop

    It'll open a window with the camera stream and Gazebo, a joystick can be used to control the ROV.
    - `roslaunch bluerov_ros_playground gazebo_teleop.launch`

    To change the default joystick input (`/dev/input/js0`), it's possible add the parameter `joy_dev:=/dev/input/jsX` when launching the simulation.

- RVIZ

    Visualize 3D model
    - `roslaunch bluerov_ros_playground rviz.launch`

### Running with SITL ###
- Run ArduPilot SITL

    1. [Download ArduPilot and configure SITL](http://ardupilot.org/dev/docs/setting-up-sitl-on-linux.html).
    2. `$ cd ardupilot/ArsuSub`
    3. `$ sim_vehicle.py`

    - To test mavros communication with SITL:
        - `$ roslaunch mavros apm.launch fcu_url:=udp://0.0.0.0:14550@`

- Fake video stream

    - `$ gst-launch-1.0 videotestsrc ! video/x-raw,width=640,height=480 ! videoconvert ! x264enc ! rtph264pay ! udpsink host=127.0.0.1 port=5600`
        - If necessary, change video source and resolution.
        - To test the udp reception: `gst-launch-1.0 -v udpsrc port=5600 ! application/x-rtp, payload=96 ! rtpjitterbuffer ! rtph264depay ! avdec_h264 ! fpsdisplaysink sync=false text-overlay=false`

## Topics ##
If you need more information about the topics and what you can access, take a look [here](doc/topics_and_data.md).

## Software Layer Diagram ##

<pre>
                      +-----------------------+         +------------------------+
                      |     <b>Raspberry Pi</b>      |         |    <b>Topside Commputer</b>   |
                      |    <b>ip 192.168.2.2</b>     |         |     <b>ip 192.168.2.1</b>     |
                      |                       |         |                        |
+-------+  Telemetry  | +-------------------+ |         |                        |
|Pixhawk<-------------->USB         <b>MAVProxy</b>| |         |                        |
+-------+    Pilot    | +                   + |         | +--------------------+ |
            Control   | |            udpbcast<----------->:14550         <b>MAVROS</b>| |
                      | +-------------------+ |  Pilot  | |(UDP)               | |
                      |                       | Control | |                    | |
                      | +-------------------+ |         | |       (ROS)        | |
+---------+           | CSI+2       <b>raspivid</b>| |         | +------+/mavros+-----+ |
|Raspberry+------------>camera              | |         |           ^            |
| Camera  |           | port                | |         |           |            |
+---------+           | +                   | |         | +---------v----------+ |
                      | |                   | |         | |subs.py      pubs.py| |
                      | +------------+stdout+ |         | |                    | |
                      |                  +    |         | |                    | |
                      |             Raw  |    |         | |                    | |
                      |             H264 |    |         | |                    | |
                      |                  v    |         | |      <b>user.py</b>       | |
                      | +------------+ fdsrc+ |         | |                    | |
                      | |<b>gstreamer</b>          | |         | |                    | |
                      | |                   + |         | :5600 video.py       | |
                      | |             udpsink+----------->(UDP)                | |
                      | +-------------------+ |  Video  | +---------^----------+ |
                      |                       | Stream  |           |            |
                      +-----------------------+         |           +            |
                                                        | +--------/joy--------+ |
                                                        | |<b>joy</b>     (ROS)       | |         +--------+
                                                        | |                  USB<----------+Joystick|
                                                        | +--------------------+ |  Pilot  +--------+
                                                        |                        | Control
                                                        +------------------------+
</pre>
