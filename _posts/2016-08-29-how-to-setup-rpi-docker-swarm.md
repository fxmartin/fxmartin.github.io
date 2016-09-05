---
layout: post
title: How to setup a Docker Swarm cluster with Raspberry Pi's
author: FX
---
I've been playing with [Raspberry Pis](https://en.wikipedia.org/wiki/Raspberry_Pi) for several years now. My very first one was a PI1+, from 2014. I also discovered [Docker](https://www.docker.com/) in 2015, after having used heavily virtual machines with [VMware Fusion](http://www.vmware.com/products/fusion.html) on my Mac and [Vagrant/Homestead](https://laravel.com/docs/5.3/homestead) for my pet project PHP/Laravel.

I'm a strong believer of Docker and find it a natural successor of virtual machines. Docker has evolved extremely fast in the past 12 months, from [Docker Toolbox](https://docs.docker.com/machine/get-started/), which was relying on a virtual machine (Virtualbox or VMWare), to [Docker for Mac and Windows](https://docs.docker.com/docker-for-mac/docker-toolbox/), relying on native OS virtualization, [hyperkit](https://github.com/docker/HyperKit/) for MacOs and [Hyper-V](https://en.wikipedia.org/wiki/Hyper-V) for Windows and now [Swarm mode](https://docs.docker.com/engine/swarm/key-concepts/), which is a true revolution as far as deploying redundant high availability architecture is concerned. Read [MySQL on Docker: Introduction to Docker Swarm Mode and Multi-Host Networking](http://severalnines.com/blog/mysql-docker-introduction-docker-swarm-mode-and-multi-host-networking) if you're not convinced.

![MySQL on Docker: Introduction to Docker Swarm Mode and Multi-Host Networking](/images/2016-08-29-how-to-setup-rpi-docker-swarm-img02.jpg)

So what's best than to combine the cheapest still powerful PI3 with Docker to create a cheap 5 nodes PI3 swarm farm ?

This is exactly what I did and I will explain in this post the various steps involved.

![PI cluster](/images/2016-08-29-how-to-setup-rpi-docker-swarm-img01.jpg)

