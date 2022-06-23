- Feature Name: deprecate-dunder-utils
- Start Date: 20022-06-23
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

Deprecate and remove all usage of the ``__utils__`` Salt loader.

# Motivation
[motivation]: #motivation

## Why are we doing this? What use cases does it support? What is the expected outcome?

The Salt loader is a **controlled** hack around Python's import system which allows
loading(or prevent loading) of python modules which contribute to the Salt ecosystem:

* Execution modules
* State execution modules
* Runners
* etc ...

The **controlled** part of the loader is extremely important and the Salt team
goes to great lengths to maintain that controlled environment bug free and working
as supposed.

The ``__utils__`` loader was added to facilitate the utility functions usage.
For example, it helps by reducing the ammount of imports at the top of the module.
In the case where a utility module is targetted to a specific platform(for example,
Windows), we can prevent that utility module from loading on the wrong platform.

While all of this suggests it was a great idea, Salt lost the **controlled** part
on that loader.

See the following for background:

* https://github.com/saltstack/salt/pull/62006
* https://github.com/saltstack/salt/pull/62007
* https://github.com/saltstack/salt/pull/62021

Just to name a few.

Being utility functions, and given the undesired side-effects shown above, and the
potential drawdowns of packing all of them in a Salt loader(the loader does not
support multiple levels of nesting(ie, ``__utils__["utilmod.submod.func"]``),
it's wise to stop abusing the Salt loader in this way and just treat utility
functions for what they are, utility functions, which should be imported when
their use is required. And, if need be, wrap them in a ``try/except`` block
to prevent import errors when trying to load them on the platforms that they
were not meant to load.

# Design
[design]: #detailed-design

The implementation of this SEP should be done by:

* Deprecate the ``__utils__`` loader and issue warnings about that usage.
* Create an automated way to remove the ``__utils__`` loader usage and prevent
 from said usage to creep in. Done in
[salt-rewrite](https://github.com/s0undt3ch/salt-rewrite/blob/2.0.0/src/saltrewrite/salt/fix_dunder_utils.py).
* Fix any issues found by the removal of the ``__utils__`` loader usage, ie,
 import errors and any other issues found. Being addressed in
 [#62208](https://github.com/saltstack/salt/pull/62208).


## Alternatives
[alternatives]: #alternatives

None considered.

## Unresolved questions
[unresolved]: #unresolved-questions

None so far.

# Drawbacks
[drawbacks]: #drawbacks

## Why should we *not* do this? Please consider:

- Fixing all of the ``__utils__`` loader usage touches a lot of code in the Salt
 code base with potential breakage.
