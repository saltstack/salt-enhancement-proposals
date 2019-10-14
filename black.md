- Feature Name: blackened-seasoning
- Start Date: 2019-10-04
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

Adopt `pre-commit`, `black`, and `isort` for Salt.

# Motivation
[motivation]: #motivation

There have been a *number* of PRs to Salt that have had minor style changes
requested, either to bring the coding style in to line with the general Salt
project or within the specific module, when it deviates from Salt's typical
standards.

Additionally there are a number of commits that are produced just to satisfy
the linter. If we automate the formatting of our code, this goes away. We also
achieve consistency throughout the Salt codebase.    


# Design
[design]: #detailed-design

### Black, Pre-commit, and You
 
Salt will adopt [Black](https://github.com/psf/black) as our code formatter of
choice. By adopting an opinionated formatter we are taking the approach that
there is only “one way to do it”. Accepting double quotes was extremely
difficult for Tom, but if he can do it, anyone can.

Salt will also adopt [pre-commit](https://pre-commit.com) in order to automate
the use of Black and lint. As `pre-commit` runs `black`, `isort` (using `force_single_line=True`) and lint
checks, it will allow us all to focus on the logic and design of the PRs
which will increase the stability and quality of Salt.

#### Lint Names

Additionally, on the rare occasion that we need to disable lint rules, we will
disable based on name, instead of number. The existing disabled lint numbers
will be changed to the corresponding names as part of the PR that implements
this SEP. 

### Git Blame? 

Because the introduction of Black will absolutely touch every file in the
codebase (except perhaps a `__init__.py` file or two), a member of the Salt
core team will create that commit using the `--author` flag set to `Salt
Refactor <jenkins@saltstack.com>` so it’s
obvious in `git blame` that those changes are due to `black`ening the codebase.

This commit hash will also be added to a `.git-blame-ignore-revs` file, so it
can easily be used in the `git blame --ignore-revs-file=.git-blame-ignore-revs`
command, as well as automatically be used with the `git hyper-blame` extension.

### Updated Docs

Adopting these tools for automation will require us to update our documentation
to make it simple for Salt contributors.

## Alternatives
[alternatives]: #alternatives

We could use `sq-black`, a fork of Black that allows single quotes. This
would require maintenance of the fork.

## Unresolved questions
[unresolved]: #unresolved-questions

None?

# Drawbacks
[drawbacks]: #drawbacks

We could *not* do this, because it will add some overhead when researching
issues via `git blame`. Especially formatting changes. It will require updating
the documentation. There will also require rebasing of any work done before the
formatting change.