# Event Model

## Internal execution journal

Every meaningful lifecycle transition/operator action should be append-only and queryable.

Candidate event families:

```text
workflow.run.created
workflow.run.started
workflow.run.suspended
workflow.run.resumed
workflow.run.cancelling
workflow.run.succeeded
workflow.run.failed
workflow.run.cancelled
workflow.run.redriven

task.run.ready
task.run.retry_scheduled
task.run.succeeded
task.run.failed
task.run.skipped

task.attempt.created
task.attempt.dispatched
task.attempt.running
task.attempt.failed
task.attempt.lost
task.attempt.cancelled

graph.expansion.recorded
operator.action.recorded
```

## Event envelope

External events should be compatible with CloudEvents concepts:

- unique `id`,
- producer/source,
- event `type`,
- subject (workflow/task/attempt),
- timestamp,
- schema version,
- data payload.

Additional Andormu fields should include:

- namespace/workspace,
- aggregate sequence number,
- correlation id,
- causation id,
- actor/principal when applicable,
- ExecutionSnapshot digest.

## Ordering

Do not promise global total ordering. Require monotonic sequence numbers within the relevant aggregate (for example one WorkflowRun) and stable causal links.

## Publication

The preferred implementation pattern is a transactional state/event write plus outbox publication, avoiding a database/event-bus dual-write gap. This is a design direction; implementation choice remains gated.

## OpenLineage

OpenLineage can receive derived run/job lineage events, but its run-state taxonomy is not rich enough to replace Andormu's internal orchestration state/event model.
