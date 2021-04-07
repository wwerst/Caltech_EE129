# ROS for Raspberry Pi 4 Tutorial

## Initial Pi Setup (summary)

First, get Raspbian OS Full installed on your Pi. To boot headless, you will want to do the following on boot partition of the SD card:

1) Create a file called ssh `touch ssh`. This enables ssh on the Pi.
2) Follow instructions here (https://www.raspberrypi.org/documentation/configuration/wireless/headless.md) for setting up wifi
3) Modify `/boot/config.txt` relevant lines to match the following:

```
# uncomment if hdmi display is not detected and composite is being output
hdmi_force_hotplug=1

# uncomment to force a specific HDMI mode (this will force VGA)
hdmi_group=2
hdmi_mode=82
```
4) Plug the SD card in and boot the Pi.
5) SSH into pi@raspberrypi. Then, go to raspi-config and enable the VNC interface
6) VNC into the raspberrypi using RealVNC VNC Viewer.
7) Go through the initial graphical setup on the Pi.
8) Do a full-upgrade of system packages: `sudo apt update && sudo apt full-upgrade`

## ROS System Installation

This guide is based off of http://wiki.ros.org/ROSberryPi/Installing%20ROS%20Melodic%20on%20the%20Raspberry%20Pi

### Setup ROS Repositories

```
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
```

### Install Bootstrap Dependencies

```
sudo apt install -y python-rosdep python-rosinstall-generator python-wstool python-rosinstall build-essential cmake
```

### Initialize rosdep

This initializes the ros package manage and updates its dependencies. It uses apt, and functions similar to apt. Note that you should only use `sudo` for the first command, `sudo rosdep init`, and never use `sudo` for the second command, `rosdep update`. This causes permissions issues later if you use `sudo` for the second command. For more info, see: http://wiki.ros.org/rosdep 

```
sudo rosdep init
rosdep update
```

### Create ROS system workspace

This creates the workspace which we will use for installing system-wide packages:

```
mkdir -p ~/ros_catkin_ws
cd ~/ros_catkin_ws
```

### Clone the source code for ROS

We can pick between two different metapackages to clone. A metapackage is a package which only contains dependencies on other packages. This allows installing of collections of packages with a single package name. Installing `ros_comm` will install a minimal set of packages for using ROS from the command line. It takes about 20 minutes to build and install on the Pi, but many graphical packages that may be useful, such as RViz, will not be installed. Installing `desktop` will install additional graphical tools that will be useful, however this will take about 2 hrs to install. See (http://wiki.ros.org/ROSberryPi/Installing%20ROS%20Melodic%20on%20the%20Raspberry%20Pi#Create_a_catkin_Workspace) for more details.

```
rosinstall_generator desktop --rosdistro melodic --deps --wet-only --tar > melodic-desktop-wet.rosinstall
wstool init src melodic-desktop-wet.rosinstall -j4          # j4 option clones four repos in parallel, for faster cloning.
```

The first command creates a rosinstall file, which describes data sources to retrieve code from. The second command clones all of those data sources into the `src` folder. These data sources can be git repos, svn repos, or even just urls to download tar files from. Lets take a look at the rosinstall file generated from above:

```
pi@raspberrypi:~/ros_catkin_ws $ cat melodic-desktop-wet.rosinstall 
- tar:
    local-name: actionlib
    uri: https://github.com/ros-gbp/actionlib-release/archive/release/melodic/actionlib/1.12.1-1.tar.gz
    version: actionlib-release-release-melodic-actionlib-1.12.1-1
- tar:
    local-name: angles
    uri: https://github.com/ros-gbp/geometry_angles_utils-release/archive/release/melodic/angles/1.9.12-1.tar.gz
    version: geometry_angles_utils-release-release-melodic-angles-1.9.12-1
- tar:
    local-name: bond_core/bond
    uri: https://github.com/ros-gbp/bond_core-release/archive/release/melodic/bond/1.8.5-1.tar.gz
    version: bond_core-release-release-melodic-bond-1.8.5-1
- tar:
    local-name: bond_core/bond_core
    uri: https://github.com/ros-gbp/bond_core-release/archive/release/melodic/bond_core/1.8.5-1.tar.gz
    version: bond_core-release-release-melodic-bond_core-1.8.5-1
- tar:
    local-name: bond_core/bondcpp
    uri: https://github.com/ros-gbp/bond_core-release/archive/release/melodic/bondcpp/1.8.5-1.tar.gz
    version: bond_core-release-release-melodic-bondcpp-1.8.5-1
- tar:
    local-name: bond_core/bondpy
    uri: https://github.com/ros-gbp/bond_core-release/archive/release/melodic/bondpy/1.8.5-1.tar.gz
    version: bond_core-release-release-melodic-bondpy-1.8.5-1
- tar:
    local-name: bond_core/smclib
    uri: https://github.com/ros-gbp/bond_core-release/archive/release/melodic/smclib/1.8.5-1.tar.gz
    version: bond_core-release-release-melodic-smclib-1.8.5-1
```

And now, if you look in the `src` folder, you will see those folders:

```
pi@raspberrypi:~/ros_catkin_ws $ ls -la src/
total 424
drwxr-xr-x 95 pi pi  4096 Apr  7 12:24 .
drwxr-xr-x  5 pi pi  4096 Apr  7 12:32 ..
drwxr-xr-x  8 pi pi  4096 Apr  7 12:22 actionlib
drwxr-xr-x  5 pi pi  4096 Apr  7 12:22 angles
drwxr-xr-x  7 pi pi  4096 Apr  7 12:22 bond_core
drwxr-xr-x  7 pi pi  4096 Apr  7 12:22 catkin
drwxr-xr-x  8 pi pi  4096 Apr  7 12:22 class_loader
drwxr-xr-x  4 pi pi  4096 Apr  7 12:22 cmake_modules
drwxr-xr-x 12 pi pi  4096 Apr  7 12:22 common_msgs
...
```

### Install package dependencies

See http://wiki.ros.org/ROSberryPi/Installing%20ROS%20Melodic%20on%20the%20Raspberry%20Pi#Resolve_Dependencies


```
cd ~/ros_catkin_ws && rosdep install -y --from-paths src --ignore-src --rosdistro melodic -r --os=debian:buster
```

### Build and Install ROS

See http://wiki.ros.org/ROSberryPi/Installing%20ROS%20Melodic%20on%20the%20Raspberry%20Pi#Building_the_catkin_Workspace

```
sudo ./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release --install-space /opt/ros/melodic -j4
```

### Adding additional packages

See http://wiki.ros.org/ROSberryPi/Installing%20ROS%20Melodic%20on%20the%20Raspberry%20Pi#Maintaining_a_Source_Checkout

This will add packages to the system install in `/opt/ros/melodic`. It also takes care of fetching the dependencies for the package
you wish to install. It does this by creating a new rosinstall file for the package you wish to install and its dependencies, then
merging those dependencies with the current install.

### Simple demo of Rosinstall with wstool

Wstool documentation can be found here: http://wiki.ros.org/wstool

In order to avoid repeatedly typing ssh password, you may want to start an ssh-agent:

```
eval $(ssh-agent -s)
ssh-add
```

Clone main repo that contains the rosinstall file:

```
cd ~
mkdir -pv ~/demo_ws/src && cd demo_ws/src
git clone git@github.com:wwerst/caltech_ee129.git
```

Now, we use `wstool init` to initialize our project with all of the relevant repos. This command will clone our support repo:

```
cd ~/demo_ws/src
wstool init . caltech_ee129/demo.rosinstall
```

