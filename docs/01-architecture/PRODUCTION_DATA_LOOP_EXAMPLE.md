# Production Data Closed-Loop Reference Workflow

## Purpose

This document is the canonical Phase-0 production example used to test whether Andormu/Timeways abstractions remain useful, vendor-neutral, and operationally realistic.

It is illustrative rather than a finalized schema.

## Scenario

Two broad artifact classes are uploaded to object storage:

1. bag/log-style files;
2. normal data files.

Upload completion produces domain events through Kafka or an equivalent mature message system.

```text
                         Object Storage
                              │
                       upload completed
                              │
                 ┌────────────┴────────────┐
                 ▼                         ▼
          bag.file.uploaded         data.file.uploaded
                Kafka                     Kafka
                 │                         │
                 └────────────┬────────────┘
                              ▼
                         Zidormi Ingress
                              │
                     domain interpretation
                     workflow selection
                              │
                 ┌────────────┴────────────┐
                 ▼                         ▼
          bag-processing-vN        data-processing-vM
                 │                         │
                 └────────────┬────────────┘
                              ▼
                           Andormu
```

Kafka is transport, not workflow truth. Zidormi owns file/domain classification and workflow selection. Andormu must not inspect `.bag`, file extensions, or business metadata to decide which DAG runs.

## Different business DAGs

The two artifact classes do not need the same nodes.

A possible bag workflow:

```text
                     Bag ArtifactRef
                           │
                           ▼
                       Safety
                           │
                           ▼
                         Probe
                           │
                           ▼
                      Preprocess
                           │
                  ┌────────┴─────────┐
                  ▼                  ▼
               Decoder          Metadata
                  │                  │
                  ▼                  │
               Encoder               │
                  │                  │
                  └─────────┬────────┘
                            ▼
                           Index
                            │
                            ▼
                       Result Check
                            │
                            ▼
                         Publish
```

A possible normal-data workflow:

```text
                   Normal ArtifactRef
                           │
                           ▼
                       Safety
                           │
                           ▼
                      Preprocess
                           │
                  ┌────────┴────────┐
                  ▼                 ▼
               Encoder          Metadata
                  │                 │
                  └────────┬────────┘
                           ▼
                          Index
                           │
                           ▼
                      Result Check
                           │
                           ▼
                        Publish
```

Another business workflow may omit decoder, encoder, metadata, publish, or other nodes entirely.

**The domain workflow owns which nodes and edges exist.** Do not create a universal mega-workflow merely to represent every possible file type.

## Logical nodes, heterogeneous execution

The logical DAG remains independent of how each Task is executed.

Illustrative mappings:

| Logical capability | Possible realization |
|---|---|
| safety scan | persistent service + inline or deferred completion |
| probe / metadata | persistent service + inline completion |
| preprocess | worker, deferred service, Flink/Spark job |
| decode | persistent decoder service + deferred completion |
| encode | persistent GPU service or compute job |
| index | persistent service or deferred bulk-index operation |
| result check | service/worker implementing domain validation |
| publish | deferred external service operation |

These are examples, not canonical backend bindings.

The same logical capability may move from service to worker or compute job over time without changing the business DAG if its logical contract remains compatible.

## Service is not the same axis as async

A service is about **executor lifecycle**.

An asynchronous/deferred task is about **completion protocol**.

### Persistent service with inline completion

```text
Andormu                  Service
   │                        │
   │ Dispatch(attempt A1)   │
   ├───────────────────────►│
   │                        │ execute
   │◄───────────────────────┤
   │ terminal result        │
```

### Persistent service with deferred completion

```text
Andormu                  Decoder Service
   │                           │
   │ Submit(attempt A1)        │
   ├──────────────────────────►│
   │◄──────────────────────────┤
   │ accepted + handle         │
   │                           │ continues processing
   │◄──── event/callback ──────┤
   │ or Observe(handle)        │
   │                           │
   ▼                           │
terminal outcome
```

### Compute/data-engine job with deferred completion

```text
Andormu
   │ TaskExecutionRequest
   ▼
Adapter
   │
   ├── Flink / Spark
   ├── Kubernetes / Slurm / Ray
   └── cloud compute
   │
   ▼
ExecutionHandle
   │
   ▼
observe terminal outcome
```

