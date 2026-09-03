# Design Principles

1. **Correctness before convenience.** Hidden retry/cancel behavior is worse than a slightly more verbose contract.
2. **Immutable definitions, append-only history.** Historical executions must remain explainable after definitions evolve.
3. **Explicit control semantics.** Finalizers, skips, redrive, and cancellation should be first-class, not accidental trigger-rule combinations.
4. **Adapters contain infrastructure coupling.** The core state machine speaks Andormu concepts only.
5. **Small inline data, large data by reference.** Workflow state is not an object store.
6. **Typed failure, simple state.** Avoid exploding the state enum; attach failure origin/class/reason separately.
7. **Logical admission vs physical scheduling.** Andormu owns whether logically ready work may be dispatched; Compute owns physical admission, placement, quota, topology, and preemption.
8. **At-least-once is a fact, not a bug.** Design idempotency explicitly.
9. **Dynamic behavior must be replayable from persisted decisions.** Never depend on recomputing unrecorded runtime branching after recovery.
10. **Design for operators.** Every transition should be explainable in the UI and event history.
11. **Own the semantics; reuse the infrastructure.** Mature messaging, storage, stream/batch processing, container, compute, GPU, database, and observability systems are dependencies/backends, not responsibilities to reimplement.
12. **Semantic portability over syntax portability.** Bronze Dragonflight owns the canonical workflow/task contract; portability means preserving execution meaning across backends, not translating one vendor YAML into another.
13. **Logical DAG nodes are not deployment types.** A Task describes logical work. Persistent service, deferred service operation, worker, Flink/Spark job, Kubernetes/Slurm/Ray job, or cloud job is an execution realization behind an adapter.
14. **Executor lifecycle and completion protocol are separate dimensions.** A persistent service may complete work inline or defer completion; asynchronous completion is not a distinct business-node type.
15. **One orchestration authority per DAG layer.** When another mature engine owns an internal execution graph, Andormu normally treats it as one opaque TaskAttempt rather than duplicating its scheduler/state machine.
16. **Depend on capabilities, not product internals.** Production may strongly depend on ACID storage, durable messaging, compute, and observability capabilities without making a particular vendor object's semantics canonical.
