- Feature Name: A new organization for modules
- Start Date: 2022-03-26
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)


## Summary
[summary]: #summary

> Disclosure
> This enhancement proposal is being written by someone with little experience in Salt.
> One of the many motivators of writing this is trying to contribute to Salt to improve it’s positioning on the
> configuration management/remove execution ecosystem, by making the software easier to use,
> and going more mainstream outside of the Enterprise world. 

At the moment that are over 530 modules builtin in Salt, over 170k lines of code for modules only. At first sight, this seems positive, since users can find pretty much anything they need in one of the existing modules. However, this poses other challenges, both for maintainers, and for users. This is the proposal do remove the majority of Modules from Salt Core code base, and open a new code-base of Salt Community Modules.
Salt had a `salt-contrib` repo in the past (archived in 2018), which would concentrate module development, but still in a centralized manner. This proposal is for the decentralization of modules themselves, and the centralization of a “space” so to say, where the community can share modules.

## Motivation
[motivation]: #motivation

1. Documentation
When writing states, one of the main sources of information is the documentation of the modules being used. This is frequently being consulted and used throughout all states written.
O good documentation is crucial. Not only Python parameters are needed, but examples, values, descriptions, allowed keys, allowed arguments, default arguments etc.
At the moment, there are a significant percentage of modules that lack specific 

2. Public Perpecption
When having such a large number of volumes, it becomes increasingly difficult to manage them all. Maintenance is expensive and time consuming. It’s only natural that some modules will have bugs, small quirks, and other details that might make it less them optimal to work with.
Apart from the functionality itself, the sometimes lacking documentation may also collaborate to a frustrating experience to the user.
This has been seen in forums, where users demonstra their frustrating with specific modules, modules that could be interpreted as less than essencial.
When having modules with bugs being a part of the core Salt project, this detriments Salt’s image as a robust software, and may not be healthy to Salt’s  as whole.

3. Core developers efficiency (sanity?)
Maintaining, responding to issues, writing documentation, and enforcing style guides for such a high number of modules is expensive and time-consuming. By having only a core pack of modules (~50 maybe?), maintainers can greatly increase efficiency on the core code base (internals).

4. Community Engagement
By having a *specific* space for community driven modules development, the sense of community increases, the learning curve for contributors decreases, and this might have a potential positive effect on community engagement.

## Design
[design]: #detailed-design

If such made is done, it’s not reasonable to implement a high barrier to usage for community modules. One potential implementation would be to install Salt with batteries included `pip3 install salt-master[community]` which would include all community modules (that are managed from a separated repository).

Other potential implementation could include the installation of a different package `pip3 install salt-community` or even installing modules one by one, or in batch.

For a successful implementation, a specific “space” for Salt Community Modules would be beneficial, a so called “App Store”. Ansible Galaxy is a good example of what can be considered a successful implementation of community driven plugin space.

## Documentation
By having fewer modules to be managed by the core developers, it’s fair to say that improvement in documentation would be easier to write and maintain. Although Salt module’s documentation is fairly rich, it’s mostly technical, resembling the reference part of a typical software documentation. By organizing in a user-first approach (less technical and more user friendly) it could also help making the final user learning curve less steep. I will write a separate topic on documentation shortly.

## Future Considerations
Having a separated `community modules` repository would potentially improve future code sustainability, while decreasing code complexity. By not tying modules to the core code base, Salt would become more extensible than already is.

## Alternatives
[drawbacks]: #drawbacks

Having the entire modules ecosystem embedded in Salt (as is) remains a valid alternative. In that scenario, the suggestion would change for strict reinforcement of documentation rules (a separate topic of discussion).

## Unresolved Questions & Potential Drawbacks
[unresolved]: #unresolved-questions

If the majority of modules leave the core codebase, a dynamic module import system would have to be implemented. This could potentially be achieved through Python’s native `__import__` which would help compatibility with Python 3.1 or earlier versions (if I’m not mistaken). Or using `importlib` .

