# youBot_from_scratch

This repository contains the required files and description of the process to perform autonomous navigation using a KUKA youBot, controlled via an external PC/Laptop. This work was performed at RoboCup@Work Lab in Hochschule Bonn-Rhein-Sieg. 

## Pre-requisites

* A KUKA youBot

* A Laptop with Ubuntu v18.04 and ROS Melodic Morenia

## Steps to follow

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
 

### Configuring KUKA youBot drivers

* Now, we'll install the youBot drivers and test-run the youBot:

  - Build the youbot_driver package:
  
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
  
  - Now, navigate to the youbot_application package and build it using:
  
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

### Configuring Hokuyo Lidar Sensor

* Now, we'll integrate the Hokuyo Lidar Sensor with the external laptop:

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
    
    - if it works with super user permission, then add yourself to the dialout group:
    
    ```
    sudo adduser <YOUR_USERNAME> dialout
    ```
    
    - Log out and log back in. Check if you're in the `dialout` group:
    
    ```
    groups
    ```
  
  - Now get the device ID/Sensor ID:
  
  ```
  rosrun urg_node getID /dev/ttyACM0 --
  
  ```
  
  - Now we'll write udev rules to give Hokuyo consistent device names:
  
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
  ---
  ToDo: mapping, localization, navigation
