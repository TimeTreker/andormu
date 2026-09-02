# Failure Propagation

## Default mode: continue independent work

A failure prevents dependents whose predicates can no longer be satisfied, while independent branches may continue.

The WorkflowRun ultimately fails if a required non-optional path fails.

## Optional fail-fast mode

A workflow may request `FAIL_FAST`:

- stop scheduling new normal tasks after an unrecoverable required failure,
- optionally request cancellation of active cancellable attempts,
- still evaluate required finalizers.

The exact “cancel in-flight” option must be explicit rather than implied by fail-fast.

## Failure origin vs state

Keep `FAILED` as a simple state and attach a structured `FailureDescriptor`:

- origin: workload / adapter / compute / engine / user,
- class,
- stable error code,
- human message,
- retryable hint,
- details/log reference,
- causal attempt/event ids.

## Optional tasks

If optional-task semantics are supported, an optional failure must be represented as an explicit policy and visible in the run outcome, not silently rewritten to success.
