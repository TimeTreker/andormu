# Production Data Loop Reference Workflows

## Purpose

This is the canonical Phase-0 regression scenario for Andormu's workflow contract. It tests whether Bronze-owned contracts can express two real production DAGs while keeping domain selection, transport, service topology, compute backends, and physical scheduling outside graph semantics.

The examples are conceptual and deliberately do **not** define a JSON, YAML, or Proto schema.

## Ownership and ingress

```text
Object Storage
      │ upload-completed domain event
      ▼
Kafka or equivalent transport
      │ at-least-once delivery
      ▼
Zidormi
      │ classify artifact + select exact workflow revision
      ▼
bag-processing-v3 OR normal-file-processing-v5
      │ WorkflowSpec + ArtifactRef + submission idempotency key
      ▼
Andormu
```

Kafka is transport, not workflow truth. Zidormi owns file classification and workflow selection. Andormu never inspects `.bag`, filename extensions, or domain metadata to choose a DAG.

## Canonical business graphs

### bag-processing-v3

```text
                safety
                   │
                   ▼
                 probe
                   │
                   ▼
              preprocess
                   │
          ┌────────┴─────────┐
          ▼                  ▼
       decoder            metadata
          │                  │
          ▼                  │
       encoder               │
          │                  │
          └─────────┬────────┘
                    ▼
                  index
                    │
                    ▼
              result_check
                    │
                    ▼
                 publish
```

### normal-file-processing-v5

```text
               safety
                  │
                  ▼
              preprocess
                  │
          ┌───────┴────────┐
          ▼                ▼
       encoder          metadata
          │                │
          └───────┬────────┘
                  ▼
                index
                  │
                  ▼
             result_check
                  │
                  ▼
               publish
```

The graphs intentionally differ. The normal-file workflow has no `probe` or `decoder`; another valid workflow may omit any other node. Zidormi owns those business choices. Andormu must not require a universal mega-workflow.

## Logical contract view

Representative exact logical contracts could be:

| Node | TaskDefinition revision | Capability |
|---|---|---|
| `safety` | `data.artifact.safety` revision `v2` | `data.artifact.safety@v2` |
| `probe` | `data.bag.probe` revision `v1` | `data.bag.probe@v1` |
| `preprocess` | `data.artifact.preprocess` revision `v4` | `data.artifact.preprocess@v4` |
| `decoder` | `data.bag.decode` revision `v3` | `data.bag.decode@v3` |
| `encoder` | `data.embedding.encode` revision `v2` | `data.embedding.encode@v2` |
| `metadata` | `data.metadata.extract` revision `v2` | `data.metadata.extract@v2` |
| `index` | `data.index.write` revision `v4` | `data.index.write@v4` |
| `result_check` | `data.result.validate` revision `v2` | `data.result.validate@v2` |
| `publish` | `data.result.publish` revision `v3` | `data.result.publish@v3` |

These names are stable regression examples, not a normative global catalog.

A representative decoder occurrence says only:

```text
TaskSpec decoder
  TaskDefinition = data.bag.decode revision v3
  depends-on = preprocess
  input artifact = output(preprocess, artifact)
  output decoded-artifacts = ArtifactRef[]
```

No DAG node says HTTP, gRPC, Kafka, service, worker, Flink, Kubernetes, Slurm, cloud provider, endpoint, or GPU node.

## Environment execution view

Execution realization is resolved separately and pinned into the `ExecutionSnapshot`:

| Capability | Provider | Completion | Illustrative target |
|---|---|---|---|
| `data.artifact.safety@v2` | `SERVICE` | `INLINE` | `safety.production` |
| `data.bag.probe@v1` | `SERVICE` | `INLINE` | `bag-probe.production` |
| `data.artifact.preprocess@v4` | `EXTERNAL_RUNTIME` | `DEFERRED` | `preprocess.runtime.v4` |
| `data.bag.decode@v3` | `SERVICE` | `DEFERRED` | `decoder.production` |
| `data.embedding.encode@v2` | `COMPUTE` | `DEFERRED` | `embedding.encoder.v2` |
| `data.metadata.extract@v2` | `WORKER` | `DEFERRED` | `metadata.worker.v2` |
| `data.index.write@v4` | `SERVICE` | `DEFERRED` | `index.production` |
| `data.result.validate@v2` | `WORKER` | `DEFERRED` | `result-check.worker.v2` |
| `data.result.publish@v3` | `SERVICE` | `DEFERRED` | `publish.production` |

This table represents one possible production environment. Another environment may bind the same exact capabilities differently without changing either WorkflowSpec or TaskDefinition.

### Service and completion are separate axes

```text
short service call       = SERVICE + INLINE
long decoder operation   = SERVICE + DEFERRED
GPU-backed encoding      = COMPUTE + DEFERRED
```

`SERVICE`, `WORKER`, `COMPUTE`, and `EXTERNAL_RUNTIME` are binding provider classes, not parallel Task kinds. `INLINE` and `DEFERRED` are completion models.

