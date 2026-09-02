# ADR-0013: Task is logical work, not a process

**Status:** Accepted

## Context

Andormu must orchestrate both persistent service-based processing and compute-backed jobs. Defining a task as a Pod, process, or container would couple the core workflow model to one execution substrate and make service workflows unnatural.

## Decision

A canonical Andormu Task is a **logical unit of work with observable completion semantics**.

A TaskAttempt may be fulfilled by a persistent service, worker pool, asynchronous service operation, external runtime, or compute job through an execution adapter.

The canonical model must not require service lifecycle or physical resource allocation to equal task lifecycle.

## Consequences

- TaskRun/TaskAttempt semantics remain backend-neutral.
- Long-running service tasks are first-class.
- Execution adapters need synchronous and asynchronous completion patterns.
- Workflow definitions should prefer capability/target references over transient physical endpoints.
- Andormu does not become a service deployment or compute scheduling platform.
