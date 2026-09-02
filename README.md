# Andormu

**Shared DAG Workflow Execution Engine for Bronze Dragonflight**

> **Phase 0 — Product & Architecture Design Baseline**
>
> This repository intentionally contains **no implementation code**. The current goal is to land the product goals, execution semantics, architecture boundaries, contracts, ADRs, and design-gate criteria before implementation begins.

## Mission

Andormu is a **durable, event-driven workflow execution engine** that turns a versioned `WorkflowSpec` into an observable, recoverable `WorkflowRun` and can orchestrate persistent service tasks, asynchronous jobs, and compute-backed tasks through declarative DAGs.

```text
Domain Platforms
  Zidormi / Anachronos / Evaluation / Simulation / ...
                         │
                         │ WorkflowSpec
                         ▼
                    ┌─────────┐
                    │ Andormu │
                    └────┬────┘
                         │ TaskExecutionRequest
                         ▼
               Execution / Runtime Adapters
                         │
                         ▼
                  Compute Platform
                 CPU / GPU / NPU / ...
```

Andormu owns **workflow semantics and workflow execution state**. It does **not** own domain semantics or physical compute placement.

## Core boundary

**Andormu owns**

- Workflow validation and immutable execution snapshots.
- DAG dependency evaluation and runnable-node determination.
- Workflow, task-run, and task-attempt lifecycle state.
- Retry, timeout, cancellation, suspension, finalizer, and failure-propagation semantics.
- Durable execution history and recovery/redrive semantics.
- Dynamic graph expansion under explicit, bounded rules.
- Task dispatch contracts and execution-handle tracking.
- Persistent service-task and asynchronous-job orchestration semantics.
- Workflow/task backpressure, logical admission, and concurrency controls.
- Workflow events, auditability, observability, and operational UI semantics.

**Andormu does not own**

- Training, data, evaluation, or simulation domain semantics.
- GPU/CPU/NPU placement, quota, admission, topology, preemption, or cluster scheduling.
- Kubernetes/Slurm-specific APIs in the core domain model.
- User task business logic.
- Service deployment, replica management, or service discovery infrastructure.
- Dataset/model/artifact storage systems; Andormu passes references.
- Generic lineage ownership; it may emit compatible lineage events.
- Cron/product scheduling as a Phase-1 core requirement.
- Exactly-once external side effects.

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

We **do not copy any one product wholesale**. In particular, Andormu remains independent of Kubernetes and independent of physical resource scheduling.

## Foundational invariants

1. **Domain agnostic.** Andormu never needs to know that a task is “training”, “annotation”, “evaluation”, or “simulation”.
2. **Immutable execution snapshot.** Once a `WorkflowRun` starts, its resolved workflow revision and referenced task revisions are pinned.
3. **Acyclic execution graph.** Runtime fan-out and dynamic expansion are allowed only when the expanded execution graph remains acyclic.
4. **TaskRun != TaskAttempt.** Retries create new attempts without destroying the logical identity of a task occurrence.
5. **Persist before progress.** Durable state/event records must exist before the engine relies on a transition or external side effect.
6. **At-least-once dispatch, not exactly-once effects.** Execution adapters must deduplicate repeated dispatches for the same attempt; user side effects still require idempotency or compensation.
7. **Recovery is explicit.** Automatic crash recovery, task retry, workflow redrive, restart, and future targeted rerun are distinct operations.
8. **Resources stay external.** Andormu declares/passes resource intent; the Compute Platform admits and places work.
9. **Finalization is explicit.** Cleanup/finalizers have defined semantics and are not hidden trigger-rule tricks.
10. **Task is logical work, not a process.** A persistent service, async operation, worker, or compute job may fulfill a TaskAttempt.
11. **Backpressure is first-class.** Dependency-ready work must not imply unlimited dispatch to downstream services.
12. **No coding before Design Gate.** See `review/PHASE0_REVIEW.md` and `coding/ENTRY_CRITERIA.md`.

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
Platform boundary
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

Start with [`docs/00-goals/PRODUCT_GOALS.md`](docs/00-goals/PRODUCT_GOALS.md), then [`docs/01-architecture/OVERVIEW.md`](docs/01-architecture/OVERVIEW.md).
