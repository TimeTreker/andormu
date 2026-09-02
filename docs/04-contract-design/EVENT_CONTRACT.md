# Event Contract Proposal

External events should use a CloudEvents-compatible envelope and Andormu-specific data payload.

Required conceptual fields:

- event id,
- source,
- type,
- subject,
- time,
- schema version,
- workflow namespace,
- WorkflowRun id,
- optional TaskRun / TaskAttempt id,
- aggregate sequence,
- causation/correlation ids,
- actor if applicable,
- data.

Delivery is at least once; consumers deduplicate by event id.
