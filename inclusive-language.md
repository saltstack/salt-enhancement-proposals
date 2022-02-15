- Feature Name: SEP34: Inclusive Language
- Start Date: 2021-07-06
- SEP Status: Draft
- SEP PR: https://github.com/saltstack/salt-enhancement-proposals/pull/54
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

The Salt Project has always sought to be inclusive and not offensive. Original designs and names used were done so
with the intent of being inclusive. Over the last decade this landscape has evolved and certain changes need to be made.

This SEP proposes how we can make a solid design that can be sustained as this landscape continues to evolve.

This SEP is intended to supersede SEP #28

# Motivation
[motivation]: #motivation

The use of inclusive language needs to be a continual discussion, but the project also needs to document
why certain choices were made and the liabilities that come from specific changes to language.

Understanding the intent derived from original language choices, as well as formalizing what have previously
been informal policies will allow us to maintain a clean situation moving forward.

# Design
[design]: #detailed-design

In 2020 a very valid surge of concern around inclusive language surfaced and has touched many aspects
of society. It has also highlighted that our collective understating of inclusive language is
evolving. The Salt Project has no intention of perpetuating language, practices, culture, or code
that is offensive or non-inclusive.

The evolving nature of the problem issues a concern on how to best manage this situation in the
long term. The Salt Project is not staffed internally to adequately address these concerns. We
also feel that the creation of a single static document to track this effort is not adequate.

Fortunately, the primary sponsor of The Salt Project - VMware - maintains a significant effort to
maintain inclusion and diversity. This SEP proposes that we strive to adhere to the guidelines
presented by VMware for inclusive language.

Details on the Diversity, Equity, and Inclusion efforts of VMware can be found here:

https://www.vmware.com/company/diversity.html

Fortunately, the Salt Project already adheres to the vast majority of rules set forth by
the VMware Diversity, Equity, and Inclusion policies. Only a few things need to be addressed.

Please note that these topics have been discussed at length and are actively being addressed.

While we share a strong desire to adhere to inclusive language, we should not break user
installs. We feel that if this initiative were to create added and onerous work, confusion, and/or
frustration for our users then it could backfire in the overall intent to create a more understanding
and peaceful world. This means that standard deprecation paths will be adhered to for changes and
the impact of a change will need to be carefully considered.

## Whitelist/Blacklist

The primary change that needs to be made, and has already started to be executed, is to
remove references to Whitelist and Blacklist.

The usage of whitelist will be replaced with allowlist
The usage of blacklist will be replaced with blocklist

## Usage of the term "Master"
Salt uses the term "Master" extensively to refer to the system which issues commands out to Salt Minions.
I (Thomas Hatch) chose the term Master/Minion to refer to this process with the original intent of making
the choice inclusive. The Salt Project team has extensively evaluated the usage of the term "Master" and has
concluded that any attempt to change the term in its entirety would be deeply detrimental to our users.
Due to the deep and extensive usage of the term "Master" we feel that an outright change would not be possible.
Changing the term "Master" would break nearly every install of Salt worldwide and would therefore be detrimental
to the project and, I feel, to the intent of the movement.

We also feel that the term Master/Minion is non-derogatory due to the fact that the word Master is not paired
with a derogatory term for a subservient entity. This was the original intent for the choice made in our verbiage.

With this said, we feel that forcing the ongoing usage of the term "Master" in presentation
materials will potentially be an issue. Therefore we propose adding a new startup script allowing the Salt Master
to be started up via an alternative name. This alternative name will also be available for use in presentations.

## Master Git Branch

The "master" branch used in git will be changed to "main".

## Interacting With Existing Systems

Many systems currently exist which do not use inclusive language. In instances where the interaction or
management of underlying systems does not use inclusive language, Salt will use the language of the underlying,
managed system.

## Alternatives
[alternatives]: #alternatives

Many alternatives have been discussed previously and have been deemed untenable. These include maintaining
a project level inclusiveness program as well as complete renames of the Salt Master. We feel strongly that
these alternatives cannot be reliably executed given our available resources and goals.

## Unresolved questions
[unresolved]: #unresolved-questions

The main unresolved question of the alternative name that will be allowed for the Salt Master.

# Drawbacks
[drawbacks]: #drawbacks

Drawbacks have been listed in this document for consideration.
