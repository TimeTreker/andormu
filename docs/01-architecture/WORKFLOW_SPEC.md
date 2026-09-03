# WorkflowSpec Design

This document is **conceptual, not a finalized JSON, YAML, or Proto schema**.

## Responsibility

A `WorkflowSpec` describes a versioned business execution graph. It says which logical work occurs, how data references flow between nodes, and which dependency conditions control progress.

It does not describe how a service is reached, which runtime launches work, which cluster supplies resources, or which vendor object represents an execution.

## Canonical layering

```text
WorkflowSpec
    │ contains
    ▼
TaskSpec (one logical occurrence in this graph)
    │ references an exact revision
    ▼
TaskDefinition (reusable logical contract)
    │ requires
    ▼
Capability (versioned semantic requirement)
    │ resolved for an environment
    ▼
ExecutionBinding (pinned execution realization)
    │ selects
    ▼
Execution Adapter
```

Only the first two objects define graph topology. `ExecutionBinding` is resolved outside the DAG and pinned into the run's `ExecutionSnapshot`.

## Conceptual contents

```text
WorkflowSpec
  identity + revision
  contract version
  interface
    inputs
    outputs
  nodes[]
    id
    logical node kind
    task-definition revision (for task nodes)
    input/output bindings
    dependencies + trigger rule
    logical policy overrides
  finalizers[]
  workflow policies
    timeout
    failure mode
    logical max parallelism
  annotations/tags
```

The notation above only identifies semantic fields for review; it is not a serialization proposal.

## Logical task example

```text
node decoder
  task-definition = data.bag.decode revision v3
  depends-on = preprocess
  input artifact = output(preprocess, artifact)
  output decoded-artifacts = ArtifactRef[]
```

The node asks for the logical work defined by `data.bag.decode@v3`. It does not say `HTTP`, `gRPC`, `Kafka`, `Flink`, `Kubernetes`, `Slurm`, `Volcano`, `Pod`, `GPU node`, IP address, endpoint, worker queue, or cloud product.

## Different workflows may have different graphs

Bag processing and normal-file processing are separate domain workflows. They do not need identical nodes or a universal superset graph. Zidormi may select either workflow, but Andormu only validates and executes the submitted graph.

See `PRODUCTION_DATA_LOOP_EXAMPLE.md` for the canonical Phase-0 regression scenario.

## Why graph-oriented rather than arbitrary workflow code

A graph-oriented contract:

- can be validated before execution;
- can be visualized consistently;
- is easier to version and diff;
- does not require deterministic replay of user orchestration code;
- lets domain platforms use any implementation language to generate the same contract.

## Prohibited execution leakage

The canonical graph must not contain:

- transport protocols or endpoints;
- provider/runtime kinds used as business node kinds;
- backend-native job, service, queue, or cluster objects;
- physical resource placement or allocation details;
- environment-specific admission limits.

A provider change from a service to a worker or compute adapter must not require a WorkflowSpec revision when the logical TaskDefinition contract remains compatible.

## Versioning invariant

No running `WorkflowRun` follows a mutable alias. Before the first task is dispatched, its `ExecutionSnapshot` pins:

- the exact WorkflowSpec/WorkflowRevision;
- every referenced TaskDefinition revision;
- every resolved ExecutionBinding revision;
- applicable policy revisions and bound workflow inputs.

Changing an environment binding affects only future snapshots. It never mutates an active or historical run.
