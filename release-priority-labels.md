# Summary

The current GitHub labels are complicated and not very intuitive. This SEP is
designed to simplify the way we use labels to track what should make it into a
release, and clarify the high-level purpose of specific labels.

# Motivation

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
release**. Under normal circumstances, these MUST be fixed before the next
major or point release. These are issues that cause data loss, prevent Salt
from starting, cause Salt to crash, etc.

The next highest priority is for issues that are serious issues, but **not
serious enough to be considered a release blocker**. These are issues that will
be addressed by the core team or the community, as time allows.

There will be another label for issues that the **core team may address as time
allows**, but community help is more than welcome.

The final label will be issues that are unlikely to be addressed by the Salt
core team, and will likely require a community contribution to get them
accomplished.

These labels will be Release Blocker, High, Medium, Low.

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

# Alternatives

## Keep It Up

We could keep doing what we’ve been doing, and simply do better at publicizing
/ documenting the changes.

## Different Colors of Bikeshed

We could use other label names, such as “Critical”, “Major”, “Minor”,
“Cosmetic”.

# Unresolved Questions

Do we need visibility labels, meaning something to replace P1, P2, P3, P4? For
example, High Visibility (many or most Salt users will see it), Medium
Visibility (some), Low Visibility (few people will see it). This measure is
somewhat subjective, though visibility could help in prioritizing the backlog.

# Drawbacks

We have to update documentation and re-train ourselves. But even with current
documentation, it seems like there are enough complications that we *still*
would need to train ourselves. By simplifying things it should be easier. We
may need to go through and re-label existing issues.
