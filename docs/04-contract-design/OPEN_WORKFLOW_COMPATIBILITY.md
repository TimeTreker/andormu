# Open Workflow Specification Compatibility

The Open Workflow Specification 1.0 family is a useful interoperability reference because it defines common concepts such as fork, switch, try/retry, wait, listen, emit, and nested workflows.

## Current proposal

Do **not** make it Andormu's canonical internal/public schema in Phase 0.

Reasons:

- Andormu is graph-first for AI-infra generated DAGs.
- It needs explicit resource-intent and execution-adapter boundaries.
- Artifact-reference and immutable snapshot semantics are central.
- The Open Workflow DSL is broader and more imperative/service-oriented.

## Future direction

Consider an importer/exporter or compatibility layer for the supported semantic subset after Andormu's canonical contract stabilizes.
