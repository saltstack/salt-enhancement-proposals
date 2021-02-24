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
installed on the root of the system directory. Salt is currently installed on
the root of the ``C:`` drive. This seems to be a paradigm brought over from
Linux. In Windows:

- Non-changing binaries should be in ``Program Files``
- Volatile data should be in ``ProgramData``
- Configuration should be in the registry, though we would probably make an
  exception for salt due to its reliance on a config file
- Some config could be placed in the registry to tell salt where the config file
  resides

We want the salt installer to be consistent with other enterprise installers
that follow the Windows paradigm for software installation

# Design
[design]: #detailed-design

### Directory Structure
[directory-structure]: #directory-structure

In order for this to work we will separate out separate binary and
volatile/config data. It is proposed that this be broken out as follows:

**Binary Data**:

```
c:\salt\*.files -> C:\Program Files\<salt>\*.files

c:\salt\bin     -> C:\Program Files\<salt>\bin
```

**Config/PKI Data**:

```
c:\salt\conf -> C:\ProgramData\<salt>\conf
```

**Volatile Data**:

```
c:\salt\var -> C:\ProgramData\<salt>\var

c:\salt\srv -> C:\ProgramData\<salt>\srv
```

There is some discussion to be had over the name of the <salt> directory. Here
are a few proposals:

- Salt
- SaltProject
- Salt Project <===
- saltproject.io

We recommend ``Salt Project``

### Registry

[registry]: #registry

Another issue of concern is how salt will know where its config file resides. It
is proposed that a registry entry be set at the following location:

- Path: ``HKLM\\SOFTWARE\\Salt Project\\salt``
- Key Name: ``config``
- Value: ``C:\\ProgramData\\Salt Project\\salt\\conf``

This is assuming we go with ``Salt Project``. This registry setting will be
created by the installer after the user selects the location for the salt
installation.

### Program Files / ProgramData

[program_files_programdata]: #program-files-programdata

There are scenarios where the system drive contains only the system files with
all other software installed on a separate drive, ``D:\Program Files`` for
example. This is done by setting the environment variable ``ProgramFiles``. The
default location for program data can also be set using ``ProgramData``. It is
proposed that salt use these environment variables to determine the default
location of the ``Program Files`` and ``ProgramData`` directories.

### Directory Permissions

[directory-permissions]: #directory-permissions

Salt will be seperated into binary data and config/volatile data. Binary data
will be located in the ``Program Files`` directory by default. Config/volatile
data will be located in the ``ProgramData`` directory.

Binary data installed in ``Program Files`` will inherit the permissions applied
to the ``Program Files`` directory. If salt installed in any other location then
the permissions will be the same as those applied to ``C:\salt``.

The salt directory in ``ProgramData`` will have the same permissions as those
applied to ``C:\salt`` with the additional restrictions applied to the ``pki``
directory. Only the process that is running salt should have access to those
directories. It should be locked down to standard users.

### Upgrading Existing Installations

[upgrading-existing-installations]: #upgrading-existing-installations

In situations where there is an existing salt installation the default behavior
will be to upgrade salt in place. If using the GUI, the existing location will
be displayed but will be grayed out. This would be the same behavior for the old
and new directory paradigms.

If salt is installed in the old location (``C:\salt``) a dialog box will be 
displayed prior to the location dialog box that says:

```
An existing salt installation was found at C:\salt. Salt now supports installing
to the Program Files directory. Would you like to migrate the existing salt
installation to the Program Files directory?
```

If the user selects ``Yes``, then the installer will attempt to move the
contents of the ``C:\salt`` directory to ``Program Files`` and ``ProgramData``
directories accordingly. Then the new default installation path will be
displayed in the GUI and will be grayed out. An upgrade will then be performed
by the installer in the new location. The registry will reflect the location of
the new config directory.

If the user selects ``No``, then the old installation path will be displayed in
the GUI and will be grayed out. The installer will upgrade salt as before
without attempting to modify the existing installation location. The registry
will reflect the location of the old config directory.

The only time you will be able to select the installation location will be
during a new install. The GUI will display a dialog box with a directory picker
that will allow you to browse to a directory. The default will be the directory
defined in the %ProgramFiles% environment variable.

### Installer Flags

[installer-flags]: #installer-flags

*install_location*

This option will allow you to silently set the install location for salt. The
default will be ``%ProgramFiles%``. This would apply to new installations only.

*migrate_existing_install*

This option will migrate an existing salt installation under ``C:\salt`` to the
``%ProgramFiles`` and ``%ProgramData%`` directories accordingly. Other existing
install locations not be affected by this flag. The *install_location* flag will
be ignored.

If this flag is not set and there is an existing salt installation at
``C:\salt``, then salt will be upgraded in place.
