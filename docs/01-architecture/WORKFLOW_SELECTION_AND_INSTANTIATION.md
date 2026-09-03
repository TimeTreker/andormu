# Workflow Selection and Instantiation

## Principle

Andormu executes a submitted WorkflowSpec/WorkflowDefinition revision. It does **not** infer domain-specific workflow choice from business data.

## Data-loop example

A Zidormi file-processing system may receive:

- MP4 video,
- ROS bag,
- Cyber log,
- point-cloud archive,
- metadata-only package.

The selection belongs to Zidormi:

```text
FileArtifact
    │
    ▼
Domain metadata / policy
    │
    ▼
Workflow selection or compilation
    │
    ▼
WorkflowSpec + inputs
    │
    ▼
Andormu
```

Andormu must not contain rules such as `if filename.endswith(".bag")`.

## Reusable definition, per-input run

Prefer reusable definitions:

```text
video-processing-v4
rosbag-processing-v3
cyberlog-processing-v5
```

Each concrete input normally creates a separate WorkflowRun with pinned inputs and ExecutionSnapshot. The caller supplies a stable workflow-submission idempotency identity so duplicate event delivery returns the existing run instead of creating another.

## Granularity rule

`1 file = 1 WorkflowRun` is appropriate when the unit of work is operationally meaningful and long enough to justify individual tracing/recovery.

It is inappropriate for millions of tiny operations where orchestration overhead exceeds useful work. In that case the domain platform should batch/partition inputs or generate bounded fan-out inside a coarser WorkflowRun.

## Andormu responsibility

Andormu owns:

- instantiating a run from the supplied definition/revision and inputs,
- enforcing workflow-submission idempotency within the declared caller/scope,
- pinning the resolved execution snapshot,
- validating graph and bindings,
- executing and observing the run.

The caller owns:

- business policy that chooses the workflow,
- domain-specific batching/partitioning decisions,
- interpretation of file/data semantics.
