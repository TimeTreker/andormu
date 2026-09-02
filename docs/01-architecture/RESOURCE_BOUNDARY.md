# Resource Boundary

## ResourceIntent is a request, not an allocation

A TaskSpec may carry or reference a resource intent such as:

```text
cpu: 32
memory: 128Gi
accelerator:
  kind: GPU
  model: H100
  count: 8
```

Andormu may validate structural correctness but must not decide where those resources come from.

## Compute Platform responsibilities

- tenant resource quota,
- queues/admission,
- physical placement,
- topology/NUMA/NVLink constraints,
- reservations,
- preemption,
- capacity health,
- cloud/cluster selection.

## Andormu responsibilities

- decide the task is logically ready,
- attach resource intent to TaskExecutionRequest,
- track admission/execution references returned by adapters,
- interpret resource/backend failures as structured attempt failures,
- apply task retry/cancellation semantics.

## Logical concurrency is allowed

Andormu may own workflow-level logical controls such as `max_parallelism=100`, because that constrains DAG progression rather than choosing physical resources.

Tenant hardware fairness belongs to Compute.
