# First Commit Readiness

## Decision

**Ready for the first repository commit as a Phase-0 design baseline.**

This does **not** mean the Design Gate is passed and does **not** authorize implementation coding.

## What the first commit establishes

- product mission and non-goals,
- Bronze Dragonflight platform boundaries,
- domain-agnostic DAG/workflow responsibility,
- durable/reconciliation execution direction,
- WorkflowRun / TaskRun / TaskAttempt design direction,
- retry, timeout, cancellation, finalizer, redrive, event, and dynamic-graph proposals,
- independent Compute Platform boundary,
- service-task and asynchronous-task first-class requirements,
- logical task admission/backpressure requirements,
- domain-owned workflow selection,
- contract-design topics and unresolved questions,
- explicit Design Gate before coding.

## What remains intentionally unresolved after the first commit

See `PHASE0_REVIEW.md`. Major items include:

- exact public state model,
- retry-counter semantics on redrive,
- timeout clocks during suspension,
- exact finalizer outcome rules,
- dynamic fan-out Phase-1 scope/limits,
- event journal source-of-truth architecture,
- capability-to-execution-target resolution ownership,
- async completion protocol minimums,
- logical fairness/admission scope,
- shared ResourceIntent / ArtifactRef / SecretRef contracts.

## Recommended commit message

```text
docs: establish Andormu Phase-0 design baseline
```

## Rule after first commit

Continue Phase-0 review/design commits. Do not add implementation code until `coding/ENTRY_CRITERIA.md` and `PHASE0_REVIEW.md` satisfy the Design Gate.
