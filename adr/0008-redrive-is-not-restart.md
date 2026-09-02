# ADR-0008: Redrive and restart are distinct

**Status:** Proposed

Redrive continues the same failed WorkflowRun under the same ExecutionSnapshot and reuses successful work. Restart creates a new WorkflowRun.
