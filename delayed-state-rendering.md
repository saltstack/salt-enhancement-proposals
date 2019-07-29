- Feature Name: Delayed State Rendering
- Start Date: 2019-06-21
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

On Salt-cloud and Highstates - delayed rendering
A fundamental aspect of Salt is the ability to statefully control things by executing a highstate. Conceptually, there is a desired use for statefully creating infrastructure - not just configuring it. Much of salt-cloud is not idempotent, and even when it is, it does not easily fit in to a highstate. This can be changed. The goal is to be able to have salt-cloud / fractus be able to be embedded in a highstate that it is idempotent and informs further states. In other words, information about what has been created is available to later states to be operated on, such as everything typically in a salt-cloud full_query.

As an example, say you want a state to spawn an ec2 instance, than another to make some network configuration based on its public IP address. This conceptually fits into what you might want in a highstate, but is problematic. The public IP address is only known *after* the first state is ran that spawns the ec2 instance.

A solution is to layer rendering of the state. To be clear, this is a bit complex, and Salt doesn’t do this right now. The state could be written to contain jinja in the second state that is only evaluated after the first state has completed. The Jinja could be passed through in the state file by using Jinja escaping. This would have to be coupled with a new flag in the yaml that signals to Salt that a second rendering of this state needs to happen just before execution.

The rest of Salt could benefit from this system too. It isn’t hard to think of a state that should inform another state.

How delayed rendering is different from slots:
Slots (currently at least) only runs remote execution calls. Delayed rendering brings with it everything that Jinja typically has in a state, including logic, grains, pillars, `salt`.
Slots simply returns the result of an execution call. They do not interact with Jinja to allow any logic or parsing.

Examples:

1. Stateful hardware operations whose outcomes are not predictable. For example, writing to disk and then querying for the exact physical location on disk of a file. Writing a file to disk is stateful, but we cannot predict the location of the file on disk.
1. Any operation that outputs a random string or hash. For example, using a service that creates any kind of account and an UUID
1. Cloud operations:
