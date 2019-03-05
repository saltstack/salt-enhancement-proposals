# Salt Enhancement Proposals

Many changes, including bug fixes and documentation improvements, can be
implemented and reviewed via the normal GitHub pull request workflow.

Some changes though are "substantial", and we ask that these be put through a
bit of a design process and produce a consensus among the Salt core team.

The "SEP" (Salt Enhancement Proposal) process is intended to provide a
consistent and controlled path for new features to enter the project.

This process is being **actively developed**, and it will still change as more
features are implemented and the community settles on specific approaches to
feature development.

## When to follow this process

You should consider using this process if you intend to make "substantial"
changes to Salt or its documentation. Some examples that would benefit from an
SEP are:

   - A new feature that creates new API surface area
   - The removal of features that already shipped
   - The introduction of new idiomatic usage or conventions

The SEP process is a great opportunity to get more eyeballs on your proposal
before it becomes a part of a released version of Salt. Quite often, even
proposals that seem "obvious" can be significantly improved once a wider group
of interested people have a chance to weigh in.

The SEP process can also be helpful to encourage discussions about a proposed
feature as it is being designed, and incorporate important constraints into the
design while it's easier to change, before the design has been fully
implemented.

Changes that do **NOT** require an SEP:

  - Rephrasing, reorganizing or refactoring
  - Bug fixes
  - Addition or removal of warnings
  - Additions only likely to be _noticed by_ other implementors-of-Salt,
    invisible to users-of-Salt.

## What the process is

In short, to get a major feature added to Salt, one must submit the SEP via
pull-request. After a comment period, the SEP will be either **Accepted** or
**Rejected**. An **Accepted** SEP may then be implemented with the goal of
eventual inclusion into Salt.

## The SEP life-cycle
The following is a more detailed explanation of the process.

### 1. Proposal
The SEP is proposed by submitting a pull request to this repo, by copying
`0000-template.md` and modifying it. If the SEP pertains to any open issues,
reference them in the **Salt Issue(s)** entry.

When copying the file, do _not_ assign it a number. Simply name the file with a
short description of the SEP (e.g. `subspace-transport.md`). Once the pull
request has been opened, a SaltStack core engineer will assign the SEP a number
and the SEP will enter the initial **Draft** status. At this time, the
following changes can be made to the SEP file:

1. Add the pull request number to the **SEP PR** entry at the top of the file
2. Rename the file to include the assigned SEP number, then commit and push to
   update the pull request
    ```bash
    $ git mv subspace-transport.md 0123-subspace-transport.md
    $ git commit -am 'Assigned SEP number'
    $ git push origin branchname
    ```

### 2. Discussion
The pull request will remain open and serve as the comment thread for the SEP.
The initial comment period will last no fewer than two (2) weeks, and may be
extended as deemed necessary based on comment activity.

Once the initial comment period has ended, the SEP will enter **Final Comment**
status. One (1) week will be allowed for any further comments. At the end of
the **Final Comment** period, a decision will be made on whether or not to
accept the SEP. Acceptance requires approval from five (5) members of the core
development team. As project creator, [Thomas
Hatch](https://github.com/thatch45) will have a final veto on any SEP.

Additionally, as
[discussed](https://github.com/saltstack/salt-enhancement-proposals/pull/1#issuecomment-468823572), SaltStack will be publishing a SEP to form a Community Advisory Board (CAB).

### 3. Acceptance / Rejection
At the end of the **Final Comment** stage, the SEP will be either **Accepted**
or **Rejected**. Either way, the pull request will be merged.

Note that acceptance does not mean that the feature will be immediately
implemented, or that it will be implemented at all; It merely means that the
core development team has agreed to it in principle. Additionally, the fact
that an SEP pull request has been merged does not necessarily mean that the SEP
has been accepted; pull requests for rejected SEPs are merged so that they are
visible to others who might otherwise open an SEP for a previously-rejected
topic.

### 4. Implementation
An **Accepted** SEP may proceed to be implemented. If no issues on the Salt
issue tracker are listed under **Salt Issue(s)**, then create one and open a
pull request to update the SEP with the issue number. All SEPs which have
reached the implementation step must have at least one associated issue.

We should strive to write each SEP in a way that it will reflect the final
design of the feature; However, if during implementation things change, the SEP
document should be updated accordingly.

The SEP author (like any other developer) is welcome to post an implementation
for review after the SEP has been accepted. However, the author of an SEP is
not obligated to implement it.

If you are interested in working on the implementation for an accepted SEP, but
cannot determine if someone else is already working on it, feel free to ask
(e.g. by leaving a comment on the associated issue).

## Summary of SEP Statuses
The below statuses were discussed above:
- **Draft**: The initial status, from submission through the initial discussion
  period
- **Final Comment**: A one-week period after the initial comment period has
  ended
- **Accepted**: The SEP has been approved for future implementation
- **Rejected**: The SEP has been rejected during discussion phase

In additon, SEPs can be assigned the following statuses:
- **Withdrawn**: The SEP has been voluntarily withdrawn from consideration
- **Implemented**: The accepted SEP has been implemented
- **Obsolete**: The accepted SEP is no longer relevant due to other changes in
  Salt, but should be considered for re-evaluation. The re-evaluation will be
  done in a separate SEP. Once the new SEP is opened, the **Obsolete** SEP will
  be considered **Replaced**.
- **Replaced**: The SEP has been superseded by another SEP

The SEP's status can be viewed in two ways:

1. In the **Status** entry at the top of the SEP file
2. Via GitHub labels applied to the SEP's pull request

## Reviewing SEPs
SaltStack staff will post information about open SEPs to the **#sep** channel
in the community Slack, as well as our community IRC and mailing list on a
regular basis to encourage discussion.

**This SEP process owes its inspiration to the [React RFC process], [Yarn RFC
process], [Rust RFC process], and [Ember RFC process]**

[React RFC process]: https://github.com/reactjs/rfcs
[Yarn RFC process]: https://github.com/yarnpkg/rfcs
[Rust RFC process]: https://github.com/rust-lang/rfcs
[Ember RFC process]: https://github.com/emberjs/rfcs
