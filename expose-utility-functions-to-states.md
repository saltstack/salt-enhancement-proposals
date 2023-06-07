- Feature Name: Expose utility functions to states, but not to the CLI or template context
- Start Date: 2023-06-06
- SEP Status: Draft
- SEP PR: https://github.com/saltstack/salt-enhancement-proposals/pull/70
- Salt Issue: https://github.com/saltstack/salt/pull/64287#pullrequestreview-1451646261

# Summary
[summary]: #summary

Provide a means for the loader to access utility functions, but keep them from
being added to the CLI or template context.

# Motivation
[motivation]: #motivation

It is a policy of the Salt core team not to add any new utility functions to
the `__salt__` loader (thus exposing them to the Salt CLI and the template
context). The core team's proposed alternative, however, is to do an OS check
in the state module, then a late import of the salt execution module, and
finally to invoke the utility function directly. While this option technically
works, it is unequivocally a bad idea, with several complications:

1. The state modules were never supposed to include platform-specific code. I
   was even [advised to
   remove](https://github.com/saltstack/salt/pull/3019#issuecomment-11680999)
   grains checks from a state module some 10 years ago.

2. Doing an OS check in a state module duplicates work already done in the
   execution module's `__virtual__` function. If the conditions in that
   `__virtual__` function are changed at a later date, all ad-hoc duplications
   of this logic in state modules must be located and updated. It is only a
   matter of when, not if, this will cause bugs to arise.

3. For execution modules which are virtual (e.g. `service`, `pkg`, etc.),
   additional care needs to be taken to craft these ad-hoc duplications, in
   order to catch minions which use the
   [providers](https://docs.saltproject.io/en/latest/ref/configuration/minion.html#providers)
   config option to manually assign a module as the virtual provider. If this
   very obscure and specific case isn't manually accounted for in one's ad-hoc
   platform checks,
   [providers](https://docs.saltproject.io/en/latest/ref/configuration/minion.html#providers)
   functionality is broken.

In the past, complications `2` and `3` above were unnecessary, as we would
simply add a function to the platform-specific execution module, and then use
loader membership (e.g. `if "pkg.somefunc" in __salt__`) to determine whether
or not to run them. The `pkg` state module is littered with such loader checks
to handle the various idiosyncrasies of different package managers. This method
has the benefit of automatically supporting minions which use
[providers](https://docs.saltproject.io/en/latest/ref/configuration/minion.html#providers),
and keeps duplicated platform checks out of state modules, at the expense of
exposing dubiously-useful functions to the CLI and template context.

# Design
[design]: #detailed-design

## Option 1: Augmentation of LazyLoader
[option-1]: #option-1

An optional module-level dunder could be added to remote-execution modules, to
suppress availability of utility functions. For example:

```python
__utility_funcs__ = ('foo', 'bar')
```

Any function name found in `__utility_funcs__` would be ignored by the loader
by default, but they could be conditionally loaded by passing an argument when
instantiating the `LazyLoader` class. This would allow `salt.loader.states()`
to instantiate its own copy of the `minion_mods` loader that contains the
utility functions, while the CLI and template context get a copy with them
absent, preventing them from being exposed to end users.

## Option 2: Move the utility functions to a new loader type
[option-2]: #option-2

This solution is far less elegant and more intrusive, but I'm including it as
an alternative. It was my first idea for solving this problem, but I believe
that [Option 1](#option-1) is the superior solution.

Utility functions would be moved to a different module, which would be in a
directory processed by a loader instance. That loader would be added to the
dunders which get packed onto the state modules.  Meanwhile, the execution
modules can import these utility functions directly.

The obvious first choice would be to use `__utils__` for this purpose, but I
noticed there is already [a SEP to remove the `__utils__`
loader](https://github.com/saltstack/salt-enhancement-proposals/pull/66). So,
a different directory then? Assuming this directory is called `foo`, then you
would for example have a `salt/modules/aptpkg.py` and a `salt/foo/aptpkg.py`.

Splitting the code into separate modules for remote-execution and utility
functions would appear at first to result in a duplicated `__virtual__`
function, but objects beginning and ending in double-underscores are not
name-mangled and thus can be imported. So, the `__virtual__` can be defined in
the utility module and imported into the remote-execution module.

## Alternatives
[alternatives]: #alternatives

This SEP already contains two potential implementations. The alternative to
implementing this SEP would be to duplicate the logic from the `__virtual__` at
every location where utility functions need to be used, which is an open
invitation for the two (or more) copies of that logic to drift from one
another.

## Unresolved questions
[unresolved]: #unresolved-questions

- For [Option 1](#option-1), Salt once had a test which parsed all docstrings
  in every execution module, ensuring that CLI examples were included. I'm not
  sure if this test still exists, but utility functions would not need CLI
  examples and this test case (if still present) would need to be updated to
  account for this fact.

- As noted above for [Option 2](#option-2), with the fate of `__utils__` in
  question, the actual name of the hypothetical loader type is unknown.

# Drawbacks
[drawbacks]: #drawbacks

- For [Option 1](#option-1), this means that `states` loader instances would no
  longer be re-using the existing `minion_mods` loader instance that is used by
  the CLI and tempalte context, and would need to generate its own to use as
  its `__salt__` dunder. The performance impact of this should be minimal,
  however.

- For [Option 2](#option-2), adding a new loader type is a non-trivial
  undertaking, so this would be a much less-attractive solution.
