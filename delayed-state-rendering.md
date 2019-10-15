- Feature Name: Delayed State Rendering
- Start Date: 2019-06-21
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary

This SEP proposes a way of delaying rendering of portions of states until after other states have executed.

A fundamental aspect of Salt is the ability to statefully control things by executing a highstate. This is currently limited when encountering a particular, fairly common circumstance. Consider a state that is truly stateful, but has an unpredictable effect. That is, this state is idempotent, but you can't know ahead of time a particular effect it has - like a hash. If you want to write a second state that needs this unpredictable information at its render time, then these two states cannot exist in the highstate together. This is because the rendering of the entire highstate happens prior to the execution of any of the states - the second state needs the first to complete before its render.

Currently, to get around this issue, I often write orchestration states. Other Salt tools like beacons and reactors can be used too. These tools have great value, but when the desired outcome is just part of the idempotent desired state of a minion, I really just want them in a highstate. What I propose adds complexity to the highstate, but I think it's a worthy trade to get rid of the complexity of these workarounds. At least, the choice would then be left to the developer to use delayed rendering, or an alternate solution using, for example, orchestration.

Here are a few potential use cases to demonstrate the utility of this feature:

1. Writing to disk and then querying for the exact physical location on disk of a file. Writing a file to disk is stateful, but we cannot predict the location
1. Managing partitions, giving them certain attributes, but needing to query for others
1. Cryptography: Any operation that outputs a (pseudo)random string or hash, that the master cannot reproduce on its own.
1. Instancing any cloud device or service that has an associated id or other attribute that is not predictable.

# Design
[design]: #detailed-design

## Design Summary

We will need a way to indicate

1. what section of an sls file to delay the render of, and
2. when to start the render and execution of these delayed section.

To do this, we add a new requisite called `delayed_render`, that takes a list of things render and execute after the state that has the requisite in it finishes. This list contains either sections enclosed by a custom jinja tag similar to `block` called `delayed_block`, or the names of entire sls files in the same way as given in other requisites.

