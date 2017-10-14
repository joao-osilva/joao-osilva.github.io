---
layout: post
title: "Cloud Messaging with RabbitMQ and AMQP"
author: "João Silva"
categories: technology
tags: [technology]
image:
  feature: rabbit.jpg
---
A typical corporation generally have a large number of different systems, that must communicate with each other in order to implement a defined business process.

<img src="/assets/img/enterprise-message-1.png" width="350">

In a scenario like this, some key factors must be considered before deciding how this communication will be implemented:

**Loose coupling:** does the systems depend on each other or can operate independently?

**Real-time workload processing:** how fast is the communication between the systems?

**Scalability:** how does the system scale when more systems are added and the workload increases?

**Maintainability:** how hard is to maintain the integrated systems?

**Extensibility:** how easy is to integrate new systems?
&nbsp;&nbsp;
## Enterprise message system
<img src="/assets/img/enterprise-message-2.png" width="350">

In an EMS(or Message-oriented Middleware) a message is a first-class citizen, or the central unit of communication if you will. It consists in:

**Header:** has some metadata about the message.

**Body:** has the actual data, represented in a specific format.

- The message system provides:
    - Mechanisms to validate, store, route and transform messages
  - Each system creates a message based on the protocol defined by the broker
  - Clustering capabilities
  - Implements different types of communication protocols

Some of the communication patterns used with it are:

**Point-to-Point:** One sender and one receiver. The sender does not receive a response.

**Publish-Subscribe:** One sender and multiple receivers(subscribers). The sender does not await for a response once the message is sent to the broker.

**Request-Reponse:** One sender and one receiver, that sends a response to the sender of the message.
&nbsp;&nbsp;
## Advanced Messaging Queueing Protocol(AMQP)
AMQP comes from the financial sector and it was conceived as a co-operative effort, started in 2003 by JPMorgan Chase.

The great advantage behind AMQP, is that it is a wire-level protocol, meaning that it's only a description of the format of the data(as a stream of bytes) that is sent across the network.

So any tool that create/interpret messages that conform with this data format, can interoperate regardless of its implementation language.

#### The AMQP 0.9.1 model
<img src="/assets/img/amqp-model.png" width="500">
- The workflow consists in:
  - Messages that are published to exchanges
  - These exchanges distribute the messages to queues based on a binding(rule)
  - The consumers then fetch/pull messages from these queues

Some of the model entities are:

**Exchanges:** Entities where the producer sends messages. The exchanges will use bindings to route the message to the correct queue.

**Bindings:** Each binding is a rule that specifies how the exchanges should route messages to queues.

**Queues:** Store the messages(in memory or on disk) coming from one or more exchanges until they are consumed by applications.
