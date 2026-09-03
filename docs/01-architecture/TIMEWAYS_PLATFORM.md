# Timeways Platform Boundary

## Status

Accepted platform positioning. Detailed Andormu execution semantics remain subject to the Phase-0 Design Gate where marked Proposed.

## Definition

**Timeways** is the vendor-neutral execution platform of Bronze Dragonflight.

**Andormu** is the durable control plane of Timeways.

Timeways is not a new monolithic runtime. It is the architectural composition of:

1. Bronze-owned workflow/task execution contracts;
2. Andormu durable orchestration and logical admission;
3. execution adapters;
4. mature standard/cloud/on-prem execution and infrastructure systems.

```text
                         Bronze Dragonflight

 Domain Platforms
 Zidormi / Anachronos / Evaluation / Simulation / ...
                         │
                         │ Bronze WorkflowSpec
                         ▼
┌──────────────────────────────────────────────────────────────┐
│                          TIMEWAYS                            │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │                    ANDORMU                             │  │
│  │              Durable Control Plane                    │  │
│  │                                                       │  │
│  │ Workflow semantics | logical admission | recovery     │  │
│  │ Task identity       | durable history    | audit       │  │
│  └──────────────────────────┬─────────────────────────────┘  │
│                             │ TaskExecutionRequest            │
│                             ▼                                 │
│                    Execution Adapters                         │
│                             │                                 │
│        ┌────────────────────┼────────────────────┐            │
│        ▼                    ▼                    ▼            │
│ Services / Workers    Data Engines          Compute Jobs      │
│ HTTP/gRPC/MQ          Flink/Spark           K8s/Slurm/Ray     │
└─────────────────────────────┬────────────────────────────────┘
                              ▼
                  Cloud / On-Prem Infrastructure
      Alibaba | Volcano | Tencent | Baidu | private/edge clusters
```

## Ownership model

### Domain platforms own meaning

Domain platforms answer:

- What does this artifact/data/training/evaluation request mean?
- Which business workflow should run?
- Which logical capabilities are required?
- Which nodes and edges belong in that workflow?

Examples:

- Zidormi distinguishes bag-file processing from normal-data-file processing.
- Anachronos selects/compiles training workflows.
- Evaluation owns evaluation semantics.

### Andormu owns logical execution truth

Andormu answers:

- What exact WorkflowSpec/revision is being executed?
- Which logical nodes are blocked/ready/terminal?
- Which dependency predicate is satisfied?
- Which work is logically admitted for dispatch?
- What is the authoritative logical task/attempt history?
- What retry/timeout/cancel/recovery/redrive semantics apply?
- Why is the workflow currently in its observed logical state?

### Execution systems own work execution

Services, workers, Flink/Spark, training runtimes, cloud APIs, and compute jobs own their internal execution mechanics.

Andormu normally interacts through a small adapter boundary such as:

```text
Dispatch(TaskAttempt)
Observe(TaskAttempt / ExecutionHandle)
Cancel(TaskAttempt / ExecutionHandle)
```

A backend may additionally support callback/event/watch/heartbeat/capacity capabilities.

### Compute/cloud platforms own physical resources

Compute/cloud systems own:

- quota and physical admission;
- CPU/GPU/NPU capacity;
- placement and topology;
- Kubernetes/Slurm/Ray scheduling;
- MIG/vGPU/MPS or other GPU virtualization;
- reservations and preemption;
- cluster/node selection.

Andormu may pass `ResourceIntent` but does not implement these systems.

## Strategic invariants

### SI-1 — Canonical contract ownership

Bronze Dragonflight owns its workflow/task execution contracts. Vendor workflow definitions and backend-native objects are integration targets, not the canonical model.

### SI-2 — Semantic execution portability

The same Bronze logical workflow semantics must remain meaningful across supported cloud and on-prem execution environments.

This is stronger than syntax translation. Retry, cancellation, logical task identity, recovery, and downstream dependency meaning must not change merely because the backend changes.

### SI-3 — Logical admission ownership

Andormu owns admission of logically ready work into execution.

Examples include:

- per-workflow logical parallelism;
- per-capability/service active-attempt limits;
- dispatch pacing/rate limits;
- queue-depth/backpressure policies;
- bounded fan-out;
- priority/fairness policy where accepted.

