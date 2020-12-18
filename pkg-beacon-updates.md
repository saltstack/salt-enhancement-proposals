- Feature Name: Changes to PKG Beacon
- Start Date: 2020-12-18
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

Changes to the PKG beacon to fire events when software packages are removed and installed, in addition to when upgrades are available.
Because of the way beacons work, this change will also require changes to all PKG modules to ensure that there is the option in the form of a keyword argument to pull an updated list of installed packages and bypass the ___context dunder.   By default the list_pkgs functions will still continue to pull a list of installed packages from the context dunder dictionary.

# Motivation
[motivation]: #motivation

This change would be an enhancement to the PKG beacon, brining additional functionality.

# Design
[design]: #detailed-design

Currently the PKG beacon will monitor the list of installed packages and fire an event when an upgrade is available.

This change would update that functionality to fire an event the status of a package changes from installed to uninstalled and vice versa, in addition to when an updated version is available.

# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this? Please consider:

- This would change the format and the contents returned from the PKG beacon to the event bus.  This change will need to be communicated out to anyone relying on the existing format.
