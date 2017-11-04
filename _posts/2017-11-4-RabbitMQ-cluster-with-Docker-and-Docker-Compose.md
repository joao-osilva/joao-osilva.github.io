---
layout: post
title: "RabbitMQ cluster with Docker and Docker Compose"
author: "João Silva"
categories: technology
tags: [technology, rabbitmq, docker, docker-compose, haproxy]
image:
  feature: container.jpeg
published: true
---
Following my previous post about [Enterprise Messaging with RabbitMQ and AMQP]({{ site.baseurl }}{% post_url 2017-10-15-Enterprise-Messaging-with-RabbitMQ-and-AMQP %}), let's get a cluster up running using Docker and Docker Compose.

## Target Architecture
The idea here is to create a RabbitMQ cluster composed by 3 nodes, having a reverse proxy sitting in front of the cluster distributing the load between the nodes.

<div class="all-img">
<img src="/assets/img/reverse-proxy.png">
</div>

## RabbitMQ and Docker
I'm going to use the official RabbitMQ docker image available at [Docker Hub](https://hub.docker.com/_/rabbitmq/), since it already has the management plugin and it's really easy to configure.

In order to define and run our multi-container environment I'm goint to use [Docker Compose](https://docs.docker.com/compose/), which will help us to organize everything in one place.

To use Docker Compose you need 2 things:

&nbsp;&nbsp;1. Define your environment in a `Dockerfile`.

&nbsp;&nbsp;2. Define your services in a `docker-compose.yml` file.

## HAProxy
As I mentioned early, I'm using a reverse proxy to balance the load on my cluster. There are many options in the market such as NGINX, Apache HTTP Server and HAProxy, which is what I'm using here, given it has support for HTTP and TCP protocol as well.

First we need to create a configuration file with all the necessary information for HAProxy.

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

**_global_**
| Variable    | Description |
| :----------|:-------------|
| _log_     | indicates where to send the logs, its facility and level |
| _chroot_  | isolates the app(in a directory) from the rest of the system to increase the security level([more about](https://help.ubuntu.com/community/BasicChroot))      |
| _maxconn_ | maximum number of concurrent connections      |
| _daemon_  | makes the process run in the background      |
| _user_    | name of the user dedicated to HAProxy in the OS      |
| _group_   | name of the group that the user belongs to      |

**_defaults_**
| Variable    | Description           |
| :----------|:-------------|
| _log_     | apply log settings from the global section |
| _option dontlognull_  | disable logging of null connections      |
| _option persist_ | forward requests firstly to servers that are allegedly down      |
| _option redispatch_  | in case the server its really dead, redirect the request to another one      |
| _retries_    | number of retries to perform on a server after connection failure      |
| _timeout connect_   | maximum time to wait for a connection attempt to a server to succeed      |
| _timeout client_   | maximum inactivity time on the client side      |
| _timeout server_   | maximum inactivity time on the server side      |

**_listen haproxy-stats_**
| Variable    | Description           |
| :----------|:-------------|
| _bind_     | listening address:port |
| _mode_     | which protocol is being used |
| _stats enable_     | enable statistics reporting |
| _stats hide-version_     | hide HAProxy version reporting |
| _stats refresh_     | statistics refresh rate |
| _stats uri_     | the URI prefix to access the statistics page |
| _stats realm_     | statistics authentication realm |
| _stats auth_     | enable statistics basic authentication and grant access to an account(user:pass) |

**__listen rabbitmq__**
| Variable    | Description           |
| :----------|:-------------|
| _bind_     | listening address:port |
| _mode_     | which protocol is being used |
| _option tcplog_     | advanced logging of TCP connections with session state and timers |
| _balance roundrobin_     | used load balancing algorithm |
| _server_  | declares a rabbitmq server with hostname "rabbitmq-node-x" that listen at port "5672", with a health check interval of 5000ms. This server can be considered operational after 3 consecutive successful health checks(rise), and it can only be considered dead after 5 consecutive unsuccessful health checks(fall) |

You can find more information about HAproxy configuration [here](https://cbonte.github.io/haproxy-dconv/1.7/configuration.html).

### HAProxy Dockerfile
Now we can create a Dockerfile for our HAProxy that will use our previously created configuration.

Create a file named `Dockerfile` and place the following content on it:
```dockerfile
FROM haproxy:1.7

ENV HAPROXY_USER haproxy

RUN groupadd --system ${HAPROXY_USER} \
    && useradd --system --gid ${HAPROXY_USER} ${HAPROXY_USER}

COPY haproxy.cfg /usr/local/etc/haproxy/haproxy.cfg

CMD ['haproxy', '-f', /usr/local/etc/haproxy/haproxy.cfg]
```
This Dockerfile will:

&nbsp;&nbsp;1. get a HAProxy 1.7 docker image

&nbsp;&nbsp;2. add a "haproxy" user/group in the system

&nbsp;&nbsp;3. copy our configuration file over to the container

&nbsp;&nbsp;4. run HAProxy with our configuration

Now we need to build the image:
```sh
$ docker build -t haproxy-rabbitmq-cluster:1.7 .
```
You can check your newly created image by running `$ docker images`
```sh
REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
haproxy-rabbitmq-cluster   1.7                 fbe7dc7dddad        23 hours ago        137MB
haproxy                    1.7                 4bb854517f75        3 weeks ago         136MB
rabbitmq                   3-management        fb11f4e0a6b6        2 months ago        124MB
```

## Linking everything with Docker Compose
Create a file named `docker-compose.yml` and place the following content on it:
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
Our compose file has 4 services and 1 network:
- **rabbitmq-node-1:** creates a container based on a RabbitMQ image, defines its hostname, exposed ports, networks, storage volumes and some specific RabbitMQ environment variables
- **rabbitmq-node-2:** the same for node 2
- **rabbitmq-node-3:** the same for node 3
- **haproxy:** creates a container based on our HAproxy image, exposes port "5672" for TCP communication with RabbitMQ servers, and port "1936" for accessing HAProxy statistics portal
- **cluster-network:** creates a private internal network named "cluster-network" that enables the containers to communicate with each other

Now we can run docker compose:
```sh
$ docker-compose up -d
```
You can view your containers running, just execute `$ docker ps`:
```
CONTAINER ID        IMAGE                          COMMAND                  CREATED             STATUS              PORTS                                                                     NAMES
20438b543486        haproxy-rabbitmq-cluster:1.7   "/docker-entrypoin..."   24 hours ago        Up 24 hours         0.0.0.0:1936->1936/tcp, 0.0.0.0:5672->5672/tcp                            haproxy
ae3c75f20878        rabbitmq:3-management          "docker-entrypoint..."   24 hours ago        Up 22 hours         4369/tcp, 5671-5672/tcp, 15671/tcp, 25672/tcp, 0.0.0.0:15672->15672/tcp   rabbitmq-node-1
193ab53159c4        rabbitmq:3-management          "docker-entrypoint..."   24 hours ago        Up 24 hours         4369/tcp, 5671-5672/tcp, 15671/tcp, 25672/tcp, 0.0.0.0:15673->15672/tcp   rabbitmq-node-2
227d2d32d86e        rabbitmq:3-management          "docker-entrypoint..."   24 hours ago        Up 23 hours         4369/tcp, 5671-5672/tcp, 15671/tcp, 25672/tcp, 0.0.0.0:15674->15672/tcp   rabbitmq-node-3
```

## Creating a cluster
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
