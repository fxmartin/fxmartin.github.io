---
layout: post
title: Installing and configuring apcupsd (UPS Management Software) - Part 1
author: FX
---
My PI3 cluster is connected to an APC SmartUPS using a usb cable and I would like to detect a power failure. If power is not restored the server must shutdown when the battery is exhausted. How do I configure and use my APC SmartUPS under Debian operating system for power management?

The purpose of this blog is to explain how I answered the above questions.

![APC Back-UPS 950](/images/2016-09-02-Installing-and-configuring-apcupsd-part-1.jpg)

Linux comes with GPL licensed open source apcupsd server ( daemon ) that can be used for power management and controlling most of APC’s UPS models on Linux, BSD, Unix and MS-Windows operating systems. Apcupsd works with most of APC’s Smart-UPS models as well as most simple signalling models such a Back-UPS, and BackUPS-Office. During a power failure, apcupsd will inform the users about the power failure and that a shutdown may occur. If power is not restored, a system shutdown will follow when the battery is exhausted, a timeout (seconds) expires, or runtime expires based on internal APC calculations determined by power consumption rates.

# Step 1 - Check that the UPS is connected

Enter ```lsusb```to check that the UPS is recognized. In my case it appears as ```Device 006``` in the output below.

```
Bus 001 Device 006: ID 051d:0002 American Power Conversion Uninterruptible Power Supply
Bus 001 Device 005: ID 0658:0200 Sigma Designs, Inc.
Bus 001 Device 004: ID 0b95:772a ASIX Electronics Corp. AX88772A Fast Ethernet
Bus 001 Device 003: ID 0424:ec00 Standard Microsystems Corp. SMSC9512/9514 Fast Ethernet Adapter
Bus 001 Device 002: ID 0424:9514 Standard Microsystems Corp.
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

# Step 2 - Install Apcupsd

Type the following command under Debian / Ubuntu Linux install apcupsd software: ```sudo apt-get install apcupsd```.

In ```/etc/default/apcupsd``` change ```ISCONFIGURED``` from no to yes:

```sed -i -e "s/ISCONFIGURED=no/ISCONFIGURED=yes/g" /etc/default/apcupsd```

# Step3 - Configure apcupsd

The **apcuspd** daemon is configured through one single file: ```/etc/apcupsd/apcupsd.conf```. This standard installation comes with a default set-up, which only needs minimal tweaks.

First, give your UPS a name. This is particulary useful if you have multiple UPSes. This does not set the EEPROM. It should be 8 characters or less:

```UPSNAME mainups```

Next, defines the type of cable connecting the UPS to your Linux server. In this example, I’m connecting my UPS using usb:

```UPSCABLE usb```

Possible generic choices for cable are – simple, smart, ether, and usb. For USB UPSes, please leave the DEVICE directive blank. For
other UPS types, you must specify an appropriate port or address (see config file for detailed information):

```
## set ups type to usb ##
UPSTYPE usb
DEVICE
```

The below commands will perform these changes automatically:

```
sed -i -e "s/#UPSNAME/UPSNAME mainups/g" /etc/apcupsd/apcupsd.conf
sed -i -e "s/UPSCABLE smart/UPSCABLE usb/g" /etc/apcupsd/apcupsd.conf
sed -i -e "s/UPSTYPE apcsmart/UPSTYPE usb/g" /etc/apcupsd/apcupsd.conf
sed -i -e "s_DEVICE /dev/ttyS0_DEVICE_g" /etc/apcupsd/apcupsd.conf
```

## Configuration parameters used during power failures

I personally sticked to the default values.

The ONBATTERYDELAY directive defined the time in seconds from when a power failure is detected until we react to it with an onbattery event:

```ONBATTERYDELAY 6```

If during a power failure, the remaining battery percentage (as reported by the UPS) is below or equal to BATTERYLEVEL, **apcupsd** will initiate a Linux system shutdown:

```BATTERYLEVEL 5```

If during a power failure, the remaining runtime in minutes (as calculated internally by the UPS) is below or equal to MINUTES, **apcupsd**, will initiate a system shutdown.

```MINUTES 3```

If during a power failure, the UPS has run on batteries for TIMEOUT many seconds or longer, **apcupsd** will initiate a system shutdown. A value of 0 disables this timer:

```TIMEOUT 0```

*Note: If you have an older dumb UPS, you will want to set this to less than the time you know you can run on batteries.*

## Configuration of apcupsd network information server

There are two major ways of running **apcupsd** on your system. The first is a standalone configuration where **apcupsd** controls a single UPS, which powers the computer. This is the most common configuration. The second configuration is a master/slave configuration, where one UPS powers several computers, each of which runs a copy of **apcupsd**. The computer that controls the UPS is called the master, and the other computers are called slaves. The master copy of **apcupsd** communicates with and controls the slaves via an ethernet connection.

You can use CGI or GUI programs from remote system to get information about your UPS. Turn on network information server:

```NETSERVER on```

For the master node as explained above for the second configuration, use the loopback address (127.0.0.1) to accept connections only from the local machine (default).

```NISIP 127.0.0.1```

The default port is set to 3551 for sending STATUS and EVENTS data over the network:

```NISPORT 3551```

# Step 4 - Test acpupsd

Type the following command: ```sudo apctest```. You should get an output like below:

```
2016-09-02 18:10:01 apctest 3.14.12 (29 March 2014) debian
Checking configuration ...
sharenet.type = Network & ShareUPS Disabled
cable.type = USB Cable
mode.type = USB UPS Driver
Setting up the port ...
Doing prep_device() ...

