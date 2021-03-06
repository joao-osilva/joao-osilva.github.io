---
layout: post
title: "Enterprise Messaging with RabbitMQ and AMQP"
author: "João Silva"
categories: technology
tags: [technology, message-oriented-middleware, rabbitmq, amqp]
image:
  feature: rabbit.jpg
---
I think it's reasonable to assume that any company that has a well defined business process, even complex at times, struggles at some point with the communication hell generated by the integration of so many different systems.

I would go further and say that this is the quintessential problem in the Cloud-era, where everything is distributed, and must somehow be able to spread the information across the globe.

In a scenario like this, some key factors must be considered before deciding how this communication will be implemented:

**Loose coupling:** does the systems depend on each other or can operate independently?

**Real-time workload processing:** how fast is the communication between the systems?

**Scalability:** how does the system scale when more systems are added and the workload increases?

**Maintainability:** how hard is to maintain the integrated systems?

**Extensibility:** how easy is to integrate new systems?

## Enterprise message system to the rescue
<div class="all-img">
<img src="/assets/img/enterprise-message-2.png">
</div>

In an Enterprise message system(or Message-oriented Middleware) a message is a first-class citizen, or the central unit of communication if you will. It consists in:

**Header:** has some metadata about the message.

**Body:** has the actual data, represented in a specific format.

The message system provides:

&nbsp;&nbsp;1. Mechanisms to validate, store, route and transform messages

&nbsp;&nbsp;2. Each system creates a message based on the protocol defined by the broker

&nbsp;&nbsp;3. Clustering capabilities

&nbsp;&nbsp;4. Implements different types of communication protocols

Some of the communication patterns used with it are:

**Point-to-Point:** One sender and one receiver. The sender does not receive a response.

**Publish-Subscribe:** One sender and multiple receivers(subscribers). The sender does not await for a response once the message is sent to the broker.

**Request-Reponse:** One sender and one receiver, that sends a response to the sender of the message.

## Advanced Messaging Queueing Protocol(AMQP)
AMQP comes from the financial sector and it was conceived as a co-operative effort, started in 2003 by JPMorgan Chase.

The great advantage behind AMQP, is that it is a wire-level protocol, meaning that it's only a description of the format of the data exchanged across the network.

So any tool that create/interpret messages that conform with this data format, can interoperate regardless of its implementation language.

The AMQP 0.9.1 model consists in:
<div class="all-img">
<img src="/assets/img/amqp-model.png">
</div>

&nbsp;&nbsp;1. Messages that are published to exchanges

&nbsp;&nbsp;2. These exchanges distribute the messages to queues based on a binding(rule)

&nbsp;&nbsp;3. The consumers then fetch/pull messages from these queues

Some of its entities are:

**Exchanges:** Entities where the producer sends messages. The exchanges will use bindings to route the message to the correct queue.

**Bindings:** Each binding is a rule that specifies how the exchanges should route messages to queues.

**Queues:** Store the messages(in memory or on disk) coming from one or more exchanges until they are consumed by applications.

The model also provide 4 types of exchanges:

**Direct exchange:** A one-to-one relationship with a queue through its binding. There is default exchange(that is a direct exchange) that uses a queue's name as a routing key for its binding.

<div class="all-img"><img src="/assets/img/default-exchange.png">
<img src="/assets/img/direct-exchange.png"></div>

**Fanout exchange:** Delivers a message to all the queues that are bound to the exchange. It can be used as broadcast mechanism, similar to the publish/subscribe pattern.

<div class="all-img">
<img src="/assets/img/fanout-exchange.png">
</div>

**Topic exchange:** Is similar to the direct exchange, the only difference is that it accepts wildcards(\*) for the routing keys.

<div class="all-img">
<img src="/assets/img/topic-exchange.png">
</div>

**Headers exchange:** It will route the message based on the message header attributes. You need to indicate whether you want the headers to match exactly by adding x-match:all(header:key), or match any by adding x-match:any (header:any).

<div class="all-img">
<img src="/assets/img/headers-exchange.png">
</div>

## RabbitMQ
An Erlang implementation of a message broker, basically. It has all the Erlang's native support for building highly-reliable and distributed applications, and it can run on any OS. It implements version 0-9-1 of AMQP with some extensions.

Some of its features are:

**Support for multiple protocols:** AMQP, STOMP, MQTT and HTTP

**Support for multiple programming languages:** a variety of supported clients

**Reliable delivery:** guarantees successful message delivery using acknowledgements.

**Clustering:** provides a mechanism to implement scalable applications

**Federation:** an alternative mechanism to implement scalable applications by transferring messages between exchanges and queues in different broker instances without the need to create a RabbitMQ cluster

**High availability:** ensures that if a broker fails, communication will be redirected to a different broker instance.

**Pluggable architecture:** provides a mechanism to extend its functionalities with RabbitMQ plug-ins

That's it for today, I'll do another post in the future showing how to set up RabbitMQ on docker.
