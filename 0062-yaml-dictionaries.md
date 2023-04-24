- Feature Name: yaml-dictionaries
- Start Date: 2022-02-17
- SEP Status: Draft
- SEP PR:
- Salt Issue:

# Summary
[summary]: #summary

Allow use YAML dictionaries instead of arrays in Salt SLS files

# Motivation
[motivation]: #motivation

Many people are embarassed when see that Salt uses arrays to describe key-value structures in SLS files, like this:
```yaml
/etc/foo.conf:
  file.managed:
    - source:
      - 'salt://foo.conf?saltenv=dev'
    - user: foo
    - group: users
    - mode: '0644'
    - attrs: i
```
In this structure we are describing key-value structure of arguments to Salt state module,
where keys must be unique, but for some reason we use YAML array instead of dictionary!

So it will be much more convenient to use dictionaries for those structures, like this:
```yaml
/etc/foo.conf:
  file.managed:
    source:
      - 'salt://foo.conf?saltenv=dev'
    user: foo
    group: users
    mode: '0644'
    attrs: i
```
As I understand, it happined for some historical reasons (can anybody describe why?),
so maybe it's time to fix this mistake?

Most of SaltStack competitors like Ansible Playbooks, Puppet Manifests, Chef Recipes
use in that places only dictonaries! So when they see Salt state files with such strange
usage of arrays, they got confused!

And resolving this discrepancy even should make SaltStack more attractable for new users,
who choosing between competitors.

Also this way should automatically get rid of key duplicates in YAML files at parser stage (to see such errors immediately in YAML editors)! Because now we can got mistakes with duplicates without any errors in YAML format, like this:
```yaml
/etc/foo.conf:
  file.managed:
    - source:
      - 'salt://foo.conf?saltenv=dev'
    - user: foo
    - group: users
    - mode: '0644'
    - attrs: i
    - user: bar
```
this duplicated `user` key is hard to detect, because array allows inserting of even identical strings.

As result, this feature will make SLS files look more structured and less confusing for newbies!

I hope we can implement this feature without breaking old behavior, to not ask to rewrite old state files into new format.

# Design
[design]: #detailed-design

Rework YAML parser module (or deeper modules) to accept dictionaries as salt modules arguments, _together_ with arrays, that are used now. This way should not break old behavior, so users will
not forced to upgrade their old state files to new format.

This can be implemented via looking to key format, and if it is an array - treat using current parser, if dictionary - treat using new parser.

So those mixed code in one file shoud work well:
```yaml
/etc/foo.conf:
  file.managed:
    - source:
      - 'salt://foo.conf?saltenv=dev'
    - user: foo
    - group: users
    - mode: '0644'
    - attrs: i

/etc/bar.conf:
  file.managed:
    source:
    - 'salt://bar.conf?saltenv=dev'
    user: bar
    group: users
    mode: '0644'
    attrs: i
```

Together with reworking, will be good to improve documentation with recommending
using of dictionaries over arrays, and swith to use dictionaries in new examples
and "best practice" articles.


## Future considerations

Additionally to make SLS files more structured, this conversion will afford us to insert several groups of arguments to a single module call, like this:
```yaml
foo_bar_files:
  file.managed:
    - name: /etc/foo.conf
      source:
      - 'salt://foo.conf?saltenv=dev'
      user: foo
      group: users
      mode: '0644'
      attrs: i

    - name: /etc/bar.conf
      source:
      - 'salt://bar.conf?saltenv=dev'
      user: bar
      group: users
      mode: '0644'
      attrs: i
```

instead of forcing users to make separate blocks for each action like this:
```yaml
foo_file:
  file.managed:
    - name: /etc/foo.conf
    - source:
      - 'salt://foo.conf?saltenv=dev'
    - user: foo
    - group: users
    - mode: '0644'
    - attrs: i

bar_file:
  file.managed:
    - name: /etc/bar.conf
    - source:
      - 'salt://bar.conf?saltenv=dev'
    - user: bar
    - group: users
    - mode: '0644'
    - attrs: i
```

But this feature is an idea for next cool step in SaltStack improvement.

## Alternatives
[alternatives]: #alternatives

Alternatively to make support of both formats in a single file, we can consider making
a new separate format of SLS file which uses only dictionaries for module arguments.

But I think this is a bad idea, because support of both formats can be implemented together in single file without interferentions.

## Unresolved questions
[unresolved]: #unresolved-questions

How hard is implementing support for dictionaries without breaking old behavior with arrays?

# Drawbacks
[drawbacks]: #drawbacks

I see only one drawback: having support of two formats at same time will confuse people,
so we must update all documentation with note about this and recommendations to use
new dictionary format over old arrays.
