- Feature Name: move non-supported releases to a separate domain name
- Start Date: 2020-05-20
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary
With the recent CVE events, we have observed and have been alerted to
situations where clients and users were accessing and downloading older
unsupported releases that contain the vulnerability. The ease of access,
coupled with the potential for error that we observed, mean that we need to
make a decision on what to do with older, unsupported releases.

We have three potential paths: release updated packages for older versions,
remove older versions all together, or move older versions to an archive
location that is separated from currently supported versions. From a time and
effort standpoint, the option to update is simply not viable and would set the
project back considerably. The option to completely remove the older packages
would leave many stranded while trying to upgrade to later versions and is
therefore not viable either. This leaves us with the approach of moving the
older, unsupported versions to an archive location. In examining this, we
assessed other open projects and their approach to older, unsupported releases.
We found many following the same proposed approach that we have in this SEP.
Examples include Ubuntu and Debian, which make this distinction easy by using
two separate domain names: old-releases.ubuntu.com and archive.ubuntu.com for
Ubuntu, and archive.debian.org and deb.debian.org for Debian. We should emulate
their behavior by moving our old unsupported releases to a separate URL.

On the same note, unsupported PyPi releases should also be moved to a static
HTML page hosted by SaltStack and taken down from PyPi. If possible, the
release pages on PyPi should be updated to direct users to the new place where
old releases are hosted. If not possible, the next major release of Salt, and
following, should include this information around the top.

# Motivation
[motivation]: #motivation
We are doing this so people have to make a deliberate decision to use an
unsupported, and possibly insecure, release. It will become obvious when a
release is no longer supported. It also means people syncing our repo won’t
have to sync as much to get all supported releases. This reduces bandwidth
costs for them and SaltStack.

# Design
[design]: #detailed-design
Whenever a major release goes out of CVE support, we will move the contents to
https://archive.repo.saltstack.com, for example:

```
https://repo.saltstack.com/yum/redhat/7/x86_64/2018.3/
```
would move to

```
https://archive.repo.saltstack.com/yum/redhat/7/x86_64/2018.3/
```
and all of the point releases under

```
https://repo.saltstack.com/yum/redhat/7/x86_64/archive/
```
that are part of that major release would move to

```
https://archive.repo.saltstack.com/yum/redhat/7/x86_64/archive/
```

If a point release is affected by a CVE that is part of a major supported
branch, that point release will also be moved to
https://archive.repo.saltstack.com. For example,

```
https://repo.saltstack.com/yum/redhat/7/x86_64/archive/3000/
```
was affected by the recent CVE so it would be moved to

```
https://archive.repo.saltstack.com/yum/redhat/7/x86_64/archive/3000/
```

This would mean someone using the bootstrap script would have to specify `-R
archive.repo.saltstack.com` if they wanted to install an unsupported/insecure
release.  Other methods of installation would also require updating the URL
with the new domain name.

https://archive.repo.saltstack.com will be S3 syncable just like the main repo.

We will add a notice to the main page of repo.saltstack.com explaining this
behavior.

With regards to the source tarballs hosted on PyPi, we will host them on a web
server and they will be made available on a static HTML page. The reason to do
this instead of hosting our own PyPi server is to make it explicit, and not
easy, to install older versions.

### Communication Plan and Timeline
We are sharing this SEP via Slack, IRC, and the Salt-Users and Salt-Announce
Mailing Lists.

In light of the CVE and potential for exploiting unsupported Salt releases,
this SEP is urgent and running on an expedited timeline. The SEP will be open
for comments from May 20-26, 2020. The detailed timeline is below:
- May 20, 2020: SEP Announcement
- May 21, 2020: SEP discussion during the Community Open Hour
- May 26, 2020: SEP closed to comments
- May 27, 2020: SEP closes
- June 1, 2020: Unsupported releases moved to separate URL

## Alternatives
[alternatives]: #alternatives
- We remove unsupported releases from being publicly available at all. However,
  this would force users to upgrade to the latest version.
- We could leave vulnerable versions where they are. This would likely lead to
  inadvertent installation of vulnerable versions, probably compromising those
  servers.

## Unresolved questions
[unresolved]: #unsresolved-questions
- None so far

# Drawbacks
[drawbacks]: #drawbacks
- We will not update rpms that setup the repo, such as
  https://repo.saltstack.com/yum/redhat/salt-repo-2015.5-1.el5.noarch.rpm which
  may inconvenience people who would like to use them.
- We would have to update documentation describing the new behavior.
- It will take effort for someone to move unsupported versions to the archive
  domain name.
