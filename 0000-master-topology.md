- Feature Name: (fill with a unique identifier, ex: my-awesome-feature)
- Start Date: (fill with today's date, YYYY-MM-DD)
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

There is a well proven want and need of Salt users to be able to group minions
into a single master logically and/or physically. Add the ablity
to link master's together and target different masters.

# Motivation
[motivation]: #motivation

There is a well proven want and need of Salt users to be able to group minions
into a single master logically and/or physically. Historically Synic has been
used to achive these kinds of groupings. The intention of this SEP is to create
a more robust solutions which will eventually deprecate Synics by building this
functionality directly into Salt Masters.

# Design
[design]: #detailed-design

Add the ability to define a topology of masters. Each node under topology is a
master. Users will be able to target jobs based on which master the minion is
connected to in addition to our existing targeting options.

```
topology:
  - west:
    - west-dc-1:
      - rack1
      - rack2
      - rack3
    - west-dc-2:
      - rack1
      - rack3
      - rack3
    - west-dc-2:
      - rack1
      - rack3
      - rack3
  - east:
    - east-dc-1:
      - rack1
      - rack2
      - rack3
    - east-dc-2:
      - rack1
      - rack3
      - rack3
```


## Alternatives
[alternatives]: #alternatives

What other designs have been considered? What is the impact of not doing this?

## Unresolved questions
[unresolved]: #unresolved-questions

What parts of the design are still TBD?

# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this? Please consider:

- Implementation cost, both in term of code size and complexity
- Integration of this feature with other existing and planned features
- Cost of migrating existing Salt setups (is it a breaking change?)
- Documentation (would Salt documentation need to be re-organized or altered?)


There are tradeoffs to choosing any path. Attempt to identify them here.
