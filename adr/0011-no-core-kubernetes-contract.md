# ADR-0011: Core contracts are not Kubernetes contracts

**Status:** Accepted

Kubernetes/Slurm/Ray-specific objects belong behind execution adapters. Canonical WorkflowSpec and TaskSpec remain backend-neutral.
