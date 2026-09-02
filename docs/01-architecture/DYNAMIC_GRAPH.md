# Dynamic Graph Semantics

Dynamic behavior is useful for map/fan-out, conditional branches, and subworkflows, but arbitrary mutable DAGs make recovery unsafe.

## Invariant

**The concrete execution graph of one WorkflowRun must remain acyclic.**

## Proposed dynamic mechanisms

### Conditional gate

A gate records one durable branch decision; untaken nodes become skipped.

### Map / fan-out

A map node consumes a finite input collection and expands it into uniquely identified child node instances.

### Dynamic expansion task

A dedicated expansion-capable node may return a validated subgraph description. Andormu must persist the expansion result before scheduling any child.

### Subworkflow

A node can invoke a pinned child WorkflowRevision/ExecutionSnapshot and track parent-child execution ids.

## Forbidden behavior

- arbitrary mutation of already-executed graph structure,
- creation of an edge that introduces a cycle,
- non-durable branching decisions,
- resolving child workflow “latest” after the parent has started.

## Why this design

Flyte's dynamic nodes demonstrate the value of runtime expansion; Temporal demonstrates why recovery cannot depend on recomputing non-deterministic decisions. Andormu combines both lessons by persisting expansion decisions as part of execution history.
