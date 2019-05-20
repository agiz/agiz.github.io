---
layout: post
title: Run commands on a remote docker host
---


Setup:

computer-A (docker host):
* docker has to be installed
* exposed tcp connection to the unix socket: `docker run -v /var/run/docker.sock:/var/run/docker.sock -d --restart=always --privileged -p 2375:2375 --name docker-sock-proxy docksal/socat`

computer-B (client):
* docker has to be installed
* See all running docker containers: `docker -H tcp://computer-A ps`
