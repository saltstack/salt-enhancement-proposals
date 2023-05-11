- Feature Name: salt-ssh-configurable-compression
- Start Date: 2021-08-30
- SEP Status: Draft
- SEP PR:
- Salt Issue: [#60479](https://github.com/saltstack/salt/issues/60479)

# Summary
[summary]: #summary

we should be able to configure the type and strength of compression used on the
salt thin tarball for distribution via salt-ssh.

# Motivation
[motivation]: #motivation

thin tarball distribution can be multiple times the actual runtime of salt when
target hosts are network-distant from where salt-ssh is run. in poor network
situations, distribution can fail, and require being re-run. this would
help prevent those problems by allowing the use of more aggressive compression.

in my tests, a simple switch from `gzip` to `zstd` resulted in the tarball
being reduced from ~11MB to ~6MB, which is a massive improvement in my environment.

# Design
[design]: #detailed-design

Ideally, salt would provide builtin options for any common form of compression
supported by the [python tarfile library](https://docs.python.org/3/library/tarfile.html),
since that's what's doing the tarball generation already. right now, this
includes `gzip`, `bzip2`, and `xz`. There should also be an option to specify
the compression level used, although this may have to be specific to each
codec.

As an added enhancement, it would be very helpful to be able to provide an arbitrary
command for tarball compression / decompression, so that any arbitrary codec
(such as `zstd`) can be supported without much trouble. The commands could be
required to take data on stdin and return data on stdout, so that they can
be used inline by salt to/from the tar operations.

The options should be settable from the cli when running `salt-ssh`, from the
usual config files, and per-host in the roster. The per-host options is necessary
for targets that don't support the full suite of codecs for whatever reason.
Naming should be consistent with other thin tarball related options, so something
like the following:

```
thin_compression_codec: [gzip/bzip2/xz]
thin_compression_level: [defined per compressor, usually a value from 0-9]
thin_compression_command: zstdmt -19 --stdout -
thin_decompression_command: zstdcat
```

The [code location](https://github.com/saltstack/salt/blob/5e6f2d69600b3ae20493f4dd28c940f1e733d087/salt/utils/thin.py#L676)
already switches on compression type, so it should be fairly easy to add more options
based on user configuration.

## Alternatives
[alternatives]: #alternatives

* if the thin client could be provided to the hosts in question via https,
  they could have a caching web proxy configured, and this could provide even
  greater speed boosts. however this would require starting a web server as part
  of the salt-ssh run.
* if the thin client tarball could be provided via ipfs, that would also auto-scale
  the distribution. however this would require ensuring the target host has an
  ipfs client properly configured, which is certainly not a guarantee, or
  require using public ipfs gateways.

both of these approaches require significant additional setup and outside resources, and are overkill for something that can
be solved simply by using a different compression algorithm.

## Unresolved questions
[unresolved]: #unresolved-questions

* @waynew had concerns around native minions on switch hardware and the like.
  Would `salt-ssh` ever interact with native minions? and is per-host compression
  exceptions in the roster sufficient to address this?

# Drawbacks
[drawbacks]: #drawbacks
