# Task Capability Model

## Goal

Decouple a workflow's logical requirement from environment-specific execution realization and transient backend routing.

## Object responsibilities

| Object | Question answered | Must not contain |
|---|---|---|
| `TaskDefinition` | What logical work and contract are required? | provider, adapter, endpoint, environment capacity |
| `Capability` | What exact semantic ability must a realization satisfy? | target instance, transport, infrastructure object |
| `ExecutionBinding` | How is that capability realized in this environment? | DAG topology or business dependency rules |
| `ExecutionTarget` | Where does the selected adapter route this attempt? | logical task meaning or mutable instance address in the workflow |

## Capability

A capability is a stable, versioned semantic identifier, for example:

```text
data.bag.decode@v3
data.embedding.encode@v2
data.metadata.extract@v1
```

It is selected by a domain-authored TaskDefinition, not inferred by Andormu from a filename or task name. It is an exact compatibility requirement, not a request for “latest”.

The interface and behavioral guarantees live in the TaskDefinition. The capability provides the match key; it does not duplicate the full contract.

## ExecutionBinding

An ExecutionBinding is an immutable, environment-scoped mapping from an exact capability to an execution realization. It selects:

- provider class;
- completion model;
- adapter reference;
- stable logical ExecutionTarget;
- supported observation/cancellation behavior;
- optional AdmissionPolicy reference;
- optional ResourceIntent for compute realizations.

The full design is in `EXECUTION_BINDING.md`.

## ExecutionTarget

`ExecutionTarget` is deliberately narrower than the earlier overloaded concept. It is a stable logical route understood by the chosen adapter, such as a service name, worker type, runtime template, or external integration key.

It is not:

- a TaskDefinition or Capability;
- an ExecutionBinding;
- a transient IP, Pod name, allocation id, or backend run id;
- the opaque `ExecutionHandle` returned after dispatch.

An adapter or service registry may resolve an ExecutionTarget to healthy physical instances at dispatch time without changing the pinned binding.

## Resolution ownership

Before execution, an environment resolver matches every required exact capability to one compatible active ExecutionBinding revision. Andormu pins the result in the `ExecutionSnapshot`.

The implementation boundary remains open: resolution may be provided by Andormu configuration, a shared registry, or a Bronze execution gateway. Regardless of implementation, the semantic result is the same:

```text
exact capability
      │ resolve in environment
      ▼
pinned ExecutionBinding revision
      │ dispatch through adapter
      ▼
stable ExecutionTarget -> physical instance/backend execution
```

## Compatibility invariants

- Resolution must fail before dispatch when no compatible binding exists.
- One snapshot never follows a mutable binding alias after activation.
- Routing may select a different healthy physical instance without changing the logical contract.
- A binding update affects future snapshots only.
- A provider/runtime migration needs no WorkflowSpec or TaskDefinition revision when the logical contract remains compatible.
- If inputs, outputs, retry safety, idempotency requirements, or other logical guarantees change incompatibly, the TaskDefinition/capability revision must change.

## Why not fixed endpoints

Embedding endpoints or native runtime objects in WorkflowSpec couples reproducible workflow semantics to ephemeral deployment details and prevents independent execution-plane evolution. Semantic portability requires a stable logical contract plus an environment binding, not portable-looking vendor YAML.
