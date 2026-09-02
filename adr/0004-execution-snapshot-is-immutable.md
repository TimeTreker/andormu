# ADR-0004: Workflow execution uses an immutable snapshot

**Status:** Proposed

Before dispatching work, Andormu resolves workflow/task references and inputs into an immutable ExecutionSnapshot. Redrive uses the same snapshot.