You are using a USB cable type, so I'm entering USB test mode
Hello, this is the apcupsd Cable Test program.
This part of apctest is for testing USB UPSes.

Getting UPS capabilities...SUCCESS

Please select the function you want to perform.

1)  Test kill UPS power
2)  Perform self-test
3)  Read last self-test result
4)  View/Change battery date
5)  View manufacturing date
6)  View/Change alarm behavior
7)  View/Change sensitivity
8)  View/Change low transfer voltage
9)  View/Change high transfer voltage
10) Perform battery calibration
11) Test alarm
12) View/Change self-test interval
 Q) Quit

Select function number:
```

You can test your ups or read test results.

To see the current status of your UPS enter the following command: ```apcaccess```. You should an output similar to the one below:

```
APC      : 001,036,0866
DATE     : 2016-09-02 16:22:51 +0000
HOSTNAME : swarm-master
VERSION  : 3.14.12 (29 March 2014) debian
UPSNAME  : MAINUPS
CABLE    : USB Cable
DRIVER   : USB UPS Driver
UPSMODE  : Stand Alone
STARTTIME: 2016-09-02 16:17:03 +0000
MODEL    : Back-UPS XS 950U
STATUS   : ONLINE
LINEV    : 230.0 Volts
LOADPCT  : 11.0 Percent
BCHARGE  : 100.0 Percent
TIMELEFT : 50.4 Minutes
MBATTCHG : 5 Percent
MINTIMEL : 3 Minutes
MAXTIME  : 0 Seconds
SENSE    : Medium
LOTRANS  : 155.0 Volts
HITRANS  : 280.0 Volts
ALARMDEL : 30 Seconds
BATTV    : 13.4 Volts
LASTXFER : Low line voltage
NUMXFERS : 0
TONBATT  : 0 Seconds
CUMONBATT: 0 Seconds
XOFFBATT : N/A
SELFTEST : NO
STATFLAG : 0x05000008
SERIALNO : 3B1618X24140
BATTDATE : 2016-05-08
NOMINV   : 230 Volts
NOMBATTV : 12.0 Volts
NOMPOWER : 480 Watts
FIRMWARE : 925.T2 .I USB FW:T2
END APC  : 2016-09-02 16:22:55 +0000
```

That's it, **apcupsd** is now running. In case of power failure and if you're connected via ssh, you'll get broadcast messages:

```
HypriotOS/armv7: pirate@swarm-master in ~

Broadcast message from root@swarm-master (somewhere) (Fri Sep  2 16:25:20 2016)

Power failure on UPS MAINUPS. Running on batteries.


Broadcast message from root@swarm-master (somewhere) (Fri Sep  2 16:25:46 2016)

Power has returned on UPS MAINUPS...
```

To view the history of events, enter the following command ```tail -f /var/log/apcupsd.events```, and you'll get something like that:

```
2016-09-01 17:24:16 +0200  apcupsd shutdown succeeded
2016-09-01 17:24:26 +0200  apcupsd 3.14.12 (29 March 2014) debian startup succeeded
2016-09-02 12:23:52 +0000  apcupsd 3.14.12 (29 March 2014) debian startup succeeded
2016-09-02 16:09:31 +0000  apcupsd exiting, signal 15
2016-09-02 16:09:31 +0000  apcupsd shutdown succeeded
2016-09-02 16:17:03 +0000  apcupsd 3.14.12 (29 March 2014) debian startup succeeded
2016-09-02 16:25:14 +0000  Power failure.
2016-09-02 16:25:20 +0000  Running on UPS batteries.
2016-09-02 16:25:45 +0000  Mains returned. No longer on UPS batteries.
2016-09-02 16:25:45 +0000  Power is back. UPS running on mains.
```

My next post will cover the installation of **apcupsd** on a server, which is not directly connected to the UPS.

# Credits
- [Installing and configuring apcupsd (UPS Management Software)](http://raspisimon.dlinkddns.com/ups.php)
- [Linux: Configure and Control APC SmartUPS During a Power Failure](http://www.cyberciti.biz/faq/debian-ubuntu-centos-rhel-install-apcups/)
- [Apcupsd Post Installation Configuration](http://www.jusch.ch/dokus/apcupsd/configure.html)
