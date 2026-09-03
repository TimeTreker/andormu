# Andormu

**Durable Control Plane for Timeways — Bronze Dragonflight's vendor-neutral execution platform**

> **Phase 0 — Product & Architecture Design Baseline**
>
> This repository intentionally contains **no implementation code**. The current goal is to land the product goals, execution semantics, architecture boundaries, contracts, ADRs, and design-gate criteria before implementation begins.

## Mission

**Timeways** is the vendor-neutral execution platform of Bronze Dragonflight. It combines Andormu's durable control-plane semantics with standard service, data-processing, compute, messaging, storage, and observability infrastructure.

**Andormu is the durable control plane of Timeways.** It turns a versioned `WorkflowSpec` into an observable, recoverable `WorkflowRun`, owns canonical DAG/task execution semantics and logical dispatch admission, and orchestrates heterogeneous execution backends across cloud and on-prem environments.

```text
Domain Platforms
  Zidormi / Anachronos / Evaluation / Simulation / ...
                         │
                         │ Bronze WorkflowSpec
                         ▼
┌───────────────────────────────────────────────────────────────┐
│                          Timeways                              │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                    Andormu                              │  │
│  │              Durable Control Plane                     │  │
│  │ workflow/task semantics | logical admission | recovery │  │
│  └───────────────────────────┬─────────────────────────────┘  │
│                              │ TaskExecutionRequest            │
│                              ▼                                 │
│                 Execution / Runtime Adapters                   │
│                              │                                 │
│        ┌─────────────────────┼──────────────────────┐          │
│        ▼                     ▼                      ▼          │
│   Services/Workers      Flink/Spark/etc.       Compute Jobs    │
│                                              K8s/Slurm/Ray/... │
└──────────────────────────────┬────────────────────────────────┘
                               ▼
                 Cloud / On-Prem Infrastructure
       Alibaba | Volcano | Tencent | Baidu | private clusters
```

Timeways is **not** a monolithic reimplementation of these systems. Kafka/MQ, Flink/Spark, Kubernetes, Slurm, Ray, GPU virtualization, object storage, databases, service discovery, and observability systems remain standard infrastructure.

Andormu owns **logical orchestration truth**. Domain platforms own business meaning. Execution backends perform work. Compute/cloud platforms own physical resources.

## Core boundary

**Andormu owns**

- The Bronze Dragonflight canonical workflow/task execution contract.
- Workflow validation and proposed immutable execution snapshots.
- DAG dependency evaluation and runnable-node determination.
- Logical dispatch admission, workflow/capability concurrency, and backpressure semantics.
- Workflow, task-run, and task-attempt lifecycle semantics.
- Retry, timeout, cancellation, suspension, finalizer, and failure-propagation semantics.
- Durable execution history and recovery/redrive semantics.
- Dynamic graph expansion under explicit, bounded rules.
- Task dispatch contracts, stable attempt identity, and execution-handle tracking.
- Persistent-service, deferred/asynchronous, worker, data-engine, and compute-job execution integration through adapters.
- Workflow events, auditability, observability, and operational UI semantics.
- Semantic portability across supported cloud and on-prem execution environments.

**Andormu does not own**

- Training, data, evaluation, or simulation domain semantics.
- Domain workflow selection from file type, dataset type, model family, or business policy.
- GPU/CPU/NPU placement, quota, physical admission, topology, preemption, or cluster scheduling.
- Kubernetes/Slurm/Ray/cloud-native objects in the canonical Andormu domain model.
- Kafka/MQ, Flink/Spark, Kubernetes, container runtime, GPU virtualization, object storage, databases, service discovery, or observability implementations.
- User task business logic.
- Service deployment, replica management, or cluster autoscaling.
- Dataset/model/artifact storage systems; Andormu passes references.
- Generic lineage ownership; it may emit compatible lineage events.
- Cron/product scheduling as a Phase-1 core requirement.
- Exactly-once external side effects.

## Infrastructure philosophy

**Own the semantics; reuse the infrastructure.**

Andormu may depend strongly on mature infrastructure capabilities in production, but those systems must not define the canonical workflow semantics.

Examples:

```text
Kafka delivery id       != TaskAttempt identity
Kubernetes Job phase    != TaskRun state
Flink internal DAG      != Andormu DAG
GPU/MIG/vGPU placement  != Andormu logical admission
Cloud workflow syntax   != Bronze WorkflowSpec
```

When another mature system already owns internal execution semantics, Andormu normally treats that execution as one opaque TaskAttempt and integrates through `Dispatch`, `Observe`, and `Cancel`-style adapter operations.

See [`docs/01-architecture/TIMEWAYS_PLATFORM.md`](docs/01-architecture/TIMEWAYS_PLATFORM.md).

## Logical DAG vs execution realization

A canonical DAG contains **logical work and graph-control nodes**, not deployment/runtime process types.

```text
Task Node: data.bag.decode@v3
             │
             ▼
        TaskAttempt
             │
      execution realization
             │
     ┌───────┼────────────┬───────────────┐
     ▼       ▼            ▼               ▼
 inline   deferred      worker        compute/data
 service   service       queue          engine/job
```

A persistent service describes the lifecycle of the executor. Inline vs deferred/asynchronous describes how one TaskAttempt reaches completion. These dimensions must not be confused with DAG topology.

## Design synthesis

The Phase-0 design deliberately combines proven ideas from several workflow systems:

