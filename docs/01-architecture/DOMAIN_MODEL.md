# Domain Model

```text
WorkflowDefinition
      │ 1..N
      ▼
WorkflowRevision ───────────────┐
                               │ resolved/pinned
TaskDefinition revisions ──────┤
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

A `TaskDefinition` describes a reusable executable contract. A `TaskSpec` is one occurrence of that task inside a workflow, with graph position, inputs, dependency rules, overrides, and resource intent.

## TaskRun vs TaskAttempt

A TaskRun is the logical node execution. It may have multiple attempts due to retries or recovery. Attempt history is immutable.
