- Feature Name: Use Tiamat built packages for distribution of salt
- Start Date: 2020-06-12
- SEP Status: Accepted
- SEP PR: https://github.com/saltstack/salt-enhancement-proposals/pull/34
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

Instead of packaging salt with dependencies on the system’s python and python library versions, we use
[tiamat](https://gitlab.com/saltstack/pop/tiamat) to build the packages.


# Motivation
[motivation]: #motivation

Tiamat bundles all libraries with the package.  This allows us to package for a much larger number of distros, switch
vendors, oses, etc. without worrying about the large differences in library versions between them.

It also means that testing is much easier as there are fewer combinations of libraries, oses, and distros that need to
be tested.  There is more of a guarantee that we test the same thing that gets deployed to end users. Essentially,
rather than testing a combination of installed libraries and their particular versions on an OS, you just have to test
on the OS.

Another benefit is that because we don’t have to worry about building the dependencies for all of the oses, it makes it
easier to build the packages in a pipeline.  The way package building is currently done is hard to automate because
every time a new dependency is added or adjusted, we have to then build that dependency if the os doesn’t have it
already.  Because there are no dependencies on the system’s python libraries, it would now be more feasible to have
salt packages be built on every PR.

It also means that for salt-ssh, we will no longer rely on the end system having python installed as we can include
python, and all relevant libraries with the tarball that salt-ssh deploys.

It also means that we can have a single binary as an option (similar to a go binary) that can be used for use cases
that it would be helpful in.


# Design
[design]: #detailed-design

[Tiamat](https://gitlab.com/saltstack/pop/tiamat) is a wrapper around [pyinstaller](https://www.pyinstaller.org/) which
allows building a python environment that includes python, all needed python libraries, and all c libraries.  This
means that we can control all dependencies without interfering with any other software on the system.

We would not use Tiamat for Windows or Mac since they already have a python environment bundled with their installers.
It doesn't make sense to disrupt people already familiar with how those packages work when they are already fast to
build, and we have the same control over the python environment that we would with Tiamat packages.

We would build packages for each OS in the same packaging method we always have (exe, msi, rpm, deb, pkg).  For the
Linux distros we support, we would build them on containers matching the os version we intend to support. See
https://gitlab.com/saltstack/open/cicd/containers for those.  This solves problems with glibc compatibility.

Because we have no dependencies, we can build whenever we want without requiring manual effort to add packages for
dependencies.  The effort required for maintenance of the packaging would be reduced tremendously.  Because of this, we
would be able to do nightly builds via ci.  See https://gitlab.com/saltstack/open/salt-pkg for more on that.  This also
means that could build on every PR and have an artifact of the pipeline be packages that could be installed.  We don’t
intend on signing them on PR.

The resulting packages from nightly builds would go on artifactory in the
[open-debian-staging](https://artifactory.saltstack.net/ui/repos/tree/General/open-debian-staging) and
[open-rpm-staging](https://artifactory.saltstack.net/ui/repos/tree/General/open-rpm-staging) repos.  This would allow
anyone to install from the nightly builds in order to test actual packages during the release cycle.

The combination of building on every PR and nightly builds would dramatically increase the ability of the community to
test salt on their own machines during the release cycle.

Tiamat would be tested that it can build at a minimum, salt, before we release new versions of Tiamat.  We would
ideally test that other applications can be built as well before release.

## Alternatives
[alternatives]: #alternatives

- We keep packaging the way we are which means we have added slowness in our release cycle and have a harder time
  testing the right versions of libraries.

## Unresolved questions
[unresolved]: #unresolved-questions

- Will we package arm64 versions now that packages are architecture specific?


# Drawbacks
[drawbacks]: #drawbacks

- It would be a learning curve for people to understand how to add python libraries to the new environment.
- Adding python libraries to the new environment might be harder in an air gapped network.
- People would have to re-install their python packages potentially if we updated the python in the tiamat environment
  on upgrades.
