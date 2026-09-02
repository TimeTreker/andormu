# Design Status

**Phase:** 0 — Goal / Architecture / Product / Contract design

**Coding:** blocked by design gate

## Accepted boundary decisions

- Andormu is an independent shared DAG workflow engine.
- Domain semantics stay in domain platforms.
- Physical compute/resource scheduling stays in Compute Platform.
- Core contracts are backend-neutral, not Kubernetes-native.
- A Task is logical work, not a process; persistent service and asynchronous task execution are first-class.
- Phase 0 is design-only.

## Major proposed decisions still requiring review

- Immutable ExecutionSnapshot.
- TaskRun/TaskAttempt split.
- Reconciliation + durable event journal architecture.
- At-least-once dispatch semantics.
- Redrive behavior.
- Dynamic expansion rules.
- Finalizer rules.
- Detailed state/trigger-rule sets.
- Capability/execution-target resolution model.
- Logical backpressure/admission and fairness scope.
- Async task callback/poll/watch protocol details.

See `review/PHASE0_REVIEW.md`.
