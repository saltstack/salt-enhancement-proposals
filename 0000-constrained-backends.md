- Feature Name: constrained backends
- Start Date: 2023-06-23
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

Apply constraints to individual backends to allow collaboration with "foreign" backends.

# Motivation
[motivation]: #motivation

Why are we doing this? What use cases does it support? What is the expected outcome?

This will open parts of the configuration for other developers (maybe in another department or even external) to modify, but without giving them root rights, even if the minion runs as root.
Currently the publisher_acl is only applied via the salt api - but if a state enters the salt master via a (git) backend, there is no constraint checking, allowing anyone that can change an backend to basically change almost everything, if not everything, within reach of the minion process.
For example: This hinders to allow "foreign" git repositories to be used as backend without extensive merge approval process. (supply chain attacks)

In the end, an application administrator could contribute states / functions / ... that are specific to a software and enable more complex orchestrations while having the benefits of auto state ordering, without having a merge approver to know everything about every application to spot (unintentional or intentional) malicious changes and an expert keeps being an expert for just one/assigned scope/s, without the need to be trusted with everything.

While scoping backends inside a host are beneficial for security, they also can have the benefit of being able to use state auto ordering and depending upon each other, maybe even calling functions published by other repositories, without the need of defining them themselves.

# Design
[design]: #detailed-design

This is the bulk of the SEP. Explain the design in enough detail for somebody familiar
with the product to understand, and for somebody familiar with the internals to implement. It should include:

- Definition of any new terminology
  - "foreign" backend: a backend that can be modified by other users than the salt master administrator.
- Examples of how the feature is used:
      - Target minions on backends
        Users able to change webserver.git aren't always the same users that should be able to modify backupservers.
        Basically every minion can have a view on an environment based on the targeting. An overlay might not be applied if the conditions aren't met.
      - Apply these states constrained to be applied as user x / only with a certain function set.
        - This and more could be accomplished by forcing a render pipeline, that acts as gateway that wraps the renderers for unprivileged backends with another renderer, that applies/filters parts by dropping privileges. Or simply raise an exception to stop further processing of a configuration state.
      - Allow just a certain list of renderers.
         - Some renderers  (like py) might have more capabilities than others to try to circumvent scopes.


      ```
                gitfs_remotes:
                  - https://foo.com/main.git
                  - https://foo.com/webserver.git:
                    - target: web-group-a\*
                    - renderer-prepend: malware_scan
                    - renderer-append: force_user_x
                    - renderer-allow:
                      -  jinja|yaml
                  - https://foo.com/webserver.git:
                    - target: web-group-b\*
                    - renderer-append: force_user_y
                    - renderer-allow:
                      -  jinja|yaml
                      -  py
                  - https://foo.com/backups.git:
                    - target: backup\*
      ```
  
  The last part, is still unpolished:
      * Render (some) state files in a sandbox.
         * Rendering itself can be harmful on the salt minion user, as the renderer (for example: py) might get access to data or write data as the minion user that is not in scope for the targeted user, like the minion keys. Restricting the information that is available/accessable/modifyable.
         * The minion forks itself (or starts a subminion process) to render the statefile as a user or even in a container/jail/... environment. The parent process acts as proxy/filter to let subminions gather data from pillars/etc and communicate upwards to the master.
  ```
                gitfs_remotes:
                  - https://foo.com/main.git
                  - https://foo.com/webserver.git:
                    - renderer-allow:
                      -  py
                    - renderer-sandbox:
                      - runas: user_x
                      - ## TODO: syntax for passing scoped pillar data ##
  ```

- Outline of a test plan for this feature. How do you plan to test it? Can it be automated?
  - Create multiple repositories and check if only data in a scope is readable.
  - Create a state file for the py renderer and write data as another user. Check if data has been written (it should not).
  - Create a state file for the py renderer and run it in a container/jail and write data. Check if data has been permanently written on the minion host (it should not).
  - Create functions distributed over multiple repositories and use them together in an orchestrate runner.
  - Create states distributed over multiple repositories and use them together with state auto order.
  - ...


## Alternatives
[alternatives]: #alternatives

What other designs have been considered?
* Running multiple minions as non root in their respective contexts.
  * Hinders state auto ordering.
    * Orchestration Runners have to be used, while targeting just one host.
  * May have a bigger cpu/memory footprint.
  * Not every context on a host needs a distinct communication towards the master.

What is the impact of not doing this?
* Collaboration via different backends is not possible if anyone does not fit in the scope of salt master trustees.
  * Not everyone, who should be able to define a state on a host for a unprivileged scope, needs access to every data or function while depending on states that need to be resolved in privileged contexts.

## Unresolved questions
[unresolved]: #unresolved-questions

What parts of the design are still TBD?

Sandboxing of renderers.

# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this? Please consider:

- Implementation cost, both in term of code size and complexity
  - Sandboxing and subscoping (pillar)data/functions might have a higher complexity
- Integration of this feature with other existing and planned features
  - no
- Cost of migrating existing Salt setups (is it a breaking change?)
  - no
- Documentation (would Salt documentation need to be re-organized or altered?)
  - no 


There are tradeoffs to choosing any path. Attempt to identify them here.
