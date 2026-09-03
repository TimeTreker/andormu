# ADR-0014: Andormu is the durable control plane of Timeways

**Status:** Accepted

## Context

Bronze Dragonflight needs one coherent execution-platform boundary across domain-generated DAGs and heterogeneous service/data/compute backends.

Calling Andormu merely another DAG scheduler understates the intended ownership boundary, while making Andormu a monolithic execution platform would duplicate mature infrastructure.

## Decision

**Timeways** is the vendor-neutral execution platform of Bronze Dragonflight.

**Andormu** is the durable control plane of Timeways.

Andormu owns logical workflow/task orchestration semantics, logical dispatch admission, durable execution state, recovery, and operator-visible execution truth.

Timeways execution is fulfilled by standard cloud/on-prem services, workers, data engines, compute systems, messaging, storage, and observability infrastructure.

## Consequences

- Domain platforms submit Bronze `WorkflowSpec` contracts to Andormu.
- Andormu is not the physical execution substrate.
- Timeways is an architectural platform composition, not a requirement to build replacements for standard infrastructure.
- The Andormu API/domain model must remain suitable across cloud and on-prem execution environments.
