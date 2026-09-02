# ADR-0010: Finalizers are first-class workflow semantics

**Status:** Proposed

Cleanup/finalization is modeled explicitly. Graceful cancellation can run finalizers; emergency termination does not guarantee them.
