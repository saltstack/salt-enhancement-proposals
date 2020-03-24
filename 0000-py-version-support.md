- Feature Name: Python Version Support
- Start Date: 2020-03-24
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

This SEP is being created to add a policy around Python versions Salt will support and
detail how we will support Python versions going forward. This policy change is to align Salt Releases
with Python's End of Life schedule.

# Motivation
[motivation]: #motivation

Documentation and Communication
-------------------------------
One of the motivating factors around this policy change is to make it more clear what our Python
version support is for each Salt releases. If it is clearly defined, then packagers, community members,
and Salt users will know when a Python version will be dropped for a given Salt release.
In the past we had never clearly defined how our Python version support lined up with Python's
End of Life schedule. We previously made a best effort to support the Python versions that lined
up with the operating systems Python versions, but never clearly defined this and did not always
follow this in every case. For example, when we dropped Python 2.6 support we still supported
Redhat 6, which had Python 2.6 installed by default. To continue supporting Redhat 6 while also
removing support for the End of Life Python 2.6 version, we included Python 2.7 in our packages
for that operating system.

Including Python in Packages
----------------------------
Another motivating factor to make this policy change is our shift to including Python in our
packages. For the Sodium release we will be using salt-bin (https://github.com/saltstack/salt-bin) to
package for our Fully Supported Platforms.
(http://get.saltstack.com/rs/304-PHQ-615/images/SaltStack-Supported-Operating-Systems.pdf)
This will allow us to easily continue supporting operating systems that include End of Life Python
versions by default by providing the Python binary in the Salt package included in a Salt release.

Security and Maintenance Concerns
---------------------------------
The last motivating factor surrounds security and maintenance concerns. Continuing to support an End
of Life Python version adds additional maintenance around testing and development for each version.
Supporting these versions also puts users at a security risk, since no more security fixes will be
submited and released with that Python version if the operating system does not choose to update
it themselves.

# Design
[design]: #detailed-design

The design goal of this SEP is to define a clear policy around Salt's Python version support.
This new policy defines only supporting major Python versions until they reach End of Life. You can see
the current status of supported Python branches here https://devguide.Python.org/#status-of-Python-branches
with the associated End Of Life date for each branch. You can also see details about Python versions that
have already become End Of Life here: https://devguide.Python.org/devcycle/#end-of-life-branches.

What if a Python version drops support relatively close to a Salt Release?
--------------------------------------------------------------------------
We may drop support for a Python version if the End of Life date lands relatively close to a Salt release.
For example if a Python version is End of Life a couple weeks before a Salt release, support for that
Python version may drop in that Salt version. This will need to be clearly communicated beforehand
with the community.

How will this impact the Sodium release?
----------------------------------------
Because Python 3.4 became End Of Life on 2019-03-18, we will only support Python versions >= 3.5

How does this impact all supported Operating Systems
----------------------------------------------------
The details below will include the impact for our supported operating systmes included here
and some future operating systems we will be supporting:
http://get.saltstack.com/rs/304-PHQ-615/images/SaltStack-Supported-Operating-Systems.pdf


AIX/Solaris
-----------
AIX/Solaris are Enterprise packages and will use salt-bin for their packaging mechanism,
so they will include the correct Python version.

Windows
-------
Python does not come pre-installed on Windows operating systems. We have always included
the Python binary in the package and will include to do that going forward. If someone
wants to pip install Salt onto windows they will need to install a supported Python version
first, just as they always have. In the case for the Sodium release this will mean they
will need to install Python >=3.5

OSX
---
Apple does not clearly state when their operating systems become End of Life, but they do
always support the three latest versions of OSX. We can estimate these End of Life on some of the
operating systems based on when the latest ones released. The dates included in the table
below are not confirmed by Apple and are estimates to help determine how this policy will
affect macosx.

|     OS     |    End of Life      | Default Python |
|------------|---------------------|----------------|
|   10.13    |    September 2020   |    2.7         |
|   10.14    |    September 2021   |    2.7         |
|   10.15    |    TBD              |    2.7         |
|   10.16    |    TBD              |   no default   |

We have always included the Python binary with the mac packages, so that will not change for
users that use the mac Python packages. Starting with the Sodium release we will no longer
support Python 2.7, so if users install Salt on OSX using pip or any other means they will
need to install Python >= 3.5.

Fedora
------
Fedora releases a new version every 6 months and supports these releases for approximately 13 months
. Our current support mirrors that of the upstream and they are currently supporting Fedora 30, 31 and
32. They do not have current EOL dates for these releases yet here:
https://endoflife.software/operating-systems/linux/fedora

|     OS     |    End of Life              |  Default Python |
|------------|-----------------------------|-----------------|
|     30     |    TBD (possibly june 2020) |      3.7        |
|     31     |    TBD (possibly oct 2020)  |      3.7        |
|     32     |    TBD (possibly apr 2020)  |      3.8        |

Based off of these dates and Python's End of Life dates, Fedora will not
be of any concern with this change in policy. Fedora releases fairly regularly
and includes the latest Python version. Both packages and pip installs
will work against supported Fedora versions going forward.

SLES/OpenSuse
----
The General End of Life dates do conflict with the current Python Version
End of Life dates. Since Suse does the packaging here they have been informed
of this policy change and will include a resolution around this with their packages
when these version conflicts occur.

Other Linux
-----------

|     OS     |    End of Life      | Default Python |
|------------|---------------------|----------------|
|     Rhel 6 |    30 Nov 2020      |      2.6       |
|     Rhel 7 |    30 Jun 2024      |      2.7       |
|     Rhel 8 |    05 May 2029      |      None      |
|     Ubuntu 16.04  |    April 2021    |    2.7/3.5 |
|     Ubuntu 18.04  |    April 2023    |    3.6     |
|     Ubuntu 20.04  |    2030          |    3.8     |
|     Debian 8      |    30 June 2020  |  2.7, 3.4  |
|     Debian 9      |    June 2022     |  2.7, 3.5  |
|     Debian 10      |    June 2024    |  2.7, 3.7  |

Based on this chart above all of these Operating Systems will or already have conflicted with
the Python's End of Life Support dates. We will need to include Python in our packages for these
Operating Systems especially at any given time when their default Python version is considered
End of Life. This does impact users who would want to pip install Salt. They will need to install
a newer version of Python before they can install Salt.

Other Reasonable-Effor Support Linux
------------------------------------

Arch
----
Arch is a rolling release and is consistantly updating their Python version, so this is not
of any concern for this policy.

FreeBSD
-------
According to this https://wiki.freebsd.org/Python/PortsPolicy FreeBSD currently already aligns
with this policy. They only support Python versions until they are End of Life so there are
no concerns with this operating system.

SmartOS
-------
SmartOS includes the latest Python version so will not be impacted by this change.


What about dependencies?
------------------------
Before dropping an End of Life Python version we should have an idea beforehand if Salt's dependencies
do not support a particular Python version with our testing suite. Currently our tests use the dependencies
in the requirements directory in the main Salt project. This does of course mean that if we are not testing
a module we would not have this information beforehand. We are now requiring tests to be added
for PRs so this will help ensure this is caught for those modules, but there are still some modules that do
not have tests currently.

For the Sodium Release, I installed the libraries from anything defined with :depends: in the documentation
and the requirements defined in our requirements directory. Once these were installed I ran
this tool https://pypi.org/project/checkmyreqs/ against all installed Python libraries. This brought to light
the following libraries that will not support Python 3, which will impact some Salt components:

 - pycassa (salt/modules/cassandra.py, salt/returners/cassandra_return.py)
 - vbox (salt/modules/vbox_guest.py, salt/modules/vbox_guest.py,
         salt/modules/vboxmanage.py, salt/utils/virtualbox.py,
         salt/cloud/clouds/virtualbox.py)

Both of these projects have not had commits recently added to the project in years. This will need to be documented
for the Sodium release to make it clear that these modules will not work unless the upstream projects add
Python 3 support.

Going forward adding more test coverage will help to uncover these projects before we drop a Python version,
but if we do not have enough test coverage, this exercise should be done any time a Python Version is dropped.
There might be a conflict between a Salt dependency and its Python version support that will impact many users.
If this conflict does occur we may need to delay dropping support for the End of Life Python version until that
dependency is updated to the appropriate Python version.

## Who is Affected

Salt Contributors
-----------------
When a Python Version is dropped any contributions will need to ensure they align with the syntax
of the lowest supported Python version. These contributors can look at Python's Release Notes for
details around what is added into a release or removed. For example for Python 3.6
https://docs.Python.org/3/whatsnew/3.6.html outlines some of the changes and removals added to this version.

Salt User
---------
If a Salt user is using the packages from https://repo.saltstack.com they will not be affected
as the Python binary will be included in the package. If they are pip installing the package and
their operating system Python version is an End of Life Python version they will need to upgrade
they're Python version before being able to pip install Salt.

Users creating their own packages
---------------------------------
If a user creates their own packages they will only be impacted if they are packaging for an operating
system that includes an End of Life Python Version. They would need to either create their packages using
salt-bin or include a supported version of Python in their packages.

When will this happen
---------------------
This Python Version Support change will go into affect as soon as its approved, but will first impact
the Sodium Release. The Sodium Release is due to be released in June and will support Python versions >= 3.5.

## Alternatives
[alternatives]: #alternatives

- Alternative is to keep supporting insecure Python versions that line up with supported operating systems

## Unresolved questions
[unresolved]: #unresolved-questions

- Will need to document our Python version support. This documentation should include which versions of Salt
  support which versions of Python. This document will need to be updated each release.

# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this? Please consider:

- Would impact operating systems we do not package for, if they only have an older Python version available.
