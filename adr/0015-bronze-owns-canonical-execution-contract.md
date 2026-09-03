# ADR-0015: Bronze owns the canonical execution contract

**Status:** Accepted

## Context

Using a cloud workflow product, Kubernetes object, Slurm/Ray job, or other backend-native definition as the canonical workflow model would couple Bronze Dragonflight execution semantics to one deployment/runtime ecosystem.

Cross-cloud and on-prem execution requires more than syntax translation: task identity, dependency meaning, retry/cancel/recovery semantics, and operator interpretation must remain coherent.

## Decision

Bronze Dragonflight owns the canonical workflow/task execution contract.

Andormu executes that vendor-neutral contract and preserves its logical semantics across supported backends through adapters.

Cloud/vendor/native workflow and compute definitions may be generated, invoked, or observed behind adapters, but they do not define canonical Bronze workflow semantics.

## Consequences

- `WorkflowSpec` and related shared contracts must not require Alibaba, Volcano, Tencent, Baidu, Kubernetes, Slurm, Ray, or other vendor-native objects.
- Portability is semantic, not merely YAML/DSL translation.
- ExecutionSnapshot/versioning design must distinguish stable logical/contract revisions from transient physical instances.
