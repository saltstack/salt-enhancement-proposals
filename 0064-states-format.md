- Feature Name: A new paradigm for writing state files
- Start Date: 2022-03-30
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)


## Summary
[summary]: #summary

Currently Salt states are very simple and powerful to write. Even though it’s powerful and flexible, Salt implements several abstraction layers and implicit logic in state terminology. This SEP proposes a new design/standard for state files that can be easier to write and to understand, specially for beginners to Salt.

## Motivation
[motivation]: #motivation

The main motivation for this SEP is the steep learning curve new users tend to have with Salt, which in turn makes Salt less than top-of-mind when comparing to other remote execution/configuration management tools (more on this argument in a different space).

The idea is that the implicit logic greatly contributes for a steeper learning curve for new users, which may scare people away from Salt. By having a simpler way to write states, the learning process can be easier for newcomers.

A good example of implicit logic is this snippet from the documentation:
```yaml
vim:
  pkg.installed: []
```

The first line `vim` is a name of a package, while also being an unique identifier for a single state block. The second line, has three informations: the module (`pkg`), the function `install` and the arguments (an empty array `[]`).
An argument could be made that although this is straight forward to write, is not straightforward to understand (specially for beginners), and not straightforward to parse either, which demands lots of logic on the backend.
There are a few ways to interpret this:
1. Option 1
    - The first line should be packages to be installed, and the second line the module/function with an array with arguments
    - If that’s the case, how one installs multiple packages?
    - If I install multiple packages passing the `name` parameter, then what does `vim` stands for? 
2. Option 2:
    - The first line is an identifier.
    - If that’s the case, one should pass the name of packages to be installed in the empty array `[]`. So how does Salt knows that it has to install `vim`?

Either way, the logic behind the state is implicit.
Things get more complicated when looking at different examples on the documentation.
This snippet is a good example:

```yaml
vim:
  pkg.install: []

/etc/httpd/conf/httpd.conf:
  file.managed:
    - user: root
    - group: root
    - mode: 644
```

In the above snippet, it’s fair to understand why this can be confusing for newcomers. After reasoning with one of the two main options above for the first line, the second state block poses a new challenge: it’s a folder path. Does it have any real utility/meaning?
 
If the first line of a state block is a unique identifier (no meaning), how does Salt knows what it have to do with the File? If the first line had any meaning, does it mean that when writing `vim` like state blocks I need to know the path of the `apt` repository?

Apart from those aspects, the current renderer Jinja2, puts a lot of responsibility on the templating engine itself. Even though it’s extremely powerful, it can be overwhelming for beginners, and make the code less readable. Jinja logic block implements a less than simple syntax with it’s `{%` notation.

