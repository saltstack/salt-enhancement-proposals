- Feature Name: Inclusive Language
- Start Date: 2020-10-05
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

Following examples set by others in the tech industry and even the Linux kernel itself, Salt should adopt inclusive language to replace racially-charged jargon.

# Motivation
[motivation]: #motivation

**Why are we doing this?** If Salt is to continue to grow and gain wider adoption, it needs to drop references to terms that do not have a place in this world.

**What use cases does it support?** This will affect all current and future users of Saltstack.

**What is the expected outcome?** Salt will join a growing number of companies and projects that are making changes to use inclusive language.

**If this SEP is not accepted?** I don't believe there is an alternative way around this.

# Design
[design]: #detailed-design

**Proposed Updated Terminology**
Salt does use non-inclusive language for a key component. While I will submit what is becoming commonly accepted alternatives, I think the Salt Master/Minon terms will require some larger considerations. I understand that it's not just updating a configuration term, but falls into the larger product branding. I understand this will likely touch a lot of files, but should not require any significant refactoring of code. Mostly just agreeing upon new terms and making sure they mach up as they are updated. 

| Avoid non-inclusive language | Prefer inclusive versions |
| ---------------------------- | ------------------------- |
| Whitelist | Allowlist |
| Blacklist | Blocklist |
| Master/slave | Leader/follower, primary/replica, primary/standby |
| Grandfathered | Legacy |
| Gendered pronouns (e.g. guys) | Folks, people, you all, y'all |
| Gendered pronouns (e.g. he/him/his) | They, them, their |
| Man hours | Person hours, engineer hours |
| Sanity check | Quick check, confidence check, coherence check |
| Dummy value | Placeholder value, sample value |

## Alternatives
[alternatives]: #alternatives

**What is the impact of not doing this?** I don't believe it is possible to have an alternative to this. You are either making a change or not. Not making the change just leaves the old terminology intact.

## Unresolved questions
[unresolved]: #unresolved-questions

Specific terminology to use should be discussed, agreed upon and then formalized. This should prevent competing terms from making their way into the code.

# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this? Please consider:

Without digging into the code, I do not have a full picture of what the impact will be. This will consume time. This will likley cause backwards compatibility issues a likely break communications between different verisons of releases. Documentation and formal terminology will need to be updated. Since a `salt-master` is a significant part of Salt's branding, there is likely to be marketing costs for making this change. Regarding alternatives, I cannot imagine an alternative way around solving this problem.
