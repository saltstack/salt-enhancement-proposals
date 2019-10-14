[I- Feature Name: Delayed State Rendering
- Start Date: 2019-06-21
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary

This SEP proposes a way of delaying rendering the jinja and yaml of portions of states until after other states have executed.

A fundamental aspect of Salt is the ability to statefully control things by executing a highstate. This is currently limited when encountering a particular, fairly common circumstance. Consider a state that is truly stateful, but has an unpredictable effect. That is, this state is idempotent, but you can't know ahead of time a particular effect it has - like a hash. If you want to write a second state that needs this unpredictable information at its render time, then these two states cannot exist in the highstate together. This is because the rendering of the entire highstate happens prior to the execution of any of the states - the second state needs the first to complete before its render.

Currently, to get around this issue, I often write orchestration states. Other Salt tools like beacons and reactors can be used too. These tools have great value, but when the desired outcome is just part of the idempotent desired state of a minion, I really just want them in a highstate. What I propose adds complexity to the highstate, but I think it's a worthy trade to get rid of the complexity of these workarounds. At least, the choice would then be left to the developer to use delayed rendering, or an alternate solution using, for example, orchestration.

Here are a few potential use cases to demonstrate the utility of this feature:

1. Writing to disk and then querying for the exact physical location on disk of a file. Writing a file to disk is stateful, but we cannot predict the location
1. Managing partitions, giving them certain attributes, but needing to query for others
1. Cryptography: Any operation that outputs a (pseudo)random string or hash, that the master cannot reproduce on its own.
1. Instancing any cloud device or service that has an associated id or other attribute that is not predictable.

# Design
[design]: #detailed-design

We will need a way to indicate

1. what section of an sls file to delay the render of, and
2. when to start the render and execution of these delayed section.

Here are several examples of how state files could be accomplished.

### Simple example calling with `delayed_blocks`:

```salt
# init.sls
my_state1:
  mod.fun1:
    - do_something
    - delayed_blocks:
        - dependant_block_A

{% delayed_block dependant_block_A %}
my_state2:
  mod.fun2:
    - {{ data }}
{% end_delayed_block %}
```

In this example, `my_state` indicates that `dependant_block_A` will be rendered and its states executed after it finishes. In this way, `delayed_blocks` can be thought of as a new kind of requisite. It is similar to `require_in` in that it helps determin an order of execution. It is different, in that `delayed_blocks` determines what will be executed **imediately after** this state. It also does not simply point to another state. Instead, `delayed_blocks` points to a block of Jinja defined by a custom Jinja tag.

Upon initial render of this state (as in a highstate or otherwise), the `delayed_block` Jinja tag is effectively cuts all of the text in the block, and stores it in memeory on the Salt master, and associates it with the original job. When `my_state1` finishes, the minion calls back to the master, indicating that the `delayed_block` `dependant_block_A` should be rendered, and any states within it executed, on the same minion.

### Simple example calling with `delayed_slses`:

```salt
# init.sls
my_state1:
  mod.fun1:
    - do_something
    - delayed_slses:
        - my_state
```

```salt
# state_file.sls
my_state2:
  mod.fun2:
    - {{ data }}
```

This is a very similar example, except that instead of a `delayed_blocks`, indicating a particular `delayed_block`, `delayed_slses` specifies an entire sls file to render and execute when `my_state1` finishes.

### More complicated examples

```salt
# init.sls
my_state1:
  mod.fun1:
    - do_something
    - delayed_slses:
        - state_file1
        - state_file2
    - delayed_blocks:
        - dependant_block_A
        - dependant_block_B
```

This state must be a precursor to many things. It calls two sls files and two blocks.

### Edge Cases


### Implementation Notes




## Alternatives
[alternatives]: #alternatives

How delayed rendering is different from slots:
Slots (currently at least) only run remote execution calls. Delayed rendering brings with it everything that Jinja typically has in a state, including logic, grains, pillars, `salt`.
Slots simply returns the result of an execution call. They do not interact with Jinja to allow any logic or parsing.
