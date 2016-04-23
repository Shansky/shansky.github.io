---
layout: post
title: Docker and Mac OSX
---

## Intro

Running docker on mac osx environment is working but when you're running some serious project with many dockers it's really slowing down and creates many problems.

## Docker Machine

There's no native support for docker in mac osx so to use docker, you must use ```docker-machine``` (used to known as: ```boot2docker```) which talks with Virtualbox API like ```vagrant``` to create tiny virtual machine with linux distro: busybox (in fact it's by default uses old boot2docker.iso but you can use your custom iso) with preinstalled docker daemon. Now docker client installed on mac osx will talk to docker daemon located on this linux virtualmachine throw forwarded port. So all networking base on virtualbox network interfaces ```vboxnet#``` and inside vm there's ```vboxsf``` for mounting files from your osx host.

## Problem Case #1

Let's say we want to create container with dns based on consul, so we could autodiscover our other containers. Let's assume that we won't change dhcp ip range in virtualbox configuration for our docker-machine virtual machine, so we create interface which will be used by our docker dns container.

```bash
docker-machine create -d virtualbox --virtual-memory 4096 vm_1
eval "$(docker-machine env vm_1)"
docker-machine ssh vm_1 "sudo ifconfig eth1:1 172.17.255.253 network 255.255.255.0"
```

We will use this dns server in other containers so we need to change some docker daemon options:

```bash
docker-machine ssh vm_1 "/usr/local/bin/docker daemon -D -g /var/lib/docker -H unix:// -H tcp://0.0.0.0:2376 --bip=172.17.0.1/16
--dns=172.17.255.253  --label provider=virtualbox --tlsverify --tlscacert=/var/lib/boot2docker/ca.pem --tlscert=/var/l
ib/boot2docker/server.pem --tlskey=/var/lib/boot2docker/server-key.pem -s aufs &"
```

bip it's docker0 interface ip address class from virtual machine and dns is static address of our dns container. When daemon starts we need to do one more thing. On host. But first get ip of our docker daemon URL

```bash
docker-machine ls
NAME       ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER    ERRORS
vm_1      *        virtualbox   Running   tcp://192.168.99.100:2376           v1.11.0
```

Now we need add static route to our docker network

```bash
sudo route -n add 172.17.0.0/16 192.168.99.100
```

Check if our route uses vboxnet interface to route traffic to 172.17 network

```bash
netstat -nr | grep 172.17
```

Now we can test if our traffic goes to our dockers:

```bash
docker run -it ubuntu bash
root@containerhash:/# ifconfig # get ip of container
 nc -l 7777
```

from host:

```bash
telnet 172.17.0.8 777
```

Sweet if connected :wink:

## Problem case #2

TBD