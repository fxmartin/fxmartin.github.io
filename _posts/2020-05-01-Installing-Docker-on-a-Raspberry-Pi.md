---
layout: post
title: Installing Docker on a Raspberry Pi and hacking a ScrollPhat
author: FX
---

I started playing with Docker in 2016. I am still amazed by its power and simplicity. Being confined at home and kind of bored on this rainy day I resumed hacking a Raspberry Pi with Docker, connected to IPad and now plugging a ScrollPhat I found to monitor cpu, temp and number of docker images running.

# Install Docker
From your SSH session type in ``curl -sSL https://get.docker.com | sudo -E sh``, this will setup Docker on your Pi.

Now let the Pi user have access to the Docker daemon and reboot by typing in:

```
$ sudo usermod -aG docker pi
$ sudo reboot
```

To check everything worked you can list the running containers (which should come back empty)

```
$ docker ps
```

I found a ScrollPhat from Pimoroni in one of my cupboard and I decided to use it to monitor the Raspberry Pi.

The idea is to write a Python program to display temperature, CPU load and number of containers running on Docker.

It will also be the occasion to explore a bit more Python and especially parameters management.

The final program is available on GitHub [github.com/fxmartin/ScrollPhatDockerPi](https://github.com/fxmartin/ScrollPhatDockerPi)

All the examples I found were clearing/updating the full matrix of LEDs resulting in a kind of blinking effect. I looked for a smarter way to shut on/off the LEDs in order to avoid the blinking effect.

Below a short video to demonstrate how it works. I took this video while compiling CV2 library so it explains why the CPU load is high. It was a good way to test the program and the PI.

![ScrollPhat demo](/images/ScrollPhatVideo.MOV)

Initially I included the monitoring of docker containers but when trying to run the Python program as a service then I faced an error message with ``import docker`` so after 2h of investigations I decided to remove it.

# Credits
- [Revsol/blinkt-cpu-info](https://github.com/Revsol/blinkt-cpu-info)
- [How to monitor docker and automatically start containers?](https://fxmartin.github.io/How-to-monitor-docker-and-automatically-start-containers/)
- [Pimoroni ScrollPhat](https://shop.pimoroni.com/products/scroll-phat)
- [Alexellis Workshop on Docker and IoT](https://github.com/alexellis/docker-blinkt-workshop)