- Feature Name: PR Merge Requirements
- Start Date: 2019-05-21
- SEP Status: Accepted
- SEP PR: https://github.com/saltstack/salt-enhancement-proposals/pull/13
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

Lay out the requirements to merge a PR

# Motivation
[motivation]: #motivation

As we begin our working groups, to make sure that our community is all on the same page, we want to
clearly define our requirements to merge a PR. This should benefit developers, reviewers and those
merging the PRs.

# Design
[design]: #detailed-design

All PR requirements
  - Approval Reviews: 1 approval review from core team member OR
                      1 approval review from captain of working group
  - All tests pass.
  - Cannot merge your own PR until 1 reviewer approves from defined list above that is not the author.

Bug Fix PR requirements:
  - Test Coverage: regression test written to cover bug fix.
  - Point to the issue the PR is resolving. If there is not an issue one will need to be created.

Feature PR requirements:
  - Test Coverage: tests written to cover new feature.
  - Release Notes: Add note in release notes of new feature for relative release.
  - Add `.. versionadded:: <release>` to module's documentation
  
Exceptions:
- Changes that do not require writing tests: documentation, cosmetic changes (ex. log.debug -> log.trace), 
                                             fixing tests, pylint, changes outside of the salt directory.
- If an exception needs to be made around the requirement for tests passing or writing a test alongside
  a bug or feature it requires 3 approvals before it can be merged.
- If a test is not included in a bug fix or feature and the author cannot write a test, keep the PR open
  for 3 months with the “Needs Testcase” label.
  
Clarifications:
  - Contributers only need to write test coverage for their specific changes. 
  
Requirements for Merge Access:
  - Two factor authentication is required for any user with merge access.

## Unresolved questions
[unresolved]: #unresolved-questions

- Will need to document this if approved and merged. Documentation needs to include everything specified in
this SEP and the following details:
    * Document current test infrastructure and how contributors can mirror the test runs to help troubleshoot
      test failures
    * Clarify where the release notes are located and how to determine the version to include in versionadded.
      If it is still not clear the contributor can include TBD and the reviewer would clarify the version.
    * Document when integration tests are more applicable over unit tests.
    * Document what test coverage still needs to be added.

# Drawbacks
[drawbacks]: #drawbacks

Requiring tests and ensuring all tests pass will slow down the PR merge process, but will provide more
stability in salt.