This still could be leverage for advanced users, but simple use cases ( `if` statements and `for loops` could be embedded in the parser itself, eliminating the necessity of writing Jinja code for beginners.

This arguably would make `state` files simpler, easier and lighter (to the eyes). And even though this is a stretch of an argument, it’s fair to say that a simple language **tends** to be more elegant than a language with a significant amount of symbols.

It’s only fair to mention that a few of this issues could be resolved via documentation (and the documentation has improved a lot in the last two years in that send). Indeed the new Salt User Guide is a game changer in that aspect, making it easier to understand. But this is not a replacement of a clearer documentation, but rather a new paradigm.


## Design
[design]: #detailed-design

The idea is that a Salt state becomes more verbose, but more explicit. Instead of leveraging identifiers as keys, this implementation would have fixed keys, with values being user input. This would make more obvious for the user to understand and write a SaltState, and would potentially simplify Salt’s internals (easier to parse the state file). This walks closely with the idea of SEP XYZ that proposes the usage of dictionaries instead of lists.

This design suggestion leverages ideas from Kubernetes Manifests, Docker Compose files, and Ansible Playbooks. All of those arguably have great community adoption outside the enterprise world.

```yaml
---
# The new statefile proposal would use yaml 
# By using a well known extensions, IDE would better work with linters 
# Change of language on the IDE and other custom language file mapping or extensions wouldn't be necessary 


# The header would start with simple definitions 
# Similar to what docker-compose and Kubernetes manifests use 
# This would allow for future versioning of the renderer
# Even though not many changes were to be expected between versions, having the flexibility would help 
# The engine key would help the parser to decide how to handle the yaml file.
# If it's Jinja, it first parses the template, if it's yaml, uses the embeeded functionality of the main parser. 
name: My Custom State
version: v2
engine: yaml  # jinja2 or any other

# Adding metadata in the state declaration file 
# could help newcomers to better understand the objective of the file
# also could be leveraged by the interpreter to make state files more porwerful
# On the future, similar to what Kubernetes has, leveraging metadata labels to make rule-based decisions in run time could be implemented 
metadata:
  description: This is a long string with the state description
  hosts: this is a custom key, the metadata would accept anything.

# The statefile would still have the same attributes as it does now, albeit with more explicit keys. 
include_states:
  - stateA
  - stateB

# A state file is an array of single states. 
# Organizing the array on a single "spec" (k8s based) section 
# which defines the specification of the state. 
# would make it more readable, easier to maintain and easier for humans 
# to understand whats going on. 
spec:

  # A single state would be organized on the key:value based approach. 
  # Instead of having the task_id as key itself, we would have the explicit key 
  # and then the values. 
  # This would help differenciate what is a keyword for salt 
  # and what is the user input 
  # This would also avoid different interpretations of what each input means 
  - id: my_state_id
    name: My State String
    pkg:
      method: installed
      attr_key0: attr_value0
      attr_key1: attr_value1

  # Modules/Functions could be combined 
  # This would result in the dot notation shortcut.
  # The renderer would parse this, and return the state as a Python Object 
  # This kind of flexibility would allow for a easier learning curve for newcomers 
  # coming from Ansible, for example, this would be a smooth transaction
  - id: my_state_id
    name: My State String
    pkg.installed:
      attr_key0: attr_value0
      attr_key1: attr_value1

  # As these are all keyword arguments, this would also be possible. 
  # This would shorten the single_state definition and should also be accepted 
  # Although it shouldnt be reiforced in the documentation and considered an "advanced" use case
  - id: other_state_id
    name: My State String
    module: func="fdsf" name="net-tools" attr="attrvalue"

  # If the renderer now has greater responsability on the parsing/conversion 
  # It would be possible to make common cases where Jinja was needed 
  # directly on the YAML file. 
  # This would help Editors linters (no need for a .sls file with IDE Extensions) 
  # And would make the state declaration cleaner 
  - id: other_state_id
    name: My State String
    module_name:
      attr_key0: attr_value0
      attr_key1: attr_value1
    when: "grains['os_family'] == 'debian'"

  # Something similar for common scenarios
  # Most already are contemplated on the current Salt State files
  - id: other_state_id
    name: My State String
    pkg.install:
      name: httpd
    except: "grains['os_family'] !== 'debian'"

  # Similar to the example above 
  # For loops could also be implemented, similar to what ansible has. 
  # Leveraging the well known Jinja variable syntax 
  - id: other_state_id
    name: My State String
    pkg.install:
      - items: "{{ item }}"
    items: "pillar['myItems']"
```

Even though this is a big change in terms of how to write states, the implementation could be gradual.
At first, using this state format, it would be possible to send this state to the `renderer v2`. So a simple `if` statement inside the current default renderer could send this to a new renderer, so it would be fairly easy code change.
Internally the `renderer v2` could initially just transform the `State V2` file into the default simple Salt State file, just as it’s written now.

## Future Considerations
In the future, by having more predictable keys, it would be possible to simplify the logic of parsing states (state > high state > low state), and the renderer itself could already return low state data, hence eliminating a couple of steps on the way. I'm not familiar with the internals, so would be great to listen to experienced community members.

Old formats could still be accepted. By not using the `version` argument, the renderer would still be the default one, so it would be possible to keep backwards compatibility.

Keys that currently have no use (like the `name` attribute, or `metadata` ) section at first would just be disregarded, and slowly it would be possible to aggregate information to the state, and the information passed/received from master/minion.

## Alternatives
[alternatives]: #alternatives

It could be the case to make small changes to the current `state` logic, making it clearer what is an `state identifier` and what is an argument. In the same lines of the SEP 62 [https://github.com/saltstack/salt-enhancement-proposals/pull/62], changing arrays to dictionaries could also result in a great improvement on readability.


## Unresolved Questions
[unresolved]: #unresolved-questions

At last, I imagine this probably will make little sense and/or little difference for current Salt users that already are accustomed on how to write state files.
This proposal is geared towards new users, or rather, making Salt more attractive/easier to new users. The fact that having such a format could accommodate for the current state format, also makes it less intrusive of current state file style/


## Drawbacks
[drawbacks]: #drawbacks
This would make it necessary to write a new renderer, that actually **removes** information from the statefile in order to make the change smooth. In the future, the renderer would stop remove information. I


## References:
[salt-enhancement-proposals/0062-yaml-dictionaries.md at yaml-dictionaries · Murz-forks/salt-enhancement-proposals · GitHub](https://github.com/Murz-forks/salt-enhancement-proposals/blob/yaml-dictionaries/0062-yaml-dictionaries.md)
[TECH DEBT Add ability to use yaml dictionaries instead of arrays in Salt SLS files · Issue #61649 · saltstack/salt · GitHub](https://github.com/saltstack/salt/issues/61649)
