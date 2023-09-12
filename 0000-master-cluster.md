- Feature Name: Master Cluster
- Start Date: 2023-08-09
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

Add the ability to create a cluster of Masters that run behind a load balancer.

# Motivation
[motivation]: #motivation

The current [high availability features](https://docs.saltproject.io/en/latest/topics/highavailability/index.html) in the Salt ecosystem allow minions to have back up masters. There are two flavors of Multi Master which can be configured on a Minion.

Minions can connect to [multiple masters simultaneously](https://docs.saltproject.io/en/latest/topics/tutorials/multimaster.html).

<img src='/diagrams/000-multi-master.png' width='400px'>

Minions can also be configured to connect to one master at a time [using fail over](https://docs.saltproject.io/en/latest/topics/tutorials/multimaster_pki.html#multiple-masters-for-a-minion).

<img src='/diagrams/000-multi-master-failover.png' width='400px'>

This results in jobs targeting lots of minions being pinned to a single master. Another drawback to the current HA implementation is that minions need to be re-configured to add or remove masters.


<img src='/diagrams/000-mm-large-job.png' width='400px'>

It would be much more ideal if jobs could scale across multiple masters.


<img src='/diagrams/000-mc-large-job.png' width='400px'>

# Design
[design]: #detailed-design

In order to accomplish this, we will need to change the way jobs execute.
Currently new jobs get sent directly to the publish server from the request
server.

<img src='/diagrams/000-current-job-pub.png' width='400px'>

If we forward IPC Events between Masters, we can get the return flow to be shared, as shown below:


<img src='/diagrams/000-cluster-job-pub.png' width='400px'>

To get job publishes to work, we need to make sure publishes also travel over the IPC Event bus.


<img src='/diagrams/000-cluster-fwd.png' width='400px'>

Jobs can come and go through all the masters in our master pool. From a minion's perspective, all of the masters in our pool are completely the same. We can remove the need of minions to know about multiple masters by putting our pool behind a load balancer. Minions will not need to be re-configured to add master resources.


<img src='/diagrams/000-cluster-arch.png' width='400px'>


Events from master's including job returns are sent to all masters in the cluster. This requires that all masters run on stable local network.

<img src="/diagrams/000-master-event-bus.png" width="400px">

> [!IMPORTANT]
> The current work for this SEP can be found [here](https://github.com/saltstack/salt/pull/64936)


## Alternatives
[alternatives]: #alternatives

We currently have two alternatives to achieve "high availablity". This is a
third, more robust approach that alleviates the issues with the current
options. This is not intending to deprecate the current HA functionality.


## Unresolved questions
[unresolved]: #unresolved-questions

None as of this time.

# Drawbacks
[drawbacks]: #drawbacks

The biggest drawback is the fact that we will need to maintain three ways of
doing HA. This adds complexity however, if successfull. We can potentially
depericate some of, or all of, the exiting HA functionality.
