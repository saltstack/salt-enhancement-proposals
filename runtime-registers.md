- Feature Name: (runtime-registers)
- Start Date: (2020-08-25)
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

This proposal aims to add runtime variables, referred to hereafter as 'registers', to the salt execution runtime.

# Motivation
[motivation]: #motivation

A common problem I have noticed while using salt is that it does not have an obvious way to store the result of a function execution and use it as an input to another method, beyond using the fairly limited `slots`, or writing a python module. I believe this:
1) Stifles the overall utility of states, orchestrations, and thorium reactor modules.
2) Causes workarounds to be made every time this limitation is felt.

To elaborate on point 1:
I understand that salt encourages and enables users to write python more readily than say, ansible, but this is still a daunting task for many devops folk who are more comfortable with the prescribed format of using .sls files to define a custom workflow. Additionally, the way custom modules and states are stashed away under _modules and _states further makes it feel out of place.
If registers were a global feature it would allow people to use vanilla .sls to create states, orchestrations, and even execution modules that could make decisions based on the output of previous execution steps.

As for point 2:
I believe salt lacking this very fundamental feature manifests as many different symptoms depending on the problem at hand, and as such has had a compounding effect over the years -- spawning many well-meaning but narrowly scoped features that were likely created because of this missing concept.
Some clear examples would be `slots`, `mod_aggregate`, `file.accumulated`, requisites like `unless` and even `onchanges/watch`.. These are all features that would either be obsoleted or simplified by a register concept.


# Design
[design]: #detailed-design

This proposal piggybacks on the thorium concept of registers, which already proves out a fair amount of the concept to some degree, though it is missing the final step of interpolating the registers in commands.

With that in mind: I propose that the salt runtime keeps a global dictionary of registers that can be accessed from any .sls step at runtime to either read to or write from. This dictionary could be accessed directly by modules, but more importantly the state definition itself could also refer to the registers and use them as function arguments, and/or nominate them as a named return values.

Let's start by looking at a common use case.
Here's a simple example to run a command to find a files, and then move it to another location:
```sls
# Capture a result into a register.
Find some file a script might have thrown in /tmp:
  module.run:
    - file.find:
      - /tmp
      - type: f
      - name: 'some-binary-v*'
    - register: output_of_find

# Use a helper state to make runtime decisions, if needed
Check if we found something:
  check.changed_if:
    - {| output_of_find | first |}
    - expected: not None
    - comment: "Yep it's there"

# the 'check' module has translated the register into a normal result,
# so we can trigger other tasks, events, alerts... and use the register in them:
Move file:
  onchanges: Check if we found something
  file.copy:
    - name: /usr/bin/some-binary
    - source: {| output_of_find | first |}
    - force: true
  file.absent:
    - name: {| output_of_find | first |}
```

And with this we've built a find-and-move state.


## Alternatives
[alternatives]: #alternatives

Further enhance `slots` to be able to recall results in multiple locations without rerunning the function, make it able to collect multiple execution outputs, have the result evaluated/tested, and be processed with jinjas' NativeEnvironment to allow filtering and formatting.

I wouldn't mind this approach but I feel like registers model the problem and solution more correctly, by integrating it more as a normal part of the states as they run.

## Unresolved questions
[unresolved]: #unresolved-questions

What do we use to interpolate the registers? I personally think jinja's NativeEnvironment works really well, and have proposed something already in thorium: (LINK)

How advanced do we want the 'check' helper module to be? If jinja can perform the actual test, we don't need to mimic complicated tests like loop.until_no_eval does. However I can still think of some features to improve the use of registers.

How advanced does variable capture need to be? We can extract from a list/dict upon registering the new value in order to save on complexity later. We can also append or merge results into an existing register. I think these are both very useful features.

# Drawbacks
[drawbacks]: #drawbacks

- Users may find the concept clashes with existing features, especially slots.
- Will need a lot of documentation.
- Initial implementation effort may be large, though it should be easy to maintain.
