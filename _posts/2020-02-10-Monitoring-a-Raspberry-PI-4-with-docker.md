---
layout: post
title: Monitoring a Raspberry Pi 4 with docker
author: FX
---
These are quick notes which will need to be completed later.

Taken from a previous post on monitoring PI with docker.

Added docker service for Blinkt

```
sudo ufw allow 8080/udp
sudo ufw allow 9000/udp
```

# Credits
- [Revsol/blinkt-cpu-info](https://github.com/Revsol/blinkt-cpu-info)
- [How to monitor docker and automatically start containers?](https://fxmartin.github.io/How-to-monitor-docker-and-automatically-start-containers/)
- [Pimoroni Blinkt](https://shop.pimoroni.com/products/blinkt)