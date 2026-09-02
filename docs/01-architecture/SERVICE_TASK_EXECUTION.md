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

`TaskSpec` describes **what logical capability is required and the execution contract**, while adapters/runtime infrastructure decide how that capability is reached.

The canonical spec SHOULD NOT embed physical instance addresses such as fixed IPs or transient Pod names.

Prefer a stable capability or execution-target reference:

```text
capability: data.video.decode@v2
```

rather than:

```text
endpoint: http://10.23.34.18:8912
```

## Completion semantics

A service-backed TaskAttempt must have explicit observable completion semantics. Supported patterns may include:

1. synchronous request/response for short work;
2. asynchronous submit + callback/event;
3. asynchronous submit + poll/observe;
4. worker lease / task queue acknowledgement;
5. backend execution handle observation.

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
