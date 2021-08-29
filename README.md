# youBot: Installation from Scratch

# Create a workspace

For our tutorial, we will create a workspace called **youbot_from_scratch**.

```
mkdir youbot_from_scratch
cd youbot_from_scratch
mkdir src
catkin build
```

# youBot drivers

## Installation

Go to [youBot Github page](https://github.com/youbot) and clone the **youbot_driver** repository into the src folder of the youbot_from_scratch workspace.

```
cd youbot_from_scratch
git clone <link>
```
## Build the package

```
cd youbot_driver
mkdir build
cd build
cmake ..
make
```
In order to find the location of the package, we will give it's location in the bashrc.

```
gedit ~/.bashrc
export YOUBOTDIR = ~/youbot_from_scratch/src/youbot_driver
```

# Ethernet port

We will set the name of our ethernet port in the configuration file of the youbot_driver

To check the name of our ethernet port, type in the terminal:

```
ifconfig
```

Now, go to the **youbot_ethercat.cfg** file that is present in the **config** folder of the **youbot_driver**
package.

```
cd youbot_driver/config
vim youbot_ethercat.cfg
-> EthernetDevice = <name_of_the_port> 
```

# youBot applications

## Installation

To check whether the drivers are installed properly, we will install **youbot_applications** package from [youBot Github page](https://github.com/youbot) in the src folder of the youbot_from_scratch workspace and try to run the **hello_world** application.

```
cd youbot_from_scratch
git clone <link>
```

## Build the package
```
cd youbot_applications
mkdir build
cd build
cmake ..
make
```
Also, export the ROS_PACKAGE_PATH in the bashrc:

```
gedit ~/.bashrc
-> export ROS_PACKAGE_PATH = ${ROS_PACKAGE_PATH}:<path_to_driver_folder>/youbot_driver:<path_to_applications_folder>/youbot_applications
source ~/.bashrc
```

## Run the application

To run the hello_world demo application, we have to run the binary file

```
cd youbot_applications/<name_of_the_application>/bin
sudo ./<name_of_the_binary>
```

# Configure the sensors

As in our case, we are using our laptop to configure the youBot instead of using the internal PC, we have to connect the sensors to the USB port of our laptop.

To check if the sensor is showing up, type the following command in the terminal:

```
dmesg
```
Now, check the port to which the sensor is connected to. In our case, its name was: **ttyACM0**.

Check if the following command works

```
od /dev/<port_name>
```
if not check

```
sudo od /dev/<port_name>
```
if it works add yourself to the dialout group and resatrt your system

```
sudo adduser <YOUR_USERNAME> dialout
groups   (check if your username appears)
```

## Write rules for the sensor

We will first find the ID of the sensor

```
rosrun urg_node getID /dev/<port_name>
```

**NOTE:** This ID keeps on changing, so we will update the rules to get a consistent ID.

```
cd ./etc/udev/rules.d
sudo vim <name_of_the_rule>
```
copy rules from the roswiki page of urg_node (change the version according to your machine)



