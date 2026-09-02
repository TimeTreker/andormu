# Platform Boundaries

## Andormu owns workflow readiness

Andormu answers:

- Are all required upstream conditions satisfied?
- Should this node run, wait, skip, retry, or fail?
- Is a finalizer eligible?
- Is a retry budget exhausted?
- Has the workflow timed out?
- Is a redrive allowed under the pinned snapshot?

## Compute Platform owns resource allocation

Compute answers:

- Is the request admitted?
- Which queue/pool/cluster should serve it?
- Which CPU/GPU/NPU/host is selected?
- Should work be preempted?
- Does tenant quota permit the allocation?

## Execution adapter owns backend translation

An adapter translates generic task-execution intent into a concrete backend API and returns an opaque `ExecutionHandle`.

The core model should not contain `Pod`, `SlurmJob`, `RayJob`, or equivalent backend objects.

## Domain platform owns domain compilation

Anachronos may compile a `TrainingRun` into a `WorkflowSpec`; Zidormi may compile a data pipeline into a `WorkflowSpec`. Once submitted, Andormu sees only the generic workflow contract.

## Artifact systems own payload storage

Andormu stores metadata and references. Large datasets, checkpoints, logs, models, and arbitrary binary outputs belong in external systems.
