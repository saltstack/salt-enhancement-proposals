- Feature Name: [security] run salt-master non-root by default
- Start Date: 2020-02-28
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

Running any server application, typically accessible via the internet, as root user is a security risk.
Bugs _do_ happen, in any application.   Apps should run with as little rights as possible.

Salt-master by default, runs as root user and salt documents how you can run salt-server as non root.(https://docs.saltstack.com/en/latest/ref/configuration/nonroot.html) 

This is the wrong way around.  It should run as non-root by default. And document when/why/how you can run it as root.

If I read the documentation correctly its PAM external auth that breaks when running as non-root (https://github.com/saltstack/salt/issues/7762)
If that really cannot be fixed in salt, then the Salt PAM external auth documentation should mention how you can change your salt installation to run as root.

Brief explanation of the feature.

# Motivation
[motivation]: #motivation

Filed as SEP as suggested by Ch3LL in https://github.com/saltstack/salt/issues/55886


# Design
[design]: #detailed-design

Salt master should be installed an run unpriviliged.

Installation scripts should install it as the proper user, chmod the proper directories etc.



## Alternatives
[alternatives]: #alternatives

To get a secured by default installation, I don't think there are any alternatives.

## Unresolved questions
[unresolved]: #unresolved-questions

How to handle loading of configuration files has to be addressed.

I see multiple possible sollutions for this.
A) We can start salt-master as an unpriviledged user, and that unpriviledged user has read access to the configuration files via group read permissions.
B) We can start salt-master as root, and let the application fallback to an unpriviledged users after reading the configuration files. (similar to what nginx for example does)



# Drawbacks
[drawbacks]: #drawbacks

The documentation (https://docs.saltstack.com/en/latest/ref/configuration/nonroot.html) mentions PAM external auth as a reason why salt master runs as root.

I'm not 100% familiar with PAM external auth, but I suspect that particular issue can even be fixed.
pam_unix has a setuid helper (unix_chkpwd) that seems exactly intended for this. For applications, without read access to /etc/shadow/ to do authentication.


Even if the PAM external auth issue cannot be fixed, then still we should run salt master, by default, as non root. And document what a uses needs to change to use PAM external auth. (which most probably don't use)
