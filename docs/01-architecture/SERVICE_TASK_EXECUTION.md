# Service Task Execution

## Purpose

Andormu must treat a task as a **logical unit of work**, not as a process, container, Pod, or machine allocation.

A task may be fulfilled by a long-running service that already exists before the WorkflowRun starts and remains alive after the TaskRun completes.

```text
TaskRun
   │
   ▼
TaskAttempt
   │
   ▼
TaskExecutionRequest
   │
   ▼
Execution Adapter
   │
   ├── existing HTTP/gRPC service
   ├── worker pool
   ├── asynchronous service job
   └── compute-backed job
```

## Why this is first-class

Bronze Dragonflight workloads include file-processing pipelines where many persistent services cooperate:

- file validation,
- archive/probe services,
- video decoding,
- ROS bag / Cyber log parsing,
- frame extraction,
- point-cloud conversion,
- anonymization,
- metadata extraction,
- embedding / deduplication,
- publication.

Starting a new process for every logical task would be the wrong abstraction for many of these workloads.

## Canonical rule

`TaskDefinition` describes **what logical capability and execution contract are required**. `TaskSpec` places one exact definition in the graph. An environment `ExecutionBinding` selects provider, completion model, adapter, and stable target; adapter/runtime infrastructure then decides how that target is physically reached.

The canonical spec SHOULD NOT embed physical instance addresses such as fixed IPs or transient Pod names.

The TaskDefinition requires a stable capability:

```text
capability = data.video.decode@v2
```

rather than:

```text
endpoint = http://10.23.34.18:8912
```

Even a stable ExecutionTarget belongs in the ExecutionBinding, not in the WorkflowSpec.

## Completion semantics

A service-backed TaskAttempt must have an explicit completion model in its pinned ExecutionBinding. Phase 0 distinguishes `INLINE` from `DEFERRED`; deferred observation may use:

1. callback;
2. event;
3. poll/observe;
4. watch.

A persistent TCP/HTTP connection must not be required for the whole task duration.

## Ownership boundary

Andormu owns:

- dispatch identity,
- TaskAttempt lifecycle,
- retry/timeout/cancel semantics,
- correlation and event history,
- logical concurrency/backpressure policies.

Andormu does not own:

- service deployment,
- service replica count,
- service discovery infrastructure,
- physical CPU/GPU allocation,
- cluster autoscaling,
- service health remediation beyond task-level observation/failure classification.

## Design consequence

The execution-adapter abstraction must be broad enough that a service task and a compute job share the same TaskRun/TaskAttempt semantics without pretending they share the same backend lifecycle.
