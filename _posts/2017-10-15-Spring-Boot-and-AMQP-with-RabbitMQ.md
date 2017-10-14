---
layout: post
title: "Spring Boot and AMQP with RabbitMQ"
author: "João Silva"
categories: technology
tags: [technology]
image:
  feature: mailboxes.jpg
---

Integration between systems it's probably one of the most critical subjects in a enterprise environment. In a scenario where you have so many different components interacting, it's very easy to put yourself in a situation where you end up in a [Tower of Babel](https://en.wikipedia.org/wiki/Tower_of_Babel), with each system communicating in a different "language".

That's where the communication protocols comes handy. By allowing you to establish a common mean of communication, they prevent unnecessary transformation(of message type) between systems and make the communication more seamlessly. Besides that, most of them have implementations in a variety of technologies, so even if you're technology stack is very diverse you'll probably find an implementation it.

In a recent project I had a scenario where the communication was implemented in an asynchronous way through a message broker(RabbitMQ), using a very interesting protocol called [AMQP](https://www.amqp.org), that stands for *Advanced Messaging Queuing Protocol*.

In this post I will demonstrate how you can implement an adapter that can produce or consume messages from a message broker(RabbitMQ) using AMQP.

## Technical Stack

I was in a context where the vast majority of the applications were written in Java, so I decided to choose:

**Spring Boot:** To get a stand-alone application up and running as quickly as possible, and still be able to benefit from Spring's components.

**Spring AMQP:** A component that implements the AMQP protocol, providing templates for sending/receiving messages, listener container for asynchronous processing of messages and some admin features that provides you a higher level of abstraction when dealing with the message broker.

**RabbitMQ:** An open source message broker that implements the AMQP 0-9-1 protocol. It is written in Erlang and supports multiple protocols, programming languages, clustering, high availability, reliable delivery and so on.

**Docker:** So we can have a RabbitMQ instance locally for tests, as well as an image for our application.

## Before we start

My main focus here is the implementation itself, so I'm assuming that you are already familiar with RabbitMQ and AMQP. If you are not sure whether your knowledge is sufficient, check out my other post where I talk about [Cloud Messaging with RabbitMQ]({{ site.baseurl }}{% post_url 2017-10-14-Cloud-Messaging-with-RabbitMQ %})

## Create a Spring Boot project

Go to http://start.spring.io and create a simple Java/Maven project with AMQP and DevTools as dependencies.

## Configure RabbitMQ properties

Go to ```src/main/resources/application.properties``` and add:
```
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
```

## Implement a
