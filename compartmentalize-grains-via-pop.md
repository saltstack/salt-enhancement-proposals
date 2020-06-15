- Feature Name: compartmentalize-grains-via pop
- Start Date: 2020-06-22
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

Grains is currently a static component of salt.  We want it to be developed in a more compartmentalize fashion.
The engineers at saltstack have created kernel-specific projects that functionally replace the grains
provided by salt with POP.  We propose that this project, codenamed `corn` replaces grains in salt while
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
individual grains projects independently from salt.

# Design
[design]: #detailed-design

Grains are currently created by calling functions in the salt loader.  We would simply need to call
external POP functions to collect grains.  We need to maintain backwards compatibility with custom user grains.

- All salt grains have already been ported to and tested on the new platform.
- The new platform has many more features than the salt-grains implementation, but is compatible with salt grains.
- None of the grain names have been changed, but the output of grains is more consistent/complete between platforms.
- This will resolve many bugs that are open against grains and will migrate those bugs OUT of the salt
platform -- making salt easier to develop
- Grains in the new platform are independently and fully tested.
- Most grains tests will be able to be removed from salt; speeding up it's test suite.
- This comprises a complete rewrite of ALL grains.  Backwards compatibility is expected, but we need this to get
in early so that any unintentional issues can be resolved quickly.
- Salt releases will be able to pin idem-grain dependencies with each release.
- Users will be to update their grains implementation to a custom idem-grain version.

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
