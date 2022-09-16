- Feature Name: (deprecate_exsi_modules_in_favor_of_vmware_extensions)
- Start Date: (fill with today's date, 2022-09-16)
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

The current execution modules for esxi, esxcluster, exsdatacenter and esxvm should be deprecated and users pointed
to using the new VMware extensions.


# Motivation
[motivation]: #motivation

Given these excution modules currently do not get much attention and all VMware support is currently occuring on
the VMware extensions. Any functionality currently provided by this modules and not support by VMware extensions,
should be added to VMware extensions.

Should have a single point of development for VMware support, which is currently VMware extensions.
Remove chance of confusion as to which execution module to use.

# Design
[design]: #detailed-design

Design is already active in VMware extensions.
Updates to Salt's documentation for the execution modules for esxi, esxcluster, exsdatacenter and esxvm directing
users to VMware extensions documentation will need to be made.

## Alternatives
[alternatives]: #alternatives

Alternative would be to have leave things as they are leading to confusion as to which execution module to use.

## Unresolved questions
[unresolved]: #unresolved-questions

What parts of the execution modules for esxi, esxcluster, exsdatacenter and esxvm that are not provided by VMware extensions.
Requires a search through the more recent VMware releases for features supported, but given that the execution modules for
esxi, esxcluster, exsdatacenter and esxvm have not had much development since 2016 and only offer basic support, the search
should only be an excerise in ensuring no surprises.

# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this?

- Cost of migratign existing Salt installations:
    There will be a cost to migrating existing installations.
        However most of those users are probably VMware customers and there will be a period of time allowing them
        to migrate, and the enhanced and supported features of the VMware extensions should give encouragment to migrate.

 - Documentation for Salt.
    There will be work in updating the exisitng execution modules for esxi, esxcluster, exsdatacenter and esxvm documentation
    but it should be trival to add a line of text and a link redirecting users to the VMware extension documentation.


