- Feature Name: Retiring Python 2 Support
- Start Date: 2019-03-25
- SEP Status: Accepted
- SEP PR: https://github.com/saltstack/salt-enhancement-proposals/pull/8
- Salt Issue: (leave this empty)



# Summary
[summary]: #summary

In light of Python 2.7 reaching its end of life (EOL) on Jan 1st 2020, Python 2
will be unsupported by SaltStack no earlier then the Sodium release, that is
either the Sodium release or a later release.


# Motivation
[motivation]: #motivation

Python 2.7 will reach its official end of life (EOL) on [January 1st,
2020](https://devguide.python.org/#branchstatus) and post that, it will no
longer be supported by its maintainers. Consequently, SaltStack team will
retire Python 2.7 support in SaltStack.

The end of Python 2.7 support will enable SaltStack to focus its effort on
better supporting Python 3.4+.

[Python 2.x is legacy, Python 3.x is the present and future of the
language](https://wiki.python.org/moin/Python2orPython3)


# Design
[design]: #detailed-design

SaltStack will drop Python 2 support no earlier than the Sodium release
(tentative release date is Feb 2020), that is either the Sodium release or a
later release. Keeping in mind SaltStack's policy of announcing 'end of
support' at least 2 major releases before the intended changes takes place,
Sodium or a later release has been chosen as a suitable deadline to drop Python
2 support.

**Migration:** After carefully evaluating the list of Python 3 versions
supported by different operating systems and by also considering the EOL for
all these operating systems [1]* (as denoted in below grid) and [EOL of Python
branches](https://devguide.python.org/#branchstatus)[2]*, SaltStack team
proposes to support Python 3.4 from Sodium or a later release. Versions of
SaltStack released before the cutoff release will continue to support python
2.7 until they reach end of life.

[![Grid.png](https://i.postimg.cc/V62Wh9nY/Grid.png)](https://postimg.cc/xJymJz67)

### Note 1

SaltStack currently supports master and minions running on different
versions of Python such as salt master running on Python 2, minion running on
Python 3 and vice versa. This support will not change in this transition as a
change in Python interpreter does not mandate a change in network protocol.

### Note 2

The version of OpenSSL that SaltStack will support will be determined
by the Python version supported by SaltStack. Check the [Python SSL
docs](https://docs.python.org/3/library/ssl.html) for the versions of OpenSSL
supported by each Python version, and check the [pyOpenSSL release
page](https://pypi.org/project/pyOpenSSL/#history) to identify the which
versions of Python are supported by pyOpenSSL.

### Note 3

A future SEP will define a schedule for ongoing depreciation of Python versions
as they also reach their end-of-life.

### Note 4

A future SEP will define the salt-ssh Python deprecation schedule.

## Alternatives
[alternatives]: #alternatives

- Support Python2 forever. Given that many of Salt's dependencies have pledged [to drop Python2 support](https://python3statement.org/) this would mean also supporting and vendoring our dependencies.
- Drop support for Python 2.7 in **Neon** (tentative release date is August 2019).

## Packaging Considerations
[Packaging]: #Packaging-Considerations

Which Python packaging solutions to use - EPEL (Extra Packages for Enterprise
Linux)/SC (Software Collections) repository or monolithic packaging? 

While the exact nature of packages delivered by SaltStack is not being laid out
in this SEP, SaltStack team guarantees that package updates will be made
available for smoother transition.


# Actions
[Actions]: #Actions

SaltStack users will see the Python 2 deprecation implemented in the next few
releases as follows: 

1. Future releases of SaltStack will have a Python 2 deprecation warning.
2. SaltStack will create packages that will allow for a smooth transition for users.
3. The six library will be removed from SaltStack slowly time as well as all
   compat library use.
5. Existing monolithic installers will move solely to Python 3 only for the
   cutoff release, such as the Windows installer.
 

# References
[References]: #References

[1]* Below is the list of all the sources from which EOL for the below
Operating system versions were retrieved 

|Operating Systems                          |Sources                         |
|-------------------------------|-----------------------------|
|RHEL 6              |https://access.redhat.com/support/policy/updates/errata 'End of Maintenance Support 2 (Product retirement)' |
|RHEL 7              | https://access.redhat.com/support/policy/updates/errata 'End of Maintenance Support 2 (Product retirement)' |
|Amazon Linux 2      | https://aws.amazon.com/amazon-linux-2/faqs/ |
|SLES 11             | https://www.suse.com/lifecycle/ |
|SLES 12             | https://www.suse.com/lifecycle/ |
|SLES 15             | https://www.suse.com/lifecycle/ |
|OpenSUSE 15         | https://en.opensuse.org/Lifetime |
|Debian 9 (Stretch)  | https://wiki.debian.org/LTS/ |
|Debian 8 (Jesse)    | https://wiki.debian.org/LTS/ |
|Ubuntu 18.04        | https://wiki.ubuntu.com/Releases |
|Ubuntu 16.04        | https://wiki.ubuntu.com/Releases |
|Ubuntu 14.04        | https://wiki.ubuntu.com/Releases |
|FreeBSD 11          | https://lists.freebsd.org/pipermail/freebsd-announce/2018-September/001842.html |


[2]*  Excerpt from EOL of Python branches 

[![States-of-python-branches.png](https://i.postimg.cc/sXrxxvYV/States-of-python-branches.png)](https://postimg.cc/Mc9qrZ34)
