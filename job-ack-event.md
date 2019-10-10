- Feature Name: job-ack-event
- Start Date: 2019-10-24
- SEP Status: Draft
- SEP PR: (leave this empty)
- Salt Issue: (leave this empty)

# Summary
[summary]: #summary

Trigger job acknowledge events from minion, before it starts executing the job.

# Motivation
[motivation]: #motivation

Currently, there seems to be no possibilty to detect if Minion successfully obtained job event. Although there is `saltutil.running`, which could be used to determine it, it creates the same problem as it's another job execution.

The main focus is for integrations to determine whether the job has been started on minion, or if it went stale (e.g. minion is not executing the job although it should), so it can be safely retried.

There are existing ideas how to achieve job retries (such as [RFC 0003 - Job retry](https://github.com/saltstack/salt/blob/develop/rfcs/0003-job-retry.md)), but the approach is different.


# Design
[design]: #detailed-design

Overall idea is to introduce new job event type `salt/job/<JID>/ack/<MID>`. The event would be fired by Salt Minion immediatelly after it receives new job. 
This event could be intercepted by Salt engines / returners, so integrated services would know whether the job was really received and is being executed.

The aim of this SEP is to introduce this new event type, not to implement job retries as such, as the retry scope relies on all minions firing this event.

Because it might be undesirable for minions to fire more events than necessary, minion-sided config option could be introduced to enable/disable the feature.
Due to the design of this SEP, the implementation should be agnostic to any underlying transport layer.


Overall implementation could be as simple as

```diff
class Minion(MinionBase):
    @classmethod
    def _thread_return(cls, minion_instance, opts, data):
        '''
        This method should be used as a threading target, start the actual
        minion side execution.
        '''
        minion_instance.gen_modules()
        fn_ = os.path.join(minion_instance.proc_dir, data['jid'])

        salt.utils.process.appendproctitle('{0}._thread_return {1}'.format(cls.__name__, data['jid']))

        sdata = {'pid': os.getpid()}
        sdata.update(data)
        log.info('Starting a new job %s with PID %s', data['jid'], sdata['pid'])

+       # send ack
+       acktag = tagify([data['jid'], 'ack', opts['id']], 'job')
+       minion_instance._fire_master(tag=acktag)
```

With similar adjustments to metaproxy minion if necessary. The final event could look like this

```
salt/job/20191010142457031142/ack/test-minion.tld	{
    "_stamp": "2019-10-10T14:24:57.414999",
    "cmd": "_minion_event",
    "data": {},
    "id": "test-minion.tld",
    "pretag": null,
    "tag": "salt/job/20191010142457031142/ack/test-minion.tld"
}
```

## Alternatives
[alternatives]: #alternatives

Another option to achieve this is to implement detection into transport protocol itself, relying on whether we can safely determine if the event was received by minion. However we would be only able to tell if event was received, not that job is starting and for various reasons, this might be impossible to implement.

## Unresolved questions
[unresolved]: #unresolved-questions

Should the triggered event be part of `_return` cmd, some new one, or is `_minion_event` enough? 

# Drawbacks
[drawbacks]: #drawbacks

Main drawback might be pushing another event to Salt Master even if it's not necessary for many users. For this reason the firing could be optional with specific config option.
