- Feature Name: Differentiate between onchanges and onchanges_any
- Start Date: 2020-04-14
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

The `onchanges_any` requisite for state runs was implemented along with a handful of other _`_any`_ requisite state arguments.  When these changes were added it was done with the assumptions that `onchanges` would only be triggered if ALL the state functions returned True.  This resulted in duplication funcationality in the `onchanges` and `onchanges_any` requisites.

This SEP is propsing to update the `onchanges` requisite to trigger only when ALL the referenced state functions return True and for the `onchanges_any` to remain in it's current state, triggering when any of the state functions return True.

# Motivation
[motivation]: #motivation

Currently having two requisites with duplicate funcationality is confusing.

# Design
[design]: #detailed-design

The proposed design is to change the `onchanges` requisite to trigger when ALL state functions return True rather than the current implementation which triggers when ANY of the state functions return True.

## Alternatives
[alternatives]: #alternatives

An alternative approach would be to remove `onchange_any` and leave the `onchanges` requisite as is, triggering when any state function returns True.

## Unresolved questions
[unresolved]: #unresolved-questions

None

# Drawbacks
[drawbacks]: #drawbacks

There is a drawback to making a change to the funcationality of an existing requisite that users are relying on working a specific way.  Because of this the change should be announced to be made two releases after the SEP is approved and merged.
