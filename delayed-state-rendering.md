- Feature Name: Delayed State Rendering
- Start Date: 2019-06-21
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)
- Author: Joseph Nix, nixjdm at terminallabs dot com, [nixjdm on GitHub](https://github.com/nixjdm)

# Summary

This SEP proposes a way of delaying rendering of portions of states until after other states have executed.

A fundamental aspect of Salt is the ability to come to a desired endstate by executing a single highstate. This is currently limited when encountering a particular, fairly common circumstance. Consider a state that is truly stateful, but has an unpredictable effect. That is, this state is idempotent, but you can't know ahead of time a particular effect it has - like a hash. If you want to write a second state that needs this unpredictable information at its render time, then these two states cannot exist in the highstate together. This is because the rendering of the entire highstate happens prior to the execution of any of the states, i.e. the second state needs the first to complete before its render.

Currently, to get around this issue, I often write orchestration states. Other Salt tools like beacons and reactors can be used too. These tools have great value, but when the desired outcome is just part of the idempotent desired state of a minion, I really just want them in a highstate. What I propose adds complexity to the highstate, but I think it's a worthy trade to get rid of the complexity of these workarounds. At least, the choice would then be left to the developer to use delayed rendering, or use an alternate solution like orchestration.

Here are a few quick use cases to demonstrate the utility of this feature:

1. Writing to disk and then querying for the exact physical location on disk of a file. Writing a file to disk is stateful, but we cannot predict the location(s) of the file.
1. Managing partitions, giving them certain attributes, but needing to query for others.
1. Cryptography: Any operation that outputs a (pseudo)random string or hash that the master cannot reproduce on its own.
1. Instancing any cloud device or service that has an associated id or other attribute that is not predictable.
1. Managing any data with an associated UUID, such as a Docker container, or entries in a database.

# Design
[design]: #detailed-design

## Design Summary

We will need a way to indicate

1. what section of an sls file to delay the render of, and
2. when to start the render and execution of these delayed sections.

To do this, we add a new requisite called `delayed_render`, that takes a list of things to render and execute after the state that has the requisite in it finishes. This list contains either 1) blocks enclosed by a custom jinja tag, similar to `block`, called `delayed_block`, or 2) the names of entire sls files, in the same way as given in other requisites.

