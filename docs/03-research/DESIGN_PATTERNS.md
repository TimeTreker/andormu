# Design Patterns to Absorb

## 1. Immutable run definition

Step Functions redrive uses the same state-machine definition; Flyte recovery uses the same workflow version. Andormu should pin an ExecutionSnapshot.

## 2. Separate logical task from execution attempt

Airflow task-instance history, Tekton TaskRuns, Prefect task-run states, and retry-capable engines all demonstrate that attempts/history must not be overwritten.

## 3. Reconciliation

Flyte Propeller repeatedly compares workflow desired/observed state and schedules eligible nodes. This is a strong fit for declarative Andormu specs.

## 4. Durable event history

Temporal demonstrates how durable event history enables recovery and postmortem explainability. Andormu should keep a durable append-only transition journal even if it does not replay workflow code.

## 5. Explicit finalization

Argo exit handlers and Tekton finally tasks show that cleanup should be a first-class semantic path.

## 6. Retry policy with backoff + jitter + typed errors

Temporal, Argo, Step Functions, and Conductor all converge on declarative retry policies. Conductor explicitly supports jitter to avoid retry storms.

## 7. Redrive distinct from restart

Step Functions and Conductor expose recovery operations distinct from a fresh execution. Andormu should make this distinction visible in API/UI.

## 8. At-least-once awareness

Conductor explicitly documents at-least-once worker message delivery. Andormu should design stable attempt ids and idempotency from day one.

## 9. Dynamic expansion as a recorded decision

Flyte dynamic nodes show runtime graph expansion is valuable. Persist the expansion result before executing child nodes.

## 10. CloudEvents-compatible external events

CloudEvents gives a stable envelope without forcing Andormu to outsource its internal state semantics.


## 11. Logical task != process lifecycle

Service-oriented workflow systems such as Conductor demonstrate that a task can be fulfilled by persistent workers/services. Andormu should model logical completion and keep service/process lifecycle external.

## 12. Backpressure before dispatch

High-volume durable task systems such as Hatchet reinforce that dependency-ready work cannot imply unbounded dispatch. Logical task admission/worker capacity belongs in workflow orchestration even though physical resource placement remains external.

## 13. Keep large payloads out of the control plane

Modern workflow-engine designs increasingly avoid moving large task outputs through controller queues. Andormu should carry ArtifactRefs and compact metadata/events, not multi-GB files or model artifacts.
