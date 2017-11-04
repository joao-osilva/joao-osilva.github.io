---
layout: post
title: "RabbitMQ cluster with Docker and Docker Compose"
author: "João Silva"
categories: technology
tags: [technology, rabbitmq, docker, docker-compose, haproxy]
image:
  feature: container.jpeg
published: false
---

Following my previous post about [Enterprise Messaging with RabbitMQ and AMQP]({{ site.baseurl }}{% post_url 2017-10-15-Enterprise-Messaging-with-RabbitMQ-and-AMQP %}), let's get a cluster running using Docker and Docker Compose.

## Target Architecture

The idea here is to create a RabbitMQ cluster composed by 3 nodes, and have a reverse proxy that sits in front of the cluster and distributes the load between nodes.

<div class="all-img">
<img src="/assets/img/reverse-proxy.png">
</div>

## RabbitMQ and Docker
I'm going to use the official RabbitMQ docker image available at [Docker Hub](https://hub.docker.com/_/rabbitmq/), since it's shipped with the management plugin and it's really easy to configure.

In order to define and run our multi-container environment I'm goint to use [Docker Compose](https://docs.docker.com/compose/), which will help us to organize everything in one place.

To use Docker Compose you need 2 things:

&nbsp;&nbsp;1. Define your environment in a `Dockerfile`.

&nbsp;&nbsp;2. Define your services in a `docker-compose.yml` file.

## HAProxy
As I mentioned early, I'm using a reverse proxy to balance the load on my cluster. There are many options in the market such as NGINX, Apache HTTP Server and HAProxy, which is what I'm using here, given it has support for HTTP and TCP as well.

Fist we need to create a configuration file with all the necessary information for HAProxy.

Create a file named `haproxy.cfg`, and copy the following content on it:
```cfg
global
  log     127.0.0.1 alert
  log     127.0.0.1 alert debug
  chroot /var/lib/haproxy
  maxconn 3000
  daemon
  user    haproxy
  group   haproxy

defaults
  log     global
  option  dontlognull
  option  persist
  option  redispatch
  retries 3
  timeout connect 5000
  timeout client  50000
  timeout server  50000

listen haproxy-stats
    bind  *:1936
    mode  http
    stats enable
    stats hide-version
    stats refresh 5s
    stats uri     /haproxy?stats
    stats realm   HHaproxy\ Statistics
    stats auth    haproxy:haproxy

listen rabbitmq
    bind    *:5672
    mode    tcp
    option  tcplog
    balance roundrobin
    server  rabbitmq-node-1 rabbitmq-node-1:5672 check inter 5000 rise 3 fall 5
    server  rabbitmq-node-2 rabbitmq-node-2:5672 check inter 5000 rise 3 fall 5
    server  rabbitmq-node-3 rabbitmq-node-3:5672 check inter 5000 rise 3 fall 5
```
I imagine that you hate(as much as I do) to just copy things over, so here's a brief explanation:
##### global
- **chroot:** isolates the app(in a directory) from the rest of the system to increase the security level([more about](https://help.ubuntu.com/community/BasicChroot))
- **maxconn:** maximum number of concurrent connections
- **daemon:** makes the process run in the background
- **user:** name of the user dedicated to HAProxy in the OS
- **group:** name of the group that the user belongs to

##### defaults
- **log:** apply log settings from the global section
- **option dontlognull:** disable logging of null connections
- **option persist:** forward requests firstly to servers that are allegedly down
- **option redispatch:** in case the server its really dead, redirect the request to another one
- **retries:** number of retries to perform on a server after connection failure
- **timeout connect:** maximum time to wait for a connection attempt to a server to succeed
- **timeout client:** maximum inactivity time on the client side
- **timeout server:** maximum inactivity time on the server side

##### listen haproxy-stats
- **bind:** listening address:port
- **mode:** which protocol is being used
- **stats enable:** enable statistics reporting
- **stats hide-version:** hide HAProxy version reporting
- **stats refresh:** statistics refresh rate
- **stats uri:** the URI prefix to access the statistics page
- **stats realm:** statistics authentication realm
- **stats auth:** enable statistics basic authentication and grant access to an account(user:pass)

##### listen rabbitmq
- **:**
- **:**
- **:**
- **:**

You can find more information about the configuration [here](https://cbonte.github.io/haproxy-dconv/1.7/configuration.html).

HAProxy Dockerfile
```dockerfile
FROM haproxy:1.7

ENV HAPROXY_CONFIG /usr/local/etc/haproxy/haproxy.cfg
ENV HAPROXY_USER haproxy

RUN groupadd --system ${HAPROXY_USER} \
    && useradd --system --gid ${HAPROXY_USER} ${HAPROXY_USER}

COPY haproxy.cfg ${HAPROXY_CONFIG}

CMD ['haproxy', '-f', ${HAPROXY_CONFIG}]
```


Build Docker image
```sh
$ docker build -t haproxy-rabbitmq-cluster:1.7 .
```

```yaml
version: '2'

services:
 rabbitmq-node-1:
  image: rabbitmq:3-management
  container_name: rabbitmq-node-1
  hostname: rabbitmq-node-1
  ports:
   - "15672:15672"
  networks:
   - cluster-network
  volumes:
   - $PWD/storage/rabbitmq-node-1:/var/lib/rabbitmq
  environment:
   - RABBITMQ_ERLANG_COOKIE=cluster_cookie
   - RABBITMQ_DEFAULT_USER=admin
   - RABBITMQ_DEFAULT_PASS=Admin@123  

 rabbitmq-node-2:
  image: rabbitmq:3-management
  container_name: rabbitmq-node-2
  hostname: rabbitmq-node-2
  ports:
   - "15673:15672"
  networks:
   - cluster-network
  volumes:
   - $PWD/storage/rabbitmq-node-2:/var/lib/rabbitmq
  environment:
   - RABBITMQ_ERLANG_COOKIE=cluster_cookie
   - RABBITMQ_DEFAULT_USER=admin
   - RABBITMQ_DEFAULT_PASS=Admin@123

 rabbitmq-node-3:
  image: rabbitmq:3-management
  container_name: rabbitmq-node-3
  hostname: rabbitmq-node-3
  ports:
   - "15674:15672"
  networks:
   - cluster-network
  volumes:
   - $PWD/storage/rabbitmq-node-3:/var/lib/rabbitmq
  environment:
   - RABBITMQ_ERLANG_COOKIE=cluster_cookie
   - RABBITMQ_DEFAULT_USER=admin
   - RABBITMQ_DEFAULT_PASS=Admin@123

 haproxy:
  image: haproxy-rabbitmq-cluster:1.7
  container_name: haproxy
  hostname: haproxy
  ports:
    - "5672:5672"
    - "1936:1936"
  networks:
   - cluster-network

networks:
 cluster-network:
  driver: bridge
```



Run docker compose
```sh
$ docker-compose up -d
```


Create cluster

Node 2
```sh
docker exec -ti rabbitmq-node-2 bash -c "rabbitmqctl stop_app"
docker exec -ti rabbitmq-node-2 bash -c "rabbitmqctl join_cluster rabbit@rabbitmq-node-1"
docker exec -ti rabbitmq-node-2 bash -c "rabbitmqctl start_app"
```
Node 3
```sh
docker exec -ti rabbitmq-node-3 bash -c "rabbitmqctl stop_app"
docker exec -ti rabbitmq-node-3 bash -c "rabbitmqctl join_cluster rabbit@rabbitmq-node-1"
docker exec -ti rabbitmq-node-3 bash -c "rabbitmqctl start_app"
```
