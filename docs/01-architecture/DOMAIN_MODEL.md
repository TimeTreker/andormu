# Domain Model

```text
WorkflowDefinition
      │ 1..N
      ▼
WorkflowRevision ───────────────┐
                               │ resolved/pinned
TaskDefinition revisions ──────┤
ExecutionBinding revisions ────┤
AdmissionPolicy revisions ─────┤
Input bindings ────────────────┤
Policy bindings ───────────────┤
                               ▼
                       ExecutionSnapshot
                               │
                               ▼
                         WorkflowRun
                               │
                 ┌─────────────┴─────────────┐
                 ▼                           ▼
             TaskRun                    RunEvent
                 │
                 ▼ 1..N
            TaskAttempt
                 │
         ExecutionHandle
```

## WorkflowDefinition / WorkflowRevision

A definition is a reusable identity. A revision is immutable. A caller may also submit an ad-hoc WorkflowSpec, but Andormu must still assign a content digest and preserve the exact submitted revision.

## ExecutionSnapshot

Created before execution. It resolves mutable references such as `task@latest` into exact revisions and binds the workflow input and policies. Recovery and redrive refer to this snapshot, never to an ambient latest definition.

## TaskDefinition vs TaskSpec

A `TaskDefinition` describes a reusable logical execution contract and requires an exact capability. A `TaskSpec` is one occurrence of that definition inside a workflow, with graph position, inputs, dependency rules, and permitted logical policy overrides. Execution provider, completion model, target, adapter, physical resource intent, and environment admission policy are resolved through a separate `ExecutionBinding`.

## Capability / ExecutionBinding / ExecutionTarget

A `Capability` is the exact versioned semantic match key required by a TaskDefinition. An immutable, environment-scoped `ExecutionBinding` maps that capability to a provider, completion model, adapter, stable logical `ExecutionTarget`, and execution-policy references. The target routes work; the opaque `ExecutionHandle` identifies the backend execution created for one attempt.

## TaskRun vs TaskAttempt

A TaskRun is the logical node execution. It may have multiple attempts due to retries or recovery. Attempt history is immutable.