This post is strongly inspired by the post of [Alasdair Allan on Make:](http://makezine.com/projects/build-a-compact-4-node-raspberry-pi-cluster/). I introduced my own variations though, 5 nodes instead of 4, the use of [HypriotOS](http://blog.hypriot.com/post/releasing-HypriotOS-1-0/) instead of the stock [Raspbian](https://www.raspbian.org/) amongst other things.

# Design

The main objective was to build a portable, compact PI3 swarm cluster. To reach this goal the following design concepts have been retained:

1. A Form factor close to the [RPI3](https://en.wikipedia.org/wiki/Raspberry_Pi#Specifications) board size: 85.60 mm × 56.5 mm
2. Slave nodes connected to a master node via an internal sub-network
3. A master node connected to the outside world with a second LAN port, providing DHCP and forwarding packets to allow slaves to access the outside world
4. A LCD 4x40 to display useful information like eth0/eth1 address, Node status, etc.
5. One singe power cable

# Pre-requisites

To be able to build your very own PI3 cluster, you'll need several components:

- 5 [Raspberry PI 3](http://www.kubii.fr/1628-nouveau-raspberry-pi-3-modele-b-1-gb-640522710850.html)
- 5 [micro SD card](https://www.amazon.fr/gp/product/B00BLHWYWS/ref=oh_aui_detailpage_o06_s00?ie=UTF8&psc=1)
- 5 [Flat RJ45cable 25 cm](https://www.amazon.fr/gp/product/B00UPHNFRI/ref=oh_aui_detailpage_o01_s00?ie=UTF8&psc=1)
- 1 [Vullers Tech plexi case](https://www.amazon.fr/gp/product/B00NB1WPEE/ref=oh_aui_detailpage_o05_s00?ie=UTF8&psc=1)
- 4 [intermediary layers for plexi case](https://www.amazon.fr/gp/product/B00NB1WQZW/ref=oh_aui_detailpage_o01_s00?ie=UTF8&psc=1)
- 1 [USB charger 5 ports](https://www.amazon.fr/gp/product/B013FJ3XFQ/ref=oh_aui_detailpage_o04_s00?ie=UTF8&psc=1)
- 1 [Ethernet switch 5 ports](https://www.amazon.fr/gp/product/B000FNFSPY/ref=oh_aui_detailpage_o04_s00?ie=UTF8&psc=1)
- 1 [LCD I2C 4x20 display](https://www.amazon.fr/gp/product/B01BI785JQ/ref=oh_aui_detailpage_o05_s00?ie=UTF8&psc=1)
- 4 [Jumper cables female-female](https://www.amazon.fr/Kuman-120pcs-Multicolored-Female-Breadboard/dp/B01BV3Z342/ref=sr_1_2?ie=UTF8&qid=1473052598&sr=8-2&keywords=jumpers+female+female)
- 1 [USB RJ45 adaptor](https://www.amazon.fr/gp/product/B00007IFED/ref=oh_aui_detailpage_o00_s00?ie=UTF8&psc=1)
- 5 [USB cable](https://www.amazon.fr/gp/product/B00OOOHPN8/ref=od_aui_detailpages00?ie=UTF8&psc=1)

The USB hub I used was the thing that actually inspired me to do the build in the first place: It’s a 5-port hub from Anker and has coincidentally about the same footprint as the Raspberry Pi itself. With five ports, there’s one port for each of my five Raspberry Pi boards.

My choice of Ethernet switch was entirely driven by two factors, size and voltage. I wanted it to more-or-less have the same footprint as Raspberry Pi, but I also desperately wanted it to be powered from my USB hub. So it had to take a 5V supply.

> **Note**: I ordered a 6 ports USB charger, slightly larger, and will try to plug the switch in it as instructed in the post of Alasdair Allan.

At the time of this writing, total cost for these parts has been 407,59 €. Not cheap but so much fun.

# Building the hardware

This is extremely straight forward and requires only a screwdriver. I used double sided adhesive tape to secure the switch, the case and the USB hub.

![RPI3 Swarm cluster](/images/2016-08-29-how-to-setup-rpi-docker-swarm-img03.jpg)

# Configuring the cluster

The first thing we need to do is grab a disk image of the latest version of HypriotOS and copy it to five SD cards, one for each of our Raspberry Pi boards. I chose HypriotOS for several reasons:

1. It includes Docker out-of-the-box (latest Docker Engine 1.12.1 with Swarm Mode)
2. It comes with a [flash tool](https://github.com/hypriot/flash), which allows to specify several parameters while flashing like hostname
3. It is optimised in several ways: Enhanced security out of the box, Maximum performance out of the box , Now 50% smaller in size, even smaller than Raspbian Lite, nice pre-installed features like Avahi-daemon

## Flash all the SD cards

To flash the SD cards I used the tool provided by hypriot and set the hostname with the parameter ```--hosname```.

```
flash --hostname swarm-master hypriotos-rpi-v1.0.0-rc1.img
flash --hostname swarm-slave-1 hypriotos-rpi-v1.0.0-rc1.img
flash --hostname swarm-slave-2 hypriotos-rpi-v1.0.0-rc1.img
flash --hostname swarm-slave-3 hypriotos-rpi-v1.0.0-rc1.img
flash --hostname swarm-slave-4 hypriotos-rpi-v1.0.0-rc1.img
```

For now I’m taking a short cut and using my home router to allocate IP addresses to each of the nodes.

First consider checking that all nodes are up and running on your current network. You need to obtain the IP addresses of each of the Raspberry Pi’s running on your LAN. I used Hypriot’s custom bash function to do a lookup against the Avahi-daemon configured on each Raspberry Pi:

> NOTE: Default password for the “pirate” user is “hypriot”

```
function getip() { (traceroute $1 2>&1 | head -n 1 | cut -d\( -f 2 | cut -d\) -f 1) }
export MASTER=$(getip swarm-master.local)
echo $MASTER
export SLAVE_1=$(getip swarm-slave-1.local)
echo $SLAVE_1
export SLAVE_2=$(getip swarm-slave-2.local)
echo $SLAVE_2
export SLAVE_3=$(getip swarm-slave-3.local)
echo $SLAVE_3
```

## perform initial set-up

Logging into the nodes in turn I did the standard setup on each node by running,

```
sudo raspi-config
```

and then going ahead and expanding the file system to the size of the SD card, giving me a few extra gigabytes to play with. I also changed the password for each node to something a bit more secure.

## Set-up the external USB LAN connector

Check that the USB LAN adapter is indeed recognised and mounted by running the command ```lsusb```. You should get an output similar to the one below:

```
Bus 001 Device 004: ID 0b95:772a ASIX Electronics Corp. AX88772A Fast Ethernet
Bus 001 Device 003: ID 0424:ec00 Standard Microsystems Corp. SMSC9512/9514 Fast Ethernet Adapter
Bus 001 Device 002: ID 0424:9514 Standard Microsystems Corp.
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

Use ```ifconfig``` to get the MAC address of the USB LAN Adapter.

```
veth44e7beb Link encap:Ethernet  HWaddr 42:16:f8:6c:53:1b
          inet6 addr: fe80::4016:f8ff:fe6c:531b/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:46 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:648 (648.0 B)  TX bytes:8344 (8.1 KiB)
```

The first issue I faced while applying the instructions of [Alasdair Allan on Make:](http://makezine.com/projects/build-a-compact-4-node-raspberry-pi-cluster/) was that after each reboot, the name of the external USB ethernet adaptor was changing. So I had to find a way to fix it.

Create a new UDEV rule to name it automatically:

```
sudo nano /etc/udev/rules.d/010-usb-net-interfaces.rules
KERNEL=="veth*", SYSFS{address}=="3a:54:a8:91:6c:cf", NAME="eth1"
```

Duplicate the ```/etc/network/interfaces.d/eth0``` for this new ```eth1```

```
sudo cp /etc/network/interfaces.d/eth0 /etc/network/interfaces.d/eth1
allow-hotplug eth1
iface eth1 inet dhcp
```

## Set-up simplified secure SSH connection by sending public key

Copy the public key to all nodes. I assume here that you have a private/public RSA key. If not then go and generate one. There are plenty of instructions on the Internet on how to do it.

```
ssh-copy-id -i .ssh/id_rsa.pub pirate@swarm-master.local
```

Now try logging into the machine, with: ```ssh 'pirate@swarm-master.local'```.

Repeat the action for all other nodes, swarm-slave-X in my case.

## Set-up DHCPD on the master node

Define ```eth0``` with a static address:

```
sudo nano /etc/network/interfaces.d/eth0
iface eth0 inet static
  address 192.168.50.1
  netmask 255.255.255.0
  network 192.168.50.0
  broadcast 192.168.50.255
```

From the configuration you’ll notice that I’m intending to leave ```eth0``` — the onboard Ethernet socket — connected to the Ethernet switch and serve as the internal connection to the cluster , while ```eth1``` is connected to the outside world.

You should bear in mind that, since the MAC address of our adaptor facing the home router is going to change, our “external” IP address for the head node is also going to change.

Next we need to install the DHCP server: ```sudo apt-get install isc-dhcp-server```

and then edit the file ```/etc/dhcp/dhcpd.conf``` file as follows,

```
sudo nano /etc/dhcp/dhcpd.conf

#Specific dhcpd.conf setup for defining internal cluster of swarm nodes

# You must prevent the DHCP server from receiving DNS information from clients,
# set the following global option (this is a security feature):
ddns-update-style none;

# The authoritative directive indicate that the DHCP server should send DHCPNAK
# messages to misconfigured clients. If this is not done, clients will be unable
# to get a correct IP address after changing subnets until their old lease has
# expired, which could take quite a long time.
authoritative;

# Enable logging
log-facility local7;

# No service will be given on this subnet
subnet 192.168.1.0 netmask 255.255.255.0 {
}

# The internal cluster network
group {
   option broadcast-address 192.168.50.255;
   option routers 192.168.50.1;
   default-lease-time 600;
   max-lease-time 7200;
   option domain-name "swarm-cluster";
   option domain-name-servers 8.8.8.8, 8.8.4.4;
   subnet 192.168.50.0 netmask 255.255.255.0 {
      range 192.168.50.14 192.168.50.250;

      host swarm-cluster {
         hardware ethernet b8:27:eb:8e:ea:c1;
         fixed-address 192.168.50.1;
      }
      host swarm-slave-1 {
         hardware ethernet b8:27:eb:89:90:6b;
         fixed-address 192.168.50.11;
      }
      host swarm-slave-2 {
         hardware ethernet b8:27:eb:d2:63:58;
         fixed-address 192.168.50.12;
      }
      host swarm-slave-3 {
         hardware ethernet b8:27:eb:c3:70:43;
         fixed-address 192.168.50.13;
      }
      host swarm-slave-4 {
         hardware ethernet b8:27:eb:ea:d3:d2;
         fixed-address 192.168.50.14;
      }
   }
}
```

Here we’re defining an internal cluster network, and allocating each of the four nodes their own static IP address on the internal network. Be careful to replace with your own MAC addresses, which you'll be able to find using ``ìfconfig```. Then edit the ```/etc/default/isc-dhcp-server``` file to reflect our DHCP server setup

```
sudo nano /etc/default/isc-dhcp-server
DHCPD_CONF=/etc/dhcp/dhcpd.conf
DHCPD_PID=/var/run/dhcpd.pid
INTERFACES="eth0"
```

Restart the DHCP daemon ```sudo service isc-dhcp-server restart```.

To check what’s going on ```sudo cat /var/log/messages | grep dhcpd```.

DHCP leases assigned by the daemon can be found in ```cat /var/lib/dhcp/dhcpd.leases```.

Next go ahead and edit the /etc/hosts file on all four of the nodes to reflect the changes — for now you can still reach them at their old IP addresses. Do not forget to adapt the ```127.0.0.1```details for each node.

```
sudo nano /etc/hosts
127.0.0.1       localhost
127.0.0.1       swarm-master # added by device-init
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

127.0.1.1 swarm-master swarm-master.local swarm-master.lan swarm-master.cluster

192.168.50.1    swarm-master swarm-master.local swarm-master.lan swarm-master.cluster
192.168.50.11   swarm-slave-1 swarm-slave-1.local swarm-slave-1.lan swarm-slave-1.cluster
192.168.50.12   swarm-slave-2 swarm-slave-2.local swarm-slave-2.lan swarm-slave-2.cluster
192.168.50.13   swarm-slave-3 swarm-slave-3.local swarm-slave-3.lan swarm-slave-3.cluster
192.168.50.14   swarm-slave-4 swarm-slave-4.local swarm-slave-4.lan swarm-slave-4.cluster
```

You can see that ```eth0``` has the static internal IP address we allocated to it, while ```eth1``` has a new IP address allocated by our home router. If all goes according to plan you should be able to ssh into the head node using its new external IP address, and see something like this,

```
ifconfig
eth0      Link encap:Ethernet  HWaddr b8:27:eb:8e:ea:c1
          inet addr:192.168.50.1  Bcast:192.168.50.255  Mask:255.255.255.0
          inet6 addr: fe80::ba27:ebff:fe8e:eac1/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:38 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:6498 (6.3 KiB)

eth1      Link encap:Ethernet  HWaddr d8:eb:97:bf:1c:5b
          inet addr:192.168.0.90  Bcast:192.168.0.255  Mask:255.255.255.0
          inet6 addr: fe80::daeb:97ff:febf:1c5b/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:200 errors:141 dropped:0 overruns:0 frame:0
          TX packets:83 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:31240 (30.5 KiB)  TX bytes:13279 (12.9 KiB)
```

and this,

```
route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.0.1     0.0.0.0         UG    0      0        0 eth1
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
172.18.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker_gwbridge
192.168.0.0     0.0.0.0         255.255.255.0   U     0      0        0 eth1
192.168.50.0    0.0.0.0         255.255.255.0   U     0      0        0 eth0
```

Hopefully you can reach the head node and ssh into it from the external network. From there, you should be able to ping both external hosts on the 192.168.0.* network, and internal hosts on the 192.168.50.* network.

However, at least right now, if we ssh into one of the slave nodes, while they can see the head node — and each other — they can’t yet see the outside world. We’re going to have to forward packets from the internal to the external networks before that’s possible.

On the head node go ahead and, ```sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"```

and then edit the  ```/etc/sysctl.conf``` file uncommenting the line saying, ```net.ipv4.ip_forward=1```.

After activating forwarding we’ll need to configure iptables,

```
sudo iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE
sudo iptables -A FORWARD -i eth1 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT
sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
```

and then add at the bottom of the ```/etc/network/interfaces.d/eth0``` file a line to load the tables on boot,

```
allow-hotplug eth0
iface eth0 inet static
  address 192.168.50.1
  netmask 255.255.255.0
  network 192.168.50.0
  broadcast 192.168.50.255
  pre-up iptables-restore < /etc/iptables.ipv4.nat
```

## Create ssh key for cluster and copy it to slave nodes

As we did from our host to the master node, let's copy a public key from the master node to all slave nodes. First an SSH-key on the master Raspberry Pi is generated.

```
ssh pirate@swarm-master.local
ssh-keygen -t rsa -C "root@swarm-master.local"
```

Copy the public key to all nodes

```
ssh-copy-id -i .ssh/id_rsa.pub pirate@swarm-slave-1.local
ssh-copy-id -i .ssh/id_rsa.pub pirate@swarm-slave-2.local
ssh-copy-id -i .ssh/id_rsa.pub pirate@swarm-slave-3.local
ssh-copy-id -i .ssh/id_rsa.pub pirate@swarm-slave-4.local
```

## Additional customisations

### Set-up the time zone

Add to .profile

```
# Set-up timezone France/Paris
export TZ='Europe/Paris'
```

### Setting Locale Settings
The locale settings can be set (to en_US.UTF-8 in the example). Add to .profile

```
export LANGUAGE=en_US.UTF-8
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
run the following command
sudo locale-gen en_US.UTF-8
```

### Setting-up NTP client

One of the disadvantages of designing so cheap computer is that you have to give up some features which are too expensive. One of these features is RTC - real-time clock. RTC chip has its own battery to store actual time even when device isn`t plugged in. This chip costs about 10$. This is quite enough to persuade RPi designers to do not implement this feature.

Problem with storing actual time is partially solved by fake-hwclock package which writes actual time to text file every hour and at system reboot. At start, if no better source of time is present, time is set to one stored in that text file. Since RPi is disposing of a RJ-45 port, we can use it as very precise time source with NTP. First we have to install a package called ntp: ```sudo apt-get update && sudo apt-get install -y ntp```

Thats all, RPi is now synchronizing its time to NTP servers. By default it uses NTP servers which are generally too far from you. It has bad influence to time accuracy. So first go to page pool.ntp.org and find a location as near as possible. For me it is France. To set NTP servers open ntp configuration file.

```
sudo nano /etc/ntp.conf
# pool.ntp.org maps to about 1000 low-stratum NTP servers.  Your server will
# pick a different set every time it starts up.  Please consider joining the
# pool: <http://www.pool.ntp.org/join.html>
server 0.fr.pool.ntp.org
server 1.fr.pool.ntp.org
server 2.fr.pool.ntp.org
server 3.fr.pool.ntp.org
```

Now you can restart ntp service: ```sudo  /etc/init.d/ntp restart```

Here is a command to check if time is synchronizing properly. To list NTP servers with which RPi is synchronizing: ```ntpq -pn```/

You will get some output like this.

```
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
 149.202.97.123  213.251.128.249  2 u   47   64    1  340.026  127.835   0.001
 129.250.35.250  .INIT.          16 u    -   64    0    0.000    0.000   0.000
 194.57.169.1    145.238.203.14   2 u   45   64    1  658.247  289.613   0.001
 5.196.160.139   10.21.137.1      2 u   44   64    1  809.818  364.128   0.001
```

Now when master RPi has synchronized its time, you can set it to pass this time information to devices in your local network. You only have to open configuration file of ntp daemon: ```sudo nano /etc/ntp.conf``` and add a string that describes the hosts which requests will be answered.

```
    restrict 192.168.50.0 mask 255.255.255.0
```

And also add these 2 lines. They will enable sending of broadcasts and multicasts containing time information for devices which accept them (Cisco, Juniper...)). Change first address in bold to broadcast address of your LAN. Do not change multicast address 224.0.1.1 since this address is assigned to NTP service by IANA and some network devices join this multicast group automatically.

```
    broadcast 192.168.50.255
    broadcast 224.0.1.1
```

Now close the configuration file (CTRL+X...) and save changes (...press "y" and Enter). Last step is to restart ntp daemon with: ```/etc/init.d/ntp restart```.

For the other nodes, slave-1 to 4, proceed similarly but edit the file ```ntp.conf``` as follows:

```
sudo nano /etc/ntp.conf
# You do need to talk to an NTP server or two (or three).
server swarm-master.local

# pool.ntp.org maps to about 1000 low-stratum NTP servers.  Your server will
# pick a different set every time it starts up.  Please consider joining the
# pool: <http://www.pool.ntp.org/join.html>
#server 0.debian.pool.ntp.org iburst
#server 1.debian.pool.ntp.org iburst
#server 2.debian.pool.ntp.org iburst
#server 3.debian.pool.ntp.org iburst
```

Now you can restart ntp service: ```sudo  /etc/init.d/ntp restart```.

No use of the pool of servers but replication to swarm-master as ntp server. Here is command to check if time is synchronizing properly. To list NTP servers with which RPi is synchronizing: ```ntpq -pn```/

You will get some output like this.

```
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
 192.168.50.1    151.80.19.218    3 u   64   64    3    0.428   -9.072   0.685
```

# Set-up the Docker Swarm

## Initialise the swarm mode

First connect to the master node ```ssh pirate@swarm-master.local```and initialise the swarm: ```docker swarm init```. You should get an output like below:

```
Swarm initialized: current node (2iqsernn2wilfi0xy4ny1bzj3) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-33q1bffqfy42tecajzozqmw0df4b4jz5usi0jmvy0oo8oyzg8h-bn371hggsnqmr91f9t14arpla \
    192.168.0.105:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

## Add additional Docker Swarm nodes

Connect to a slave mode ```ssh pirate@swarm-slave-1``` and join the swarm manager:

```
docker swarm join \
    --token SWMTKN-1-33q1bffqfy42tecajzozqmw0df4b4jz5usi0jmvy0oo8oyzg8h-bn371hggsnqmr91f9t14arpla \
    192.168.0.105:2377
```

Repeat for all nodes.

To check the number of nodes execute the following command on the swarm manager: ```docker info```.

# Set-up GIT

Tell Git your name so your commits will be properly labeled.

```
git config --global user.name "FX Martin"
config --global user.email "mail@fxmartin.me"
```

Fork a repository directly on the github website. On GitHub, navigate for example to fxmartin/rpi-apcupsd-cgi repository. In the top-right corner of the page, click Fork.

```
git clone https://github.com/fxmartin/using_docker_in_dev.git
```

When you fork a project in order to propose changes to the original repository, you can configure Git to pull changes from the original, or upstream, repository into the local clone of your fork.

```
git remote add upstream https://github.com/using-docker/using_docker_in_dev.git
git remote -v
origin    https://github.com/fxmartin/using_docker_in_dev.git (fetch)
origin    https://github.com/fxmartin/using_docker_in_dev.git (push)
upstream    https://github.com/using-docker/using_docker_in_dev.git (fetch)
upstream    https://github.com/using-docker/using_docker_in_dev.git (push)
```

Because 2-factors authentication has been enabled, a token must be generated and used instead of the password to be able to execute ```git push origin```.

To store the password in memory for 15 minutes: ```git config --global credential.helper "cache --timeout=3600"```

# Credits
- [Alasdair Allan](http://makezine.com/author/aallan/): [Build a Compact 4 Node Raspberry Pi Cluster](http://makezine.com/projects/build-a-compact-4-node-raspberry-pi-cluster/)
- [How to setup a Docker Swarm cluster with Raspberry Pi's](http://blog.hypriot.com/post/how-to-setup-rpi-docker-swarm/)
