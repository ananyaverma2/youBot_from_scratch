# youBot_from_scratch

This repository offers a detailed description to perform autonomous navigation using a KUKA youBot controlled via an external Laptop. This work was performed at RoboCup@Work Lab in Hochschule Bonn-Rhein-Sieg. 

  
# Motivation

The intention behind performing this work is to get in-depth knowledge of `How to configure a robot with ROS from scratch`. For this purpose, we have used a **KUKA youBot** and an **external Laptop**. 

# Pre-requisites

* A KUKA youBot

* A laptop with the following specifications:
  * Ubuntu v18.04 
  * ROS Melodic Morenia

# Steps to follow

## Create a catkin workspace

* First create a workspace, we call it `youBot_from_scratch` and then build it.

```
mkdir youBot_from_scratch
cd youbot_from_scratch
mkdir src
catkin build
cd src
```

* Inside `src`, we now need to clone the following packages:

  - youbot_driver: `git clone https://github.com/youbot/youbot_driver.git`
  
  - youbot_description: `git clone https://github.com/youbot/youbot_description.git`
  
  - youbot_driver_ros_interface: `git clone https://github.com/youbot/youbot_driver_ros_interface.git`
  
  - youbot_applications: `git clone https://github.com/youbot/youbot_applications.git`
  
  
## Configuring Hardware

> Note: Power the youBot and make sure to turn-off the internal PC.

### KUKA youBot drivers

* We'll install the youBot drivers and test-run the youBot:

  - Build the `youbot_driver` package:
  
  ```
  cd youbot_driver
  mkdir build
  cd build
  cmake ..
  make
  ```
  
  - We need to edit some configuration files in `youbot_driver` package: 
  
    - Connect the ethernet cable from KUKA youBot to your Laptop
    
    - Check your PC's Ethernet device ID using `ifconfig` (It is usually something like eth0).
    
    - Go to `config` folder inside `youbot_driver` package, and open `youbot-ethercat.cfg` in edit-mode and change`EthernetDevice = <Your Ethernet Device ID>`
  
  - Navigate to the `youbot_application` package and build it using:
  
  ```
  cd youbot_applications
  mkdir build
  cd build
  cmake ..
  make
  ```
  
  - Export the respective package paths so that it can be discovered by other packages in ROS:
  
  ```
  echo "export ROS_PACKAGE_PATH=\$ROS_PACKAGE_PATH:/home/<user>/youBot_from_scratch/src/youbot_driver:\
  /home/<user>/youBot_from_scratch/src/youbot_applications" >> ~/.bashrc
  ```
  
  > Note: Change the <user> tag to your default system user and change the workspace name if different.
  
  - Now we'll check if the drivers are installed successfully by running a demo script.
    
    ```
    cd youbot_applications/hello_world_demo/bin
    sudo ./<name_of_binary_file>
    ```

### Hokuyo lidar sensor

* We'll integrate the Hokuyo Lidar Sensor with our external laptop:

  > Note: Since we are using an external laptop instead of KUKA youBot's internal PC, we need to connect the Hokuyo Lidar sensors to the USB port of the laptop. 

  - Connect the sensor to the Laptop and check if the sensor is found by typing `dmesg` in a terminal window. Keep a note of its port ID (something like ttyACM0)
  
  - Check if you have the rights to open the port:
  
  ```
  od /dev/<port ID>
  ```
  
    - If it doesn't work, try it with super user permission:
    
    ```
    sudo  od /dev/<port ID>
    ```
    
    - if it works with super user permission, then add yourself to the `dialout` group:
    
    ```
    sudo adduser <YOUR_USERNAME> dialout
    ```
    
    - Log out and log back in. Check if you're in the `dialout` group:
    
    ```
    groups
    ```
  
  - Get the device ID/Sensor ID:
  
  ```
  rosrun urg_node getID /dev/ttyACM0 --
  
  ```
  
  - Write udev rules to give Hokuyo consistent device names:
  
    ```
    cd /etc/udev/rules.d
    ```
    
    - Create a following rule file:
    
    ```
    sudo vim 80-youBot.rules
    ```
    
    - Paste the following rule and save it:
    
    ```
    SUBSYSTEMS=="usb", KERNEL=="ttyACM[0-9]*", ACTION=="add", ATTRS{idVendor}=="15d1", ATTRS{idProduct}=="0000", MODE="666",      PROGRAM="/opt/ros/melodic/lib/urg_node/getID /dev/%k q", SYMLINK+="sensors/hokuyo_%c", GROUP="dialout"
    ```
  
## Running youBot with ROS

- Create a catkin package inside the `src` :
  
  ```
  catkin_create_pkg youBot std_msgs roscpp rospy
  ```
  
### Create youBot driver node
  
