# Task and Attempt Detail

## TaskRun panel

- logical task id,
- state + reason,
- dependency predicate and upstream status,
- resolved TaskDefinition revision,
- retry policy,
- total timeout,
- attempts list,
- outputs/artifact references.

## TaskAttempt panel

- attempt id and number,
- state and transition timestamps,
- execution adapter,
- pinned ExecutionBinding revision, provider/completion model, and logical target,
- opaque backend execution handle,
- failure descriptor,
- resource intent/allocation reference,
- log/metric links,
- cancellation requests,
- dispatch idempotency key.

This distinction is important for postmortems: “Task failed three times” must be visible as three immutable attempts.
