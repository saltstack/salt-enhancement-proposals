- Feature Name: develop-first-policy
- Start Date: 22-03-2019
- SEP Status: Draft
- SEP PR:
- Salt Issue: 51997

# Summary
[summary]: #summary

Currently we have a [merge
forward](https://docs.saltstack.com/en/latest/topics/development/contributing.html#merge-forward-process)
policy in place, where all the fixes are first developed against the
oldest supported product version of Salt, and from there are rebased
to the more modern versions, until it reach the `develop` (`master`)
branch.

The rebase is not done per commit, but per branch. This generate very
big commits that accumulate several changes (and slow down the merging
process as there are multiple conflicts), that are pushed from the
old version to the new version. We can see that those big commits
contains relevant refactoring changes (like the rename of modules and
states) among the fixes, and that can affect [hundreds of
files](https://github.com/saltstack/salt/pull/51990).

The net effect is that meanwhile the big patch is moving from older
versions to newer, until it reach `develop`, a developer is working
with an effective out of date version of `develop`, creating problems
when a feature is backported or duplicating work when a new is
introduced.

This SEP propose a change in the workflow, where each commit (fix or
feature) is always first done against `develop`, reviewed there and
once is merged, cherry picked from there to the product branch.

# Motivation
[motivation]: #motivation

The current workflow have several drawbacks that I would like to
address:

* `develop` branch gets outdated frequently for several weeks.

  The problem with this is that complicate consider `develop` as the
  reference branch of the project, that includes all the fixes and
  reflect all the technical decision done in the project.

* Is hard to track what is missing in `develop`.

  To see what commits are living in an old product branch, but not in
  `develop`, you need to do some git juggling. This per-se is not a
  problem, but makes an required task more complicated that it should,
  and also error-prone.

  Revealing the history of a single change is complicate with those
  big commits. Instead a `cherry-pick -x` make the track extremely
  straightforward, as is already documented in the description of the
  commit.

  The tracking problem makes more hard to detect missing or lost
  commits. When a commit that is living in an old release is not found
  in a newer one nor in `develop`, makes hard to decide is a error in
  the rebase or a conscious decision.

* Hard to diagnose problems in the big commit.

  Creating big commits that aggregate that belongs to a delta of other
  commits that are in an old release, make hard to detect what is
  wrong when the CI complains. Integrating smaller changes makes this
  process more direct.

* Review and merge process is slow

  Of course big commits makes the review complicated, and also the
  probability of a conflict increases. Note that moving from an old
  code-base to a new code-base is also moving from a more simple
  code-base to a more convoluted one (usually, not always true). This
  makes the analysis of a rebase more and more complicated when we
  move to a more recent version of Salt.

  Instead, moving from a new code base to a old one can be more
  simple.

# Design
[design]: #detailed-design

I propose a change in the workflow where:

* `develop` first.

  All fix, refactoring, documentation change or new feature will be
  done first against `develop`. The review process will happens there,
  and needs to be merged (and the CI fixed and the documentation in
  place) before it can be backported.

* No backport without an issue.

  Bugs in the product require the creation of an issue. Here we can
  discuss the versions that are affected. The developer can create the
  fix for `develop`, wait for the review and wait for the merge, and
  after that create the backports to the products based on this
  commit. The commit message will point to the issue number.

* Backport with cherry-pick -x

  Doing a cherry-pick -x will make easy to discover from where comes
  the commit. Ideally all the new commits that are on top of a release
  are all cherry-picks of code that is already in `develop`.

* No more mega-commits from the past.

  Because we have an issue for the fix, and we have an already merged
  code in `develop` that was reviewed, passed the CI and discussed,
  the review for the backport can be minimal and very fast.

  Once all the backports are merged, the issue can be closed, so there
  burden of the required janitor tasks are more simple, and can be
  done via GitHub filters.

## Alternatives
[alternatives]: #alternatives

Of course we can maintain the current model of development, accept a
subset of suggestions from here, or do something completely different.

Any change that makes `develop` the reference code base will improve
the situation for developers, and also once the fork is done to
generate a new product, the quality of the initial point will be
better (will require less effort to stabilize before the release).

## Unresolved questions
[unresolved]: #unresolved-questions

One open question is what to do when, even enforcing this policy, we
detect code changes in a product that is not living in `develop`. This
can happen easily when a developer with commit rights simply change
the code without reviews or backporting.

Some git hocks can help to detect and signalize the problem. In that
case we can remind about the new process, and ask for the creation of
the change in `develop` first.

During a period of time maybe both models needs to live together.

# Drawbacks
[drawbacks]: #drawbacks

All changes are complicated, require new mental models and create new
problems that need to be addressed later. IMHO this proposal try to
improve the situation, based on experiences on different and bigger
projects.
