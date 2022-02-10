- Feature Name: Pillar Fileserver 
- Start Date: 2022-02-10
- SEP Status: Draft
- SEP PR:
- Salt Issue:

# Summary
[summary]: #summary

A secondary fileserver local to the master for internal pillar handling.

# Motivation
[motivation]: #motivation

This will ease pillar writing as it allows the ability to treat what is 
currently only available in pillar_roots to all ext_pillars, and would allow 
splitting of fetch and render tasks within pillar. Which can be used to speed 
up and help with pillar scaling. 

This would allow things such as pillar.file_exists to work universally. 
Solving [#61508](https://github.com/saltstack/salt/issues/61508)

As well as allowing ext_pillars like pillarstack and file_tree to work with git 
or s3 with minimal changes.

This is not a full replacement of ext_pillars. But a way of giving the file 
based ext_pillars a central place for their files. And setting up fetch based 
pillars universal rendering through a standard top system.

Here is a list of bugs/feature requests this would help remove. 

* [#61508](https://github.com/saltstack/salt/issues/61508)
* [#59443](https://github.com/saltstack/salt/issues/59443) since the render engine is separated from the fetch engine would 
         all true env separation
* 

# Design
[design]: #detailed-design

We can refer to this new system as pfs. For Pillar file server. 

The first thing added would be a creation of a pillar fileclient api that reads
local cache only. This would be used for the fetch functionality. Allowing any 
ext_pillar or pillar_roots to read from this fileserver as well as it be 
maintained locally by the master/minion. 

Internally files within the fileserver should be referenced by `pillar://` and 
from a pillar render perspective
would operate much like `salt://` does for states. The different sources would 
be combined into environments based on pillarenv. This includes in rendering 
engines

All fetch operations would happen on the loop interval. Allowing rendering to 
happen when a minion requests pillar updates. This Separation of fetch and 
render would actually allow current ext_pillars to operate much faster as they 
do not have to do both fetch and render within the same operation. 

modules/runners needed to support this would be a pfs runner/module that 
handles forcing fetch operations. as well as lookup operations such as listing
files, dirs, and envs. an slsutil runner that can be used for rendering 
`pillar://` allowing for debugging. An update the slsutil module that allows 
it to access to `pillar://` in a masterless minions only. 

Once the fileserver api is added the next item would be to separate what is 
currently pillar_roots into two sub modules that are both enabled by default. 
One that handles rendering and he other that handles fetching. the fetch 
section would work much like the `salt://` roots currently works, and would use
current `pillar_root` config options to adjust how things are pulled into the 
pfs. 

The second part of the pillar_roots separation is the render engine. Currently 
I believe this part should be turned into an ext_pillar. Called pillar_tops. 
It would operate much like the state system does for rendering on the minion. 
It would read from `pillar://` and rendering based on a top file located 
within. 

This would need to put the current pillar_root system into deprecation but it 
should be able to co-exist with this split system, without much problem. 
And a minion by default would not know either way. 

later changes. 

* git_pillar moved to fetch only, renamed to git_pfs.
* s3_pillar moved to fetch only, renamed to s3_pfs
* hg_pillar moved to fetch only, renamed to hg_pfs
* svn_pillar moved to fetch only, renamed to svn_pfs
* csv_pillar updated to be able to read from `pillar://` addresses
* salt.pillar.file_tree be updated to be able to read from `pillar://` addresses
* salt.pillar.makostack be able to be able to read from `pillar://` addresses
* salt.pillar.gpg updated to be able to read from `pillar://` addresses
* salt.pillar.pepa updated to be able to read from `pillar://` addresses
* salt.pillar.saltclass updated to be able to read from `pillar://` addresses
* salt.pillar.stack be updated to be able to read from `pillar://` addresses
* salt.pillar.varstack be updated to be able to read from `pillar://` addresses

Testing this should actually be less problematic than current implementations 
of ext_pillars. As the pfs only has to test fileserver integration and meshing. 

Removing the need to test render engines over and over again for each different
fileserver type.

Addition functionality could be added including encryption layers, and cross 
master pillar fileservers. but those are not the focus of this first pass.

Configuration from a users perspective. 

The default configuration should be a standard pillar_roots like system. In 
that it reads form /srv/pillar by default for the base env. 

the standard config would be rather like the state fileserver setup. Where you
declare the fileserver backends in `pillar_backends` or `pfs_backends` with a 
default for `pillar_roots` 
then there are the secondary options.

The following would be a replica of how the current system would look in this 
new system. with an enabled git backend



``` code=yaml
pfs_backends:
  - pillar_roots
  - git_pfs

git_pfs:
  - master https://git-server/pillardata-https.git

ext_pillar:
  - render:
    - backends: [pillar_roots]
    - root: 'pillar://'
  - render:
    - backends: [git_pfs]
    - root: 'pillar://'
```

however if the backends setting is left off all backends are used together all 
merged in pfs_backends order. 

another example with pillarstack and only the above git pillar

```
pfs_backends:
  - git_pfs

git_pfs:
  - master https://git-server/pillardata-https.git

ext_pillar:
  - pillarstack:
    - root: 'pillar://pillarstack/'
```
This would allow the pillar stack to be a sub mount directory of the 
pillar fileserver. 



API end points would need to be created. And they would need to be treated 
securely. Only accessible directly within the master or minion they are 
spawned.

some endpoints i see needed.

* pfs_refresh(backend) used to rebuild a backends local copy. 
* pfs_clear(backend) used to clear the cache of a pfs backend
* pfs_dir_list(backend) used to get a directory listing from a pfs backend
* pfs_file_list(backend) used for file listing from a pfs backend
* pfs_get_file(path, backend) used to fetch a file from a pfs backend. file 
  should be returned as unaltered string
* pfs_file_exists(path, backend) bool returns true of the file requested exists 
  in the pfs backend.

## Alternatives
[alternatives]: #alternatives

I'm open to other solutions to the problem. The problem being pillar fetch and
render causing slowdowns. This would allow fetch and render to be separate
operations that can be handled independently. Allowing more flexibility in how 
to setup and administrate salt. As well as helping with scaling issues from
each minion forcing a refresh of the fetching of files. 


## Unresolved questions
[unresolved]: #unresolved-questions

Still to TBD is if the render system should become an ext_module or it's 
own thing. Being an ext_pillar module means it will be treated the same as 
other ext_pillars. Including merge order. This would also allow it to co-exist
with current pillar_roots implementations until they are decommissioned.

I am sure there are other pieces that need thought out I have not brought up. 

# Drawbacks
[drawbacks]: #drawbacks

* Development time. This is a huge redesign of the current pillar system in 
  whole. And will touch almost every pillar module in some way. Luckily this 
  will not disrupt any existing custom pillar modules. and all current modules
  can be brought over slowly once the main system is brought up to speed.
* A second issue with development time is how this will have minor effects all
  modules that can be used within pillar rendering. 
* Documentation would not only need to be changed by almost rewritten for all 
  pillar based information. 
* So I believe this can be done with decommissioning in mind Without much 
  stress. But once fully decommissioned this will be breaking if moving from a 
  system before changes start to decommissioned. As such i would recommend a 
  longer decommission time than normal. 
