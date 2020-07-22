- Feature Name: salt-file-protocol-file-listing-details
- Start Date: 20200723
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary
cp.list_master does not provide enough information to determine if files on the minion are up to date, and if they should still exits. This proposal is to look into expanding the information available on files and directories. This would lead to cp.get_dir performance improvements.

# Motivation
[motivation]: #motivation

salt:// file fetching does not support syncing files between the master and minion in an efficient  manner. For example deleting the files in the cache and re-fetching the files is often used to ensure what is on the minion matches what is on the master.

By providing more detail information about files and folders it should be possible to reduce the time it takes to sync files between the master and the minion. Agree to data structure to return is just the first step. This SEP is about agreeing to the data that is required.

# Design
[design]: #detailed-design
The following is the proposal
```
cp.list_master detail=True directories_only=False path=/ # maybe expand cp.list_master or introduce a new function
'.':
   Time: Unix 64bit epoch time
   ETag: <Maybe a checksum of all the filenames and modification str times>
   Size: <items in directory> or None
   Type: d
'file1':
   Time: Unix 64bit epoch time. Last Modified time.
   ETag: string/checksum
   Size: 64bit int big-endian (as per msgpack)
   Type: f(ile) or d(irectory)
'file2':
   Time: 7483278389
   ETag: jfds8jlfsjd8ereteghyrbvvdffeeejfljdl
   Size: 7184
   Type: f
'dir':
   Time: seconds since 1970
   ETag: <Maybe a checksum of all the filenames and modification str times>
   Size: <items in directory> or None
   Type: d
'dir1\file3':
   Time: 6367554432
   ETag: rfejlwjkldggdgfddgthdvhyhyytrfwe
   Size: 9373
   Type: f
```
The following should allow a minion to determine if the files in a directory have changed, and which files have changed.

eTag is a string which must be unique for the file/directory, when a minion fetches the file it should cache the eTag value. Minion does not need to calculate the eTag. If the reported eTag changes the minion knows the file has changed.

## Unresolved questions
[unresolved]: #unresolved-questions

What salt functions need to be updated?
Will this cause any incompatibility issues?

# Drawbacks
[drawbacks]: #drawbacks
There maybe compatibility issues between different versions of salt when introduced. As the salt:// protocol is not versioned
