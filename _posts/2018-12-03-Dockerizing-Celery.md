---
layout: post
title: Dockerizing celery
---

The first time I became familiar with [Celery](http://www.celeryproject.org/) is when I was working on a WLAN's Slovenia project [PiplMesh](https://github.com/wlanslovenija/PiplMesh) way back in 2012. Celery is a Message Queue written in python. A Message Queue is a data structure that enqueues messages produced by producer on one end and consumer(s) that dequeue those messages on the other end (Figure 1).

The reason we decided to use a message queue is the following. There are multiple producers writing to the postgres SQL database. It often happens that the same record tries to be written by not just one producer. This produces duplicate key exception and we want to avoid those. By putting all the messages in queue you get synchronous stream from asynchronous inputs. Each entry is then split into own transaction. This guarantees that workers do not need to handle exceptions on their part.

![_config.yml]({{ site.baseurl }}/images/celery-1.svg)
*Figure 1: Overview of a message queue.*

Another angle to this project is that we want to run Celery in a docker container. Docker is a container technology for GNU/Linux that allows a developer to package up an application with all of the parts it needs. We are exposing access to the message queue on the host so each producer (also running in a separate docker container) can send messages to the queue. We will add another ingredient just in case we need to visually inspect what is going on in the queue. Meet [Flower](https://flower.readthedocs.io/). Flower is a web based tool for monitoring and administrating Celery clusters.


So we can see that our consumer consists of three independent services that are tightly coupled. We can neatly encapsulate all these services in a logical unit with the help of Docker Compose (Figure 2). Compose is a tool for defining and running multi-container Docker applications.

![_config.yml]({{ site.baseurl }}/images/docker-compose.svg)
*Figure 2: Overview of a system.*


Next time, we will dive into kubernetes.
