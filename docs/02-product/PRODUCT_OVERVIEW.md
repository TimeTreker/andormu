# Product Overview

The Phase-1 product should prioritize **operations and debugging**, not low-code workflow authoring.

## Primary navigation

```text
Andormu
├── Workflow Definitions
├── Runs
├── Task Definitions
├── Events / Incidents
└── Administration
```

## Golden user journey

```text
Validate WorkflowSpec
       ↓
Preview DAG + resolved revision
       ↓
Submit WorkflowRun
       ↓
Observe graph/timeline
       ↓
Investigate task attempt if needed
       ↓
Suspend / cancel / redrive when necessary
       ↓
Audit final result and execution history
```

The UI should always present Andormu semantic objects first. Backend Kubernetes/Slurm objects, if exposed at all, appear only as linked diagnostic details from an adapter.
