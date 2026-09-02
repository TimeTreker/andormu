# Architecture Overview

## External architecture

```text
┌───────────────────────────────────────────────────────────────┐
│                     Domain Platforms                          │
│ Zidormi | Anachronos | Evaluation | Simulation | ...          │
└───────────────────────────────┬───────────────────────────────┘
                                │ WorkflowSpec
                                ▼
┌───────────────────────────────────────────────────────────────┐
│                         Andormu                               │
│                                                               │
│  API / Validation                                             │
│        │                                                      │
│        ▼                                                      │
│  Revision + Snapshot Store                                    │
│        │                                                      │
│        ▼                                                      │
│  Run Manager ──► Dependency Evaluator / Reconciler            │
│                       │        │                              │
│                       │        ├── Timers / Retry              │
│                       │        ├── Finalization                │
│                       │        └── Recovery                    │
│                       ▼                                       │
│                 Dispatch Gateway                              │
│                       │                                       │
│  State Projection ◄── Event Journal ──► Event Outbox          │
└───────────────────────┬───────────────────────────────────────┘
                        │ TaskExecutionRequest / ExecutionHandle
                        ▼
┌───────────────────────────────────────────────────────────────┐
│               Execution / Runtime Adapters                    │
│  service | async service | worker | compute job | subworkflow │
└───────────────────────┬───────────────────────────────────────┘
                        │ resource intent / backend actions
                        ▼
┌───────────────────────────────────────────────────────────────┐
│                    Compute Platform                           │
│ quota | queue | admission | placement | preemption | topology │
└───────────────────────────────────────────────────────────────┘
```

## Architectural style

The preferred design combines:

- **Immutable desired state:** an ExecutionSnapshot pins the graph and policies.
- **Durable observed state:** WorkflowRun / TaskRun / TaskAttempt transitions are persisted.
- **Reconciliation:** workers repeatedly determine which nodes are now runnable and what side effects are required.
- **Durable event history:** transitions and operator actions are append-only and queryable.
- **Adapter-driven execution:** backend-specific submission/observation/cancellation is isolated.

This intentionally borrows Flyte's reconciler strengths without coupling Andormu to Kubernetes, and Temporal's durable-history strengths without requiring user workflow code to be deterministic/replayed.

## Key split: planner vs executor

Andormu is a **workflow planner/orchestrator**, not the physical execution substrate.

```text
Dependency state says Task B is READY
               │
               ▼
Andormu creates TaskAttempt B#1
               │
               ▼
Dispatch adapter receives stable attempt identity
               │
               ▼
Compute/runtime ecosystem executes work
               │
               ▼
Adapter reports attempt state/result
               │
               ▼
Andormu persists transition and re-evaluates graph
```


## Execution target diversity

Andormu deliberately supports multiple task-execution shapes behind the same logical TaskRun/TaskAttempt model:

```text
TaskAttempt
    │
    ├── persistent service call
    ├── asynchronous service operation
    ├── durable worker/task-queue execution
    ├── external runtime job
    └── compute-backed job
```

This is why task lifecycle, service lifecycle, and compute allocation lifecycle are separate concepts.

## Data-loop integration example

For file/data processing, the domain platform (for example Zidormi) selects or compiles the appropriate WorkflowSpec from file metadata/policy. Andormu only executes the resulting workflow. Large file payloads remain in object/artifact storage and flow through `ArtifactRef`, not through Andormu's control-plane database or event queue.
