- Feature Name: copy_python_exe
- Start Date: 2019-06-12
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

For maintenance and bug-reporting, it would be beneficial if the salt client processes in the Windows task manager would be named `salt-minion.exe` instead of `python.exe`

# Motivation
[motivation]: #motivation

For maintenance and bug-reporting.

# Design
[design]: #detailed-design

copy python.exe to salt-minion.exe and reference the latter, e.g in salt-minion.bat

## Alternatives
[alternatives]: #alternatives

Setproctitle [does not and will never work on Windows](https://github.com/dvarrazzo/py-setproctitle/issues/61)

Windows task manager continues to show  `python.exe`

## Unresolved questions
[unresolved]: #unresolved-questions

None

# Drawbacks
[drawbacks]: #drawbacks

None