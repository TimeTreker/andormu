# Backpressure and Dispatch Admission

## Why this belongs in Andormu

A workflow engine can overwhelm a healthy downstream service even when physical compute capacity is sufficient.

Example:

```text
100,000 READY decode tasks
           │
           ▼
video-decoder service capacity = 100 concurrent
```

Dispatching all 100,000 requests immediately is an orchestration failure, not a GPU placement problem.

## Logical admission vs resource admission

Andormu may own **logical dispatch admission**:

- per-workflow max parallelism,
- per-task/capability concurrency limit,
- tenant/workflow rate limit,
- priority among READY work,
- queue depth / dispatch pacing,
- bounded dynamic fan-out,
- service-specific backpressure hints.

Compute Platform owns **physical resource admission**:

- quota,
- queue placement,
- accelerator/CPU availability,
- physical node placement,
- preemption,
- topology,
- reservations.

## AdmissionPolicy placement

An environment `ExecutionBinding` may reference an immutable `AdmissionPolicy` revision such as:

```text
AdmissionPolicy decoder.production revision 4
  max-active-attempts = 200
```

The policy protects a logical capability/target; it is not part of TaskDefinition or TaskSpec. It also does not describe physical CPU/GPU capacity. Phase 0 does not yet define the complete policy schema or fairness dimensions.

## Proposed readiness split

A task may be dependency-ready but not dispatch-admitted.

```text
BLOCKED
   ↓ dependencies satisfied
READY
   ↓ logical admission granted
DISPATCHABLE
   ↓ adapter accepts work
RUNNING / WAITING
```

Whether `DISPATCHABLE` becomes a persisted public state or an internal scheduling condition is still open for review.

## Capacity signal

Andormu must not require one capacity mechanism. Capacity may come from:

- configured concurrency limits,
- worker slot advertisement,
- service-provided capacity/429 signals,
- queue depth,
- adapter feedback,
- Compute Platform admission status as execution feedback, without treating physical capacity as the logical policy itself.

## Fairness

Hardware fairness belongs to Compute Platform. Workflow/task fairness may belong to Andormu when it decides which logically ready task should be dispatched next.

The Phase-0 review must define whether cross-tenant logical fairness is core or delegated to a shared admission service.
