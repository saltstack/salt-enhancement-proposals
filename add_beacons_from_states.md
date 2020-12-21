- Feature Name: Add Beacons from Salt States
- Start Date: 2020-12-21
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

Adding a new global option to Salt state system that would audomatically add a beacon on the targeted minion for specific functions in specific state modules.

# Motivation
[motivation]: #motivation

Adding this new option would allow beacons easier and associating them with Salt managed resources.

# Design
[design]: #detailed-design

The design and implementation of this new option would done in a similar way to the mod_watch system.  During a state run
the various individual states will be scanned and those that include the beacon option will be run through a function that
calls a "mod_beacon" function in the desired state module.  This "mod_beacon" function would create the beacon using the
configured resources associated with the state in question, eg. the PKGs, files, or services.

## Alternatives
[alternatives]: #alternatives

N/A

## Unresolved questions
[unresolved]: #unresolved-questions

- If we want to allow the configured beacon to be configurable or only have specific states use specific beacons.
- Allow the beacons for specific use cases to be combined, eg. a common beacon which monitors PKGs.

# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this? Please consider:

- This addition would add another level of complexity to the state system.
