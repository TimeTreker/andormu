# ADR-0009: Dynamic expansion must preserve an acyclic execution graph

**Status:** Proposed

Runtime branch/fan-out/subgraph decisions are persisted before child scheduling and may not introduce cycles or mutate already-executed history.
