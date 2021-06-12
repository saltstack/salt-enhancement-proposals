- Feature Name: Master Key Rotation for disconnected Minions
- Start Date: 2021-05-27
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

There are several reasons to rotate keys:
- Exposure or a compromised key.
- Security departments require to limit future exposure. (e. g. [eset protect](https://help.eset.com/protect_admin/80/en-US/certificate_replacement.html)).


# Motivation
[motivation]: #motivation

If a minion is offline when the master changes its key, there is no way to get it after the fact. The minion will not be able to reconnect to the master after the master has changed its key. This is a problem e.g. with end user desktops/laptops that aren't ALWAYS connected to the master.

The goal is a key management that allows for
- Key rotation/replacement.
- Distribution of the replacement key while the current key is valid or at install.
- Keeping the private replacement key offline to safeguard against a compromised master.


Related issues:
- https://github.com/saltstack/salt/issues/59149
- https://github.com/saltstack/salt/issues/59342


# Design
[design]: #detailed-design

- Definition of any new terminology
  - Current key (k1): the key that is currently in use.
  - Replacement key (k2): a key which is unused as long k1 works.
  - Rotation: replacing k1 with k2.
- Examples of how the feature is used
  - Create replacement key k2.
  - Distribute k2 to all minions, which could require to wait for offline minions,
  - After complete distribution of k2, the master can rotate keys from k1 to k2.
  - When a minion reconnects with k1 and fails, he reconnects with k2.
- Additionial command
  - `salt * key_util.set_replacement_key public_key=/full/path/to/file`
    - The return of `key_util.set_replacement_key` should inform if a former replacement key was replaced and if they differed.
- How do you plan to test it?
  - Distribute k2 to minions m1 and m2.
  - Disconnect m2.
  - Rotate key on master.
  - Verify m1 rotates key.
  - Reconnect m2.
  - Verify m2 rotates key.


## Alternatives
[alternatives]: #alternatives

What other designs have been considered?
  - Expiring key: a key that is only valid until its expire time. This seems to add no benefit.

What is the impact of not doing this?
- No key rotation when minions disconnect
- No Key rotation API


# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this? Please consider:

- Implementation cost, both in term of code size and complexity
  - No change on the master required.
  - A minion must additionally store the replacement key.
  - The key rotation action is essentially moving a file.
- Integration of this feature with other existing and planned features
  - The advice [ROTATE A MASTER KEY](https://docs.saltproject.io/en/latest/topics/hardening.html#rotate-a-master-key) contains a security hole because the minion must blindly trust the host that responds as `master`.
  - There is no API to rotate keys, only a [rekey script](https://github.com/dwoz/salt-rekey/) that requires all minions to be connected at the same time.
- Cost of migrating existing Salt setups (is it a breaking change?)
  - No migration costs, not a breaking change.
- Documentation (would Salt documentation need to be re-organized or altered?)
  - Only added.
