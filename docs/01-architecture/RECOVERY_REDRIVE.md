# Recovery and Redrive

Andormu must distinguish several operations that workflow products often blur together.

## 1. Engine crash recovery — automatic

If an Andormu process dies, another process reconciles the same durable run. Existing external attempts should be re-observed through stored ExecutionHandles rather than blindly relaunched.

## 2. Task retry — automatic/policy-driven

A new TaskAttempt is created under the same TaskRun after a retryable failure.

## 3. Redrive — explicit, same WorkflowRun

Inspired by Step Functions redrive:

- use the same ExecutionSnapshot,
- preserve successful TaskRuns and outputs,
- reactivate failed/unreached work,
- append `RedriveAttempt` history,
- never silently adopt a newer WorkflowRevision.

**Open question:** whether per-task retry counters reset on redrive. Step Functions resets retries for redriven states; Andormu should make this a deliberate contract choice.

## 4. Restart — new WorkflowRun

Create a new run from the beginning using an explicitly selected revision/snapshot and inputs.

## 5. Targeted rerun/invalidation — future

Rerunning an arbitrary successful middle task can invalidate downstream outputs and repeat side effects. This should not enter Phase 1 until descendant invalidation and idempotency semantics are designed.

## Lost attempts

If an adapter cannot find an execution previously believed active:

1. mark the TaskAttempt `LOST` with evidence,
2. apply the TaskRun policy,
3. never fabricate success,
4. preserve the lost attempt in history.
