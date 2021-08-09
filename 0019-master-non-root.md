- Feature Name: [security] Run salt master services as non root user by default
- Start Date: 2021-08-03
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

Salt has historically assumed that all salt components run as root. 
Although support has been added for running the master and minion as non root users, this is not simple to set up and the documentation discourages it.
This has had a further impact that the salt cli tools have to be run as root also.

It has long been a standing best-practice to run serving daemons under regular user privileges, opening up additional capabilities as necessary.
The `salt-master` and `salt-api` processes do not require root privileges for normal operation.

The `salt-master` and `salt-api` both handle connections from potentially untrusted clients. By reducing the privilge of these processes we
can reduce the impact of remotely exploitable vulnerabilities in those services.

The `salt-minion` process generally requires root privileges in order to effectively manage the systems it runs on. It may be possible to modify the
`salt-minion` process to run with lower privilege and gain additional privileges as required, but that is out of scope for this SEP due to the
additional complexity that would require and because the `salt-minion` process is not require to accept incoming connections from potentially
untrusted clients.

An earlier version of this SEP also covered changes to config file locations, but that has been removed for simplicity and can be dealt with as
a seperate SEP if required.


# Motivation
[motivation]: #motivation

* Initially filed as SEP as suggested by Ch3LL in [#55886](https://github.com/saltstack/salt/issues/55886)
* Reduce the impact of future vulnerabilities by having salt master components run as unprivileged users
* Remove requirement for users of salt to have root privileges to run salt cli tools
* Salt master services should be configured to use minimum privilege by default and not require additional configuration by users to do so
* Salt cli tools should be usable by non root users, with good default groups and ACLs

# Design
[design]: #detailed-design

## Salt master services

* `salt-master` and `salt-api`
* run as dedicated user, eg `salt-master`
* run `salt-api` as seperate dedicated user if possible
* dedicated users are members of `salt` group
* read only access to configs in `/etc/salt` through membership of `salt` group
  * `/etc/salt/master`
  * `/etc/salt/master.d/`
  * `/etc/pki/master/`
* read/write access to specific directories under `/etc/salt` through `salt-master` user:
  * `/etc/salt/master.d/_generated.d`
  * `/etc/salt/pki/master/*/`
* read/write access via `salt` group to:
  * `/var/cache/salt/master`
  * `/var/run/salt/master`
  * `/var/log/salt`


## Salt cli tools

* `salt`, `salt-ssh`, `salt-cp`, `salt-key`, `salt-run` and `salt-ssh`
* Users requiring use of cli tools can be added to `salt` group and additional publisher_acl group
* read only access to configs in `/etc/salt` through membership of `salt` group
  * `/etc/salt/master`
  * `/etc/salt/master.d/`
  * `/etc/pki/master/`
* read/write access via `salt` group to:
  * `/var/cache/salt/master`
  * `/var/run/salt/master`
  * `/var/log/salt`
* Additional groups defined `salt-users` and `salt-admins` with suitable publisher_acl configs (eg `salt-admins` allowed to run any module/function, `salt-users` just functions on minions)


## OS package changes

* add system user/group `salt`
* add `salt-master` user
* add `salt-master` (or `salt-api` user if using) to `shadow` group
* change init-scripts to invoke services as `salt-master`
* create `/etc/salt` with ownership `root:salt` and permissions `0750`
* create `/etc/salt/master` with ownership `root:salt` and permissions `0640`
* create `/etc/salt/master.d/` with ownership `root:salt` and permissions `0750`
* create `/etc/salt/pki/master/` with ownership `root:salt` and permissions `0750`
* create `/etc/salt/pki/master/*` with ownership `salt-master:salt` and permissions `0750`
* create `/etc/salt/master.d/_generated.d/` with ownership `salt-master:salt` and permissions `0750`
* create `/var/cache/salt/master/` with ownership `root:salt` and permissions `0770`
* create `/var/run/salt/master/` with ownership `root:salt` and permissions `0770`
* create `/var/log/salt/` with ownership `root:salt` and permissions `0770`
* add `salt-users` and `salt-admins` groups
* create initial master config with publisher_acl defined


## PAM

[The documentation](https://docs.saltproject.io/en/master/topics/eauth/index.html) says that "eAuth using the PAM external auth system requires salt-master to be run as root as this system needs root access to check authenticationn and this is also documented in [#7762](https://github.com/saltstack/salt/issues/7762). Many Linux distributions have `/etc/shadow` and `/etc/gshadow` readable by the `shadow` group. By adding the `salt-master` user to this group we should be able to work round this, but this requires some testing.


## Migration paths

* On upgrade, if existing config doesn't define non root user/group in config, install as previously
* Add helper scripts to update config and directories of existing installs
* Add warnings when salt cli tools run as root on systems configured with non-root user/group

# Unresolved questions
[unresolved]: #unresolved-questions

* Ensuring files/directories written by `salt-master` user or other users maintain correct group ownership and permissions. May be solvable with setgid directories or changes to salt cli tools.
* Minimum set of files/directories that need to be readable or writable by `salt-master` user and users in `salt` group.
* Everything for `root` goes also for Windows' `SYSTEM`?
* grains; if salt-master is configured to load these, how many of those need `root` and how graceful do they fail
* Does publish_acl support `@wheel` and `@runner` syntax
* Which supported distributions support the `shadow` group

# Drawbacks
[drawbacks]: #drawbacks

* Large change in initial behaviour of installed salt master will require updates to documentation and education of users
* Migration for existing setups may be complicated


# Alternatives

* Lock down `salt-master` and `salt-api` processes using AppArmour, SELinux or similar tools
* PAM as non-root - socket auth

# References

* [Running the Salt Master/Minion as an Unprivileged User](https://docs.saltstack.com/en/latest/ref/configuration/nonroot.html)
* [Running salt as normal user tutorial](https://docs.saltstack.com/en/latest/topics/tutorials/rooted.html)
* [django-pam](https://pypi.org/project/django-pam/)
* [pamela](https://pypi.org/project/pamela/)
* [radicale-auth-PAM](https://pypi.org/project/radicale-auth-PAM/)
