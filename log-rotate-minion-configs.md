- Feature Name: Cross Platform Log Rotate in Minion Config
- Start Date: 2021-08-12
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

Purpose is to make current salt-minion log configs (for performing log rotation) available on all operating systems. Currently, Windows has two minion configs that set the max size of the log and set the max number of log files. However, these configs do not apply to other platforms. 
The work would be to streamline the log rotation by using the most _salty_ features (i.e. minion-configs), instead of relying on platform based solutions which may, or may not, be controlled by a different execution/state module, which may, or may not, be updated regularly.

# Design
[design]: #detailed-design

Salt-minion for all platforms to use these two minion-configs settings:
- log_rotate_max_bytes:
- log_rotate_backup_count:

## Alternatives
[alternatives]: #alternatives

An alternative would need to free up the minion log (file acls/permissions) file so that the log can be manipulated by external processes *while* salt-minion svc is running. 

Note: I do not like, nor recommend this alternative, as it seems to contraindicate current practices of cross-platform solution with minion configs.

## Unresolved questions
[unresolved]: #unresolved-questions

Unknown how to set *nix and MacOS platforms to perform log rotation as a function of minion configs

# Drawbacks
[drawbacks]: #drawbacks

- Time to dev
- Maybe no one else really cares
