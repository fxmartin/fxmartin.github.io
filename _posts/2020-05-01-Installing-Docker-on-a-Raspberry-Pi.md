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

# Credits
- [Revsol/blinkt-cpu-info](https://github.com/Revsol/blinkt-cpu-info)
- [How to monitor docker and automatically start containers?](https://fxmartin.github.io/How-to-monitor-docker-and-automatically-start-containers/)
- [Pimoroni ScrollPhat](https://shop.pimoroni.com/products/scroll-phat)
- [Alexellis Workshop on Docker and IoT](https://github.com/alexellis/docker-blinkt-workshop)