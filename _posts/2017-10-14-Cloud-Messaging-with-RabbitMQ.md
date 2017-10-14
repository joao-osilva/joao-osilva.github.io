---
layout: post
title: "Cloud Messaging with RabbitMQ"
author: "João Silva"
categories: technology
tags: [technology]
image:
  feature: postoffice.jpg
---
A typical corporation generally have a large number of different systems, that must communicate with each other in order to implement a defined business process.

<img src="/assets/img/enterprise-message-1.png" width="350">

In a scenario like this, some key factors must be considered before deciding how this communication will be implemented:

**Loose coupling:** does the systems depend on each other or can operate independently?

**Real-time workload processing:** how fast is the communication between the systems?

**Scalability:** how does the system scale when more systems are added and the workload increases?

**Maintainability:** how hard is to maintain the integrated systems?

**Extensibility:** how easy is to integrate new systems?

## Enterprise message system
<img src="/assets/img/enterprise-message-2.png" width="350">

In an EMS(or Message-oriented Middleware) a message is first-class citizen, or the central unit of communication if you will. It consists in:

**Header:** has some metadata about the message.

**Body:** has the actual data, represented in a specific format.

The message system provides:

- Mechanisms to validate, store, route and transform messages
- Each system creates a message based on the protocol defined by the broker
- Clustering capabilities
- Implements different types of communication protocols

Some of the communication patterns used with it are:

**Point-to-Point:** One sender and one receiver. The sender does not receive a response.

**Publish-Subscribe:** One sender and multiple receivers(subscribers). The sender does not await for a response once the message is sent to the broker.

**Request-Reponse:** One sender and one receiver, that sends a response to the sender of the message.