It is important to note that we don't want to mark a state directly to indicate its render should be delayed. Instead, we want the ability to mark *Jinja* as needing to be rendered later. We don't want to render a single state later - we want to render Jinja later that contains a state, or states. Doing this allows for loops, conditionals, [Salt's special context vars like `salt` and `grains`](https://docs.saltstack.com/en/latest/ref/states/vars.html), and the full power of the render machinery. This allows for accessing and building logic around data at delayed render time, where that data did not exist at the render of the calling state.

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

In this example, `my_state` indicates that `dependant_block_A` will be rendered and its states executed after it finishes. In this way, `delayed_render` can be thought of as a new kind of requisite. It is similar to `require_in` in that it helps determine an order of execution, and indicates what comes later. It is different, in that `delayed_render` determines what will be executed **immediately after** this state. It also does not simply point to another state. Instead, `delayed_render` points to a block of Jinja defined by the custom Jinja tag `delayed_block`.

Upon initial render of this state (as in a highstate or otherwise), the `delayed_block` Jinja tag effectively cuts all of the text in the block, and stores it in memory on the Salt master, and associates it with the original job. When `my_state1` finishes, the minion calls back to the master, indicating that the `delayed_block` `dependant_block_A` should be rendered, and any states within it executed, on the same minion.

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

This is a very similar example, except that instead of a `delayed_render` indicating a particular `delayed_block`, `delayed_render` specifies an entire sls file to render and execute when `my_state1` finishes.

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

Since the requisite and delayed render is so generalized, the user can potentially do complicated things.

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

Since delayed_blocks are encapsulated much like normal Jinja blocks, you can nest them. In this case, there are three renders. The first, for example caused by a `state.apply`, and the next two caused by two `delayed_render` requisites, in a chain.

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

Here `delayed_repeat_limit=3` is essentially a kwarg/value passed to `delayed_block`, in addition to the block name.

2. The top of an sls file:

```salt
# my_delayed_sls.sls
{% delayed_repeat_limit 3 %}

my_stateA:
  mod.funA:
    - {{ data }}
    - do_something
```

Note, in this case `delayed_repeat_limit` is another custom Jinja tag, not a kwargs as in the first example. It should be placed at the top of a file similar to an [`{% extends %}` Jinja tag](https://jinja.palletsprojects.com/en/2.10.x/templates/#child-template).

3. In the master configuration, as an alternative default value.

```salt
# master.conf
delayed_repeat_limit: 3
```

This would set the default value to `3`, instead of the typical `1`. This could also be used in a minion configuration file on a masterless minion.

## Implementation Notes

### General flow

On initial render of a state or set of states (`state.apply` or `state.highstate`), all `delayed_block`s must be scanned for. Their contents should be removed as-is, and stored for later rendering. At this step it doesn't matter at all what the contents are, be it a simple state, complex jinja, a nested `delayed_block`, or even unrenderable Jinja or Yaml. The initial render will execute until the minion hits a `delay_render` callback requisite. When that happens, the minion pauses it's chain of state executions and calls back to the master for further instructions. This callback should contain its parent's job id. E.g. if a highstate contains callbacks, that highstate's jid would be returned in the callback. That way the master has enough information to proceed.

The master then renders the block (or sls), and proceeds to execute what it says. This could nest deeper by using the same process as above. Once the minion completes this set of execution, it can proceed to the next step in the original order of state execution.

If there are no callbacks looking for a particular `delayed_block` or `delayed_sls`, it will be discarded at ultimate end of the original job.

Orchestration can use delayed rendering in a similar way. Though orchestration execution happens on the master, it may still be valuable to delay rendering before continuing to execute certain dependent functions.

### User Interface

I think it's best that states that came from a delayed render should be slightly indented in the normal output in the terminal when you run, for example, a highstate. This is to help identify visually that this happened, and because the delayed renders scope is different, meaning that state names could be repeated. This will help keep things clear to the user.

This is confounded if there is significant nesting. I'm unsure how this would be best displayed.


### Delayed Blocks

Custom `delayed_block` tags draw their inspiration traditional Jinja `block` tags. As such, their use should be as similar as possible.

#### End Tags

You should also be able to [specify named block end-tags](https://jinja.palletsprojects.com/en/2.10.x/templates/#named-block-end-tags) like `{% end_delayed_block dependant_block_A %}`. Note the traditional end tag is `endblock`. Since `end_delayed_block` is three words long, the additional of underscores seems prudent for readability.

#### Finding blocks

Like traditional `block` tags, you can only reference a block within the same template, or in a Jinja included or extended template. A `delayed_block ` in another template will not be found or used unless that template is included or extended with Jinja.

#### Delayed Scope

Like traditional `block` tags, [delayed_blocks may not access variables from outer scope](https://jinja.palletsprojects.com/en/2.10.x/templates/#block-nesting-and-scope). *Unlike* traditional `block` tags, there is no exception to this. When the minion calls back to the master, it doesn't pass anything that could be added directly to the `delayed_block`'s scope. The master also adds nothing. This could possibly be added in the future, but for now seems like unneeded complexity.

As an extension of this, other requisites cannot traverse the scopes. No requisite from the outer scope can reference a state in the inner scope, and no state in the inner scope can reference a state in the outer scope. It is possible to fanagle a few exceptions to this, but I think that would greatly increase complexity. It is much simpler to have `delayed_block`s "stand on their own" in an isolated scope.

Finally, because the inner scope is isolated, the same state id can be reused. Take the example in the [Repeated Renders](#repeated-renders) section as an example. Here, each delayed render is rendered and executed in isolation, yet the state declaration is the same each time. There is no conflict because of the isolated scopes. Each delayed render state can also still be uniquely identified by associaating it with its parent state, that caused its render.

These scoping rules apply equally to `delayed_sls` files.

### Alternative Renderers

Any renderer that has the ability to add something analogous to a custom template tag in Jinja, can also have `delayed_block`s. Mako, Genshi, and Cheetah should be fine, for instance, though the implementation may look more like function calls than blocks. Specific implementation however, needs work. The Python renderer could also achieve the same effect with a function. For instance, a [pure Python state](https://docs.saltstack.com/en/latest/ref/renderers/all/salt.renderers.py.html#full-example) could include additional functions that, when decorated by `@delayed_block`, could be evaluated in delayed fashion. As a matter of development, Jinja could be the only renderer that offers `delayed_block`s for a while, and other renderers could add the feature as they are completed.

References to `delayed_sls` files should be straight-forward.

## Alternatives
[alternatives]: #alternatives

### Slots

Slots (currently at least) only run remote execution calls, potentially informing a state, but they do not interact with Jinja to allow any logic or parsing.

### Orchestration

The goal of this feature is to have a more powerful highstate. However, as I have done to date, pieces of the would-be highstate can be broken out into separate states, called by orchestration states, or beacons and reactors. I find this to be a less intuitive and more complicated approach if you do much beyond the simplest orchestration.

## Unresolved questions
[unresolved]: #unresolved-questions

Finer details of the internal mechanisms could be solidified. I choose to not get too deep into that to leave room for the developers.


How alternate renderers could implement this could also be fleshed out further.

# Drawbacks
[drawbacks]: #drawbacks

This would be a fairly large and complicated feature to implement. It would have real cost in terms of code size and complexity, and it would touch many parts of the existing code.

This is not a breaking change. Existing setups should run unaffected.

There would probably be several documentation pages that would need to be written or updated. This probably deserves its own page. It also relates to the renderer pages, the requisites page, and pages that talk about the highstate and state system layers.

# Postscript

This has been on my mind for about a year. It first occurred to me when I was writing cloud orchestration states. I had an orchestration state that spawned a compute instance and then kicked off a highstate. It was simple enough. Then I wanted to access and use the public IP address of that compute instance. That was not so easy. Then I found similar problems for other "cloud" operations within a highstate, where I needed to use information I could not set or predict. Finally, I realized that this limitation is systemic. It doesn't just apply to cloud operations, though that may be the most frequently encountered limitation. Any state that has an unpredictable effect is currently difficult to incorporate into a streamlined highstate.

I have discussed this topic with Alan Cugler [alan-cugler@GitHub](https://github.com/alan-cugler) and Michael Verhulst [verhulstm@GitHub](https://github.com/verhulstm) several times, and they helped me flesh this out quite a bit.
