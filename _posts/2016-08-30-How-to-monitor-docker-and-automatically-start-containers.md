---
layout: post
title: How to monitor docker and automatically start containers?
author: FX
---

While looking for ways to monitor containers with open source solutions, I read a very interesting paper from Rancher labs: [Comparing Seven Monitoring Options for Docker](http://rancher.com/comparing-monitoring-options-for-docker-deployments/).

I started to play with Prometheus but it will take me some more time to be really useable. Recently I also found an interesting [Docker UI](https://github.com/kevana/ui-for-docker) while reading a [paper](http://www.vivekjuneja.in/) on ARM clustering. This topic alone could justify a dedicated post but for the moment I will stick with [Cadvisor](https://github.com/google/cadvisor) and docker-ui.

These two complementary solutions are not good enough for monitoring a docker swarm but again Prometheus looks promising, but will require more work on my side before having something to publish.

![Dockerui screenshot](/images/How-to-monitor-docker-and-automatically-start-containers.dockerui2.jpg)

Now the question I asked myself is how to ensure that the monitoring containers are launched on startup. Easy enough the [instructions](https://docs.docker.com/engine/admin/host_integration/) in the Docker administration guide are pretty straight forward.

Because I run [Hypriot OS](http://blog.hypriot.com/post/releasing-HypriotOS-1-0/) on my nodes, which is basically a stripped down Raspbian with docker pre-installed and a cool list of fancy features like Avahi, the out-of-the-box process manager is [systemd](https://en.wikipedia.org/wiki/Systemd).

In my case I obviously also used one of the [ARM version](robinthrift/raspbian-cadvisor) of the official [CAdvisor docker](https://hub.docker.com/r/google/cadvisor/).

The first step is to define the service file in ```/etc/systemd/system```.

Enter the following command:

```sudo nano /etc/systemd/system/docker-ca_advisor.service```

And copy/paste the below content:

```
[Unit]
Description=CA Advisor container
Requires=docker.service
After=docker.service

[Service]
Restart=always
ExecStart=/usr/bin/docker run --volume=/:/rootfs:ro --volume=/var/run:/var/run:rw --volume=/sys:/sys:ro --volume=/var/lib/docker/:/var/lib/docker:ro --publish=8080:8080 --name=cadvisor robinthrift/raspbian-cadvisor
ExecStop=/usr/bin/docker stop -t 2 cadvisor
ExecStopPost=/usr/bin/docker rm -f cadvisor

[Install]
WantedBy=default.target
```

> **Note**: when trying to implement my own instructions for docker-ui I encountered an issue. The container was stopping and restarting within seconds. It took me sometime to realise that you cannot *just* copy/paste the command line you use to launch manually the container. Indeed the option -d must not be specified in a service file. So the ExecStart for Dockerui is ```docker run -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock --name=dockerui hypriot/rpi-dockerui```

To start using the service, reload ```systemd``` and start the service:
```
sudo systemctl daemon-reload
sudo systemctl start docker-ca_advisor.service```

To enable the service at system startup, execute:

```sudo systemctl enable docker-ca_advisor.service```

To access the Cadvisor page, point your favorite browser to ```http://IPADRESS:8080``` where you replace IPADDRESS by the IP address of the docker node you want to monitor.

![Cadvisor screenshot](/images/How-to-monitor-docker-and-automatically-start-containers.cadvisor.jpg)

And the Docker-ui dashboard on the port 9000

![Docker-ui screenshot](/images/How-to-monitor-docker-and-automatically-start-containers.dockerui1.jpg)
