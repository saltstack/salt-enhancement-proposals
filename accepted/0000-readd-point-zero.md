- Feature Name: Add point zero to major releases for Salt
- Start Date: 2021-06-01
- SEP Status: Final Comment Period
- SEP PR: https://github.com/saltstack/salt-enhancement-proposals/pull/49
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

Originally, Salt version numbers were date based as in YYYY.MM.DD and from SEP 14 we moved away from dates and general numbers, and removed the point zero from major release version numbers. This is to add the standard format back to the major versions because it is problematic and inconsistent.


# Motivation
[motivation]: #motivation

Why are we doing this? What use cases does it support? What is the expected outcome?

When we changed from date-based version numbers to numbered releases with `3000` ideally there isn't a need for point releases, however; it isn't realistic in regards to where the stability of Salt releases are, today, it causes inconsistency in versions for example if there is a Security Release, or critical bug to be fixed, and it causes problems with packaging. The expectation is eventually only have point releases to major versions if there is a Security Release, but to also continue to use the version `3000.0` for major releases as a indication of the major release and to ensure we are not causing issues for the packaging of Salt.

# Design
[design]: #detailed-design

Revert changes made to not use the point version on major releases and henceforth and for the foreseable future use points in all releases.

## Alternatives
[alternatives]: #alternatives

What other designs have been considered? This is a reversal so we have tried the alternatives and know it didn't work.

What is the impact of not doing this? The impact of not making this change is the current pain and suffering we now see from not moving towards an iterative or phased change implementation. We did not forsee the problems with packaging in [SEP 14](https://github.com/saltstack/salt-enhancement-proposals/blob/master/accepted/0014-dev-overhaul.md) implementation, but do now and wish to be kinder to our future selves.

## Unresolved questions
[unresolved]: #unresolved-questions

What parts of the design are still TBD?
Do we need to do this with supported major versions of Salt? If so, how and when? Updated: 2021-JUN-22 Answer: No, it is not the intent of this SEP to revisit the past releases, but to set for the standard operating procedure and the process going forward. If warranted to re-release a major version of Salt originally released as `####` without the point zero, likely it would need separate analysis and discussion. Today, and for this SEP it is to adjust the processs for future major releases, only.

# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this? Please consider:

- Implementation cost, both in term of code size and complexity, exist. This change will revert complexities in the current code base and return it to a previous state, and will cost time and effort to revert the changes.
- Integration of this feature with other existing and planned features - this shouldn't be an problem, but is likely a risk.
- Cost of migrating existing Salt setups (is it a breaking change?). No this should be a correction to breaking packaging with SEP 14 changes in the first release of Salt with version `3000` but isn't an attempt to re-release.
- Documentation (would Salt documentation need to be re-organized or altered?) Yes, any reference to major release as `3000` instead of `300x.0` such as `3001` `3002` and `3003` this likely will need to be corrected in the .rst files but can be left as-is on any non-code or linked documenation such a blog posts, social media, and the like.
- Inconsisencies will exist and this SEP is one attempt at mitigation, likely there are other ways to improve and we can cite this SEP in all documentation and likely write a blog post or other content, as well.
- Unknown risks are common and likely and can be added once identified.


There are tradeoffs to choosing any path. Attempt to identify them here.
