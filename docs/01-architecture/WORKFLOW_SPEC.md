# WorkflowSpec Design

This document is **conceptual, not a finalized schema**.

A WorkflowSpec should contain:

- identity and schema version,
- declared workflow inputs/outputs,
- nodes/tasks,
- dependency relationships,
- workflow failure mode,
- workflow timeout,
- logical max parallelism (optional),
- finalizers,
- referenced task/workflow definitions,
- annotations/tags.

## Proposed canonical shape

```text
WorkflowSpec
  metadata
  interface
    inputs
    outputs
  nodes[]
    id
    kind
    depends_on[]
    trigger_rule
    ... kind-specific body
  finalizers[]
  policies
    timeout
    failure_mode
    max_parallelism
```

## Why graph-oriented rather than arbitrary workflow code

A graph-oriented contract:

- can be validated before execution,
- can be visualized consistently,
- is easier to version and diff,
- does not require Temporal-style deterministic replay of user orchestration code,
- lets domain platforms use any implementation language to generate the same wire format.

## Versioning invariant

No running WorkflowRun follows a mutable alias. All aliases are resolved into the ExecutionSnapshot before the first task is dispatched.
