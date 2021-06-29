- Feature Name: (runtime-registers)
- Start Date: (2020-08-25)
- SEP Status: Draft
- SEP PR: https://github.com/saltstack/salt-enhancement-proposals/pull/33
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

I propose that the salt runtime keeps a global dictionary of registers that can be accessed from any .sls step during their execution to either read from or write to. This dictionary could be accessed directly by modules as a global var, but more importantly -- the state definition itself could refer to the registers and interpret them as function arguments, and/or nominate them to be filled with a function return value. This would be done by using jinja's NativeEnvironment interpolation, and a single global state option for capturing return values, called 'register'.

This proposal piggybacks on thorium's (and ansible's) concept of registers, which already proves out a fair amount of the concept to some degree, though thorium is missing the final step of interpolating the registers in commands. See [this PR](https://github.com/saltstack/salt/pull/58198) for a fix, and initial PoC to this proposal.


This is easier to understand by looking at a common case. Here's a simple example to run a command to find a files, and then move it to another location:
```yaml
# First, run a function and capture a result into a register.
Find a file some previous step might have thrown in /tmp:
  module.run:
    - file.find:
      - /tmp
      - type: f
      - name: 'some-binary-v*'
    - register: output_of_find

# Since we can interpolate the register into a state module call, we can use an
# ordinary module to 'translate' function results into state results:
Check if we found something:
  check.changed_if: #Fictional module for now
    - name: {| (output_of_find | first) or False |}
    - comment: "Yep it's there"

# the 'check' module has translated the register into a normal result, so
# 'onchanges' will work here
Move file:
  onchanges: Check if we found something
  file.copy:
    - name: /usr/bin/some-binary
    - source: {| output_of_find | first |}
    - force: true
  file.absent:
    - name: {| output_of_find | first |}
```

And with this we've built a find-and-move state in pure sls, without needing any features beyond runtime interpolation and capture.

Many useful features are covered by jinja's expressions and so with minimal effort you can achieve some otherwise difficult-ish tasks.
Here are a couple of concepts:

```yaml
# (Non-ordered) Requisites:
# No more _any / _all confusion
Run something maybe..:
  gah.blah:
    - only_if: {| register_one and register_two or register_three | length > 4 |}
    - .
  ...


# Anything can basically work as an aggregate:
Read a dynamic list of packages on the minion:
  module.run:
    - file.read:
      - ./package-list
    - register: output_of_read

Install all the packages:
  pkg.install:
    - names: {| output_of_read |}

  ...

```

One huge advantage is the ability to pass data between steps in orchestrations, essentially allowing far more complex orchestrations:

```yaml
get zip name:
  salt.function:
    - name: s3.get
    - tgt: '{{ deploy_machine }}'
    - kwarg:
        bucket: 'release-blobs'
    - register: zip_names

create release workdir:
  salt.function:
    - name: temp.dir
    - tgt: '{{ deploy_machine }}'
    - kwarg:
        prefix: 'release-'
    - register: temp_path

get zip file:
  salt.function:
    - name: s3.get
    - tgt: '{{ deploy_machine }}'
    - kwarg:
        bucket: 'release-blobs'
        path: '{| zip_names | sort | first |}'
        local_file: '{| temp_path + 'migration.zip' |}

```

Here is an example of how to use thorium to get your minions to highstate, instead of the original reactor:
```yaml
minion startup:
  reg.set:
    - name: new_minion_id
    - add: id
    - match: salt/minion/*/start

is new minion ready:
  check.ne:
    - name: new_minion_id
    - value: 0

highstate new minion:
  local.cmd:
    - tgt: {| new_minion_id |}
    - func: state.highstate
    - require:
      - check: is new minion ready

unset register for next time:
  reg.set_value:
    - name: new_minion_id
    - value: 0
    - require:
      - local: highstate new minion

```
And with an additional `check`.`save_event` function, you can shorten the above example. The reason this is superior to the standard reactor is that it's *dynamic*, and requires no config changes to catch and react to the event. It also allows logic to be applied to the events, and passed between states.
You may also notice I didn't change any requisites, I just piggybacked off of states that are already in salt.

The only thing missing without additional features is being able to `loop` through a register, but this could work through a register-aware version of the `loop` module.
```yaml
Get a list of binary patches from something like a database:
  my_api.call
    - name: get_patches
    - register: bin_list

Apply new config to servers:
  loop.for_register:
    - name: bin
    - in: bin_list
    - do:
      - my_module.apply_hotfix:
        - fix_location: {| bin |}
```
The above example is a need we had recently as we needed a state to read a metadata file, and apply patches to various DBs based on the metadata.
A common counterargument would be to use jinja to make the first call (which I think is a bit of a copout), but this constrains you to needing to fetch the data at compile-time so you again cannot use the result of a previous set of steps.

It should also be possible to use something like {% set %} to make it easy to interpret function results and assign them to another register, though I haven't tested this in my PoC:
```yaml
Assign some register:
  test.nop:
    - eh: {|% set new_register = old_register['some']['value'] | title %|}

  # Can now use new_register in subsequent steps
```
... However in many cases it might be preferable to alias the output of a module without using a `test.nop`, for which I propose a dummy 'hook' that runs after executing the state:
```yaml
Run and assign subset:
  my_api.call
    - name: get_patches
    - register: api_output
    - posthook:
      - bin_list: {| api_output | list |}
      - first_item: {| bin_list | first |}
```


#### Some extra implementation details:
1) I have chosen the delimiters as `{| |}` somewhat arbitrarily, and there is no need for them to encompass the entire yaml value as I have done in the above examples.. However, complications could arise if there is no demarcation of which strings need to be interpolated by jinja at runtime.
If substring interpolation is desirable it may be feasible to mark values with an appropriately ugly delimiter, such as: `foo: __reg__ testing {| register |} 123` and have a preparser look for `__reg__` before handing the string to jinja.
A yaml tag, or a renderer could also be utilized here.

2) The register itself is just a dictionary, kept around in memory in the same manner that thorium does it. It is not expected for the registers to survive longer than a state or orchestration run.

3) I have not considered tying registers to things that alter state ordering. Though registers can functionally replace the way some requisites work, they cannot replace the core functionality of `require`. This seems fair to me, but could be unexpected for some?

## Alternatives
[alternatives]: #alternatives

This is in many ways a superset of functionality that `slots` provides, so one alternative would be to improve slots to:
- be able to recall results in multiple locations without rerunning the function
- able to collect multiple execution outputs
- have the result evaluated/tested, and combined with expressions to allow filtering and formatting

I wouldn't mind this approach but I feel like registers model the problem and solution more elegantly by bypassing the need for any custom parsing/interpreting, as well as using the very well-understood concept of variables.

## Unresolved questions
[unresolved]: #unresolved-questions

Some helper modules will need to be created to facilitate any creative use of registers that can't be done with just jinja. Loops and requisites are two I can think of.

If .sls files can now handle variables, it could possible to have them act as modules instead of states. Should this be explored?

# Drawbacks
[drawbacks]: #drawbacks

- Users may find the concept clashes with existing features, especially requisites.
- This feature needs jinja.NativeEnvironment which might be too new for some systems?
- Will need a lot of documentation.
- Initial implementation effort may be large, though it should be easy to maintain.
- This module definitely gives people more opportunities to over-engineer .sls files instead of just using python.
