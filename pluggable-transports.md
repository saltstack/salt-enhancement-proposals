- Feature Name: Pluggable Transports
- Start Date: 2021-09-01
- SEP Status: Draft
- SEP PR: 
- Salt Issue:

# Summary
[summary]: #summary

Most subsystems in Salt are pluggable.  For example, adding new External Pillar modules, Proxy Minion types, and SDB backends are all well documented and leveraged extensively.  However, while some work has been done, Salt transports are not currently pluggable.  At the moment Salt effectively has two transports: TCP, and ZeroMQ.  There is great value in exploring other transport types including things like [NATS](https://nats.io), [Kafka](https://kafka.apache.org), [RabbitMQ](https://www.rabbitmq.com).

# Motivation
[motivation]: #motivation

ZeroMQ has served the Salt Project well.  In 8 years of the project, very few failures of Salt can be tied to ZeroMQ directly.  Most transport related problems are due to things like misconfiguration, network issues, and filesystem corruption.  ZeroMQ is very fast and basically "just works".

As Salt continues to expand into larger and larger environments, ZeroMQ presents problems that are harder to solve.  For example:

1. Salt's High Availability and Disaster Recovery scenarios are not as robust as they could be.  Hot-hot masters can be an effective HA/DR tool but introduce their own issues.  Failover masters don't always failover the way you anticipate.
2. We often talk about ZeroMQ as being a "brokerless" transport but that's not entirely true in a Salt environment.  If the master goes away for some reason, the event bus also goes away.
3. While we used to recommend syndics as a solution to complex Salt topologies in recent years we are deprecating this approach and suggest users apply the patterns offered by the Salt Enterprise product (now known as vRA SaltStack Config post VMware acquisition).  This doesn't solve some of the latency, deployment, and scaling issues that customers encounter.  One example involves customers that want to bridge several on-prem datacenters with presences in the cloud.  This turns out to generate a great deal of complexity and reliability challenges.
4. ZeroMQ has always been a black box and the ZeroMQ project has resisted adding observability hooks, preferring instead to spend effort on the "just works" part of the library and/or recommending that library users layer their own observability on top.  But Salt customers continue asking questions like "is Salt dropping events? Is my bus 'overloaded'?  How do I know when I should add another master and split my minions between them?  How can I predict my event bus traffic so I can know that my Salt infrastructure will perform in a crisis?"  In general all we have had to offer them is rules of thumb and estimates.

In some cases we can solve these problems by adding additional features inside the Salt codebase.  But other scenarios are impossible without direct transport support.

Rather than trying to fix our codebase for every scenario, it would be wiser to adapt Salt to make it easier to add transports, and then leverage the features of those transports where appropriate.  This also means that users that do not need other transport features are not saddled with the additional code/configuration burden.  ZeroMQ will remain the primary transport in Salt, making it easy for users to get started with Salt and scale it significantly before needing to be concerned with other options.

Now it makes little sense to do the work to make transports pluggable without a good proof of concept.  This SEP also proposes that RabbitMQ would be the first additional pluggable transport.

# Design
[design]: #detailed-design

This is the bulk of the SEP. Explain the design in enough detail for somebody familiar
with the product to understand, and for somebody familiar with the internals to implement. It should include:

- Definition of any new terminology
- Examples of how the feature is used.
- Corner-cases
- A basic code example in case the proposal involves a new or changed API
- Outline of a test plan for this feature. How do you plan to test it? Can it be automated?

## Alternatives
[alternatives]: #alternatives

As part of early PoC efforts we also created Salt engines that bridge the ZeroMQ transport to other event buses.  This can be an effective approach, and some of these bridges will persist as parts of the product so customers can run some parts of their infrastructure with ZeroMQ and others with other transports.  Like any engineering solution these have tradeoffs (performance, duplication of network traffic, potential for misconfiguration).


## Unresolved questions
[unresolved]: #unresolved-questions

- Discuss auth concerns here?  Where will we keep keys?  Do we need a shared keystore?  How much do we talk about the auth refactor?

# Drawbacks
[drawbacks]: #drawbacks

Some reasons to NOT add pluggable transports mostly center around adding code complexity, introducing bugs during the refactor of various chunks of code, and figuring out how best to add tests with proper separation of concerns.  
