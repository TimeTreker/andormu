# Service Execution Contract Design

This document records contract requirements only; it is not yet a normative schema.

## Dispatch must carry

- stable WorkflowRun / TaskRun / TaskAttempt identity,
- immutable task-definition/snapshot reference,
- exact capability and pinned ExecutionBinding/ExecutionTarget references,
- compact resolved inputs and ArtifactRefs,
- timeout/cancel/retry context,
- trace/correlation context,
- idempotency identity,
- optional ResourceIntent resolved from the ExecutionBinding.

## Adapter must support

Minimum:

- dispatch/submit,
- observe,
- cancel.

Potential Phase-1 additions:

- callback/event completion,
- heartbeat/lease renewal,
- capacity advertisement,
- log/trace reference discovery.

## Transport invariant

Duplicate delivery for the same TaskAttempt must not create another logical attempt.

## Payload invariant

Large files and artifacts are referenced, not transported through the workflow control plane.
