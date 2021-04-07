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

## ROS Installation

This guide is based off of http://wiki.ros.org/ROSberryPi/Installing%20ROS%20Melodic%20on%20the%20Raspberry%20Pi

