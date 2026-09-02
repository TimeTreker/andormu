# UI Information Architecture

Phase-1 UI is an **operations console**, not a low-code DAG builder.

```text
Workflow Definitions
  └─ Definition Detail
      ├─ Graph
      ├─ Revisions
      ├─ Diff
      └─ Runs

Runs
  └─ Run Detail
      ├─ Overview
      ├─ Graph
      ├─ Timeline
      ├─ Tasks
      ├─ Attempts
      ├─ Inputs / Outputs
      ├─ Events
      ├─ Recovery
      └─ Audit

Task Definitions
Events / Incidents
Administration
```

Every graph node should answer “why is this state true?” without sending the user to raw backend objects.
