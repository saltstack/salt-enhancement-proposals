- Feature Name: Sub State Returns
- Start Date: 2020-08-12
- SEP Status: Draft
- SEP PR: github.com/saltstack/salt/pull/57993
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

Add the ability for states to have sub state returns.
For example, if you are running a state that runs several external states under a different engine,
you can now add those external state runs individually to the "sub_state_run" key of the state return.
They will be parsed and printed alongside the salt state runs.

# Motivation
[motivation]: #motivation

Salt has the ability to run Ansible Playbooks, Idem SLSs, and Chef configurations.
These "external state engines" run multiple states in their own way and salt currently
only reports whether or not external state runs as a whole were run successfully
(I.E Return status of True if an entire Ansible Playbook was executed successfully).

We want to add the ability for salt states to have a "sub_state_run", which gives
these external state engines the ability to report the status of each of the "sub states" they
ran independently, rather than reporting on the status of the entire group of sub states.

The result is that state output will be very clear and verbose, users will be able to see
exactly which idem state failed in their idem SLS or exactly which configurations passed
or failed in Chef, and exactly which ansible plays were successful when statefully
executing a playbook.


# Design
[design]: #detailed-design

Definitions
-----------
- External state engine: A salt module that has the ability to execute a group of configurations (I.E Ansible, Chef, Idem)
- Sub state run: A new key in the return of salt states that allows definitions of external state engine run details


How it is used
--------------
Some states can return multiple state runs from an external engine.
State modules that extend tools like Ansible, Chef, and Idem can run multiple external
"states" and then return their results individually in the "sub_state_run" portion of their return
as long as their individual state runs are formatted like salt states with low and high data.

For example, the idem state module can execute multiple idem states
via it's runtime and report the status of all those runs by attaching them to "sub_state_run" in it's state return.
These sub_state_runs will be formatted and printed alongside other salt states.

Example
-------

```python
state_return = {
    "name": None,    # The parent state name
    "result": None,  # The overall status of the external state engine run
    "comment": None, # Comments on the overall external state engine run
    "changes": {},   # An empty dictionary, each sub state run has it's own changes to report
    "sub_state_run": [
        {
            "changes": {},       # A dictionary describing the changes made in the external state run
            "result": None,      # The external state run name
            "comment": None,     # Comment on the external state run
            "duration": None,    # Optional, the duration in seconds of the external state run
            "start_time": None,  # Optional, the timestamp of the external state run's start time
            "low": {
                "name": None,        # The name of the state from the external state run
                "state": None,       # Name of the external state run
                "__id__": None,      # ID of the external state run
                "fun": None,         # The Function name from the external state run
            },
        }
    ],
}
```

Code Changes
------------

salt/state.py would just need to be modified to look for the `sub_state_run` key in 
a state return and then process that data the same way it would process a regular state
run, assigning a run_num (and incrementing the global counter), and formatting the low data.

```python
for sub_state_data in running[tag].pop("sub_state_run", ()):
    self.__run_num += 1
    sub_tag = _gen_tag(sub_state_data["low"])
    running[sub_tag] = {
        "name": sub_state_data["low"]["name"],
        "changes": sub_state_data["changes"],
        "result": sub_state_data["result"],
        "duration": sub_state_data.get("duration"),
        "start_time": sub_state_data.get("start_time"),
        "comment": sub_state_data.get("comment"),
        "__state_ran__": True,
        "__run_num__": self.__run_num,
        "__sls__": low["__sls__"],
    }
```

State Decorators would need to check for errors in these sub state runs. 
```python
sub_state_run = None
if isinstance(result, dict):
    sub_state_run = result.get("sub_state_run", ())
result = self._run_policies(result)
if sub_state_run:
    result["sub_state_run"] = [
        self._run_policies(sub_state_data)
        for sub_state_data in sub_state_run
    ]
```

The state content checker would need to be modified to verify sub states.
```python
for sub_state in result.get("sub_state_run", ()):
    self.content_check(sub_state)
```

The tests for this feature would be an exact copy of the existing state tests,
only tweaked to verify `sub_state_run` returns rather than traditional state returns.
```python
def test_sub_state_output_check_changes_is_dict(self):
    """
    Test that changes key contains a dictionary.
    """
    data = {"changes": {}, "sub_state_run": [{"changes": []}]}
    out = statedecorators.OutputUnifier("content_check")(lambda: data)()
    assert "'Changes' should be a dictionary" in out["sub_state_run"][0]["comment"]
    assert not out["sub_state_run"][0]["result"]
```

## Alternatives
[alternatives]: #alternatives

The impact of NOT doing this is that when calling Idem SLS, Ansible playbooks, and Chef configurations,
The output of state runs will not be verbose and it will not be clear what individual changes
were made after executing a playbook/sls/configuration.

## Unresolved questions
[unresolved]: #unresolved-questions

None, the proof of concept is functional, we just to double check with everyone before
making a change to something as critical to salt as the state engine.

# Drawbacks
[drawbacks]: #drawbacks

Doing this modifies the state return code, a critical part of salt.
If it is done poorly then it could cause unintended grief down the road.
