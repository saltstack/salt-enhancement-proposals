- Feature Name: More SEP Statuses
- Start Date: 2019-03-18
- SEP Status: Draft
- SEP PR: https://github.com/saltstack/salt-enhancement-proposals/pull/6
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

Enhancing SEP statuses

# Motivation
[motivation]: #motivation

In discussions about legacy RFCs and existing SEPs, it's become evident that we
need a few more statuses for our SEPs:

- **Abandoned**
- **Up for Grabs**

The primary motivator is that we have some SEPs (and legacy RFCs) that are generally
accepted as fundamentally good ideas, but they haven't built up enough momentum
to get to a point where we're happy with the SEP. Rather than leaving the PRs
just cluttering up the space, we'd like to close them without prejudice. The
`Abandonded` status would indicate that:

1. They need more work
2. Are open for anyone who wants to champion the idea

As the original champion either ran out of interest, motivation, or perhaps
free time.

For the `Up for Grabs` status, this would indicate that while the SEP has been
accepted, there is no current implementation champion. As the SEP process
states:

> Note that acceptance does not mean that the feature will be immediately
> implemented, or that it will be implemented at all; It merely means that the
> core development team has agreed to it in principle.

When a SEP is accepted, and the original champion indicates they won't
be able to implement it either explicitly through comments or messages via some
other medium, or simply through inaction, the SEP will be marked as "Up for
Grabs", meaning that whoever would like to actually implement the SEP is
welcome to do it without stepping on anyone's toes, or repeating work.

# Design
[design]: #detailed-design

## Abandoned SEPs

Abandoned SEPs, if they are renewed by the original author, will re-enter at
the same place in the workflow where they left off. If they were eligible to
enter the **Final Comment** status, they will still be eligible. If they were
abandoned during the **Final Comment** phase, they will still be in **Final
Comment** phase when they are renewed.

If they are adopted by a new author after being abandoned, they will restart
the process, requiring at least two (2) weeks initial comment period. The
exception to this rule is if the original author hands off to another champion
in the comments on the SEP. Something to the effect of "So and so can take over
on this for me." For instance where there are more than one champion and there
are foreseeable circumstances, this should minimize the disruption in the SEP
process.

## Up for Grabs

As previously mentioned, approval of an SEP is no guarantee of implementation.
For SEPs where the original author also has a desire and ability to see the
feature through to completion, it seems natural that they will do so. There may
be cases where the author lacks either the desire or the ability to finish a
feature. In most cases, the author should indicate this at some point before
the SEP is merged, at which point it will be **Accepted** with the additional
**Up for Grabs** status.

In some cases, the original champion may not be able to implement the SEP, and
may become unresponsive. After an inactive period of two weeks (14 calendar
days), if the SEP author fails to communicate on a SEP, or push code on a SEP
related branch, the SEP will be considered **Up for Grabs** and anyone who
desires to step up and champion the development will be more than welcome.


## Alternatives
[alternatives]: #alternatives

We could leave stale SEPs in open status, or reject them. As far as has been
discussed, neither approach is ideal. Rejecting them could cause confusion,
since that's an indication that we don't actually want that feature. Leaving
them open is also probably not ideal, as there's constantly a mental step
(however small) of reviewing them when looking through the SEPs.

## Unresolved questions
[unresolved]: #unresolved-questions

Is two weeks for **Up for Grabs** too long? Too short? 

How long without comment before we mark an in-progress SEP as abandoned?

# Drawbacks
[drawbacks]: #drawbacks

Maybe not any?
