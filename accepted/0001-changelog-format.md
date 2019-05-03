- Feature Name: Standarize on the format of changelogs
- Start Date: 2019-02-15
- SEP Status: Approved
- SEP PR: http://github.com/saltstack/salt-enhancement-proposals/pull/2 

# Summary
[summary]: #summary

This RFC proposes a standard changelog format for software released by the SaltStack project.

# Motivation
[motivation]: #motivation

The current format for changelog entries at Salt is unstructured and can be difficult to read.

Many users of Salt have complained that they weren't aware of a breaking change which may
be the result of a poorly structured and inconsistent changelog between releases.

# Design
[design]: #detailed-design

This RFC standardizes Salt on the 1.0.0 spec of project changelogs outlined at
https://keepachangelog.com/en/1.0.0/

Specifically, it provides for the following principles:

- Changelogs are for humans, not machines.
- There should be an entry for every single version.
- The same types of changes should be grouped.
- Versions and sections should be linkable.
- The latest version comes first.
- The release date of each version is displayed.
- Mention whether you follow Semantic Versioning.

Types of changes
----------------
- Added for new features.
- Changed for changes in existing functionality.
- Deprecated for soon-to-be removed features.
- Removed for now removed features.
- Fixed for any bug fixes.
- Security in case of vulnerabilities.

## Alternatives
[alternatives]: #alternatives

Current alternative is what we have now. ;)

## Unresolved questions
[unresolved]: #unresolved-questions

Perhaps a sample changelog written for a previous version might
be appropriate to put together and attach to this RFC?

# Drawbacks
[drawbacks]: #drawbacks

People could be used to the current version and depend on it.
This would be more work for contributors. It would take more attention from the core
team to attend to.

