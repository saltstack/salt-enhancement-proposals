- Feature Name: update-engines-without-minion-restart
- Start Date: 2019-07-16 
- SEP Status: Draft
- SEP PR: https://github.com/saltstack/salt-enhancement-proposals/pull/16
- Salt Issue: 

# Summary
[summary]: #summary

Being able to restart minion engines without having to restart the minion itself.

# Motivation
[motivation]: #motivation

As a user I need to run a new engine on a minion, but restarting the minion needs to be done carefully and may not happen when I need it.
This is already possible with beacons.

# Design
[design]: #detailed-design

The engine configuration could be either in the configuration or in the pillar.
When refreshing the pillar, loop over the engines configuration and start or stop the engines according to the new configuration.

## Alternatives
[alternatives]: #alternatives

The alternative is to manually restart the salt minion, with all the problems this can bring.

## Unresolved questions
[unresolved]: #unresolved-questions

If this is too problematic to run on every pillar refresh, then an additional function could be provided to only refresh the engines.

# Drawbacks
[drawbacks]: #drawbacks

The engines documentation would need to add this new possibility to configure the engines using pillar.