- In order to work with youBot, we first need to fire up the youBot drivers:
  
  - Create a launch file `youBot_driver.launch` in the `launch` directory of the `youBot` package and paste the following code:
  
  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <!launch>
    <include file="$(find youbot_driver_ros_interface)/launch/youbot_driver.launch" />
  </launch>
  ```
  > Note: The youBot driver launch file is already provided with the `youbot_driver_ros_interface` package. Here, we are simply calling that launch script.
  
### Create youBot Hokuyo sensor node
  
- To visualize the output of Hokuyo Lidar sensor:

  - Create a launch file `youBot_sensor.launch` in the launch directory of the `youBot package` and paste the following code:
  
  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <launch>
    <!-- Front Laser scanner-->
    <node name="hokuyo_node_1" pkg="urg_node" type="urg_node" respawn="false" output="screen">
    <param name="port" type="string" value="/dev/sensors/hokuyo_H1102594"/>
    <param name="intensity" type="bool" value="false"/>
    <param name="min_ang" value="-2.356194437"/>
    <param name="max_ang" value="2.35619443"/>
    <param name="cluster" value="1"/>
    <param name="frame_id" value="base_laser_front_link"/>
    </node>
    <!-- Back Laser scanner-->	
    <node name="hokuyo_node_2" pkg="urg_node" type="urg_node" respawn="false" output="screen">
    <param name="port" type="string" value="/dev/sensors/hokuyo_H1838912"/>
    <param name="intensity" type="bool" value="false"/>
    <param name="min_ang" value="-2.356194437"/>
    <param name="max_ang" value="2.35619443"/>
    <param name="cluster" value="1"/>
    </node> 
    <!-- Launching RViz -->
    <node name="rviz" pkg="rviz" type="rviz"/>
  </launch>
  ```
  
  > Note: 1. The value of the parameter `port` needs to be change according to your sensor ID (you can find it by navigating to /dev/sensors/hokuyo_xxxxxxx), 2. Depending upon whether you have 1 or 2 hokuyo sensors, you can edit the launch file. 


### Create youBot urdf node

- To import the youBot in RViz, 
  
  - Create a launch file `youBot_urdf.launch` in the launch directory of the `youBot_package` and paste the following code:
  
  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <launch>
	  <!-- Launching urdf -->
	  <param name="robot_description" command="$(find xacro)/xacro --inorder '$(find youbot_description)/robots/youbot.urdf.xacro'" />    
	  <!-- Start TF publishers -->		  
	  <node pkg="robot_state_publisher" type="robot_state_publisher" name="robot_base_state_publisher" output="screen" />
  </launch>
  ```
  
### Create youBot teleop node

- In order to move the youBot around in the workspace, we need to launch teleop package. We'll be using a Joystick controller to move the robot around. 

  - Create a launch file `youBot_teleop.launch` in the launch directory of the `youBot_package` and paste the following code:
  
  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <launch>
	  <node pkg="joy" type="joy_node" name="joystick" output="screen"/>
	  <node pkg="teleop_twist_joy" type="teleop_node" name="teleop" output="screen">
	  <param name="enable_button" value="5"/>
	  </node>
  </launch>
  ```
  
  > Note: The parameter `enable_button` allows to enable regular-speed movement. We set its value as `5` because it is mapped with the `L1` Key in Joystick and we use it as a standard key to enable regular-speed movement in all of our KUKA youBot (We have plenty of them!). 
  
### Create SLAM gmapping node

> Note: In order to do Simultaneous Localization And Mapping, we need to launch the `gmapping` package in ROS. 
  
- Create a launch file `youBot_mapping.launch` in the launch directory of the `youBot_package` and paste the following code:

```
<?xml version="1.0" encoding="UTF-8"?>
<launch>
  <node name="slam_gmapping" pkg="gmapping" type="slam_gmapping">
    <remap from="/scan" to="/scan"/>
    <param name="base_frame" value="base_footprint"/>
  </node>
</launch>
```
 
#### Mapping 'the Arena'

  - Build the catkin workspace:
  
  ```
  catkin build
  ```

  - To start mapping, we need to launch the following launch-files:
  ```
    `roslaunch youBot youBot_drivers.launch`
    `roslaunch youBot youBot_urdf.launch`
    `roslaunch youBot youBot_sensor.launch`
    `roslaunch youBot youBot_teleop.launch`
    `roslaunch youBot youBot_mapping.launch`
  ```

  - Move the youBot around cautiously using the Joystick controller and create a map of the whole arena. 

  - After you are done with mapping, you can simply save the map by doing `Ctrl+C` in the terminal where you have launched `youBot_mapping` node. The map files will be saved in your package.

  - It's a good practise, to create a `map` directory inside the `youBot` package and move both the newly generated files there `<map>.yaml` and `<map>.pgm`.

### Creating AMCL Node

> Note: In order to do localization, we need to launch the `amcl` package in ROS. 
  
- Create a launch file `youBot_localization.launch` inside the launch directory of the `youBot` package and paste the following code:
  
  ```
  <launch>
    <!-- MAP Server Node-->
    <arg name="map_file" default="$(find youbot)/map/test_env.yaml"/>
    <node name="map_server" pkg="map_server" type="map_server" args="$(arg map_file)" />
    <!-- AMCL Node -->
    <node name="amcl" pkg="amcl" type="amcl" output="screen">
        <remap from="scan" to="/scan"/> <!--according to package-->
        <param name="odom_frame_id" value="odom"/>
        <param name="odom_model_type" value="omni"/>
        <param name="base_frame_id" value="base_footprint"/> <!--according to package-->
        <param name="global_frame_id" value="map"/>
        <!-- If you choose to define initial pose here -->
        <param name="initial_pose_x" value="0"/>
        <param name="initial_pose_y" value="0"/>
        <param name="initial_pose_a" value="0"/>
    </node> 
  </launch>
  ```
  
- To start localization, we need to launch the following launch-files:
  
  ```
   `roslaunch youBot youBot_drivers.launch`
   `roslaunch youBot youBot_urdf.launch`
   `roslaunch youBot youBot_sensor.launch`
   `roslaunch youBot youBot_teleop.launch`
   `roslaunch youBot youBot_localization.launch`
  ```  
  - You can move the youBot around and visualize the localization of the youBot in the RViz window by subscribing to appropriate topics.

  ---
  Todo: Relevant images, rviz config file, navigation 

  
