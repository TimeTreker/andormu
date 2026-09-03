# Platform Boundaries

## Timeways is the execution platform; Andormu is its control plane

Timeways is a logical Bronze Dragonflight platform boundary, not a monolithic replacement for standard infrastructure.

```text
Domain Platforms
      │ WorkflowSpec
      ▼
   Andormu
 durable control plane
      │ TaskExecutionRequest
      ▼
Execution / Runtime Adapters
      │
      ├── persistent services / workers
      ├── Flink / Spark / data engines
      ├── Kubernetes / Slurm / Ray / compute
      └── cloud/on-prem execution systems
```

Andormu owns orchestration truth. The execution plane owns actual work. Cloud/Compute owns physical resources.

## Domain platform owns domain compilation and workflow selection

Anachronos may compile a `TrainingRun` into a `WorkflowSpec`; Zidormi may select/compile a bag-processing or normal-file-processing workflow from domain metadata and policy.

Andormu must not inspect file extensions, data types, model families, or other domain meaning to select the business DAG.

Once submitted, Andormu sees only the generic Bronze workflow/task contract.

## Bronze Dragonflight owns the canonical execution contract

`WorkflowSpec`, task semantics, shared execution observations, failure semantics, artifact/resource references, and other canonical execution contracts are Bronze-owned and vendor-neutral.

CloudFlow definitions, Argo YAML, Kubernetes objects, Slurm/Ray jobs, and other backend-native objects may be generated or invoked behind adapters, but they do not define the Bronze workflow model.

## Andormu owns workflow readiness and logical admission

Andormu answers:

- Are all required upstream conditions satisfied?
- Should this node run, wait, skip, retry, or fail?
- Is a finalizer eligible?
- Is a retry budget exhausted?
- Has the workflow timed out?
- Is a redrive allowed under the pinned snapshot?
- Among dependency-ready work, which work is logically admitted for attempt creation/dispatch now?
- Are workflow/capability concurrency, rate, queue-depth, or backpressure limits satisfied?

Dependency readiness and logical dispatch admission are separate concepts.

## Compute Platform owns physical resource admission and allocation

Compute answers:

- Is physical quota available?
- Which physical queue/pool/cluster should serve the request?
- Which CPU/GPU/NPU/host or virtualized accelerator allocation is selected?
- Which placement/topology/reservation policy applies?
- Should work be preempted?
- How are Kubernetes, Slurm, Ray, MIG/vGPU/MPS, or cloud-specific resources realized?

Andormu may pass `ResourceIntent` and observe generic admission/execution references, but it does not implement resource scheduling or GPU virtualization.

## Execution adapter owns backend translation

An adapter translates generic task-execution intent into a concrete backend API and returns generic observations plus an opaque `ExecutionHandle`.

The core model should not contain `Pod`, `SlurmJob`, `RayJob`, Flink operator state, cloud workflow state, or equivalent backend objects as canonical workflow state.

A backend-native phase may be evidence used by an adapter to produce a `TaskExecutionObservation`; it is not automatically a `TaskRun` state.

## Standard infrastructure remains external

Andormu deliberately reuses mature components rather than implementing replacements for them:

- Kafka/RocketMQ/Pulsar and message brokers;
- Flink/Spark and stream/batch data engines;
- Kubernetes/container runtimes, Slurm, Ray, and cloud compute systems;
- CUDA/device plugins/MIG/vGPU/MPS and GPU infrastructure;
- OSS/S3/TOS/COS and storage systems;
- PostgreSQL/MySQL/Redis and data stores;
- service discovery, networking, secret management, metrics, traces, and logs.

Andormu may strongly depend on capabilities provided by such systems, but its canonical semantics must remain independent of their native object models.

## External engines own their internal execution semantics

If Flink, Spark, Ray, a training runtime, a cloud workflow service, or another mature system already owns an internal graph, Andormu normally submits and observes it as one opaque TaskAttempt.

Do not create two competing orchestration authorities for the same internal DAG.

## Artifact systems own payload storage

Andormu stores metadata and references. Large datasets, checkpoints, logs, models, bags, normal data files, and arbitrary binary outputs belong in external systems.
