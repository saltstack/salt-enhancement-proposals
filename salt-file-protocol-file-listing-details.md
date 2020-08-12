- Feature Name: salt-file-protocol-file-listing-details
- Start Date: 20200723
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

cp.list_master does not provide enough information to determine if files on the
minion are up to date, and if they should still exist. This proposal is expand
the information available about files and directories on the master. This would
allow for performance improvements in ``cp.get_dir``.

# Motivation
[motivation]: #motivation

Fetching information from the file client (``salt://``) does not support syncing
files between the master and minion in an efficient manner. For example,
deleting the files in the cache and re-fetching the files is a method often used
to ensure that what is on the minion matches what is on the master.

Providing more detailed information about files and folders available on the
master should make it possible to reduce the time it takes to sync files between
the master and the minion. Agreeing on a data structure to return is just the
first step. This SEP is to investigate what data is required.

# Detailed design
[design]: #detailed-design

The following is the proposal. The following information should be provided for
each item on the file roots on the master.
```
<file or dir name>:
    Time: Unix 64bit epoch time
    Etag: <Maybe a checksum of all the filenames and modification str times>
    Size: <count of items in the dir> or None
    Type: <f(ile) or d(irectory)
```
A call to list the contents of the file roots might look like this:
```
# maybe expand cp.list_master or introduce a new function
cp.list_master detail=True directories_only=False path=/
'.':
    Time: 7483245778
    ETag: 85ac93ccbd530c96ffad3f3d0f9d7694
    Size: 4
    Type: d
'file1':
    Time: 7483258675
    ETag: 27d4a4651d7d1c296d397bb4500d8835
    Size: 8382
    Type: f
'file2':
    Time: 7483278389
    ETag: 5eed1b18cc3c545615b5126910480e31
    Size: 7184
    Type: f
'dir':
    Time: 74832788382
    ETag: c468a272e13982beb2c6ead49836c86d
    Size: 1
    Type: d
'dir1\file3':
    Time: 7367554432
    ETag: 6fce548132f22c12da8cab8b3bdc0fe0
    Size: 9373
    Type: f
```
The above should allow a minion to determine if the files in a directory have
changed, and which files have changed.

eTag is a string which must be unique for the file/directory, when a minion
fetches the file it should cache the eTag value. The minion does not need to
calculate the eTag. If the reported eTag changes, the minion knows the file has
changed.

An alternative structure would be a nested dictionary instead of a flat
dictionary as shown above.

## Unresolved questions
[unresolved]: #unresolved-questions

- What salt functions need to be updated?
- Will this cause any incompatibility issues?

# Drawbacks
[drawbacks]: #drawbacks

There maybe compatibility issues between different versions of salt when
introduced. As the ``salt://`` protocol is not versioned
