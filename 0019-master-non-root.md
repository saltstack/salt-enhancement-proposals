- Feature Name: [security] Improve the default behaviour of salt when running under different scenario's than as `root`
- Start Date: 2020-02-28
- SEP Status: Draft
- SEP PR: 25
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

Salt has always assumed running as root for all of it's deployment scenarios. 
As a consequence, it has always required multiple manual configuration and directory configurations to get salt in it's many incarnations to work under non-root privileges.

It has long been a standing best-practice to run serving daemons under regular user privileges, opening up additional capabilities as necessary.
The core salt master has very little need for these additional capabilities, and these requirements can be orchestrated otherwise.

Improving the default behaviour would additionally make salt more convenient to deploy and use in stand-alone/multi-minion scenario's (e.g. salt-ssh from a multi-user bastion) and containers. 


# Motivation
[motivation]: #motivation

* Initially filed as SEP as suggested by Ch3LL in [#55886](https://github.com/saltstack/salt/issues/55886)
* Make salt more flexible/easy to use in non-root scenario's
* Make salt-master safer to deploy by running it under locked down privileges by default
* Make salt align and integrate better with the wider ecosystem

# Design concepts
[design]: #design-concepts

## Salt behaviours

`salt-{master,minion,ssh,cloud}` start conforming to the XDG application directory standards
* run as `root` or `salt`: current syspath defaults
* run as user: `${XDG_{CACHE,CONFIG,DATA}_HOME}/salt/{master,minion,ssh,cloud}` etc.
* all necessary dirs are created if nonexistent
* the salt config `user` and SSH equivalent both default to the running user
* all configs written are written by default to a separate config dir, e.g. `_generated.d`

## OS package changes

* install system user/group `salt` 
* change init-scripts to invoke as `salt` and override XDG default back to `/etc/salt/<daemon>/daemon.conf`
* install all `/etc/salt` dirs owned by `root:salt` `750`
* install `/var/cache/salt` and `/srv/salt` etc. owned by `salt:salt` `770`
* allow writing on `/etc/salt/pki` dirs and `/etc/salt/<daemon>/_generated.d` dirs
* allow `salt` access to `/etc/shadow`

## PAM

[The documentation](https://docs.saltstack.com/en/latest/ref/auth/all/salt.auth.pam.html#module-salt.auth.pam) states it does not support authenticating as root.
Besides, at least one serious issue ([#7762](https://github.com/saltstack/salt/issues/7762)) stated for this to be broken because at any rate the salt running user needs access to `/etc/shadow`.

Because the `salt-master` executing the commands would now be running as `salt`, that can be considered the highest achievable named authority within the Salt master.
The usage of `root` in ACL's could therefore be deprecated and replaced by `salt`.


## Migration paths

* if `user` is set manually, show deprecation/path default change warning
* stop `user` config usage and support eventually, making `user` a runtime variable
* `runas` / elevation scenario's
* all user ACL's to `root` get deprecation warnings to change to `salt`

# Unresolved questions
[unresolved]: #unresolved-questions

* `salt` as top-ACL user is still a hard-code
* Everything for `root` goes also for Windows' `SYSTEM`? 
* The myriad of pillars/engines etc might require additional configuration/access/documentation if they access local files/sockets
* `XDG_` equivalents on Mac, Win, BSD, etc.
* grains; if salt-master is configured to load these, how many of those need `root` and how graceful do they fail

## `syspaths`

various levels of flexibility to the syspaths could be introduced to facilitate this, e.g.
* additional coalescing default items in `syspaths.py` and/or `config`
* generating & loading a `_syspaths.py` to cache

# Drawbacks
[drawbacks]: #drawbacks

* Everyone currently using centralized configs for their non-root invocations of salt cli's will eventually need to put in config in their profile
  * this might be slightly alleviated by altering the coalesce for e.g. salt-master socket locally
  * a salt-minion socket equivalent might be introduced?
* in my experience the vast majority just `sudo` as user bypassing this problem altogether
* still using PAM; a cursory glance of PyPI shows this not to be a popular option at all
* excessive reading of the config causes excessive cascading causes slowdowns
* introduce 'mappings' to ACLs?

# Alternatives

* socket auth
* improve SSO capabilities
* deprecate PAM altogether
* ship daemons built with system paths, ship user cli utils with XDG behaviours 

# References

* [Running the Salt Master/Minion as an Unprivileged User](https://docs.saltstack.com/en/latest/ref/configuration/nonroot.html)
* [Running salt as normal user tutorial](https://docs.saltstack.com/en/latest/topics/tutorials/rooted.html)
* [django-pam](https://pypi.org/project/django-pam/)
* [pamela](https://pypi.org/project/pamela/)
* [radicale-auth-PAM](https://pypi.org/project/radicale-auth-PAM/)
