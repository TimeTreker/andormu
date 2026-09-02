# Execution Model

## Submit

1. Validate WorkflowSpec.
2. Resolve referenced revisions.
3. Create immutable ExecutionSnapshot and digest.
4. Create WorkflowRun and TaskRuns.
5. Persist initial state/events.
6. Activate reconciliation.

## Reconcile

For each active WorkflowRun:

1. Load pinned snapshot and current durable state.
2. Apply completed external events/observations.
3. Evaluate dependency rules.
4. Expire timers/timeouts.
5. Determine newly runnable nodes.
6. Create TaskAttempts for eligible TaskRuns.
7. Persist dispatch intent before relying on dispatch.
8. Deliver idempotent requests through adapters.
9. Persist callbacks/observed changes.
10. Recompute workflow/finalizer completion.

## No in-memory-only progress

If the process crashes between two steps, a new process must be able to infer the correct next action from durable records. This is the central correctness requirement.

## Reconciliation vs replay

Andormu should not rerun arbitrary user orchestration code to recover state. It re-evaluates a declarative snapshot and persisted run state. This provides Temporal-like durability while avoiding deterministic-workflow-code constraints.
