# Success Criteria

Phase 0 is successful when the design makes these statements unambiguous.

## Semantic correctness

- The state of a WorkflowRun, TaskRun, and TaskAttempt can be determined without backend-specific knowledge.
- Retry does not erase previous attempts.
- A dependency rule produces predictable downstream behavior for success, failure, cancellation, timeout, and skip.
- Finalizers have deterministic scheduling rules.
- Suspend, cancel, terminate, retry, redrive, and restart are distinct concepts.

## Recovery correctness

- An Andormu controller restart does not require a user restart of the workflow.
- A duplicate task-dispatch command for the same attempt does not create a second logical attempt.
- Redrive preserves previously successful nodes unless the approved redrive semantics explicitly invalidate them.
- Redrive never silently switches to a newer workflow revision.

## Boundary correctness

- The core model contains no Kubernetes Pod/Job concepts.
- Physical resource allocation decisions remain outside Andormu.
- Domain platforms can use Andormu without importing Andormu implementation internals.

## Product quality

Before Phase 1 coding, the team must review:

- the end-to-end run lifecycle,
- graph/dependency semantics,
- retry/failure/cancellation semantics,
- dynamic expansion,
- event and idempotency model,
- resource/execution adapter boundary,
- product UX for debugging/recovery,
- security/audit model,
- versioning strategy.
