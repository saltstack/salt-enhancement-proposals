- Feature Name: Python 3 Only Salt-SSH
- Start Date: 2019-05-01
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

Since Python 2 will be deprecated in 2020 we need to figure out a solution for Salt-SSH
to run on machines regardless of the Python version on the targeted system.

# Motivation
[motivation]: #motivation

Python 2 will be deprecated in 2020 and Salt plans on supporting only Python 3 in the
Sodium release or later. Due to this deprecation and migration to Python 3 we will
need to find a specific solution for Salt-SSH to continue to work against hosts
that still have only Python 2 installed, since by default Salt-SSH only works against
hosts that have the same python version.

# Design
[design]: #detailed-design

The design proposal to allow Salt-SSH to work with Python 3, includes multiple options:
  1. By default, if python versions match we will simply use the system python and Salt-SSH
     will run as it always has.
  2. Add a Salt-SSH configuration (for example: pre_flight) to run raw commands before the
     Salt-SSH command. This will allow a user to create a custom script they can run to install
     Python 3. It will only run these pre_flight commands if the tarball is not copied over.
  3. Update the current ssh_ext_alternatives feature in Salt-SSH to automatically attempt to
     find the necessary python libraries that Salt-SSH uses to copy over and check to see if
     they are importable. If a user needs additional libraries other than the default they will
     need to add those manually to the configuration.
  4. Maintain and build a Python 3 binary that will be copied over with the tarball to be included
     in the salt-call run. This will also be gated by a config option that a user will need to enable
     if they always want to use the binary option.

If the python versions do not match when Salt-SSH is run a message will be printed out asking the user
if they want to copy over the binary, and informing them of the other alternative options pre_flight and
ssh_ext_alternatives.


How are we going to build the Python 3 binary?
We will build the binary statically for both x86_64 and ARM architecture. We will include the x86_64
binary by default in the Salt-SSH package. If the ARM architecture is detected we will include a warning
to the user to download the binary from a specific URL if not already downloaded. We will include the
required libraries to run salt-call and include the pip binary. If users want to include other dependencies
on top of this binary they can use the ssh_ext_alternatives feature to include the additional dependencies.

Python Binary Security Releases:
We will need to monitor and make sure our Python 3 binary and other built libraries are kept up to date
with any security releases. Since the Python 3 binary will be managed outside of the salt repo we can do
releases outside of the Salt release cycle to include these new patches and updates quickly.

Python 3 Binary Test Plan:
We will need to make sure this is thoroughly tested. We will need to ensure that changes to Salt work with
the Salt vendored Python 3. We will also need to ensure that changes to the Salt vendored Python 3 work with
the existing Salt branches. We will do this by building automated tests against the Salt vendored Python 3
for each Salt branch and supported OS.


## Alternatives
[alternatives]: #alternatives

Alternatives for Python 3 static binary:
- Use an already maintained minimal version of Python 3, but we will not have as much control over what is added into the binary.
- Use a dynamic build, but this will require building out a package per distro.

## Unresolved questions
[unresolved]: #unresolved-questions

When we build the static python build we will need to review the licenses of all the libraries included in the build.


# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this? Please consider:

- Adding anything additional to the tarball can impact performance of Salt-SSH. We will need to make sure that
the addition of this Python 3 binary is as small as it possibly can be for the current Salt-SSH run. We should
also be using similar hashing comparisons so that the Python 3 binary is only copied over once.
