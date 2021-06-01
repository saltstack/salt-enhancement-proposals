- Feature Name: Master Key Rotation for disconnected Minions
- Start Date: 2021-05-27
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

There are several reasons to rotate keys:
- Exposure or a compromised key
- Security departments require key rotation on a periodic basis
- It is common praxis to expire or revoke keys or certificates (e.g. [eset protect 1](https://help.eset.com/protect_admin/80/en-US/certificate_replacement.html), [eset protect 2](https://support.eset.com/en/kb6824-peer-certificate-is-going-to-expire-error-in-eset-security-management-center-7x))


# Motivation
[motivation]: #motivation

If a minion is offline when the master changes its key, there is no way to get it after the fact. The minion will not be able to reconnect to the master after the master has changed its key. This is a problem e.g. with end user desktops/laptops that aren't ALWAYS connected to the master.

The goal is a key management system that allows for:
- Expiring and future (post-expire) keys
- Key rotation
- Distribution of the post-expire key before the current key expires

Related issues:
- https://github.com/saltstack/salt/issues/59149
- https://github.com/saltstack/salt/issues/59342


# Design
[design]: #detailed-design

Definition of any new terminology:
- Expiring key: a key that is only valid until its expire time
- Expire time: the time at which a key will expire
- Post-expire key: a key which is unused until the expire time and which replaces the expiring key at expire time. The post-expire key is (initially) not expiring, but can be turned into an expiring key by getting an expire-time
- Rotation: replacing the current, expiring key with a post-expiring key

Examples of how the feature is used:
- Choose expire time t1
- Create non-expiring key k2
- Distribute t1 and k2 to all minions:
  - t1 becomes the expire time of the current key k1
  - k2 remains unused as long k1 has not expired
- Starting at t1 time, the master and all minions rotate keys from k1 to k2

Corner-cases
- Unknown

A basic code example for new API
- `salt * key_util.expire expire_time=2022-02-22 post_expire_key=AABBCC`
  - You generate the post_expire_key from the public key file by:
    1) Remove the first and the last line (`-----BEGIN PUBLIC KEY-----` and `-----END PUBLIC KEY-----`)
    2) Remove line breaks
  - The return informs if a potential current expire-time or post-expire key changed

Outline of a test plan for this feature:
- Tests must include all use cases

How do you plan to test it?
- Disconnect minion before key1 will expire and reconnect before key1 will expire
- Disconnect minion before key1 will expire and reconnect after key1 expired
- Keep minion connected while key1 expires
- Disconnect minion after key1 expired and reconnect

Can it be automated?
 - Yes, this is a highly automatable feature


## Alternatives
[alternatives]: #alternatives

What other designs have been considered?
- The expiring time could be omitted/overridden and the master could change its key to k2 at any time/earlier. A minon that fails to authenticate the master rotates from k1 to k2

What is the impact of not doing this?
- Key rotation remains impossible for disconnected minions
- Key rotation API remains missing


## Unresolved questions
[unresolved]: #unresolved-questions

What parts of the design are still TBD?
- Unknown

# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this? Please consider:

Implementation cost, both in term of code size and complexity
- Each minion must additionally store the post-expire key and the expire time, which is below 1 KB
- The key rotation action is essentially moving a file
- The key rotation decision must be taken before re-authentication and is essentially comparing present time with expire-time
Integration of this feature with other existing and planned features
- The advise [ROTATE A MASTER KEY](https://docs.saltproject.io/en/latest/topics/hardening.html#rotate-a-master-key) contains a security hole because the minion must blindly trust the new key
- There is no API to expire or rotate keys, only a [rekey script](https://github.com/dwoz/salt-rekey/) that requires all minions to be connected
Cost of migrating existing Salt setups (is it a breaking change?)
- No migration costs, not a breaking change
Documentation (would Salt documentation need to be re-organized or altered?)
- Documentation should be added


There are tradeoffs to choosing any path
- No, they can be combined
