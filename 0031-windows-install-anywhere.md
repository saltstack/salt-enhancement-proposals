- Feature Name: Windows Install Anywhere
- Start Date: 2021-01-19
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

Allow the user to select the install location for salt in the Windows Installer.
Default to Program Files

# Motivation
[motivation]: #motivation

Windows is not Linux

The Windows directory structure is such that installed software should reside in
the ``Program Files`` directory on the ``C:`` drive. Software should not be
installed on the root of the system directory. This seems to be a paradigm
brought over from Linux.

- Non-changing binaries should be in ``Program Files``
- Volatile data should be in ``ProgramData``
- Configuration should be in the registry, though we would probably make an
  exception for salt due to its reliance on a config file
- Some config could be placed in the registry to tell salt where the config file
  resides

We want the salt installer to be consistent with other enterprise installers
that follow the Windows paradigm for Software installation

# Design
[design]: #detailed-design

In order for this to work we will have to separate out much of the salt code. We
need to separate binary and volatile data as well as config data. It is proposed
that it is broken out according to the following:

Binary Data:

c:\salt\*.files
-> C:\Program Files\<salt>\*.files

c:\salt\bin
-> C:\Program Files\<salt>\bin

Volatile Data:

c:\salt\conf
-> C:\ProgramData\<salt>\conf

c:\salt\var
-> C:\ProgramData\<salt>\var

c:\salt\srv
- C:\ProgramData\<salt>\srv

There is some discussion to be had over the name of the <salt> directory. Here
are a few proposals:

- Salt
- SaltProject
- Salt Project **
- saltproject.io

We recommend ``Salt Project``

Another issue of concern is how Salt will know where its config file resides. It
is proposed that a registry entry be set at the following location:

Path: HKLM\\SOFTWARE\\Salt Project\\salt
Key Name: config
Value: C:\\ProgramData\\Salt Project\\salt\\conf

This is assuming we go with ``Salt Project``. This registry setting will be
created by the installer after the user selects the location for the salt
installation.

There are scenarios where the System drive contains only the system files with
all other software installed on a separate drive, ``D:\Program Files`` for
example. This is done by setting the environment variable ``ProgramFiles``. The
default location for program data can also be set using ``ProgramData``. It is
proposed that Salt uses these environment variables to determine the default
location of ``Program Files`` and ``ProgramData``

## Alternatives
[alternatives]: #alternatives

What other designs have been considered? What is the impact of not doing this?

## Unresolved questions
[unresolved]: #unresolved-questions

What parts of the design are still TBD?

# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this? Please consider:

- Implementation cost, both in term of code size and complexity
- Integration of this feature with other existing and planned features
- Cost of migrating existing Salt setups (is it a breaking change?)
- Documentation (would Salt documentation need to be re-organized or altered?)


There are tradeoffs to choosing any path. Attempt to identify them here.
