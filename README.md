# Salt RFCs

Many changes, including bug fixes and documentation improvements, can be
implemented and reviewed via the normal GitHub pull request workflow.

Some changes though are "substantial", and we ask that these be put through a
bit of a design process and produce a consensus among the Salt core team.

The "RFC" (request for comments) process is intended to provide a consistent
and controlled path for new features to enter the project.

This process is being **actively developed**, and it will still change as more
features are implemented and the community settles on specific approaches to
feature development.

## When to follow this process

You should consider using this process if you intend to make "substantial"
changes to Salt or its documentation. Some examples that would benefit from an
RFC are:

   - A new feature that creates new API surface area
   - The removal of features that already shipped
   - The introduction of new idiomatic usage or conventions

The RFC process is a great opportunity to get more eyeballs on your proposal
before it becomes a part of a released version of Salt. Quite often, even
proposals that seem "obvious" can be significantly improved once a wider group
of interested people have a chance to weigh in.

The RFC process can also be helpful to encourage discussions about a proposed
feature as it is being designed, and incorporate important constraints into the
design while it's easier to change, before the design has been fully
implemented.

Changes that do **NOT** require an RFC:

  - Rephrasing, reorganizing or refactoring
  - Bug fixes
  - Addition or removal of warnings
  - Additions only likely to be _noticed by_ other implementors-of-Salt,
    invisible to users-of-Salt.

## What the process is

In short, to get a major feature added to Salt, one must submit the RFC via
pull-request. After a comment period, the RFC will be either **Accepted** or
**Rejected**. An **Accepted** RFC may then be implemented with the goal of
eventual inclusion into Salt.

## The RFC life-cycle
The following is a more detailed explanation of the process.

### 1. Proposal
The RFC is proposed by submitting a pull request to this repo, by copying
`0000-template.md` and modifying it. If the RFC pertains to any open issues,
reference them in the **Salt Issue(s)** entry.

When copying the file, do _not_ assign it a number. Simply name the file with a
short description of the RFC (e.g. `subspace-transport.md`). Once the pull
request has been opened, a SaltStack core engineer will assign the RFC a number
and the RFC will enter the initial **Draft** status. At this time, the
following changes can be made to the RFC file:

1. Add the pull request number to the **RFC PR** entry at the top of the file
2. Rename the file to include the assigned RFC number, then commit and push to
   update the pull request
    ```bash
    $ git mv subspace-transport.md 0123-subspace-transport.md
    $ git commit -am 'Assigned RFC number'
    $ git push origin branchname
    ```

### 2. Discussion
The pull request will remain open and serve as the comment thread for the RFC.
The initial comment period will last no fewer than two (2) weeks, and may be
extended as deemed necessary based on comment activity.

Once the initial comment period has ended, the RFC will enter **Final Comment**
status. One (1) week will be allowed for any further comments. At the end of
the **Final Comment** period, a decision will be made on whether or not to
accept the RFC. Acceptance requires approval from five (5) members of the core
development team. As project creator, [Thomas
Hatch](https://github.com/thatch45) will have a final veto on any RFC.

### 3. Acceptance / Rejection
At the end of the **Final Comment** stage, the RFC will be either **Accepted**
or **Rejected**. Either way, the pull request will be merged.

Note that acceptance does not mean that the feature will be immediately
implemented, or that it will be implemented at all; It merely means that the
core development team has agreed to it in principle. Additionally, the fact
that an RFC pull request has been merged does not necessarily mean that the RFC
has been accepted; pull requests for rejected RFCs are merged so that they are
visible to others who might otherwise open an RFC for a previously-rejected
topic.

### 4. Implementation
An **Accepted** RFC may proceed to be implemented. If no issues on the Salt
issue tracker are listed under **Salt Issue(s)**, then create one and open a
pull request to update the RFC with the issue number. All RFCs which have
reached the implementation step must have at least one associated issue.

We should strive to write each RFC in a way that it will reflect the final
design of the feature; However, if during implementation things change, the RFC
document should be updated accordingly.

The RFC author (like any other developer) is welcome to post an implementation
for review after the RFC has been accepted. However, the author of an RFC is
not obligated to implement it.

If you are interested in working on the implementation for an accepted RFC, but
cannot determine if someone else is already working on it, feel free to ask
(e.g. by leaving a comment on the associated issue).

## Summary of RFC Statuses
The below statuses were discussed above:
- **Draft**: The initial status, from submission through the initial discussion
  period
- **Final Comment**: A one-week period after the initial comment period has
  ended
- **Accepted**: The RFC has been approved for future implementation
- **Rejected**: The RFC has been rejected during discussion phase

In additon, RFCs can be assigned the following statuses:
- **Withdrawn**: The RFC has been voluntarily withdrawn from consideration
- **Implemented**: The accepted RFC has been implemented
- **Obsolete**: The accepted RFC is no longer relevant due to other changes in
  Salt, but should be considered for re-evaluation. The re-evaluation will be
  done in a separate RFC. Once the new RFC is opened, the **Obsolete** RFC will
  be considered **Replaced**.
- **Replaced**: The RFC has been superseded by another RFC

The RFC's status can be viewed in two ways:

1. In the **Status** entry at the top of the RFC file
2. Via GitHub labels applied to the RFC's pull request

## Reviewing RFCs
SaltStack staff will post information about open RFCs to the **#rfc** channel
in the community Slack, as well as our community IRC and mailing list on a
regular basis to encourage discussion.

**This RFC process owes its inspiration to the [React RFC process], [Yarn RFC
process], [Rust RFC process], and [Ember RFC process]**

[React RFC process]: https://github.com/reactjs/rfcs
[Yarn RFC process]: https://github.com/yarnpkg/rfcs
[Rust RFC process]: https://github.com/rust-lang/rfcs
[Ember RFC process]: https://github.com/emberjs/rfcs