- **Temporal:** durable event history, explicit retries/cancellation, recovery after process failure.
- **Flyte Propeller:** immutable desired workflow state + observed execution state + reconciliation.
- **Argo Workflows:** DAGs, retry/backoff, suspend/resume, memoization concepts, exit handlers.
- **Apache Airflow 3:** explicit task dependencies, trigger rules, setup/teardown semantics, task-attempt history.
- **Prefect 3:** rich state information, event-driven operations, state transition visibility.
- **Tekton:** explicit `finally` semantics, graceful stop vs cancellation, granular timeouts.
- **AWS Step Functions:** retry/catch and redrive from failure while preserving successful work.
- **Conductor OSS / Orkes:** service-oriented worker tasks, dynamic fork/join, events, reusable task definitions, at-least-once delivery, and explicit recovery operations.
- **Hatchet:** durable task-queue + DAG thinking, worker-capacity controls, high-volume task dispatch, and event history.
- **Kestra 2:** stronger controller/worker separation and keeping large payloads outside workflow-control traffic.
- **Open Workflow Specification:** vendor-neutral workflow DSL concepts and CloudEvents interoperability.

We **do not copy any one product wholesale**. Cloud/vendor products and standard infrastructure may be execution dependencies or backends, but they are not the canonical Bronze Dragonflight execution contract.

## Foundational invariants

1. **Domain agnostic.** Andormu never needs to know that a task is “training”, “annotation”, “evaluation”, or “simulation”.
2. **Immutable execution snapshot.** Once accepted through the Design Gate, a `WorkflowRun` pins its resolved workflow revision and referenced task revisions.
3. **Acyclic execution graph.** Runtime fan-out and dynamic expansion are allowed only when the expanded execution graph remains acyclic.
4. **TaskRun != TaskAttempt.** Once accepted through the Design Gate, retries create new attempts without destroying the logical identity of a task occurrence.
5. **Persist before progress.** Durable state/event records must exist before the engine relies on a transition or external side effect.
6. **At-least-once dispatch, not exactly-once effects.** Execution adapters must deduplicate repeated dispatches for the same attempt; user side effects still require idempotency or compensation.
7. **Recovery is explicit.** Automatic crash recovery, task retry, workflow redrive, restart, and future targeted rerun are distinct operations.
8. **Resources stay external.** Andormu declares/passes resource intent; the Compute Platform admits and places work.
9. **Finalization is explicit.** Cleanup/finalizers have defined semantics and are not hidden trigger-rule tricks.
10. **Task is logical work, not a process.** A persistent service, deferred operation, worker, data engine, or compute job may fulfill a TaskAttempt.
11. **Logical admission is first-class.** Dependency-ready work does not imply unlimited dispatch; Andormu owns logical admission while Compute owns physical admission.
12. **Canonical contracts are Bronze-owned and vendor-neutral.** Semantic portability matters more than translating one vendor's syntax to another.
13. **Reuse standardized infrastructure.** Mature messaging, stream/batch processing, container, compute, GPU, storage, database, and observability systems are integrated, not reimplemented.
14. **One orchestration authority per DAG layer.** External workflow/data engines may fulfill an Andormu task, but Andormu must not duplicate ownership of the same internal execution graph.
15. **No coding before Design Gate.** See `review/PHASE0_REVIEW.md` and `coding/ENTRY_CRITERIA.md`.

Accepted architectural decisions are tracked in `DESIGN_STATUS.md` and ADRs. Detailed state, retry, persistence, redrive, dynamic-graph, and adapter protocol decisions remain subject to Phase-0 review where marked Proposed.

## Production reference scenario

The canonical Phase-0 production example is an OSS/Kafka-triggered data closed loop where Zidormi selects different DAGs for bag files and normal data files. The DAGs use only the business-required subset of safety, preprocess, decoder, encoder, index, result-check, and publication tasks, with execution realized by persistent services, deferred services, workers/data engines, or compute jobs.

See [`docs/01-architecture/PRODUCTION_DATA_LOOP_EXAMPLE.md`](docs/01-architecture/PRODUCTION_DATA_LOOP_EXAMPLE.md).

## Repository map

```text
andormu/
├── docs/00-goals/             Goal clarification and success criteria
├── docs/01-architecture/      Execution model and semantic design
├── docs/02-product/           Product UX / operational experience
├── docs/03-research/          Benchmark and design synthesis
├── docs/04-contract-design/   Contract drafts, not schemas yet
├── adr/                       Architecture decisions and proposals
├── contracts/                 Reserved for post-review normative schemas
├── coding/                    Future implementation TODO only
├── review/                    Phase-0 sign-off checklist
├── roadmap/                   Design-to-product roadmap
└── ui/                        UI information architecture
```

## Phase 0 sequence

```text
Goal clarification
       ↓
Platform boundary / Timeways positioning
       ↓
Production DAG semantic validation
       ↓
Execution semantics
       ↓
Product / operations design
       ↓
Contract design
       ↓
Architecture review
       ↓
DESIGN GATE
       ↓
Implementation planning
       ↓
Phase 1 coding
```

Start with [`docs/00-goals/PRODUCT_GOALS.md`](docs/00-goals/PRODUCT_GOALS.md), [`docs/01-architecture/TIMEWAYS_PLATFORM.md`](docs/01-architecture/TIMEWAYS_PLATFORM.md), and [`docs/01-architecture/PRODUCTION_DATA_LOOP_EXAMPLE.md`](docs/01-architecture/PRODUCTION_DATA_LOOP_EXAMPLE.md).