Compute owns physical resource admission.

### SI-4 — Reuse standardized infrastructure

Andormu does not reimplement mature infrastructure:

```text
Messaging:       Kafka / RocketMQ / Pulsar / ...
Data engines:    Flink / Spark / ...
Compute:         Kubernetes / Slurm / Ray / cloud compute
GPU:             CUDA / device plugins / MIG / vGPU / MPS / ...
Storage:         OSS / S3 / TOS / COS / ...
State systems:   PostgreSQL / MySQL / Redis / ...
Observability:   OpenTelemetry / metrics / logs / traces
```

Timeways is built on these systems.

### SI-5 — One orchestration authority per DAG layer

Do not place Andormu and another workflow/data engine in competing control of the same internal DAG.

If a Flink/Spark/training/cloud workflow owns an internal graph, Andormu normally sees:

```text
TaskAttempt
    │
    ▼
external engine submission
    │
    ▼
ExecutionHandle
    │
    ▼
terminal observation
```

The external engine remains authoritative for its internal operators.

## Logical DAG nodes vs execution realization

The canonical DAG describes business/logical graph semantics.

Examples of graph-semantic node kinds may include:

- Task;
- Gate/condition;
- Map/fan-out;
- Dynamic expansion;
- Subworkflow;
- Finalizer.

A `Task` is logical work. Its execution realization is separate:

```text
Logical Task
    │
    ▼
TaskAttempt
    │
    ├── persistent service + inline completion
    ├── persistent service + deferred completion
    ├── worker/task queue
    ├── Flink/Spark/data-engine job
    ├── external workflow/runtime operation
    └── compute-backed job
```

`service`, `async_service`, `kubernetes_job`, `slurm_job`, and similar deployment/runtime mechanisms should not become business DAG node kinds merely because they are how work happens to run today.

## Service vs asynchronous completion

A **persistent service** describes the executor lifecycle: the service exists before a TaskAttempt and remains after it completes.

**Inline vs deferred completion** describes the TaskAttempt protocol:

### Inline

```text
Dispatch
   ↓
service performs work
   ↓
terminal result in same interaction
```

### Deferred

```text
Dispatch
   ↓
accepted + optional ExecutionHandle
   ↓
service/job continues independently
   ↓
Observe / callback / event / watch
   ↓
terminal result
```

A persistent service may support either or both completion models.

## Dependency model

DAG edges depend on logical `TaskRun` outcomes, not backend phases.

```text
backend observation
       ↓
TaskAttempt observation
       ↓
Andormu logical reconciliation
       ↓
TaskRun outcome
       ↓
dependency evaluation
```

Examples of non-canonical shortcuts:

```text
HTTP 200          -> downstream ready       (not sufficient in general)
K8s Pod Succeeded -> TaskRun success         (adapter/reconciliation required)
Flink RUNNING     -> DAG dependency state    (backend detail)
```

## Infrastructure dependencies

Andormu may have strong production dependencies on mature components.

The design goal is not `zero dependencies`; it is **stable semantic boundaries**.

For example, an implementation may require a durable store with ACID transactions and uniqueness/conditional-update capabilities without making PostgreSQL-specific row semantics part of `WorkflowSpec`.

Likewise Kafka may be the production event transport without making Kafka offset/delivery identity equal to `TaskAttempt` or `WorkflowRun` identity.

## Cloud workflow products

Cloud workflow engines may be useful:

1. as an alternative implementation choice when Andormu is not needed;
2. as an opaque external execution backend for one Task when a domain intentionally delegates a sub-workflow.

They should not normally sit beneath Andormu as a second owner of the same Timeways DAG state machine.

## Related ADRs

- ADR-0013 — Task is logical work, not a process.
- ADR-0014 — Andormu is the durable control plane of Timeways.
- ADR-0015 — Bronze owns the canonical execution contract and semantic portability.
- ADR-0016 — Reuse standardized infrastructure; do not reimplement it.
- ADR-0017 — Andormu owns logical admission; Compute owns physical admission.
- ADR-0018 — DAG task semantics are separate from execution realization.
