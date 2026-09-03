# Service and Deferred Tasks — Product Experience

Operators should be able to distinguish the logical Andormu TaskAttempt from the external service/backend that performs it.

A Task Attempt detail view should eventually show:

- logical task / attempt id,
- exact TaskDefinition/capability,
- pinned ExecutionBinding revision, provider, completion model, and ExecutionTarget,
- dispatch/admission timestamps,
- backend execution handle,
- service/worker/runtime reference,
- callback/poll/watch history,
- timeout/retry policy,
- cancellation state,
- log/trace links,
- structured failure reason.

For high-volume workloads the UI should expose why READY work is not yet dispatched: concurrency limit, rate limit, adapter capacity, downstream backpressure, or external compute admission.
