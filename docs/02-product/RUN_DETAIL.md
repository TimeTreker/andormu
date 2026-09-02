# Run Detail

Recommended tabs:

```text
Overview | Graph | Timeline | Tasks | Attempts | Inputs/Outputs | Events | Recovery | Audit
```

## Overview

- WorkflowRun state/reason.
- Exact ExecutionSnapshot digest.
- Workflow revision.
- Started/elapsed/end time.
- Task-state summary.
- Parent/child workflow links.
- Current operator action (suspended/cancelling/etc.).

## Graph

Node color/state with an explanation drawer. Selecting a node shows dependencies, trigger rule, attempts, retry policy, resource intent, backend handle/log links, and outputs.

## Timeline

Show run/task-attempt transitions and operator actions in time order. A retry should appear as multiple attempts, never as a rewritten single row.

## Recovery

Present only semantically safe actions for the current state, with a preview of what work will be reused/re-executed.
