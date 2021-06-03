- Feature Name: Windows Install Anywhere
- Start Date: 2021-01-19
- SEP Status: Accepted
- SEP PR: https://github.com/saltstack/salt-enhancement-proposals/pull/41
- Implementation PR: https://github.com/saltstack/salt/pull/60267

# Summary
[summary]: #summary

Allow the user to select the install location (`install_dir`) for Salt in the
Windows Installer. The default install location will be
`%ProgramFiles%\Salt Project\Salt` for the binary data and
`%ProgramData%\Salt Project\Salt` for the Root Directory (`root_dir`). Add a
command line switch to allow the user to set the `install_dir` via the command
line.

# Motivation
[motivation]: #motivation

Windows is not Linux

The Windows directory structure is such that installed software should reside in
the ``%ProgramFiles%`` directory on the system drive. Software should not be
installed on the root of the system drive. Salt is currently installed on the
root of the ``C:`` drive. This seems to be a paradigm brought over from Linux.

In Windows:

- Non-changing binaries should be in ``%ProgramFiles%``
- Volatile data should be in ``%ProgramData%``
- Configuration should be in the registry, though we make an exception for Salt
  due to its reliance on a config file

We want the Salt installer to be consistent with other enterprise installers
that follow the Windows paradigm for software installation.

# Design
[design]: #detailed-design

## Directory Structure
[directory-structure]: #directory-structure

In order to follow the Windows paradigm for software installation we will
separate out binary and volatile/config data. It is proposed that this be broken
out as follows:

**Binary Data**:

```
c:\salt\*.files -> C:\Program Files\Salt Project\Salt\*.files
c:\salt\bin     -> C:\Program Files\Salt Project\Salt\bin
```

**Config/PKI Data**:

```
c:\salt\conf -> C:\ProgramData\Salt Project\Salt\conf
```

**Volatile Data**:

```
c:\salt\srv -> C:\ProgramData\Salt Project\Salt\srv
c:\salt\var -> C:\ProgramData\Salt Project\Salt\var
```

## Registry

[registry]: #registry

In a Salt installation there is a root directory (`root_dir`) and an
installation directory (`install_dir`). The `root_dir` is where all Salt
non-binary data is stored. Other directories such as the config directory
(`conf_dir`) are based on the `root_dir`. The `install_dir` indicates the
location of the Salt binaries.

In the previous method of installation the default locations were `C:\salt` for
the `root_dir` and `C:\salt\bin` for the `install_dir`. The new installation
method will split these out and default to standard Windows locations in
`%ProgramData%` for `root_dir` and `%ProgramFiles%` for `install_dir`. It will
also allow the user to change the `install_dir` in the GUI or via a command line
switch. The settings will be stored in the registry at the following locations:

**install_dir**

- Path:     ``HKLM\\SOFTWARE\\Salt Project\\Salt``
- Key Name: ``install_dir``
- Default:  ``C:\\Program Files\\Salt Project\\Salt``

**root_dir**

- Path:     ``HKLM\\SOFTWARE\\Salt Project\\Salt``
- Key Name: ``root_dir``
- Value:    ``C:\\ProgramData\\Salt Project\\Salt``

These registry settings will be created by the installer after the user selects
the location for the Salt installation. If it is an existing old method
installation the registry entries will be created using the old locations as
their values.

## Program Files / ProgramData

[program_files_programdata]: #program-files-programdata

There are scenarios where the system drive contains only the system files with
all other software installed on a separate drive, ``D:\Program Files`` for
example. This is done by setting the environment variable ``%ProgramFiles%``.
The default location for program data can also be set using ``%ProgramData%``.
It is proposed that Salt use these environment variables to determine the
default location for the ``%Program Files%`` and ``%ProgramData%`` directories.

## Directory Permissions

[directory-permissions]: #directory-permissions

Salt will be separated into binary data and config/volatile data. Binary data
will be located in the ``%ProgramFiles%\Salt Project\Salt`` directory by
default. Config/volatile data will be located in the
``%ProgramData%\Salt Project\Salt`` directory.

The Salt directory in ``%ProgramData%`` will have the same permissions as those
applied to ``C:\salt`` with the additional restrictions applied to the ``pki``
directory. Only the process that is running Salt should have access to those
directories. It should be locked down to standard users.

Binary data installed in ``%ProgramFiles%`` will inherit the permissions applied
to the ``%ProgramFiles%`` directory. If Salt is installed in any other location
then it will be incumbent upon the user to set permissions to that directory.

## Installer Flags

[installer-flags]: #installer-flags

*install_location*

A new `install_location` installer flag will be added. This option will allow
the user to silently set the install location for Salt. The default will be
``%ProgramFiles%\Salt Project\Salt``. This would apply to new installations only
and will be ignored for all other scenarios.

# Installation Scenarios 

[installation-scenarios]: #installation-scenarios

There are 3 scenarios the installer will need to handle while installing Salt:

1. Existing New Method Installation
2. Existing Old Method Installation
3. New Installation

In either case of an existing installation, old or new, we want to just upgrade
in place. This means the `install_dir` and `root_dir` would remain unchanged. If
it's an existing new method installation, the installer would use the locations
defined in the registry. For old method installations the `install_dir` and
`root_dir` would both be `C:\salt`. In both cases the GUI would display the
current install location, but it would be greyed out. The new installation is
the only scenario that allows the user to select an install location. The GUI
will display a dialog box with a directory picker that will allow the user to
browse to a directory. The default will be the directory defined in the
`%ProgramFiles%` environment variable.

The `root_dir` is not user configurable. It will be `C:\salt` for old method
installations and `%ProgramData%` for new method installations.

In the future we may add the ability to install Salt on a per-user basis. In
that case the registry data would be found in HKCU, and the `root_dir` would
default to `%LOCALAPPDATA%`.

We don't want the installer to change the locations of `install_dir` and
`root_dir` for an existing installation. We don't want the installer to move any
existing config or binary files. If the user wishes to change the locations of
those files they would have to do that manually by uninstalling and reinstalling
Salt.

Below is the logic to handle each scenario:

### 1. Check for Existing New Method Installation

Check for an existing new method installation by checking the registry for the
presence of `HKLM\SOFTWARE\Salt Project\Salt`. This indicates that Salt was
installed using the new method. If the registry key exists, do the following:

- The `root_dir` is as defined in the registry
- The `install_dir` is as defined in the registry
- The install location `install_dir` is displayed in the GUI
- The install location is greyed out in the GUI
- Upgrade Salt in place 
- **** DONE ****

### 2. Check for Existing Old Method Installation

If a new method installation is not detected, check for an existing old method
installation by looking for the existence of `python.exe` in `C:\salt\bin`. This
would indicate that Salt was installed using the old method where everything is
in `C:\salt`. If the binary exists, do the following:

- The `root_dir` is `C:\salt`
- The `install_dir` is `C:\salt`
- The install location `install_dir` is displayed in the GUI
- The install location is greyed out in the GUI
- The `root_dir` and `install_dir` are saved in the registry
- Upgrade Salt in place
- **** DONE ****

### 3. New Installation

If neither installation type has been detected, then this must be a new
installation. The installer will do the following:

- The `root_dir` is `%ProgramData%\Salt Project\Salt`
- The `install_dir` defaults to `%ProgramFiles%\Salt Project\Salt`
- The install location `install_dir` is displayed in the GUI
- The user can change the install location
- The `root_dir` and `install_dir` are saved in the registry
- Install Salt
- **** DONE ****