For deferred work, the binding may support `CALLBACK`, `POLL`, `EVENT`, or `WATCH`. DAG dependency evaluation still consumes one normalized logical terminal outcome.

## Artifact contract

Large file content never becomes WorkflowSpec or control-plane event payload. The workflow carries a durable reference with enough identity and integrity information to resolve the same object later:

```text
ArtifactRef
  provider
  uri
  immutable version
  checksum/digest
  size
  media type
  bounded metadata reference
```

For example, the ingress ArtifactRef may resolve to `oss://bucket/data/12345.bag`. Credentials are always referenced separately, never embedded in the ArtifactRef.

Task outputs flow into downstream inputs by reference:

```text
safety.safe-artifact
        │ ArtifactRef
        ▼
preprocess.artifact
        │ ArtifactRef
        ▼
decoder.artifact
```

The same rule applies to decoded frames, embeddings, checkpoints, indexes, and publication manifests.

## Workflow-submission idempotency

At-least-once delivery may repeat the same event:

```text
bag.file.uploaded event E100
        │
        ├── delivery 1 ──► Zidormi
        └── redelivery ──► Zidormi
```

Zidormi submits a stable idempotency key derived from the business trigger identity, immutable artifact/version identity, and selected workflow revision/policy identity. Within its defined scope, repeated submission with the same key and equivalent request returns the existing WorkflowRun rather than creating another.

The contract must reject reuse of the same key with a materially different submission; it must not silently alias two business requests.

Two idempotency domains remain distinct:

```text
duplicate domain trigger     -> WorkflowRun submission idempotency
duplicate dispatch delivery  -> same TaskAttempt dispatch identity
retry after task failure     -> new TaskAttempt identity
```

## Logical admission versus ResourceIntent

Suppose the environment policy allows at most 200 active decoder attempts:

```text
100,000 dependency-ready decoder TaskRuns
                     │
                     ▼
AdmissionPolicy decoder.production
          max active attempts = 200
                     │
                     ▼
              decoder attempts
```

The policy is referenced by the decoder ExecutionBinding. It is not part of `data.bag.decode@v3` and does not belong in the WorkflowSpec.

The encoder's physical requirement is different:

```text
encoder logically admitted by Andormu
              │
              ▼
TaskExecutionRequest + ResourceIntent(GPU)
              │
              ▼
Compute Platform physical admission
     quota / queue / placement / accelerator
```

Andormu owns the first decision. Compute owns the second. They must not be collapsed into a single “GPU capacity” field.

## Two validation layers

### Execution-contract validation

Andormu validates that an observation matches the pinned attempt and TaskDefinition contract before accepting success. If a decoder reports terminal success but omits required `decoded-artifacts`, the attempt cannot become logically successful.

```text
backend reports success
        │
        ▼
correlate exact TaskAttempt
        │
        ▼
validate required output contract
        │ failure
        ▼
record contract failure; do not release success dependencies
```

### Business result validation

`result_check` is a normal domain-authored task. It may verify frame counts, index completeness, data quality, or publication invariants. Andormu executes it but does not own those business rules.

The two layers are never interchangeable: structural execution-contract validity is a control-plane responsibility; business quality is explicit work in the DAG.

## External engine boundary

If `preprocess` is realized by a Flink job whose internal graph is:

```text
Source -> Parse -> Filter -> Window -> Aggregate -> Sink
```

Andormu normally sees one opaque `preprocess` TaskAttempt. The same applies to Spark, a training runtime, or a cloud workflow that owns its internal execution graph. There must be one orchestration authority per DAG layer.

## Workflow granularity

`1 large artifact = 1 WorkflowRun` is appropriate when the artifact is operationally meaningful and deserves independent tracing/recovery. For millions of tiny objects, Zidormi should batch/partition them or request bounded fan-out so orchestration overhead does not dominate useful work.

## Review results closed by this reference

This reference establishes four contract decisions:

1. DAG nodes describe logical work and graph control, never deployment/runtime kinds.
2. TaskDefinition is the reusable logical contract; TaskSpec is one graph occurrence of that exact contract.
3. ExecutionBinding maps an exact capability to an environment realization and is pinned separately from the DAG.
4. `bag-processing-v3` and `normal-file-processing-v5` are permanent Phase-0 regression workflows for later protocol, state, retry, timeout, cancellation, admission, and persistence reviews.

## Conformance questions for later designs

Every later design must preserve the following:

1. Can both reference DAGs be expressed without Andormu learning file semantics?
2. Can a capability change provider or target without changing logical graph meaning?
3. Can a persistent service use either inline or deferred completion?
4. Can data engines and compute platforms remain external execution authorities?
5. Can logical admission remain distinct from physical resource admission?
6. Can duplicate workflow submission and duplicate attempt dispatch remain separate idempotency domains?
7. Can execution success be contract-validated independently from the business `result_check` task?
8. Can every run pin exact logical and execution-binding revisions without freezing a vendor backend object into the DAG?
