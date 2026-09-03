# TaskDefinition and TaskSpec Design

This is a design proposal, not a normative schema.

**Core rule:** a task is a logical unit of work, not a process, service, container, Pod, queue message, or compute job.

## TaskDefinition: reusable logical contract

A `TaskDefinition` answers: **what work is this, and what must a conforming implementation guarantee?**

Conceptual properties are:

- stable identity and immutable semantic revision;
- required versioned `Capability`;
- input and output contracts;
- default logical timeout semantics;
- retry-safety properties;
- idempotency requirements;
- success/output and failure guarantees;
- bounded descriptive metadata.

For example:

```text
TaskDefinition data.bag.decode revision v3
  requires capability data.bag.decode@v3
  input artifact: ArtifactRef
  output decoded-artifacts: ArtifactRef[]
  declares retry safety and idempotency requirements
```

It must not identify `decoder-service-v12`, an endpoint, a worker pool, a runtime template, a cloud job, or physical resources. Those belong to an `ExecutionBinding`.

## TaskSpec: one occurrence in a WorkflowSpec

A `TaskSpec` answers: **where and how is that logical contract used in this graph?**

It contains or references:

- node id;
- exact TaskDefinition revision;
- input bindings from workflow inputs or upstream outputs;
- output bindings to workflow outputs or downstream inputs;
- dependencies and trigger rule;
- permitted logical retry/timeout overrides;
- task-occurrence idempotency metadata where required;
- finalizer/cleanup classification when applicable;
- labels/annotations.

A TaskSpec does **not** contain an inline execution descriptor, provider kind, completion transport, endpoint, adapter, execution target, physical `ResourceIntent`, or environment capacity limit.

## Capability

A `Capability` is the versioned semantic requirement used to match a TaskDefinition to compatible execution realizations. For this review it is an exact identifier such as:

```text
data.bag.decode@v3
data.embedding.encode@v2
```

The capability revision and TaskDefinition contract must be compatible. A binding must not substitute a different semantic capability for an already-pinned TaskDefinition.

## Execution separation

```text
TaskSpec
   │ exact logical contract
   ▼
TaskDefinition
   │ capability requirement
   ▼
ExecutionBinding
   │ provider + completion + target + adapter
   ▼
TaskAttempt execution
```

Logical retry/timeout semantics can be declared by the TaskDefinition and constrained by the WorkflowSpec. Backend mechanics for enforcing or observing them belong to the binding and adapter contract.

## Resource and admission separation

Neither belongs to the TaskDefinition's business meaning:

- `AdmissionPolicy` controls whether Andormu may dispatch logically ready work;
- `ResourceIntent` is passed to a physical compute platform for a particular execution realization.

An ExecutionBinding may reference both, but their ownership and effects remain separate. See `EXECUTION_BINDING.md` and `RESOURCE_BOUNDARY.md`.

## Attempt identity

Every dispatched attempt receives a stable globally unique attempt id. Repeated delivery of the same dispatch command carries the same attempt id. A retry creates a **new** attempt id under the same TaskRun.

This dispatch identity is independent of workflow-submission idempotency.

## Backend handle

`ExecutionHandle` is opaque to core DAG logic. The adapter may use it to observe or cancel the external execution, but it does not become the logical identity or state of a TaskRun.