It is important to note that we don't want to mark a state directly to indicate its render should be delayed. Instead, we want the abillity to mark *jinja* as needing to be rendered later. We don't want to render a single state later - we want to render jinja later that contains a state, or states. Doing this allows for loops, conditionals, [Salt's special context vars like `salt` and `grains`](https://docs.saltstack.com/en/latest/ref/states/vars.html), and the full power of the render machinery. This allows for accessing and building logic around data at delayed render time, that did not exist at the render of the calling state.

Here are several examples of how state files could use this

### Simple example calling a jinja block:

```salt
# init.sls
my_state1:
  mod.fun1:
    - do_something
    - delayed_render:
      - block: dependant_block_A

{% delayed_block dependant_block_A %}
my_state2:
  mod.fun2:
    - {{ data }}
{% end_delayed_block %}
```

In this example, `my_state` indicates that `dependant_block_A` will be rendered and its states executed after it finishes. In this way, `delayed_render` can be thought of as a new kind of requisite. It is similar to `require_in` in that it helps determin an order of execution. It is different, in that `delayed_render` determines what will be executed **imediately after** this state. It also does not simply point to another state. Instead, `delayed_render` points to a block of Jinja defined by a custom Jinja tag.

Upon initial render of this state (as in a highstate or otherwise), the `delayed_block` Jinja tag is effectively cuts all of the text in the block, and stores it in memeory on the Salt master, and associates it with the original job. When `my_state1` finishes, the minion calls back to the master, indicating that the `delayed_block` `dependant_block_A` should be rendered, and any states within it executed, on the same minion.

### Simple example calling an sls file:

```salt
# init.sls
my_state1:
  mod.fun1:
    - do_something
    - delayed_render:
      - sls: my_state_file
```

```salt
# my_state_file.sls
my_state2:
  mod.fun2:
    - {{ data }}
```

This is a very similar example, except that instead of a `delayed_render`, indicating a particular `delayed_block`, `delayed_render` specifies an entire sls file to render and execute when `my_state1` finishes.

### Multiple delayed_renders

```salt
# init.sls
my_state1:
  mod.fun1:
    - do_something
    - delayed_render:
      - sls: state_file1
      - block: dependant_block_A
      - sls: state_file2
      - block: dependant_block_B
```

This state must be a precursor to many things. Since `delayed_render` contains a list, we can mark multiple blocks and sls files. This state indicates that two sls files and two blocks should be rendered after it finishes executing, and they should be rendered and executed in this particular order; `state_file1`, then `dependant_block_A`, etc.

## Edge Cases

Since the requisite and delayed render is so generalized, the author can potentially do complicated things.

### Nested delayed_renders

```salt
my_state1:
  mod.fun1:
    - do_something
    - delayed_render:
      - block: dependant_block_A

{% delayed_block dependant_block_A %}
my_state2:
  mod.fun2:
    - {{ data }}
    - do_something
    - delayed_render:
      - block: dependant_block_A_a

{% delayed_block dependant_block_A_a %}
my_state3:
  mod.fun3:
    - {{ data }}
    - do_something
{% end_delayed_block %}

{% end_delayed_block %}
```

Since delayed_blocks are encapsulated much like normal Jinja blocks, you could nest them. In this case, there are three renders. The first, for example caused by a `state.apply`, and the next two caused by two `delayed_render` requisites, in a chain.

### Repeated renders

```salt
{% for i in range(5) %}
my_state{{ i }}:
  mod.fun{{ i }}:
    - do_something
    - delayed_render:
      - block: dependant_block_A

{% delayed_block dependant_block_A %}
my_stateA:
  mod.funA:
    - {{ data }}
    - do_something
{% end_delayed_block %}
```

In this situation, the minion calls back to the master *5 times* to render `dependant_block_A`, once for each state that calls it. This should give you pause. There may be real uses to do something like this, especially since `dependant_block_A` could have very flexible Jinja in it. However, both this and nesting delayed_renders can potentially cause problems. Apart from making traversing the graph difficult to understand, there is the real possibility of causing infinite loops or being too recursive.

To deal with this we introduce the `delayed_repeat_limit` flag. This flag should default to `1`, indicating that no block or sls file should be rendered more than once. If really desired though, `delayed_repeat_limit` can be set to any positive integer, or `None`. `delayed_repeat_limit: 5` would allow the above example to render `dependant_block_A` the full 5 times. `dependant_block_A: None` would do the same, by lifting the limit entirely (no limit).

If the limit is exceeded, the exceeding render should not occur, and instead return and issue an error. This is likely a mistake and the user should be alerted, however, it should not cause an entire build to fail unless `failhard: True`.

The `delayed_repeat_limit` flag can be added in three places:

1. The `delayed_block` tag:

```salt
{% delayed_block dependant_block_A delayed_repeat_limit=3 %}
my_stateA:
  mod.funA:
    - {{ data }}
    - do_something
{% end_delayed_block %}
```

2. The top of an sls file:

```salt
# my_delayed_sls.sls
{% delayed_repeat_limit 3 %}

my_stateA:
  mod.funA:
    - {{ data }}
    - do_something
```

Note in this case `delayed_repeat_limit` is another custom Jinja tag. It should be placed at the top of a file similar to an [`{% extends %}` tag](https://jinja.palletsprojects.com/en/2.10.x/templates/#child-template).

3. In the master configuration, as an alternative default value.

```salt
# master.conf
delayed_repeat_limit: 3
```

This would set the default value to `3`, instead of the typical `1`. This could also be used in a minion configuration file on a masterless minion.


## Implementation Notes




## Alternatives
[alternatives]: #alternatives

How delayed rendering is different from slots:
Slots (currently at least) only run remote execution calls. Delayed rendering brings with it everything that Jinja typically has in a state, including logic, grains, pillars, `salt`.
Slots simply returns the result of an execution call. They do not interact with Jinja to allow any logic or parsing.
