- Feature Name: Pluggable Transports
- Start Date: 2021-09-01
- SEP Status: Draft
- SEP PR: 
- Salt Issue:

# Summary
[summary]: #summary

Most subsystems in Salt are pluggable.  For example, adding new External Pillar modules, Proxy Minion types, and SDB backends are all well documented and leveraged extensively.  However, while some work has been done, Salt transports are not currently pluggable.  At the moment Salt effectively has two transports: TCP, and ZeroMQ.  There is great value in exploring other transport types including things like [NATS](https://nats.io), [Kafka](https://kafka.apache.org), [RabbitMQ](https://www.rabbitmq.com).

Note there is a closed and unmerged PR that did some refactoring and added an http transport: [saltstack/salt#57210](https://github.com/saltstack/salt/pull/57210)

# Motivation
[motivation]: #motivation

ZeroMQ has served the Salt Project well.  In 8 years of the project, very few failures of Salt can be tied to ZeroMQ directly.  Most transport related problems are due to things like misconfiguration, network issues, and filesystem corruption.  ZeroMQ is very fast and basically "just works".

As Salt continues to expand into larger and larger environments, ZeroMQ presents problems that are harder to solve.  For example:

1. Salt's High Availability and Disaster Recovery scenarios are not as robust as they could be.  Hot-hot masters can be an effective HA/DR tool but introduce their own issues.  Failover masters don't always failover the way you anticipate.
2. We often talk about ZeroMQ as being a "brokerless" transport but that's not entirely true in a Salt environment.  If the master goes away for some reason, the event bus also goes away.
3. While we used to recommend syndics as a solution to complex Salt topologies in recent years we are deprecating this approach and suggest users apply the patterns offered by the Salt Enterprise product (now known as vRA SaltStack Config post VMware acquisition).  This doesn't solve some of the latency, deployment, and scaling issues that customers encounter.  One example involves customers that want to bridge several on-prem datacenters with presences in the cloud.  This turns out to generate many complexity and reliability challenges.
4. ZeroMQ has always been a black box and the ZeroMQ project has resisted adding observability hooks, preferring instead to spend effort on the "just works" part of the library and/or recommending that library users layer their own observability on top.  But Salt customers continue asking questions like "is Salt dropping events? Is my bus 'overloaded'?  How do I know when I should add another master and split my minions between them?  How can I predict my event bus traffic so I can know that my Salt infrastructure will perform in a crisis?"  In general all we have had to offer them is rules of thumb and estimates.

In some cases we can solve these problems by adding additional features inside the Salt codebase.  But other scenarios are impossible without direct transport support.

Rather than trying to fix our codebase for every scenario, it would be wiser to adapt Salt to make it easier to add transports, and then leverage the features of those transports where appropriate.  This also means that users that do not need other transport features are not saddled with the additional code/configuration burden.  ZeroMQ will remain the primary transport in Salt, making it easy for users to get started with Salt and scale it significantly before needing to be concerned with other options.

Now it makes little sense to do the work to make transports pluggable without a good proof of concept.  This SEP also proposes that RabbitMQ would be the first additional pluggable transport.

# Design
[design]: #detailed-design

## Deployment/Configuration
Salt already has a concept of configurable transport (e.g. "zeromq" or "tcp") and it is set in master/minion config files. The proposal is to introduce a new transport of type "rabbitmq" and also include any additional metadata such as broker address, vhost, credentials, etc. 
RMQ broker may be deployed by Salt or in some cases it may be a shared enterprise resource that Salt can be pointed to. 

## Abstractions
Salt transport is already mostly abstracted as a client/server interface with a factory pattern. A factory pattern is used to instantiate a specific instance of client/server. 
The proposal is to implement the interfaces for RMQ. See https://github.com/saltstack/salt/tree/master/salt/transport.
Note that as part of this effort we have an opportunity to clean up the interfaces (e.g. decouple auth from data channel) and we shall do so opportunistically. 

## RabbitMQ topology
Management and use of RabbitMQ objects (vhosts, users, exchanges, queues, topics, etc.): 

* VHOST/tenant, user, permission configuration is performed by the tenant admin persona or equivalent; master/minion read pertinent configuration from config files or equivalent. A master/minion belong to a single tenant/vhost.
* Minions running in customer's environment should be considered potential malicious actors (e.g. initiators of message flood). Consider the blast radius if/when this occurs. The impact should not transcend the org/tenant that minion belongs to.
   * Use permissions to limit minion's access to queues and exchanges.
   * Configure maximum queue size (several thousand messages)

* Create 1 x Fanout exchange/queue used for message publishing, one per master cluster.
* Create 1 x Fanout exchange/queue used to send requests/commands to minions, one per master cluster.
* [Optional] Create 1 x exchange/queue for initial master/minion authentication. Once authenticated, the minion can switch to another channel for communication.
* Set "auto-ack" to True for consuming messages. This gives us parity with ZeroMQ. There are potentially areas where bringing the ack up the stack could be useful, but we do not know yet where. 
* Use reply queues (with auto_delete=True) and correlation ids to correlate request/response. See https://www.rabbitmq.com/tutorials/tutorial-six-dotnet.html).
  *  Note that we need to scale to 50K minions per vhost, so need to be careful about the number of queues we create.

* Associate a "dead-letter" exchange for each queue. Undelivered messages will end up there. This can be useful for troubleshooting. (See https://www.cloudamqp.com/blog/when-and-how-to-use-the-rabbitmq-dead-letter-exchange.html)
* Use durable quorum queues for consistency
  *   When it comes to trade-offs, we favour consistency over performance.
  *   Some features are not yet available for quorum queues, such as message TTL, message priority (see https://www.rabbitmq.com/quorum-queues.html)
* Configure message expiry (TTL) to be no longer than 72 hours. Expired messages will be delivered to the dead-letter exchange. Note that TTL can be configure per-message or per-queue.
* Artifact cleanup (queues, exchanges)
  * When object is declared with auto-delete=True, it will be deleted when last consumer dies (e.g. when all connections are closed). This is gives us parity with ZeroMQ and simplifies implementation.


## Third-party libraries
Python "pika" library (BSD license) is the industry standard for interacting with AMQP and RabbitMQ. It supports both sync and async patterns and also supports Tornado IO loops. 
The proposal is to use the "pika" library (https://pypi.org/project/pika/) to interact with RabbitMQ from Salt and use non-blocking/async-style connections as much as possible. 

## Testing 
Existing parameterized functional tests will be updated to cover rabbit as a new transport. These tests already iterate over collection of transports ("tcp", "zeromq") and encryption types ("clear, "aes").
New functional tests will be added to cover some edge cases, e.g. connection recovery in cases when RMQ broker restarts. 
A scale test will be performed to confirm the new transport can support 50K minions and message throughput of 1 job per minute x 5 messages per job x 1 minion.

## Alternatives
[alternatives]: #alternatives

As part of early PoC efforts we also created Salt engines that bridge the ZeroMQ transport to other event buses.  This can be an effective approach, and some of these bridges will persist as parts of the product so customers can run some parts of their infrastructure with ZeroMQ and others with other transports.  Like any engineering solution these have tradeoffs (performance, duplication of network traffic, potential for misconfiguration).

## Unresolved questions
[unresolved]: #unresolved-questions

- Discuss auth concerns here?  Where will we keep keys?  Do we need a shared keystore?  How much do we talk about the auth refactor?

# Drawbacks
[drawbacks]: #drawbacks

Some reasons to NOT add pluggable transports mostly center around adding code complexity, introducing bugs during the refactor of various chunks of code, and figuring out how best to add tests with proper separation of concerns.  
