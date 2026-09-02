# Design Principles

1. **Correctness before convenience.** Hidden retry/cancel behavior is worse than a slightly more verbose contract.
2. **Immutable definitions, append-only history.** Historical executions must remain explainable after definitions evolve.
3. **Explicit control semantics.** Finalizers, skips, redrive, and cancellation should be first-class, not accidental trigger-rule combinations.
4. **Adapters contain infrastructure coupling.** The core state machine speaks Andormu concepts only.
5. **Small inline data, large data by reference.** Workflow state is not an object store.
6. **Typed failure, simple state.** Avoid exploding the state enum; attach failure origin/class/reason separately.
7. **Logical scheduling vs resource scheduling.** Andormu decides *what is runnable*; Compute decides *where it runs*.
8. **At-least-once is a fact, not a bug.** Design idempotency explicitly.
9. **Dynamic behavior must be replayable from persisted decisions.** Never depend on recomputing unrecorded runtime branching after recovery.
10. **Design for operators.** Every transition should be explainable in the UI and event history.
