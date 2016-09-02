---
layout: post
title: Installing and configuring apcupsd (UPS Management Software) - Part 2
author: FX
---
## Configuration of apcupsd network information server

There are two major ways of running **apcupsd** on your system. The first is a standalone configuration where **apcupsd** controls a single UPS, which powers the computer. This is the most common configuration. The second configuration is a master/slave configuration, where one UPS powers several computers, each of which runs a copy of **apcupsd**. The computer that controls the UPS is called the master, and the other computers are called slaves. The master copy of **apcupsd** communicates with and controls the slaves via an ethernet connection.

These two possibilities can be represented by the following diagrams:

                    Stand Alone Configuration

           ---------------------   serial port    ------
          |                     | <============> |      |
          |  Computer running   |                | UPS  |
          |    apcupsd in       |   Power        |      |
          |    stand alone      | <============= |      |
          |       mode          |                 ------  
          |                     |
           ---------------------




                   Typical Master/Slave Configuration

           ---------------------   serial port    ------
          |                     | <============> |      |
          |  Computer A running |                | UPS  |
          |    apcupsd in       |   Power        |      |
          |    master mode      | <============= |      |
          |                     |           ||    ------  
          |                     |           ||
           ---------------------            ||
                     |                      ||
     ----------------|  Ethernet            ||
     |               |                      ||
     |     ---------------------            ||
     |    |                     |           ||
     |    |  Computer B running |           ||
     |    |    apcupsd in       |   Power   ||
     |    |    slave mode       | <=========||
     |    |                     |           ||
     |    |                     |           ||
     |     ---------------------            ||
     |                                      ||
     |                                      ||
     -----------------  Ethernet            ||
                     |                      ||
           ---------------------            ||
          |                     |           ||
          |  Computer C running |           ||
          |    apcupsd in       |   Power   ||
          |    slave mode       | <===========
          |                     |           
          |                     |
           ---------------------


For the master node as explained above for the second configuration, please refer to [my previous blog post](http://fxmartin.github.io/Installing-and-configuring-apcupsd-part-1/).

```NISIP 0.0.0.0```

The default port is set to 3551 for sending STATUS and EVENTS data over the network:

```NISPORT 3551```

For the slave nodes as explained above for the second configuration, set IP address on which NIS server will listen for incoming connections. You need to configure this setting to any specific IP address of your server and NIS will listen for connections only on that interface.
```USPCABLE ether```
```UPSTYPE net```
```DEVICE 192.168.1.50:3551```

The below commands will perform these changes automatically, providing that you defined a variable APCADDR, containing the master node IP address:
```
sed -i -e "s/UPSCABLE smart/UPSCABLE ether/g" /etc/apcupsd/apcupsd.conf
sed -i -e "s/UPSTYPE apcsmart/UPSTYPE net/g" /etc/apcupsd/apcupsd.conf
sed -i -e s_"DEVICE /dev/ttyS0"_"DEVICE $APCADDR"_g /etc/apcupsd/apcupsd.conf
```
