# NatNet 4 ROS driver

This is a motion capture system with crazyflie robot.

## Requirements
This package is only tested with the Natnet 4.0 and ROS noetic but probably will work with the older versions of both as well. 

## Crazyflie Mocap Pipeline
Natnet ROS was used as part of crazyflie drone position tracking & broadcasting in 3D. A lander was created whose position was also tracked by the mocap with sufficient accuracy. The Windows PC and the Ubuntu PC need to be connected to the same LAN, so, the Ubuntu PC needs a wired connection to the LAN to get the position stream from the Windows computer. Also, after the Wired connection, some settings of the wired connection need to be changed. 

<div align="center">
  <img src="img/mocap_broadcast.png" width="600">
</div>

## Setup Motiv
This package contains a ROS driver for the NatNet protocol used by the OptiTrack motion capture system. It supports NatNet versions 4.0 (Motive 2.2 and higher). The NatNet SDK provided by the optitrack can be found [here](https://optitrack.com/support/downloads/developer-tools.html#natnet-sdk). It will be downloaded under `deps/NatnetSDK` while building it for the first time. NatNet protocol is used for streaming live motion capture data (rigid bodies, skeletons etc) across the shared network. 

### Current Features:
  
 - Stable and with more functionality than [mocap_optitrack](https://github.com/ros-drivers/mocap_optitrack)
 - Rigid bodies are published as `geometry_msgs/PoseStamped` under name given in the Motive, i.e `/natnet_ros/<body-name>/pose`. Plus those are also broadcasting as `tf` frame for rviz
 - Markers of the rigid bodies are published ad `geometry_msgs/PointStamped` unuder the name `/natnet_ros/<body-name>/marker#/pose`
 - Unlabeled markers with the initial position and the name mentione in the `/config/initiate.yaml`are published as `geometry_msgs/PoseStamped` unuder the name `/natnet_ros/<name-in-config-file>/pose`. Plus those are also broadcasting as `tf` frame for rviz. The marker position is updated based on Iterative closest point (nearest neighbour)
 - Unlabled markers can be also published as `sensor_msgs/PointCloud`
 - Different options for publishing and logging the data

### Build the package
Install the requirements:
```
sudo apt install -y ros-$ROS_DISTRO-tf2* wget
```
Build the package:
```
cd ~/catkin_ws/src
git clone https://github.com/L2S-lab/natnet_ros_cpp
cd ..
catkin_build  
source devel/setup.bash
roslaunch natnet_ros_cpp natnet_ros.launch
```

### Setup the Motive for this package
- Disable the firewall for the network on which the data is being published.
- Open the Motive app. 
- In the motive app, open the streaming panel.
- Disable the other streaming Engines like VRPN, Trackd etc.
- Under the OptiTrack Streaming Engine, turn on the Broadcast Frame.
- Select the correct IP address in the Local Interface.
- Select the Up Axis as Z.

Here is an example of how your streaming settings should look.

![alt text](https://github.com/L2S-lab/natnet_ros_cpp/blob/noeitc/img/streaming.png)

### Understanding the launch file
Launch file `natnet_ros.launch` contains the several configurable arguments. The details are mentioned in the launch file. Following are several important argument for the connection and the data transfer. Other connection arguments are for the advanced option.

- `serverIP` : The IP address of the host PC. (The one selected in the Local Interface in Motive app)
- `clientIP` : The IP address of the PC on which the file will be launched
- `serverType` : Two possible options, `multicast` and `unicast`

### Publishing the single marker 
It is possible to track the single marker as a rigid body with constant orientation. Go to the `config/initiate.yaml` It is suggested to make a copy of the file and rename the new file.
The file contains the details on what to modify. 

The question might arise on how to check the position of the single marker. For that, you can log the frames of the incoming data in the terminal. To do so, enable the `log_frames` in the launch file.

After configuring the `initiate.yaml`, in the launch file, enable the `pub_individual_marker`. Change the name of the config file in the argument `conf_file` if needed and launch the file.

#### Replacing existing package
You can easily replace the current package with this package. In the `natnet_ros.launch` change the name of node to the node you currently using. For an example, 
If you are using the `vrpn_client_node`
changes are following

```
<node pkg="natnet_ros_cpp" type="natnet_ros_cpp" name="vrpn_client_node" output="screen" >   
```

## Resources
- Data Streaming [[website]](https://v22.wiki.optitrack.com/index.php?title=Data_Streaming)
- mocap4ros2-setup [[website]](https://docs.optitrack.com/robotics/mocap4ros2-setup)

