# Task Capability Model

## Goal

Decouple a workflow's logical requirement from a concrete transient endpoint or runtime implementation.

## Proposed concepts

### Capability

A stable semantic execution capability, for example:

```text
data.video.decode@v2
data.rosbag.parse@v3
ai.embedding.compute@v1
```

A capability identifier is not a domain decision by Andormu. Domain platforms select or emit it; Andormu only treats it as an opaque execution requirement with versioned contract metadata.

### ExecutionTarget

A resolved target or adapter route able to fulfill the capability.

Examples:

- service routing key,
- worker type,
- runtime template,
- compute job adapter,
- external system integration.

### TaskDefinition

A reusable, versioned definition may bind a capability to execution defaults, input/output contract, retry policy, timeout policy, observability hooks, and resource intent defaults.

## Why not fixed endpoints

Embedding a fixed endpoint in WorkflowSpec couples reproducible workflow semantics to ephemeral deployment details and prevents independent service evolution.

## Resolution ownership

The final resolver design is still open. Candidates include:

1. domain platform emits an already-resolved execution target;
2. Andormu calls a shared execution/service registry;
3. execution adapter resolves capability to backend target;
4. a Bronze Dragonflight Execution Gateway performs resolution.

The canonical WorkflowSpec should avoid requiring one deployment topology.

## Compatibility rule

Capability resolution may change which healthy instance executes an attempt, but it must not silently change the logical TaskDefinition revision or its contract semantics for an already-started WorkflowRun.
