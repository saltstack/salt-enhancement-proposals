- Feature Name: release-label-improvements
- Start Date: 2020-01-13
- SEP Status: Draft
- SEP PR: https://github.com/saltstack/salt-enhancement-proposals/pull/24

# Summary
[summary]: #summary

The current GitHub labels are complicated and not very intuitive. This SEP is
designed to simplify the way we use labels to track what should make it into a
release, and clarify the high-level purpose of specific labels.

# Motivation
[motivation]: #motivation

In community and internal discussions regarding labels, it's become clear that
we need a more streamlined approach to labels and their release priority.
Complicated, unclear labeling is confusing for contributors and consumers of
the Salt project.

We combine labels such as P1 (everyone will see this issue) with Medium
Severity (it’s just cosmetic). Or we see Critical (data loss) with P4 (not many
people will see it). These pairings are confusing, even if they’re technically
correct.

# What This Is Not

This SEP is not concerned with labels regarding functional areas, functional
groups, test status, or other labels. It is also not concerned with being an
exhaustive list of criteria for labels. It also does not attempt to address the
entire development and triage process.

# Design
[design]: #detailed-design

Based on the discussions we've had, there are three main perspectives that are
confused about our current labels in regards to the release process.

As bug reporter (or someone with the same issue), I just want to know: **when
will my bug be fixed?** In other words, what major or point release will
solve my problem?

As a contributor, I want to know: **what should I be working on next**?

From a release manager point of view, I want to know: **is there anything that
still needs to be done before the next release**?

This SEP will address the labels that can answer these questions.

## Description

Going forward, there will be four (4) priority labels that will make it easier
to know when an issue can be expected in a release.

The highest priority contains **only issues that are severe enough to block a
release**. These issues affect most or all users within a functional area,
cause data loss, prevent Salt from starting, cause salt to crash, or have
similarly negative effects. These issues will block a release until corrected
or determined to be a lower priority. This label will be **Critical**.

The next highest priority is for issues that are serious issues, but **not
serious enough to be considered a release blocker**. These affect a many or
most users in the functional area, may cause data loss, crashes or hangs,
and/or makes the system unresponsive. This label will be **High**

The next highest priority is for issues that affect about half the users in a
functional area, producing incorrect functionality, bad functionality, a
confusing user experience, or a confusing error message. This label will be
**Medium**.

The final label applies to issues that affect few users within a functional
area, are limited to corner/edge cases, especially when a workaround exists,
and also includes cosmetic fixes such as spelling, formatting, and colors. This
label will be **Low**

To recap, the new labels will be:

- **Critical** - release blockers, serious issues.
- **High** - serious issues, but not release blockers.
- **Medium** - painful but not serious issues, especially ones that lack a
  reasonable workaround.
- **Low** - cosmetic issues, or issues that have a simple/reasonable
  workaround.

https://github.com/saltstack/salt/blob/master/doc/topics/development/labels.rst
will be updated to reflect these new labels.

These labels will replace the following labels:

- Blocker
- Critical
- High Severity
- Medium Severity
- P1
- P2
- P3
- P4

## How Does This Fix the Problem(s)?

Moving to these labels will simplify the number of labels required. It will
also make it easy to see what issues are certain to make it into the next
release, and which issues are *not* the focus for the core team.

## Alternatives
[alternatives]: #alternatives

### Keep It Up

We could keep doing what we’ve been doing, and simply do better at publicizing
/ documenting the changes.

### Different Colors of Bikeshed

We could use other label names, such as “Blocker”, “Major”, “Minor”,
“Cosmetic”.

# Drawbacks
[drawbacks]: #drawbacks

We have to update documentation and re-train ourselves. But even with current
documentation, it seems like there are enough complications that we *still*
would need to train ourselves. By simplifying things it should be easier. We
may need to go through and re-label existing issues.
