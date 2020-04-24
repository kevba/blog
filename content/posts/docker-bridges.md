---
title: "Docker Bridges"
date: 2020-03-10T16:10:49+01:00
draft: false
---

Configuring networks is hard, and docker certainly did not make things easier. Since I am no network expert most things I know come from trial and error. This post describes the process how I setup a docker bridge. 

## Why would I need a docker bridge
A docker bridge is needed when your containers must be reachable on the network your host is connected to. Each container will have an IP address in the same subnet as the host.

Often you can just forward a port. However I needed to run multiple containers that used the same protocol, so each one of the containers needed to use the same port. 

## A simple bridged network
To setup a docker bridged network, two things need to be done. First a bridge must be created. If you're using networkd this is fairly easy.

### Setting up the bridge
First create a file called `br0.netdev`. This file creates a new interface called `br0`.

``` 
[NetDev]
Name=br0
Kind=bridge
```

Next, create a file called `br0.network`. This configures the bridge. I think this must always have a static IP if we want to use the bridge with docker, but I am not sure.
```
[Match]
Name=br0

[Network]
DNS=1.1.1.1
Address={ip-of-your-device}/24
```

You need to add an interface to the bridge. This interface must not be configured. The network behind the interface will be reachable via the bridge.
```
[Match]
Name=eth0

[Network]
Bridge=br0
```

### Creating the docker network
The next step is creating a docker network and adding it to the bridge. The gateway MUST be the IP of your bridge. 
If you use anything else, Docker will change the IP of the bridge.

The name of the network can be anything you like, as long its not is use yet. With `docker network ls` all networks are shown.
``` bash
docker network create \
    --driver=bridge \
    --subnet=${subnet}/24 \
    --gateway=${ip-of-your-bridge} \
    -o "com.docker.network.bridge.name=br0" \
    ${name-of-the-network}
```

After a reboot the network is ready. You can now start containers on this network. These containers will be reachable from anywhere in the network.

### Using the network
You can start a container on the network by setting the `--network` option when starting the container.

The container must also be assigned an IP address. If no ip address is given docker simply assigns one, which can cause IP conflicts. The IP address can be set with the `--ip` flag.

The docker run command should look something like this:
``` bash
docker run \
    --network=${name-of-the-network} \
    --ip="${ip}" \
    ${your-image}
```

