---
layout: post
title: Connecting a Raspberry PI 4 to an iPad Pro
author: FX
---
While browsing the Internet I stumbled on the [video](https://youtu.be/IR6sDcKo3V8) from the Tech Craft channel on how to connect a Raspberry Pi 4 to an iPad Pro. I've been using my iPad Pro 12"9 inches for a bit more than a year now and still trying to perform as much as possible from it, with the ultimate goal of using it as a full replacement for my laptop.

Do not get me wrong, I love the form factor, the light weight and the battery duration. Still I was struggling to perform the technical experiments I was looking for. For example I started to go through the last book from Adrian Rosebrock [Raspberry Pi for computer vision](https://www.pyimagesearch.com/raspberry-pi-for-computer-vision/). But to play with the examples you need a working Python environment with OpenCV and Numpy installed. This is not possible on the iPad. I installed [Pythonista for iOS](http://omz-software.com/pythonista/) but OpenCV is not included.

I tried to follow the instructions from the various pages in reference but there has been some slight amendments required to make it work fully.

In addition I wanted to secure the Raspberry with private/public key authentication, connection through mosh instead of SSH, setting-up NTP and more globally finalizing the set-up on the Pi side before moving on.

This page puts together my notes to achieve this result. I plugged my Raspberry Pi 4 on an external screen and keyboard & mouse and set up up directly from the terminal.

First you have to install [Raspbian](https://www.raspberrypi.org/downloads/raspbian/) on the Raspberry Pi and plug it in. I won’t provide the details on how to flash the SD card and perform the initial installation when booting the Pi for the first time (set up WIFI, timezone & location).

Considering that this set up has been possible thanks to a recent update to the Raspberry Pi 4 bootloader - it now not only enables the low power mode for USB hardware but also allows the enabling of Network boot and enables data over the USB-C port - I recommend setting up your Pi so that it automatically updates the boot loader: this means you will get new features and bug fixes as they are released.

Bootloader updates are performed by the ``rpi-eeprom`` packages which installs a service that runs at boot-time to check for critical updates.

```
sudo apt update
sudo apt full-upgrade
sudo apt install rip-eeprom
```

If you wish to control then the updates are applied you can disable the systems service from running automatically and run ``reset-eeprom-update`` manually.

```
# Prevent the service from running, this can be run before the
# package is installed to prevent it ever running automatically.
sudo system to mask rpi-eeprom-update

# Enable it again
sudo systemctl unmask rpi-eeprom-update
```

With this enabled all the same script from the Pi Zero’s should just work but here is the updated version for Raspbian Buster.

Add ``dtoverlay=dwc2`` to the ``/boot/config.txt``
Add ``modules-load=dwc2`` to the end of ``/boot/cmdline.txt``
If you have not already enabled ssh then create a empty file called ``ssh`` in ``/boot``
Add ``libcomposite`` to ``/etc/modules``
Add ``denyinterfaces usb0`` to ``/etc/dhcpcd.conf``
Install dnsmasq with ``sudo apt-get install dnsmasq``
Create ``/etc/dnsmasq.d/usb`` with following content

```
interface=usb0
dhcp-range=10.55.0.2,10.55.0.6,255.255.255.248,1h
dhcp-option=3
leasefile-ro
```

Create ``/etc/network/interfaces.d/usb0`` with the following content

```
auto usb0
allow-hotplug usb0
iface usb0 inet static
  address 10.55.0.1
  netmask 255.255.255.248
```

Create ``/root/usb.sh``

```
#!/bin/bash
cd /sys/kernel/config/usb_gadget/
mkdir -p pi4
cd pi4
echo 0x1d6b > idVendor # Linux Foundation
echo 0x0104 > idProduct # Multifunction Composite Gadget
echo 0x0100 > bcdDevice # v1.0.0
echo 0x0200 > bcdUSB # USB2
echo 0xEF > bDeviceClass
echo 0x02 > bDeviceSubClass
echo 0x01 > bDeviceProtocol
mkdir -p strings/0x409
echo "fedcba9876543211" > strings/0x409/serialnumber
echo "Ben Hardill" > strings/0x409/manufacturer
echo "PI4 USB Device" > strings/0x409/product
mkdir -p configs/c.1/strings/0x409
echo "Config 1: ECM network" > configs/c.1/strings/0x409/configuration
echo 250 > configs/c.1/MaxPower
# Add functions here
# see gadget configurations below
# End functions
mkdir -p functions/ecm.usb0
HOST="00:dc:c8:f7:75:14" # "HostPC"
SELF="00:dd:dc:eb:6d:a1" # "BadUSB"
echo $HOST > functions/ecm.usb0/host_addr
echo $SELF > functions/ecm.usb0/dev_addr
ln -s functions/ecm.usb0 configs/c.1/
udevadm settle -t 5 || :
ls /sys/class/udc > UDC
ifup usb0
service dnsmasq restart
```

Make ``/root/usb.sh`` executable with ``chmod +x /root/usb.sh``
Add ``sh /root/usb.sh`` to ``/etc/rc.local`` before ``exit 0`` (I really should add a systemd startup script here at some point)

With this setup the Pi4 will show up as a ethernet device with an IP address of 10.55.0.1 and will assign the device you plug it into an IP address via DHCP. This means you can just ssh to pi@10.55.0.1 to start using it.

# Credits
- [Ben Hardill](https://www.hardill.me.uk/wordpress/about/): [PI-4 USB-C Gadget](https://www.hardill.me.uk/wordpress/2019/11/02/pi4-usb-c-gadget/)
- [Connect your Raspberry Pi 4 to an IPad Pro](https://www.raspberrypi.org/blog/connect-your-raspberry-pi-4-to-an-ipad-pro/)
- [My favorite Ipad Pro accessory: the Raspberry Pi 4](https://youtu.be/IR6sDcKo3V8)
- [RASPBERRY PI ZERO – PROGRAMMING OVER USB! (PART 2)](https://blog.gbaman.info/?p=791)