From Andormu's perspective, deferred service work and compute/data-engine jobs share a useful high-level pattern: submit/accept, retain an opaque handle, observe later, then reconcile a terminal outcome.

## Standard infrastructure is reused

This reference workflow assumes mature infrastructure rather than replacing it.

Examples:

```text
OSS/S3/TOS/COS     artifact storage
Kafka/MQ           domain-event transport / worker transport
Flink/Spark        stream/batch data processing where appropriate
Kubernetes/Slurm   compute execution
Ray                distributed runtime where appropriate
GPU virtualization Compute Platform responsibility
OpenTelemetry      traces/metrics
```

Andormu owns none of their internal scheduling/data-processing semantics.

## Artifact flow

Large files never move through Andormu control-plane storage.

```text
Object Storage
     │
     │ ArtifactRef
     ▼
Zidormi -> WorkflowSpec(inputs=ArtifactRef)
     │
     ▼
Andormu
     │
     │ resolved ArtifactRef
     ▼
Task execution backend
```

The same applies to large decoder outputs, embeddings, checkpoints, indexes, and other artifacts.

## Workflow submission idempotency

Kafka and similar transports are at-least-once. Duplicate domain events must not accidentally create duplicate logical workflows when they represent the same business trigger.

The domain ingress should submit a stable idempotency identity derived from the business trigger, for example conceptually:

```text
trigger event identity
+ artifact immutable/version identity
+ selected workflow revision/policy identity
```

Workflow-submission idempotency and TaskAttempt dispatch idempotency are separate layers:

```text
duplicate Kafka/domain trigger
        ↓
WorkflowRun submission idempotency

duplicate execution delivery
        ↓
same TaskAttempt dispatch identity
```

The exact contract remains a Phase-0 review item.

## Logical admission example

Suppose 100,000 bag workflows reach `decode`, but the persistent decoder capability can safely process only 200 active attempts.

```text
100,000 dependency-ready Decode TaskRuns
                     │
                     ▼
             Andormu logical admission
                     │
              max active = 200
                     │
                     ▼
              Decoder capability
```

Flooding the decoder and relying only on HTTP 429 is an orchestration failure.

This is distinct from a GPU encoder task waiting because no physical accelerator is available:

```text
Encoder logically admitted
          │
          ▼
TaskAttempt / ResourceIntent
          │
          ▼
Compute Platform
          │
     physical queue
     quota / placement
     GPU / MIG / vGPU
```

Andormu owns the first decision; Compute owns the second.

## Data-engine and external-workflow boundary

If `preprocess` is implemented by a Flink job whose internal graph is:

```text
Source -> Parse -> Filter -> Window -> Aggregate -> Sink
```

Andormu normally sees one logical `preprocess` TaskAttempt.

Likewise a cloud workflow or training runtime may be one opaque task execution when it owns its internal DAG.

Do not mirror the same internal DAG in two orchestration systems.

## Result Check vs execution-contract validation

A business `Result Check` node is domain work, for example validating business completeness or data quality.

It is distinct from Andormu validating a task's declared output contract before accepting logical task success.

```text
backend says terminal success
        │
        ▼
correlate exact attempt
        │
        ▼
validate execution/output contract
        │
        ▼
persist logical outcome
        │
        ▼
release downstream dependencies
```

The exact validation contract remains to be designed.

## Workflow granularity

`1 large artifact = 1 WorkflowRun` is appropriate when the artifact is operationally meaningful and deserves independent tracing/recovery.

For millions of tiny files/operations, Zidormi should batch/partition inputs or request bounded fan-out so orchestration overhead does not dominate useful work.

## Phase-0 conformance questions

Every proposed Andormu design should be tested against this scenario:

1. Can bag and normal-data workflows use different logical DAGs without Andormu learning file semantics?
2. Can each Task change execution realization without changing DAG meaning?
3. Can persistent services use inline or deferred completion?
4. Can Flink/Spark/Kubernetes/Slurm/cloud jobs remain external engines?
5. Can logical service backpressure be enforced separately from GPU/resource scheduling?
6. Can duplicate trigger delivery and duplicate task dispatch remain distinct idempotency domains?
7. Can every transition be recovered and explained without relying on backend-native objects as canonical truth?
