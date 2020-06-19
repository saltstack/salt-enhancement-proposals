- Feature Name: compartmentalize-grains-via pop
- Start Date: 2020-06-22
- SEP Status: Draft
- SEP PR: https://github.com/saltstack/salt/pull/57681
- Salt Issue: https://github.com/saltstack/salt/issues/57680

# Summary
[summary]: #summary

Grains is currently a static component of salt.  We want it to be developed in a more compartmentalize fashion.
The engineers at SaltStack have created platform-specific projects that functionally replace the grains
provided by salt with POP.  We propose that this project, codenamed `grainsv2` becomes available in salt while
maintaining backwards compatibility.

# Motivation
[motivation]: #motivation

"The original implementation of grains was not designed to scale to the level that salt requires" - thatch45
Tom wrote POP to solve this problem and we're going to use it. 

By using Plugin Oriented Programming, grains can be separated into individual projects that can be maintained
by the community safely.

This also solves the issue of `core.py` being an unmaintainable monolith.  It allows platform specific grains
to be independently maintained.

Working group leaders and other community members focused on specific platforms can manage the release cycle for
individual idem-platform projects independently from salt.

# Design
[design]: #detailed-design

Grains are currently created by calling functions in the salt loader.  We would simply need to call
external POP functions to collect grains.  We need to maintain backwards compatibility with custom user grains.

- All salt grains have already been ported to and fully tested on the `grainsv2`.
- `grainsv2` has many more features than the salt-grains implementation, but is compatible with salt grains.
- None of the grain names have been changed, but the output of grains is more consistent/complete between platforms.
- This will resolve many bugs that are open against grains and will migrate those bugs OUT of the salt
platform -- making salt easier to develop
- Grains in the new platform are independently and fully tested.
- Development will freeze on core.py and the salt/grains folder.
- This comprises a complete rewrite of ALL grains.  Backwards compatibility is expected, but we need this to get
in early so that any unintentional issues can be resolved quickly.
- Salt releases will aggressively pin `grainsv2`/idem-platform dependencies with each release.
- Users will be to update their grains implementation to a custom idem-grain version.
- The use of `grainsv2` will be gated with a config variable; It will be disabled by default.
- New development for grains will be done in `grainsv2` projects and core.py will remain static.
- When `grainsv2` gains maturity in salt on the other side of the gate (and python3.5 is dropped) it will completely replace core.py

## Alternatives
[alternatives]: #alternatives

Keep things the way they are.  Does this mean we would suffer in stagnation?  Will we be stuck maintaining a monolith.

## Unresolved questions
[unresolved]: #unresolved-questions

What will further adoption of POP mean inside of salt?  The purpose of this sep and these features is not to provide
a definitive answer to this question -- rather to introduce plugin oriented programming into salt in a very safe way.
In the future this will determine the design so that salt can be further compartmentalized and written in POP.

# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this? Please consider:

- We will be introducing a number of new dependencies.
- The new implementation of grains/pop is written with python3.6 and later with asyncio
(The latest release of salt is also python3.6 and later).
- Some grains for specific OSes (such as SUSE) implement some grains differently from other platforms.  We need
to set up a conversation with the relevant parties to resolve such conflicts.
- Documentation will be the heaviest lifting.  We need to document how to make salt-style internal grains and generally
available idem-grains.
