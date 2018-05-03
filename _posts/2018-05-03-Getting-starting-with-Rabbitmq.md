---
title: Getting started with Rabbitmq
categories: [rabbitmq]
category: rabbitmq
last_modified_at: 2018-05-02 22:10:01 0100
---
# RabbitMQ Introduction

## Virtual Hosts

Virtual hosts provide logical grouping and separation of resources. Separation of physical resources is not a goal of virtual hosts and should be considered an implementation detail.

Connections to a vhost can only operate on exchanges, queues, bindings, and so on in that vhost. "Interconnection" of e.g. a queue and an exchange in different vhosts is only possible when an application connects to two vhosts at the same time.

## Lazy Queues

Since RabbitMQ 3.6.0, the broker has the concept of Lazy Queues - queues that move their contents to disk as early as practically possible, and only load them in RAM when requested by consumers, therefore the lazy denomination. This generally will result in higher disk I/O utilisation.

One of the main goals of lazy queues is to be able to support very long queues (many millions of messages). Queues can become very long for various reasons:

* consumers are offline / have crashed / are down for maintenance
* there is a sudden message ingress spike, producers are outpacing consumers
* consumers are slower than normal

### Making a Queue Lazy

Queues can be made to run in default mode or lazy mode by:

* setting the mode via queue.declare arguments
* applying a queue policy

## Persistence configuration

* Persistence Messages: ils sont persistés dès leur arrivé dans la Queue et ils sont gardés en mémoire.
* Messages transisent: Ils sont persistés que quand la mémoire est saturée.

The persistence layer has two components: the queue index and the message store.
The queue index is responsible for maintaining knowledge about where a given message is in a queue, along with whether it has been delivered and acknowledged. There is therefore one queue index per queue.

The message store is a key-value store for messages,

## Parameters and Policies

### Parameters

There are 2 kinds of parameters:

* vhost-scoped parameters
* global parameters.

### Policies

Key policy attributes are

name: it can be anything but ASCII-based names without spaces are recommended
pattern: a regular expression that matches one or more queue (exchange) names. Any regular expression can be used.
definition: a set of key/value pairs (think a JSON document) that will be injected into the map of optional arguments of the matching queues and exchanges

Policies can match only queues, only exchanges, or both. This is controlled using the apply-to flag when a policy is created.

Policies can be used to configure the federation plugin, mirrored queues, alternate exchanges, dead lettering, per-queue TTLs, and maximum queue length.

Example

```http
PUT /api/policies/%2f/federate-me
                    {"pattern": "^amq\.",
                     "definition": {"federation-upstream-set":"all"},
                     "priority": 1,
                    "apply-to": "exchanges"}
```

## Reliability

### Connection Failures

In general when connections fail, the client will be informed by the connection throwing an exception (or similar language construct). .NET client provides IConnection.ConnectionShutdown and IModel.ModelShutdown events for the same purpose.

### Acknowledgements and Confirms

Acknowledgements can be used in both directions - to allow a consumer to indicate to the server that it has received / processed a message and to allow the server to indicate the same thing to the producer. RabbitMQ refers to the latter case as a "confirm".

### Clustering and High Availability

If we need to ensure that our broker survives hardware failure, we can use RabbitMQ's clustering. In a RabbitMQ cluster, all definitions (of exchanges, bindings, users, etc) are mirrored across the entire cluster. **Queues behave differently, by default residing only on a single node, but optionally being mirrored across several or all nodes**. Queues remain visible and reachable from all nodes regardless of where they are located.

### At the Producer

When using confirms, producers recovering from a channel or connection failure should retransmit any messages for which an acknowledgement has not been received from the broker.

#### Ensuring Messages are Routed

To ensure messages are routed to a single known queue, the producer can just declare a destination queue and publish directly to it. If messages may be routed in more complex ways but the producer still needs to know if they reached at least one queue, it can set the mandatory flag on a `basic.publish`, ensuring that a `basic.return` (containing a reply code and some textual explanation) will be sent back to the client if no queues were appropriately bound.

### At the consumer

If a message is delivered to a consumer and then requeued (because it was not acknowledged before the consumer connection dropped, for example) then RabbitMQ will set the `redelivered` flag on it when it is delivered again (whether to the same consumer or a different one).

> **Consumer Cancel Notification**

>**Messages That Cannot Be Processed**: if a consumer determines that it cannot handle a message then it can reject it using basic.reject (or basic.nack), either asking the server to requeue it, or not (in which case the server might be configured to dead-letter it instead.

## Detecting Dead TCP Connections with Heartbeats and TCP Keepalives

Network can fail in many ways, sometimes pretty subtle (e.g. high ratio packet loss). Disrupted TCP connections take a moderately long time (about `11 minutes with default configuration on Linux`, for example) to be detected by the operating system. AMQP 0-9-1 offers a heartbeat feature to ensure that the application layer promptly finds out about disrupted connections (and also completely unresponsive peers). Heartbeats also defend against certain network equipment which may terminate "idle" TCP connections when there's no activity on them for a certain period of time.
> Values within the 5 to 20 seconds range are optimal for most environments.

> Enabling Heartbeats with the .NET Client

```csharp
var cf = new ConnectionFactory();

// set the heartbeat timeout to 60 seconds
cf.RequestedHeartbeat = 60;
```
